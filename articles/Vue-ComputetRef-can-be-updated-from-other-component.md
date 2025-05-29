:::details 追記: 🙅‍♂️->「`core.value`が子コンポーネントに更新される」 | 🙆‍♂️->「`core.value.value`が子コンポーネントに更新される」

# 追記概要

この記事は
- 「`core.value`が子コンポーネントに更新される」
ではなく
- 「`core.value.value`が子コンポーネントに更新される」
を意図して書いています！

誤解の防止のために、追記させていただきました🙌

おそらく上述の説明のみで十分かと思いますが、以下に詳細を記します。
通常はこの折り畳みを閉じてもらい、続きを読んでください。
もし後述の本編を読み、上述の追記の意味がわからなければ、説明の順序が前後しますが、下記の追記詳細を読んでください。

## 追記詳細

`core.value.value`と`core.value`（= `proxy.value`）がまぎらわしく、誤解させることに気が付きました 🙏

意図としては後述では「`core.value`が子コンポーネントに更新される」ということを言いたいわけではなく、「`core.value.value`が子コンポーネントに更新される」ということです。

具体的には、以下のようなB.vueの`updateFooBar`により、A.vueの`fooComputed`ごしに、`foo.value.bar`が更新されるということです。

（追記なので、以下は実働を確認しておりません。失礼します！）

```vue:A.vue
<template>
  <p>foo.value is {{ foo.value }}</p>
  <!-- foo.value is { bar: 42 } --> <!-- クリックをしていない状態 -->
  <!-- foo.value is { bar: 52 } --> <!-- クリック1回目 -->
  <!-- foo.value is { bar: 62 } --> <!-- クリック1回目 -->
  <!-- ... -->
  <B :foo="fooComputed" />
</template>

<script setup lang="ts">
const foo = ref({ bar: 42 })
const fooComputed = computed(() => foo.value)
</script>
```

```vue:B.vue
<template>
  <button @click="updateFooBar">update</button>
</template>

<script setup lang="ts">
const { foo } = defineProps<{
  foo: { bar: number }
}>()
function updateFooBar() {
  foo.bar += 10
}
</script>
```

ここでの.vueと、後述の.vueの概念の対応としては、以下のようになります。

- A.vue <-> HelloWorld.vue
- B.vue <-> Child.vue

:::

# 俺の`computed(() => x.value)`の値が子コンポーネントに勝手にアップデートされるんだが

- `type T`が`T extends Record<string, unknown>`で
- かつ`computed({ get, set })`でなく、`computed(() => x.value)`の形だった（`WritableComputedRef`でなかった）

な場合の話。

（[余談だけど、`T extends Object`みたいに、`Object`型は使ってないですよね？](https://typescript-eslint.io/rules/no-wrapper-object-types)）

## 結論

- 実際に確認できる環境: https://stackblitz.com/edit/vitejs-vite-we2pdvpy?file=src%2Fcomponents%2FHelloWorld.vue

![](https://storage.googleapis.com/zenn-user-upload/776c62114f80-20250528.gif)

`ComputedRef`（`computed`な値）の`proxy`と、`Ref`（`computed`の元になる値）の`core`を定義する側:

```vue:HelloWorld.vue
<template>
  <p>core.value is {{ core.value }}</p>
  <Child :proxy />
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import Child from './Child.vue'

const core = ref({ value: 42 })
const proxy = computed(() => core.value)

// オブジェクトの受け渡しはシャローコピーなので、 core.value.valueは `<Child>` 経由で更新できる。
// ただしそのとき、core.valueが更新されたわけではないので、この`watch()`は実行されない。
watch(core, () => console.log('poi: core is updated'))
</script>
```

`ComputedRef`を更新する側（こちらは`proxy`を`props`として（単なる値として）しか認識していないことに注意）:

```vue:Child.vue
<template>
  <button @click="updateProxyValue">update</button>
</template>

<script setup lang="ts">
const { proxy } = defineProps<{
  proxy: { value: number };
}>();

function updateProxyValue() {
  proxy.value += 10;
}
</script>
```

:::message

`<FooComponent :foo>`は`<FooComponent :foo="foo">`と同じ意味です。

おそらく書くならこちらの方がよいです。

書かないものは書かない方が、可読性が高いため。

また変数と`props`が、ちょうど合致していることがわかりやすいため。
（無理やり合わせろという話ではない。むしろ無理やり合わせるなら、`:foo="bar"`のようにした方がいいでしょう。）

:::

:::message

余談ですが、最近のVueはpropsを分割代入しても、reactivityは失われません。
これからはwithDefaultsを使わずに、普通の分割代入のデフォルト値構文を使っていくのがよさそう。

```typescript
const { n = 0 } = defineProps<{
  n?: number;
}>();
```

以下と同じ。

```typescript
const props = withDefaults(
  defineProps<{
    n?: number;
  }>(),
  {
    n: 0,
  },
);
const { n } = toRefs(props);
```

:::

これを防ぐ方法。

```vue:DeepReadonlyHelloWorld.vue
<template>
  <p>core.value is {{ core.value }}</p>
  <DeepReadonlyChild :proxy />
</template>

<script setup lang="ts">
import { ref, computed, readonly } from 'vue'
import DeepReadonlyChild from './DeepReadonlyChild.vue'

const core = ref({ value: 42 })
const proxy = computed(() => readonly(core.value)) // readonlyを追加
</script>
```

```vue:DeepReadonlyChild.vue
<template>
  <button @click="updateProxyValue">update</button>
</template>

<script setup lang="ts">
import type { DeepReadonly } from "vue"

const { proxy } = defineProps<{
  proxy: DeepReadonly<{ value: number }> // DeepReadonlyで期待
}>()

function updateProxyValue() {
  proxy.value += 10 // typecheck error ！
}
</script>
```

## 用語の乱用

この記事では以降、わかりやすさを重視するため、慣習・口語での説明にならい、以下の用語の乱用を行います。

- `computed`: `const x = computed(/* ... */)`のような変数`x`、もしくは`computed`関数そのもの
- `ref`: `const x = ref(/* ... */)`のような変数`x`、もしくは`ref`関数そのもの
- `props`: 子コンポーネントの`const props = defineProps<{ /* ... */ }>()`のような変数`props`

## `computed`を子コンポーネントの`props`に渡しても、変更されないのか？

**変更されます。**

よく考えたら当たり前で

```
<Child :proxy />
const proxy = computed(() => core.value)
```

のような渡し方は、関数でいうシャローコピーの渡しになるからです。

:::details （ソースを読んだ感想です）

間違っていたら教えてください。

- https://github.com/vuejs/core/blob/main/packages/reactivity/src/computed.ts#L206
- https://github.com/vuejs/core/blob/main/packages/reactivity/src/computed.ts#L131
- https://github.com/vuejs/core/blob/main/packages/reactivity/src/effect.ts#L407

:::

```
const { proxy } = defineProps<{
  proxy: { value: number }
}>()

function updateProxyValue() {
  proxy.value += 10
}
```

ここで`proxy`はシャローコピーされた、`const proxy = computed(() => core.value)`とは別のオブジェクトですが、`proxy.value`（後者で言う`proxy.value.value`）は同じ参照先です。

ですので、`updateProxyValue`は`const core = ref({ value: 42 })`の`core.value.value`を更新するというわけですね。

## 結論

オブジェクト型の`computed`のプロパティは、子コンポーネントで更新され得ます。
それを防ぐためには、`vue`の`readonly`（`DeepReadonly`）を使いましょう。

```vue:DeepReadonlyHelloWorld.vue
<template>
  <p>core.value is {{ core.value }}</p>
  <DeepReadonlyChild :proxy />
</template>

<script setup lang="ts">
import { ref, computed, readonly } from 'vue'
import DeepReadonlyChild from './DeepReadonlyChild.vue'

const core = ref({ value: 42 })
const proxy = computed(() => readonly(core.value)) // readonlyを追加
</script>
```

```vue:DeepReadonlyChild.vue
<template>
  <button @click="updateProxyValue">update</button>
</template>

<script setup lang="ts">
import type { DeepReadonly } from "vue"

const { proxy } = defineProps<{
  proxy: DeepReadonly<{ value: number }> // DeepReadonlyで期待
}>()

function updateProxyValue() {
  proxy.value += 10 // typecheck error ！
}
</script>
```

終わり！
