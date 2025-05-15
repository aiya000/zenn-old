# 【Vue3・Nuxt3】最新のVue3で、`<script setup>`で単に値をexportする（defineExposeではない）

## 結論

下記のように、先に`<script>`（`<script setup>`でない）でexportすればよいです。

- [実際に動く環境: Vue3: An example to export consts from components](https://stackblitz.com/edit/vitejs-vite-gplvpntg?file=src%2Fcomponents%2FGoodExportValue.vue)

```vue:Parent.vue
<template>
  <p>{{ goodValue }}</p>
</template>

<script setup lang="ts">
import { value as goodValue } from './components/GoodExportValue.vue'
</script>
```

```vue:GoodExportValue.vue
<template>
  <p>dummy</p>
</template>

<script lang="ts">
import { a } from '../utils/a'
import { b } from '../utils/b'

/**
 * このように、通常のscriptでexportすることによって、正しくexportできる。
 */
export const value = 42
</script>

<script setup lang="ts">
// eslintのimport/firstルールを有効にしていると、eslint --fixしたときに、上述の`export const value`をここ（<script setup>）に持ってこられるので、importはsetupでないscriptで行う。
// その場合も、importしたものは<script setup>内で使えるので、安心してください。
// import { b } from '../utils/b' // <script>でimportするようにする

const { foo = 42 } = defineProps<{
  foo: number
}>()
</script>
```

逆に、下記のように`<script setup>`でexportすることはできません。

```vue:NoGoodExportValue.vue
<template>
  <p>{{ foo }}</p>
</template>

<script setup lang="ts">
/**
 * defineExposeでインスタンスの下に生やしたいわけではなく、
 * 通常のモジュールと同様に、
 * モジュール内の変数として単に公開したい。
 * しかしこれはエラーになる。
 * ```
 * [@vue/compiler-sfc] <script setup> cannot contain ES module exports. If you are using a previous version of <script setup>, please consult the updated RFC at https://github.com/vuejs/rfcs/pull/227.
 * ```
 */
export const value = 42

const { foo = 42 } = defineProps<{
  foo: number
}>()
</script>
```

![](https://storage.googleapis.com/zenn-user-upload/e9431d3a7aa2-20250515.png)

## defineExposeは？

defineExposeはコンポーネントのインスタンスの下に生やすためのものです。
単なる値をexportするためのものではありません。

これは割と勘違いされているのではないかと思います。
わかる～。

```vue:Parent.vue
<template>
  <ChildComponent ref="child" />
  <button @click="check">Call Child Method</button>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const child = useTemplateRef<InstanceType<typeof ChildComponent>>('child')

function check() {
  if (child.value) {
    // defineExposeされたものが、インスタンスに生えている
    child.value.sayHello()
    console.log('Count:', child.value.count)
  }
}
</script>
```

```vue:Child.vue
<template>
  <div>Child</div>
</template>

<script setup lang="ts">
const count = ref(0)

function sayHello() {
  console.log('Hello from child component!')
}

defineExpose({
  sayHello,
  count,
})
</script>
```

## 結論

`<script setup>`ではなく`<script>`でexportせよ。

```vue:GoodExportValue.vue
<template>
  <p>dummy</p>
</template>

<script lang="ts">
export const value = 42
</script>

<script setup lang="ts">
const { foo = 42 } = defineProps<{
  foo: number
}>()
</script>
```

```vue:Parent.vue
<template>
  <p>{{ goodValue }}</p>
</template>

<script setup lang="ts">
import { value as goodValue } from './components/GoodExportValue.vue'
</script>
```

- [実際に動く環境: Vue3: An example to export consts from components](https://stackblitz.com/edit/vitejs-vite-gplvpntg?file=src%2Fcomponents%2FGoodExportValue.vue)
