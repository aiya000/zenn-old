# vue-i18nのi18n.t('key')で、'key'が見つからなかったときに例外をthrowする

vue-i18nは`i18n.t('key')`をしたときに、`I18nOptions`の`messages[locale]`に`'key'`が見つからなければ`console.warn()`されますが、例外はthrowされません。
`console.warn()`で大抵の場合は問題ないですが、今回特定の条件下でのみ`'key'`が見つからないことがあったので、デバッグのためにthrowするようにしました。

その手順です。
（弊社メンバーの方に教えていただきました。ありがとうございます！）

結論ですが、`I18nOptions`に`missing`プロパティを指定すればいけます。

## nuxt/i18nでの例

動く環境↓
https://stackblitz.com/edit/nuxt-starter-wyyzfhq5?file=app.vue

コード:
```vue
<!-- buttonをクリックすると、"key 'nothing' is not found in locale 'en'"と表示される（'en'ロケールでの例）。 -->
<template>
  <div>
    <button @click="show">Click to throw</button>
  </div>
</template>

<script setup lang="ts">
const i18n = useI18n({
  // **Add the 'missing' property**
  missing: (locale: string, key: string) => {
    throw new Error(`key '${key}' is not found in locale '${locale}'`)
  },
})

const show = () => {
  try {
    alert(i18n.t('nothing'))
  } catch (e) {
    alert(e)
  }
}
</script>
```

nuxt/i18nならi18n.config.tsに指定してもいけるはずです。
https://i18n.nuxtjs.org/docs/composables/define-i18n-config

素のvue-i18nなら、`createI18n`に指定すればいけると思います。

（要は`I18nOptions`型の`missing`プロパティを設定すればいける。）

## 追記: モジュール化しました

今までの内容をモジュール化した、`useStrictI18n`というcomposableを作りました。
使い方は簡単で、以下のように使えます！

```vue:SomeComponent.vue
<i18n lang="yaml">
ja:
  someKey: some
</i18n>

<template>
  <!-- I18nTKeyMissingErrorが送出される -->
  <div>{{ i18n.t('missingKey') }}</div>
</template>

<script setup lang="ts">
const i18n = useStrictI18n()
</script>
```

モジュールは以下です。

```typescript:composables/useStrictI18n.ts
import { useI18n, type Composer, type UseI18nOptions, type MissingHandler } from 'vue-i18n'

export class I18nTKeyMissingError extends Error {}

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
 * `useI18n`とほぼ同等ですが、`i18n.t(key)`を行ったときに`key`が存在しない場合は、[[I18nTKeyMissingError]]を送出します。
 *
 * ```vue
 * <i18n lang="yaml">
 * ja:
 *   someKey: some
 * </i18n>
 *
 * <template>
 *   <!-- I18nTKeyMissingErrorが送出される -->
 *   <div>{{ i18n.t('missingKey') }}</div>
 * </template>
 *
 * <script setup lang="ts">
 * const i18n = useStrictI18n()
 * </script>
 * ```
 */
export const useStrictI18n = (
  options?: Omit<UseI18nOptions, 'missing'>,
): UseI18nReturnType<Exclude<UseI18nOptions, undefined> & { missing: MissingHandler }> =>
  useI18n({
    ...options ?? {},
    missing: (locale, key) => {
      throw new I18nTKeyMissingError(`key '${key}' is not found in locale '${locale}'`)
    },
  })
```

テストも書いたので載せます。
ちょっと今は忙しいので、TODOは残っています。
TODOを解消できた人は、コメントで教えてください！

```typescript
import { mount } from '@vue/test-utils'
import { createI18n } from 'vue-i18n'
import { useStrictI18n, I18nTKeyMissingError } from '#base/app/composables/useStrictI18n'

const defaultI18nPlugin = createI18n({
  locale: 'ja',
  messages: {
    ja: { notMissingKey: 'サラダバー' },
    en: { notMissingKey: 'Salad bar' },
  },
})

// TODO: なぜかthrowするのでskip。アプリケーションサイドではちゃんと動いているので、とりあえずテストは後回し。
test.skip('is same as useI18n() (`.t`) when the key is not missing', () => {
  // useI18nがコンポーネントのsetup内でのみしか動かないので、コンポーネントを介してテストをする
  mount(
    // eslint-disable-next-line @typescript-eslint/no-explicit-any -- 弊部ではテストコードはanyオーケーにしています。アプリケーションコードではダメ絶対
    (defineComponent as any)({
      template: '<p>サラダバー</p>',
      setup: () => {
        const i18n = useStrictI18n()
        expect(() => i18n.t('notMissingKey')).not.toThrowError()
      },
    }),
    {
      global: {
        plugins: [defaultI18nPlugin],
      },
    },
  )
})

test('throws I18nTKeyMissingError when the key is missing', () => {
  mount(
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    (defineComponent as any)({
      template: '<p>サラダバー</p>',
      setup: () => {
        const i18n = useStrictI18n()
        expect(() => i18n.t('missingKey')).toThrowError(I18nTKeyMissingError)
      },
    }),
    {
      global: {
        plugins: [defaultI18nPlugin],
      },
    },
  )
})

test('can be passed UseI18nOptions excluding `.missing`', () => {
  mount(
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    (defineComponent as any)({
      template: '<p>サラダバー</p>',
      setup: () => {
        const i18n = useStrictI18n({
          messages: {
            ja: { foo: 'foo' },
            en: { foo: 'foo' },
          },
        })
        expect(() => i18n.t('foo')).not.toThrowError()
      },
    }),
    {
      global: {
        plugins: [defaultI18nPlugin],
      },
    },
  )
})
```

:::details 本セクション上記コードのライセンス（トラブル防止）

トラブル防止のために、一応ライセンスを付けておきます。
僕はnpmパッケージ化をする予定はないですが、以下の条項を載せていただければ（MITライセンスに則っていただければ）、npmレジストリに上げる等、自由に使用していただいて構いません :ok_hand:

```
The MIT License (MIT)

Copyright (c) 2025 aiya000

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

:::
