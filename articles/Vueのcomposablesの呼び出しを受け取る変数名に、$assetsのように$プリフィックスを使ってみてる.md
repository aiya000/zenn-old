# Vueのcomposablesの呼び出しを受け取る変数名に、$assetsのように$プリフィックスを使ってみてる

## 結論

下記のような感じ。

```vue
<script setup lang="ts">
const $assets = useAssets()
</script>
```

そもそも「（Vue3の）composablesとは？」は下記を参照してください。

- [コンポーザブル - Vue.js](https://ja.vuejs.org/guide/reusability/composables)

簡単に言うと、Reactのhooksと近いものです。

## $を使っていなかったときに起こった問題

例えば上記の例に則って`useAssets`を呼び出すとすると、今までは下記のようにしていました。

```vue
<script setup lang="ts">
const assets = useAssets()
</script>
```

しかしこれは、本当にアセット群を表す変数`assets`を使いたいときに、バッティングをしてしまい、都度頭を悩ませていました。

```vue
<template>
  <AssetList :assets />
</template>

<script setup lang="ts">
const assets = useAssets()
const assets = computed(() => assets.fooAssets?.value ?? []) // 名前がバッティングしている
</script>
```

`Composable`や`Comp`をサフィックスにすることも考えましたが、普通にかっこわるい…。

```vue
<template>
  <AssetList :assets />
</template>

<script setup lang="ts">
const assetsComp = useAssets() // かっこわるい
const assets = computed(() => assetsComp.fooAssets?.value ?? [])
</script>
```

## $を使うようにした

ということで、結論に戻ります。

```vue
<template>
  <AssetList :assets />
</template>

<script setup lang="ts">
const $assets = useAssets() // バッティングしないし、かっこいい！
const assets = computed(() => $assets.fooAssets?.value ?? [])
</script>
```

Nuxtプラグインとネーミングが被る気もしますが
（Nuxtプラグインについて: [Providing Helpers - plugins - Nuxt](https://nuxt.com/docs/guide/directory-structure/plugins#providing-helpers) ）

```vue
<script setup lang="ts">
const $assets = useAssets()
const { $toast } = useNuxtApp() // こちらも$プリフィックス
</script>
```

それは一貫性があるということだと思ったので、今はこれを採用しています。
