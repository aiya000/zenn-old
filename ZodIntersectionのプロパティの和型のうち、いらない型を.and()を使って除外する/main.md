# ZodIntersectionのプロパティの和型のうち、いらない型を`.and()`を使って除外する

- 全てはここにまとまっています: https://stackblitz.com/edit/vitejs-vite-7rf7xq?file=src%2Fmain.ts
- ↑実際に動かせます

```ts
// やんごとなき理由でZodIntersectionになっている（.omit()や.extend()が使えない）zodスキーマ
const xSchema = z
  .object({
    a: z.number().nullable(), // この`number | null`を`number`に制限して、新しいスキーマを作りたい
  })
  .and(
    z.object({
      b: z.string(),
    })
  );

// nullableを消すだけなら、実は.and()で足りる
const ySchema = xSchema.and(
  z.object({
    a: z.number(),
  })
);

type Y = z.infer<typeof ySchema>;
type A = Y['a']; // number
```

## ZodIntersectionは`.omit()`や`.extend()`ができない

ZodObjectなどのzodの主要な型は`.omit()`や`.extend()`ができるので、スキーマのプロパティの型を上書きできます。
例えば以下のような用法です。

```ts
const fooSchema = z.object({
  baz: z.number(),
})
const barSchema = fooSchema.extend({
  baz: z.string(),
})

// あるいは
const barSchema = fooSchema
  .omit({ baz: true })
  .and(z.object({
    baz: z.string(),
  }))
```

しかしやんごとなき理由（例えばライブラリ側でzodスキーマが提供されているなどの理由）でZodIntersectionのみが見えている場合、ZodIntersectionには`.omit()`や`.extend()`がはえていないので、上述のような**上書き**はできません。

ただ今回「和型の部分的**除外**ならできる」という気づきがあったので、共有します。

## ZodIntersectionの和型プロパティを部分的に除外する

まず以下のようなZodIntersectionがあるとします。

```ts
// やんごとなき理由でZodIntersectionになっている（.omit()や.extend()が使えない）zodスキーマ
const xSchema = z
  .object({
    a: z.number().nullable(), // この`number | null`を`number`に制限して、新しいスキーマを作りたい
  })
  .and(
    z.object({
      b: z.string(),
    })
  );
```

`.omit()`や`.extend()`はできません。

```ts
// ZodIntersectionには.omit()がないのでできない
// const ySchema = xSchema.omit({ a: true }).and(z.object({
//   a: z.number()
// }))

// ZodIntersectionには.extend()がないのでできない
// const ySchema = xSchema.extend({
//   a: z.number()
// })
```

このとき、以下のようにすることで、`.a`を`number | null`から`number`にすることができます。

```ts
const ySchema = xSchema.and(
  z.object({
    a: z.number(),
  })
);

type Y = z.infer<typeof ySchema>;
type A = Y['a']; // number
```

実はそれはそうで、これはzodスキーマに限った話ではなく、TypeScriptの素の型でそうなっています。

```ts
type _X = {
  a: number | null;
};
type _Y = _X & {
  a: number;
};
type _A = _Y['a']; // number
```

以下はその計算式です。

```
{ a: number | null } & { a: number }
= { a: (number | null) & number }
= { a: (number & number) | (null & number) }
= { a: number | never }
= { a: number }
```

この除外（和型の一部を取り出す方法）は、もちろん除外対象がnullでなくとも使えます。

```ts
const wSchema = z
  .object({
    a: z.union([z.number(), z.literal('p'), z.boolean(), z.literal('q')]),
  })
  .and(
    z.object({
      b: z.string(),
    })
  );
const vSchema = wSchema.and(
  z.object({
    a: z.union([z.number(), z.boolean()]), // 'p'と'q'を除外する
  })
);
type V = z.infer<typeof vSchema>;
type VA = V['a']; // number | boolean
```

要は上述の

```ts
const wSchema = z
  .object({
    a: /* ここ（I） */,
  })
  .and(/* 割愛 */);

const vSchema = wSchema.and(
  z.object({
    a: /* ここ（J） */,
  })
);
```

このJがIの親型になっていればよいです。

## 上書きはできない

ただしもちろん、上書きはできないです。

```ts
const zSchema = xSchema.and(
  z.object({
    a: z.string(),
  })
);
type ZA = z.infer<typeof zSchema>['a']; // never
```

この場合は`a: never`になります。
