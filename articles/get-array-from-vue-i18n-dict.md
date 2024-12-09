---
title: 【型安全な実装】型安全にvue-i18nの辞書から配列を取得して、変数に設定する【の掘り下げ方】
emoji: 🐕
type: tech
topics: [vue, i18n, nuxt, typescript, typesafe]
published: true
---
# 結論

:::message
コードに曖昧性のある場合、Nuxtの環境を想定しています。
e.g. [Auto-imports - Nuxt Concepts](https://nuxt.com/docs/guide/concepts/auto-imports)
:::

i18nの辞書ファイル
```json:ja.json
{
  "path": {
    "to": {
      "list": ["a", "b", "c"]
    }
  }
}
```

ユーティリティ関数
（今回の主な結論。`getI18nArray`を使えば達成できます。）
```typescript:utils/i18n.ts
/**
 * 引数未指定にすると、普通に`const i18n = useI18n()`とすると入ってくる型になる。
 * 型引数の使い方については、そのままuseI18nの型引数の指定方法を参照のこと。
 */
export type UseI18nReturnType<Options extends UseI18nOptions = UseI18nOptions> =
  Composer<
    NonNullable<Options['messages']>,
    NonNullable<Options['datetimeFormats']>,
    NonNullable<Options['numberFormats']>,
    Options['locale'] extends unknown ? string : Options['locale']
  >

/**
 * @example
 * ```ts
 * import { useI18n } from 'vue-i18n'
 * const i18n = useI18n() // messageは { list: ['a', 'b', 'c'] } とする
 * const list = getI18nArray(i18n, 'list') // ['a', 'b', 'c']
 * ```
 */
export const getI18nArray = (i18n: UseI18nReturnType, key: string): string[] =>
  Object.entries<VueMessageType>(i18n.tm(key)).map(([_, term]) => i18n.rt(term))
```

配列を要求する側のコンポーネント
```vue:RequringArray.vue
<template>
  <ul>
    <li v-for="item in list" :key="item">{{ item }}</li>
  </ul>
</template>

<script setup lang="ts">
defineProps<{
  list: string[] // なんらかの事情でArrayを要求している
}>()
</script>
```

配列を取得して、propsとして渡す側のコンポーネント
```vue:Using.vue
<template>
  <div>
    <!-- ... -->
    <RequringArray :list="list" />
    <!-- ... -->
  </div>
</template>

<script setup lang="ts">
const i18n = useI18n()
const list = getI18nArray(i18n, 'path.to.list')
</script>
```

Draft PRですが以下に、弊社のNuxt QuickStarterリポジトリに、実際のPRを作成しています。

- [Make main/app/utils/i18n module & Add getI18nArray - Pull Request #3 - PublicHIKKY/vket-boilerplace-nuxt](https://github.com/PublicHIKKY/vket-boilerplace-nuxt/pull/3)

# 問題

サイトを作っていると、多言語対応をしたいという要望があります。
それを実現するVue向けの方法の一つが、vue-i18nです。

- [Vue I18n](https://vue-i18n.intlify.dev/)

そして今回、多言語化をしつつ、そのi18n辞書から直接配列を取得したいというニーズが出てきました。
つまり以下のような方法では不十分ということです。

:::message
- ※ 以下では`ja.json`をi18n辞書として定義しています。今後同様に、nuxt.config.tsの`.vueI18n.locales[0].file`には`ja.json`が指定されているとします
    - [locales](https://i18n.nuxtjs.org/docs/api/vue-i18n#locales)
    - 要約すると、ここで例示するvue-i18nの環境の辞書のうち`ja`向けの設定が、`ja.json`の内容でされているということです
:::

- - -

🙅 ❌

i18nの辞書ファイル
```json:ja.json
{
  "path": {
    "to": {
      "item1": "a",
      "item2": "b",
      "item3": "c"
    }
  }
}
```

配列を取得して、propsとして渡す側のコンポーネント
```vue:Using.vue
<template>
  <div>
    <!-- ... -->
    <RequringArray :list="list" />
    <!-- ... -->
  </div>
</template>

<script setup lang="ts">
const i18n = useI18n()

// !! ここでは個別にアイテムを取得したくない
const list = [
  `${i18n.t('path.to.item1')}`,
  `${i18n.t('path.to.item2')}`,
  `${i18n.t('path.to.item3')}`,
]
</script>
```

- - -

🙆 ⭕

i18nの辞書ファイル
```json:ja.json
{
  "path": {
    "to": {
      "list": ["a", "b", "c"]
    }
  }
}
```

配列を取得して、propsとして渡す側のコンポーネント
```vue:Using.vue
<template>
  <div>
    <!-- ... -->
    <RequringArray :list="list" />
    <!-- ... -->
  </div>
</template>

<script setup lang="ts">
const i18n = useI18n()

// !! 一度に配列を取得している
const list = getI18nArray(i18n, 'path.to.list')
</script>
```

- - -

# 解決案

調べると、次のような案が見つかりました。

- - -

```vue
<template>
  <p v-for="name in tm('heroPage.names')" :key="name">
    {{ rt(name) }}
  </p>
</template>

<script>
import { useI18n } from 'vue-i18n'

export default {
  setup() {
    const { tm, rt } = useI18n();
    return { rt, tm }
  }
}
</script>
```

['here is a example:'](https://github.com/intlify/vue-i18n/issues/885#issuecomment-1014992153)より引用

- - -

しかし我々が求めているのは、`v-for="name in tm('heroPage.names')"`することではなく、直接配列を取得することです。
ただし`i18n.tm`, `i18n.rt` は覚えておいた方がよさそうです。

次に['vue-i18nでまるっとオブジェクトを取得する - Blogメモφ(..)'](https://blog.nightonly.com/2023/09/27/vue-i18n%E3%81%A7%E3%81%BE%E3%82%8B%E3%81%A3%E3%81%A8%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B/)を発見しました。
ここでは次のようなコードが例示されています。

- - -

```typescript
for (const [, item] of Object.entries($tm('items.space') as any) as any) {
  console.log(item.text)
}
```

or

```
for (const item of Object.values($tm('items.space') as any) as any) {
  console.log(item.text)
}
```

（['vue-i18nでまるっとオブジェクトを取得する - Blogメモφ(..)'](https://blog.nightonly.com/2023/09/27/vue-i18n%E3%81%A7%E3%81%BE%E3%82%8B%E3%81%A3%E3%81%A8%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%82%92%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B/)より引用）

- - -

これは完成に近そうです！
次のようにできれば、解決できそうです。
ついでに、私達の環境向けに書き替えておきます。

```typescript
const list = Object.entries(i18n.tm('path.to.list') as any).map(([_, item]) => i18n.rt(item))
```

しかし`as any`を使うのは、あまりよくないです。
`any`を使用すると、そのコードが正しいのか静的に検証できなくなるからです。
いわゆる「型安全でない」というものです。

（プロジェクト規模が小さい場合は、それでよいかもしれませんが、プロジェクト単位が中規模以上であれば`any`は使わない方がよいでしょう。）

# 解決策

`i18n.rt`と`i18n.tm`の存在、および`any`を使った解決策は発見できました。
あとはこれらを使って、`any`の代わりに適切な型を指定するだけです。

では、適切な型とは何でしょうか。
ここまできたら、lspをガンガン使って、実地調査をしていきましょう。

## `Object.entries`

まずは「`any`を使った解決策」から、`Object.entries`が使えることがわかるので、型を見てみます。

```typescript
entries<T>(o: { [s: string]: T } | ArrayLike<T>): [string, T][];
```

これをもっと簡単に考えると、次のような感じです。

```typescript
entries<T>(o: Record<string, T>): [string, T][];
```

ここの`T`に`any`の変わりを指定すれば、よさそうですね。

## `i18n.tm`

明らかに`T`になるのは、`i18n.tm`の要素の型、言い換えれば`i18n.rt`の引数の型です。
では`i18n.tm`の型を見てみましょう。

```typescript
tm<Key extends string, ResourceKeys extends PickupKeys<Messages> = PickupKeys<Messages>, Locale extends PickupLocales<NonNullable<Messages>> = PickupLocales<NonNullable<Messages>>, Target = IsEmptyObject<Messages> extends false ? NonNullable<Messages>[Locale] : RemoveIndexSignature<{
  [K in keyof DefineLocaleMessage]: DefineLocaleMessage[K];
}>, Return = ResourceKeys extends ResourcePath<Target> ? ResourceValue<Target, ResourceKeys> : Record<string, any>>(key: Key | ResourceKeys): Return;
```

なんのこっちゃ？
手動簡約するのはめっちゃがんばればいけそうですが、今回は`i18n.rt`の型定義がもっと簡単であることに望みをかけて、そちらを見てみましょう。

```typescript
rt(message: MessageFunction<VueMessageType> | VueMessageType): string;
```

ビンゴです！
`Object.entries`の型指定は、`Object.entries<MessageFunction<VueMessageType>>`もしくは`Object.entries<VueMessageType>`になりますね。

では`MessageFunction<VueMessageType>`と`VueMessageType`のどちらになるのでしょうか？
これは`i18n.tm`の型定義を調べるのが、前述の通り静的には難しかったので、簡単にChrome DevToolsで調べました。

どうやら関数ではなさそうだったので、`VueMessageType`のようです。

## 解

材料は全てあつまりました。
答えは次になります。

```typescript
const list = Object.entries<VueMessageType>(i18n.tm('path.to.list')).map(([_, item]) => i18n.rt(item))
```

やった！

# 応用: 関数化する

本稿の答えは出ましたが、関数化しておくと、より使いやすくなります。

単純に考えて、次のようになりそうです。

```typescript:utils/i18n.ts
export const getI18nArray = (i18n: ???, key: string): string[] =>
  Object.entries<VueMessageType>(i18n.tm(key)).map(([_, term]) => i18n.rt(term))
```

`???`の部分は、`useI18n`の返り値の型を指定すればよさそうです。
`useI18n`の型定義を見てみましょう。

```typescript
export declare function useI18n<
  Options extends UseI18nOptions = UseI18nOptions
>(options?: Options):
  Composer<
    NonNullable<Options['messages']>,
    NonNullable<Options['datetimeFormats']>,
    NonNullable<Options['numberFormats']>,
    Options['locale'] extends unknown ? string : Options['locale']
  >;
```

実のところ簡単に考えれば、`???`には`ReturnType<typeof useI18n<UseI18nOptions>>`を指定すればよさそうですが、実は`useI18n`は次のように型引数を取ることがあります。

```typescript
import ja from '@/locales/ja.json'
const i18n = useI18n<{ message: typeof ja }>()
```

この場合、`???`に`ReturnType<typeof useI18n<UseI18nOptions>>`を指定していると型がミスマッチするので、一応柔軟に定義しておきましょう。
次のようになります。

```typescript:utils/i18n.ts
export type UseI18nReturnType<Options extends UseI18nOptions = UseI18nOptions> =
  Composer<
    NonNullable<Options['messages']>,
    NonNullable<Options['datetimeFormats']>,
    NonNullable<Options['numberFormats']>,
    Options['locale'] extends unknown ? string : Options['locale']
  >

export const getI18nArray = (i18n: UseI18nReturnType, key: string): string[] =>
  Object.entries<VueMessageType>(i18n.tm(key)).map(([_, term]) => i18n.rt(term))
```

ついにこれで完成です！

```typescript
const i18n = useI18n()
const list = getI18nArray(i18n, 'path.to.list')
```

# まとめ

今回はvue-i18nの辞書から配列を取得して、変数に設定する方法を解説しました。
型安全も仮定しています。

もう一度見てみます。

i18nの辞書ファイルは次のようになっています。
```json:ja.json
{
  "path": {
    "to": {
      "list": ["a", "b", "c"]
    }
  }
}
```

今回の解決策は、結局のところ次のようになります。
```typescript:utils/i18n.ts
/**
 * 引数未指定にすると、普通に`const i18n = useI18n()`とすると入ってくる型になる。
 * 型引数の使い方については、そのままuseI18nの型引数の指定方法を参照のこと。
 */
export type UseI18nReturnType<Options extends UseI18nOptions = UseI18nOptions> =
  Composer<
    NonNullable<Options['messages']>,
    NonNullable<Options['datetimeFormats']>,
    NonNullable<Options['numberFormats']>,
    Options['locale'] extends unknown ? string : Options['locale']
  >

/**
 * @example
 * ```ts
 * import { useI18n } from 'vue-i18n'
 * const i18n = useI18n() // messageは { list: ['a', 'b', 'c'] } とする
 * const list = getI18nArray(i18n, 'list') // ['a', 'b', 'c']
 * ```
 */
export const getI18nArray = (i18n: UseI18nReturnType, key: string): string[] =>
  Object.entries<VueMessageType>(i18n.tm(key)).map(([_, term]) => i18n.rt(term))
```

上述の`getI18nArray`を使った例を見てみましょう。

これは配列を要求する側のコンポーネントです。
```vue:RequringArray.vue
<template>
  <ul>
    <li v-for="item in list" :key="item">{{ item }}</li>
  </ul>
</template>

<script setup lang="ts">
defineProps<{
  list: string[] // なんらかの事情でArrayを要求している
}>()
</script>
```

次が`getI18nArray`を使い、配列を取得して、propsとして渡す側のコンポーネントです。
```vue:Using.vue
<template>
  <div>
    <!-- ... -->
    <RequringArray :list="list" />
    <!-- ... -->
  </div>
</template>

<script setup lang="ts">
const i18n = useI18n()
const list = getI18nArray(i18n, 'path.to.list')
</script>
```

このようにして、vue-i18nの辞書から配列を取得して、変数に設定することができます。

おつかれさまでした！

- - - - -

Draft PRですが以下で、弊社のNuxt QuickStarterリポジトリの、実際のPRを見ることができます。

- [Make main/app/utils/i18n module & Add getI18nArray - Pull Request #3 - pUBLIcHIKKY/vket-boilerplace-nuxt](https://github.com/PublicHIKKY/vket-boilerplace-nuxt/pull/3)

- - - - -

以下、引用したコードのライセンス条項です。

:::details vue-i18n（MIT）
[vue-i18n](https://github.com/intlify/vue-i18n/blob/3cc105dd41403263214da18f1abb09dbc62a682b/LICENSE)

The MIT License (MIT)

Copyright (c) 2016 kazuya kawaguchi

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
:::
