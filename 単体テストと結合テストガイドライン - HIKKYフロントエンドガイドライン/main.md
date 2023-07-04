# 単体テストと結合テストガイドライン - HIKKYフロントエンドガイドライン

本稿では、弊社HIKKYのフロントエンドチームプロジェクトで主に使われる、フロントエンドのテストの概要についての案内をします。

- 対象読者:
    - JavaScript・TypeScriptわかる
    - テストってよくわからない
        - テストってなに？
        - どんなのがあるの？
        - どうやるの？

なお本稿は株式会社HIKKYフロントエンドチーム向けのガイドラインであるものの、内容については筆者に一任されており、**株式会社HIKKY及びその意向等とは無関係です。**

# テストの種類

テストは主に、次の2種類に分けることができます。

- 手動テスト
- 自動テスト

本稿では**自動テスト**を主題にします！

# 手動テストの概要

その名の通り、テストをする人が**実際のテスト対象（サイトや関数）を手動で確認していくもの**が、**手動テスト**です。

例えば、サイトが

- `1` フォームのバリデーション [^validation]
- `2` 問い合わせフォームを送ったあとに、期待したページへ遷移するか
- `3` 画面がちゃんとかっこいか

などを、手動で確認することです。

しかしサイトを改良していくときに、サイトや関数が仕様に沿っているかを**毎回手動で確認するのは、かなりの手間になります**。

:::message

それに手間が多い作業を人間が行うと、怠けが生じて自然なので、ちゃんとチェックしなくなりますからね！

:::

# 自動テストの概要

そこで**テストの実行を自動化するもの**が、**自動テスト**です。

例えば前述のうち

- `1` フォームのバリデーション
- `2` 問い合わせフォームを送ったあとに、期待したページへ遷移するか

などを請け負うことができます。

しかしながら、全てを自動テストで行うことは、実はむずかしいのです！

例えば

- `3` 画面がちゃんとかっこいいか

などは人間の主観に依存するので、手動テストで行う必要があります。
つまり**全てを自動化できるわけではないんです**。

しかし自動テストを使えれば、手動テストよりも活躍する場面は多いです。
例えば「粒度の細かいテスト」では手動テストよりも、自動テストの方が正確にテストを行うことができます。
（前述のとおり、多くの何度もテストを繰り返すと、人間は怠けますからね！）

- 粒度の細かいテストの例
    - ある関数の引数としてそれぞれ`41, 42, 43`を渡した場合に、戻り値がそれぞれ`'bang!', 'yummy!', 'bang!'`になるか
    - あるひとつのUIコンポーネント[^ui-component]のうち、その中のあるボタンを押した場合に、あるテキストがちゃんと更新されるか

:::details 手動テスト支援 - Storybookについて

手動テストを支援するツールは存在します。
ここではひとつの例として、Storybookを軽く紹介します。

Storybookは、UIコンポーネントのカタログを表示するためのツールです。

開発者側としての利点は、コンポーネントを一度コードに組み込むことで、継続的な変更の監視ができるところです。前述のとおり、画面がちゃんとかっこいいかなども確認することができます。

Storybookの役割は多岐にわたるのでここでは述べませんが、手動テストの大きな味方です！

さらなる情報が必要なら、下記の詳細をご覧ください。

- 詳細
    - [Storybook for Vue tutorial](https://storybook.js.org/tutorials/intro-to-storybook/vue/en/get-started/)
        - 英語読める人向け
        - 弊社はNuxt.js（Vue.jsのフレームワーク）を使うことが多いので、可能であればこちらをおすすめします
        - こちらは日本語版がない
    - [React 向け Storybook のチュートリアル](https://storybook.js.org/tutorials/intro-to-storybook/react/ja/get-started/)
        - 日本語読める人向け
        - ただしReact.js向け＆バージョンが古い場合がある
        - Storybook自体を習得する分には、読むのはこちらでも十分

:::

# 自動テスト

次に、自動テストの各手法を説明していきます。

## 単体テスト

単体テストとは、あるひとつの事柄についての自動テストです。

HIKKYのフロントエンドテストでは、主にVitest・もしくはJestを用います。

- [Vitest Blazing Fast Unit Test Framework](https://vitest.dev/)
- [Jest - Delightful JavaScript Testing](https://jestjs.io/ja/)

Vitest・Jestは関数のテストなどからUIコンポーネントのテストまで、幅広く対応しています。
今回はVitestを、関数の単体テストを実行するためのクライアントとして、説明に使用します。

### 関数の単体テスト

まずは関数の単体テストの例を見てみましょう。
Vitestは標準で、`<リポジトリ>/src/`ディレクトリ配下[^repository]の、全ての`__tests__`ディレクトリの中にある任意の`<名前>.spec.ts`を見つけて実行します。

ただし弊社フロントエンドチームの多くのプロジェクトでは、それよりも`src/test/`ディレクトリ配下の、任意の`<名前>.spec.ts`を実行するように設定してあります。

では、具体的な単体テストを見てみましょう。

```typescript:src/utils/Array.ts
function range(start: number, end: number): Array<number> {
  // 実装
}
```

```typescript:src/test/utils/Array.spec.ts
import { test, expect } from 'vitest'
import { range } from '@/utils/Array' // src/utils/Array.ts をimportする

// テスト: range(0, 10) は 10 を含むか？（含むだろう）
it('range() includes an argument number of the end', () => {
  const end = 10
  expect(range(0, end)).toContain(end)
})
```

Vitestを実行するには、次のCLIコマンドを実行します。

```shell-session
$ vitest
```

もしくはnpmを使っている場合は下記を

```shell-session
$ npx vitest
```

yarnを使っている場合は下記を実行してください。

```shell-session
$ yarn dlx vitest
```

（これ以降は`npx`と`yarn dlx`を、適宜必要に応じて、足して読んでください。）

これで単体テストが実行できるようになりました！

（結果↓）
```
$ vitest

 DEV  v0.32.2 <project-path>

  src/test/utils/Array.spec.ts (1)

 Test Files  1 passed (1)
      Tests  1 passed (1)
```

### UIコンポーネントの単体テスト

次はコンポーネントの単体テストです。
ここではVue.jsと[Test Utils](https://v1.test-utils.vuejs.org/ja/installation/testing-single-file-components-with-jest.html#%E3%82%B9%E3%83%8A%E3%83%83%E3%83%97%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%E3%83%86%E3%82%B9%E3%83%88)を使ってみます。

```vue:src/components/Component.vue
<template>
  <div>
    <p class="result">Times clicked: {{ count }}</p>
    <button class="button" @click="increment">increment</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
const increment = () => count.value++
</script>

```

```typescript:src/test/components/Component.spec.ts
import { describe, test, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Component from '@/components/Component.vue'

test('increments value on click', async () => {
  const wrapper = mount(Component)

  await wrapper.get('.button').trigger('click')
  await wrapper.get('.button').trigger('click')

  const result = wrapper.get('.result').text()
  expect(result).toBe('Times clicked: 2')
})

```

Array.tsの単体テストと似ていますが、`.get()`や`.trigger('click')`など、コンポーネントの要素へアクセスしている雰囲気がありますよね。
もちろん、実際そうですよ！

関数の単体テストよりも、**対話的なテスト**になっているのがわかりますね。

- 詳細
    - [Jest を使用した単一ファイルコンポーネントのテスト | Vue Test Utils](https://v1.test-utils.vuejs.org/ja/installation/testing-single-file-components-with-jest.html#jest-%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%9F%E5%8D%98%E4%B8%80%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%AE%E3%83%86%E3%82%B9%E3%83%88)

### スナップショットテストでの、UIコンポーネントの単体テスト

上記ではコードでのコンポーネントのテストを行いましたが、コンポーネントの単体テストは**スナップショットテスト**で行うこともあります。

スナップショットテストは、**以前のコンポーネントと現在のコンポーネントが等しいかどうか**をテストするものです。
これにより、意図せぬコンポーネントの変更を防ぐことができます。

スナップショットテストは、次の工程よりなります。

1. ある時点でのコンポーネントの内容を保存しておく
    - この「ある時点でのコンポーネントの内容[^snapshot-content]」を、狭義にスナップショットと呼びます
1. あるコンポーネントをアップデートする
1. 「最初に保存したスナップショット」と「現在の内容」が等しいかをテストする

原理は単純ですね。

具体的なコードを見てみましょう。

```vue:src/components/Component.vue
// どんな内容でもよいので、省略
```

```typescript:src/test/componentes/Component.spec.ts
test('renders correctly', () => {
  const wrapper = mount(Component)
  expect(wrapper.element).toMatchSnapshot()
})
```

`toMatchSnapshot()`といういかにもなメソッドがありますね！
これがスナップショットテストを実行してくれます。

実行は下記で行うことができます。

Vitestの場合:

```shell-session
$ vitest --update
```

Jestの場合:

```shell-session
$ jest --updateSnapshot
```

これでスナップショットが実行されました！

さらなる情報が必要なら、下記の詳細をご覧ください。

- 詳細
    - [スナップショットテスト - Jest を使用した単一ファイルコンポーネントのテスト | Vue Test Utils](https://v1.test-utils.vuejs.org/ja/installation/testing-single-file-components-with-jest.html#%E3%82%B9%E3%83%8A%E3%83%83%E3%83%97%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%E3%83%86%E3%82%B9%E3%83%88)
    - [スナップショットテスト - Jest](https://deltice.github.io/jest/docs/ja/snapshot-testing.html)

### ビジュアルリグレッションテスト（VRT）とは

皆さん読むのが疲れてきたでしょうから、最後の単体テストを紹介して、単体テストの説明を終わりにします！
**ビジュアルリグレッションテスト**です！

ビジュアルリグレッションテストとは、その名の通り「見た目に対して」「リグレッション」する（単体）テストです。

以下、ビジュアルリグレッションテストをVRTと呼称します。

#### リグレッションテスト

はて、そのうちの「リグレッションテスト」とは、なんでしょうか。
下記のサイトから引用すると

- [リグレッションテスト 【regression testing】 回帰テスト / 退行テスト](https://e-words.jp/w/%E3%83%AA%E3%82%B0%E3%83%AC%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%86%E3%82%B9%E3%83%88.html)

> ソフトウェア開発の過程で不具合が発見されプログラムが修正されることはよくあるが、その修正によってそれまで正常に動作していた部分が異状をきたすようになることがある。
> このような現象を「デグレード」「リグレッション」などという。
> こうしたことが起きないよう、機能追加やバグ修正などでコードの一部が改修されたあと、
> それまでの動作に変更や問題が起きないことを確認するために実施されるテストをリグレッションテストという。

とのことです。

実はあなたはもう、これを知っています。
**自動テストそれ自体のことです！**

より詳しく言うと、過去に書いた自動テストを実行して、結果を確認する行動のことです。
そう。蓄積された過去の自動テストの再実行は、**「デグレード」を検知する効果があるのです**。

（
実際は手動テストでも行われることもありますが、とにかくコストがかかるため、手動ではされない場合も多いと思います。
人は怠けるので、デグレードを検知する気すらなくなりますもんね！
）

#### つまり

つまり**ビジュアル**リグレッションテストとは「**見た目に対する**」「対デグレード用テスト」です！

VRTでは、あるUIコンポーネントの、過去の時点と新しい時点を、**スクリーンショットの比較**で行います。

スナップショットテストと似ていることに気が付きましたか？
そうなんです。スナップショットテストとは、UIコンポーネントの新旧を比較する点が同じです。

それでは、何が違うのでしょうか。
それは「**比較を画像で行う**」点です。

これは例えばGitHubのPullRequestを上げた際に、変更された箇所をbotコメントにより、目で確認することが可能です。
PRで更新されるコンポーネントの部分が、**人間の目で判別できる**んです！

:::message

VRTに関する作業ではほぼコードを書かず、環境の構築が主になるので、コード例は割愛します。

:::

- 詳細
    - [ビジュアルリグレッションテストを導入した話 - メドピア開発者ブログ](https://tech.medpeer.co.jp/entry/2020/04/10/160000)
    - [UI コンポーネントをテストする | Storybook Tutorials](https://storybook.js.org/tutorials/intro-to-storybook/react/ja/test/)
        - この構成では、Storybookを用いています。またStorybook！ すごいですね！

## 結合テスト（統合テスト）

今までは単一の関数や、単一のコンポーネントをテストする、単体テストを見てきました。

比べ、複数の事柄についての自動テストを、**結合テスト**・または**統合テスト**といいます。

ここでは結合テストの方法のひとつとして、E2Eテストを述べます。

### E2Eテスト

下記サイトから引用すると、E2Eテストは次のようなものです。

- [【E2Eテスト実践ガイド】E2Eテストの実施方法からおすすめツール、現場での実践ポイントまで詳しく解説](https://www.praha-inc.com/lab/posts/e2e-testing)

> E2Eテストとは、ソフトウェアで利用されるコンポーネントやレイヤーを全て結合した状態で、
> レイヤーの初めから終わりまで（End to End）テストすることを指します。
> システムテストなどの呼び方をする場合もあります。
> ユーザーが実際に利用する環境を再現するため、
> Webの場合はブラウザ、モバイルの場合はモバイルデバイスを用いてテストを行います。

うんうん。

「ユーザーが実際に操作するシチュエーション」を再現するために、「実際の環境」でテストを行うんですね！
そして**最初から最後までの動作をテストする**から、**End to Endテスト**と言う。なるほど！

あれ？

> ユーザーが実際に利用する環境を再現するため、Webの場合はブラウザ……

と書いてありましたが、あれ、実際のブラウザを使うんじゃ自動テストできないんじゃ！？

いえ、大丈夫です。
最近の技術はすさまじく、現在はChrome・Firefox・Webkitといった主要Webブラウザを、**プログラム内部で自動操作**できちゃうんです！

今回使用する、目に見えるウィンドウを持たないブラウザを、**ヘッドレスブラウザ**と言います。

- 詳細
    - [ヘッドレス Chrome ことはじめ](https://developer.chrome.com/blog/headless-chrome/)

ヘッドレスブラウザをプログラミングで操作するライブラリは、主に2つあります。
playwrightとpuppeteerです。

- [Fast and reliable end-to-end testing for modern web apps | Playwright](https://playwright.dev/)
- [Puppeteer | Puppeteer](https://pptr.dev/)

この2つを比べると、playwrightはテストに重きを置いているので、今回はplaywrightを使いましょう。

では実際に、E2Eテストのコードを、書いてみます。

```typescript
import { test, expect } from '@playwright/test';

test('has title', async ({ page }) => {
  await page.goto('https://playwright.dev/');

  // Expect a title "to contain" a substring.
  await expect(page).toHaveTitle(/Playwright/);
});

test('get started link', async ({ page }) => {
  await page.goto('https://playwright.dev/');

  // Click the get started link.
  await page.getByRole('link', { name: 'Get started' }).click();

  // Expects the URL to contain intro.
  await expect(page).toHaveURL(/.*intro/);
});
```

- 引用元: [Writing tests | Playwright](https://playwright.dev/docs/writing-tests#first-test)

コンポーネントの結合テストで見たコードと同じような、**対話的なテスト**なのがわかります！

:::details 自動テストでのタイムアウトの扱い

ここで今までスルーしてきましたが、テストは場合によって、タイムアウトが発生し得ます。

ではテストでは、タイムアウトは発生してもよいように設計すべきでしょうか？
筆者の答えとしては「タイムアウトは発生させるな！ **自動テストは必ず完遂させろ！** 絶対だぞ！」です。

というのも、開発者によってはTest Driven Development（TDD）ないし・それに近いことを行っている場合があり、テスト全体の実行時間は短ければ短いほどよいからです。
（ここでTDDの解説は本稿の対象外なので、割愛します。）

テストは停止するようにしよう！

:::

# さいごに

以下、より高度なテスト技法についての参考リンクです。

興味のある方は、ぜひ読んでみてください！

- 境界値テスト
    - [テストケースを作成する手法「境界値分析」について解説｜ソフトウェアテストのSHIFT](https://service.shiftinc.jp/column/4792/)

- Property Based Testing
    - [fast-check で Property Based Testing を試してみる](https://zenn.dev/ryo_kawamata/articles/22d4408bd1f138)
    - [dubzzz/fast-check: Property based testing framework for JavaScript (like QuickCheck) written in TypeScript](https://github.com/dubzzz/fast-check)

- doctest
    - [doctestの使い方メモ - Qiita](https://docs.python.org/ja/3/library/doctest.html)

# おわり

おつかれさまでした！

あなたは本稿で、以下のことを学びました。

- テストの種類
    - 手動テストの概要
    - 自動テストの概要
    - 自動テスト
        - 単体テスト
            - 関数の単体テスト
            - コンポーネントの単体テスト
            - ビジュアルリグレッションテスト
        - 結合テスト
            - E2Eテスト

それぞれの詳細を理解する場合は、本稿に載せた詳細リンク・公式サイト・他の解説サイトを巡ってみて、実際に使ってみるのがおすすめです！

それでは！

- - - - -

[^validation]: 例えばお問い合わせ画面（フォーム）のうち、名前などの必須項目が入力されているかどうか、などの確認のことです。
[^ui-component]: UIコンポーネント・あるいはUI部品など。あるひとつの画面のうち、部品・要素のこと。だいたいの場合、略してコンポーネントと言います。Vue.jsでは.vueファイル・React.jsでは.tsxファイルの、一つ一つがコンポーネントになります。
[^repository]: ここで`<リポジトリ>`は、git-cloneなどによりコピーされたプロジェクトのルートのことを指します。
[^snapshot-content]: この「内容」は、コードであるか・もしくは画像である場合があります。画像である場合は後述のビジュアルリグレッションテストで述べるので、ここではコードとします。具体的には['スナップショットテスト - Jest'](https://deltice.github.io/jest/docs/ja/snapshot-testing.html)を参考にしてください。
