# 単体テスト・結合テスト - HIKKYガイドライン

本稿では、弊社HIKKYのフロントエンドチームプロジェクトで主に使われる、フロントエンドのテストの概要についての案内をします。

- 対象読者:
    - JavaScript・TypeScriptわかる
    - テストってよくわからない
    - テストってなに？
    - どうやるの？

なお、本稿は株式会社HIKKYフロントエンドチーム向けのガイドラインであるものの、**株式会社HIKKY及びその意向等とは無関係であります。**

# テストの種類

テストは主に、次の2種類に分けることができます。

- 手動テスト
- 自動テスト

本稿では**自動テスト**を主題にします！

# 手動テストの概要

**手動テスト**はその名の通り、テストをする人が**実際のテスト対象（サイトや関数）を手動で確認していくもの**です。

例えば、サイトが

- フォームのバリデーション [^validation]
- 例えばお問い合わせ画面（フォーム）のうち、名前などの必須項目が入力されているかどうか、など
- 問い合わせフォームを送ったあとに、期待したページへ遷移するか
- **画面がちゃんとかっこいか**

などを、手動で確認することです。

しかしサイトを改良していくときに、サイトや関数が仕様に沿っているかを**毎回手動で確認するのは、かなりの手間になります**。

:::message
それに手間が多い作業を人間が行うと、怠けが生じて自然なので、ちゃんとチェックしなくなりますからね！
:::

# 自動テストの概要

そこで**テストの実行を自動化するもの**が、**自動テスト**です。

例えば前述のうち

- フォームのバリデーション
- 例えばお問い合わせ画面（フォーム）のうち、名前などの必須項目が入力されているかどうか、など
- 問い合わせフォームを送ったあとに、期待したページへ遷移するか

などを請け負うことができます。

ただし全てを自動テストで行うことは、実はむずかしいです！

例えば

- 画面がちゃんとかっこいいか

などは人間の主観に依存するので、手動テストで行う必要があります。
つまり**全てを自動化できるわけではないのです**。

:::details 手動テスト支援 - Storybookについて

ただし手動テストを支援するツールは存在します。
ここではStorybookを軽く紹介します。

Storybookは、UI部品 [^ui-component] のカタログを表示するためのツールです。

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

しかし自動テストを使えれば、手動テストよりも活躍する場面は多いです。
例えば「粒度の細かいテスト」では手動テストよりも、自動テストの方が正確にテストを行うことができます。
（前述のとおり、多くの何度もテストを繰り返すと、人間は怠けますからね！）

- 粒度の細かいテストの例
    - ある関数の引数としてそれぞれ`41, 42, 43`を渡した場合に、戻り値がそれぞれ`'bang!', 'yummy!', 'bang!'`になるか
    - あるひとつのUI部品のうち、その中のあるボタンを押した場合に、あるテキストがちゃんと更新されるか

## 自動テスト

いくつかの自動テストの種類を紹介していきます。

### 単体テスト

単体テストとは、あるひとつの事柄についての自動テストです。

HIKKYのフロントエンドテストでは、主にVitest・もしくはJestを用います。

- [Vitest Blazing Fast Unit Test Framework](https://vitest.dev/)
- [Jest - Delightful JavaScript Testing](https://jestjs.io/ja/)

Vitest・Jestは関数のテストなどからUIコンポーネントのテストまで、幅広く対応しています。
今回はVitestを、関数の単体テストを実行するためのクライアントとして使用します。

#### 関数の単体テスト

まずは関数の単体テストの例を見てみましょう。
Vitestは標準で、`<リポジトリ>/src/`ディレクトリ配下の、全ての`__tests__`ディレクトリの中にある、`<名前>.spec.ts`を見つけ、実行します。

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

// テスト: range(0, 10) は 10 を含むか？
it('range() includes an argument number of the end', () => {
  const end = 10
  const result = range(0, end)
  expect(result).toContain(end)
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

これで単体テストが実行できるようになりました！

（結果↓）
```
$ vitest

 DEV  v0.32.2 <project-path>

  src/test/utils/Array.spec.ts (1)

 Test Files  1 passed (1)
      Tests  1 passed (1)
```

#### UIコンポーネントの単体テスト

次はコンポーネントの単体テストです。
ここでは[Testing Library](https://testing-library.com/docs/vue-testing-library/examples/)というテスト用ライブラリを使用します。

```vue:src/components/Component.vue
<template>
  <div>
    <p>Times clicked: {{ count }}</p>
    <button @click="increment">increment</button>
  </div>
</template>

<script setup lang="ts">
const count = ref(0)
const increment = () => count.value++
</script>
```

```typescript:src/test/components/Component.spec.ts
import { render, fireEvent } from '@testing-library/vue'
import Component from '@/components/Component.vue'

test('increments value on click', async () => {
  const { getByText } = render(Component)
  getByText('Times clicked: 0')
  const button = getByText('increment')

  await fireEvent.click(button)
  await fireEvent.click(button)

  getByText('Times clicked: 2')
})
```

Array.tsの単体テストと似ていますが、`render()`や`getByText()`・`fireEvent.click()`など、コンポーネントの要素へアクセスしている雰囲気があります。
もちろん、実際そうですよ！

関数の単体テストよりも、**対話的なテスト**になっているのがわかります。

#### スナップショットテストでの、UIコンポーネントの単体テスト

上記ではコードでのコンポーネントのテストを行いましたが、コンポーネントの単体テストは**スナップショットテスト**で行うこともあります。

スナップショットテストとは、次の工程よりなります。

1. ある時点でのコンポーネントの内容を保存しておく
    - この「ある時点でのコンポーネントの内容」を、狭義にスナップショットと呼びます
1. あるコンポーネントをアップデートする
1. 「最初に保存したスナップショット」と「現在の内容」が等しいかをテストする

原理は単純ですね。

具体的なコードを見てみましょう。

```vue:src/components/Component.vue
// どんな内容でもよいので、省略
```

```typescript:src/test/componentes/Component.spec.vue
test('renders correctly', () => {
  const wrapper = mount(Component)
  expect(wrapper.element).toMatchSnapshot()
})
```

`toMatchSnapshot()`といういかにもなメソッドがありますね！
これがスナップショットテストを実行してくれます。

実行は下記で行うことができます。
（前述を参考に、`npx`と`yarn dlx`を、必要に応じて適宜足してください。）

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
    - [Examples | Testing Library](https://testing-library.com/docs/vue-testing-library/examples/)
    - [スナップショットテスト - Jest を使用した単一ファイルコンポーネントのテスト | Vue Test Utils](https://v1.test-utils.vuejs.org/ja/installation/testing-single-file-components-with-jest.html#%E3%82%B9%E3%83%8A%E3%83%83%E3%83%97%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%E3%83%86%E3%82%B9%E3%83%88)
    - [スナップショットテスト - Jest](https://deltice.github.io/jest/docs/ja/snapshot-testing.html)


#### ビジュアルリグレッションテスト（VRT）とは

ビジュアルリグレッションテストとは、その名の通り「見た目に対して」「リグレッション」する（単体）テストです。

以下、ビジュアルリグレッションテストをVRTと呼称します。

##### リグレッションテスト

はて、そのうちの「リグレッションテスト」とは、なんでしょうか。
下記のサイトから引用すると

- [リグレッションテスト 【regression testing】 回帰テスト / 退行テスト](https://e-words.jp/w/%E3%83%AA%E3%82%B0%E3%83%AC%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%86%E3%82%B9%E3%83%88.html)

> ソフトウェア開発の過程で不具合が発見されプログラムが修正されることはよくあるが、その修正によってそれまで正常に動作していた部分が異状をきたすようになることがある。
> このような現象を「デグレード」「リグレッション」などという。
> こうしたことが起きないよう、機能追加やバグ修正などでコードの一部が改修されたあと、
> それまでの動作に変更や問題が起きないことを確認するために実施されるテストをリグレッションテストという。

とのことです。

実はあなたはもう、これを知っています。
**自動テストのことです！**

より詳しく言うと、過去に書いた自動テストのことです。
そう。**蓄積された過去の自動テストは**、テストの自動化だけでなく、**「デグレード」を検知する効果があるのです**。

（
実際は手動テストでも行われることもありますが、とにかくコストがかかるため、手動ではされない場合が多いと思います。
人は怠けるので、デグレードを検知する気すらなくなりますから。
）

##### VRT

つまり**ビジュアル**リグレッションテストとは「**見た目に対する**」「対デグレード用テスト」です！

VRTでは、あるUIコンポーネントの、過去の時点と新しい時点を、**スクリーンショットの比較**で行います。

スナップショットテストと似ていることに気が付きましたか？
そうなんです。スナップショットテストとは、UIコンポーネントの新旧を比較する点が同じです。

それでは、何が違うのでしょうか。
それは「比較を画像で行う」点です。

これは例えばGitHubのPullRequestを上げた際に、変更された箇所をbotコメントにより、目で確認することが可能です。
PRで更新されるコンポーネントの部分が、人間の目で判別できるんです！

　

VRTに関する作業ではほぼコードを書かず、環境の構築が主になるので、コード例は割愛します。

- 詳細
    - [ビジュアルリグレッションテストを導入した話 - メドピア開発者ブログ](https://tech.medpeer.co.jp/entry/2020/04/10/160000)
    - [UI コンポーネントをテストする | Storybook Tutorials](https://storybook.js.org/tutorials/intro-to-storybook/react/ja/test/)
        - この構成では、Storybookを用いています。またStorybook！ すごいですね！

### 結合テスト（統合テスト）

今までは単一の関数や、単一のコンポーネントをテストする、単体テストを見てきました。

比べ、複数の事柄についての自動テストを、**結合テスト**・または**統合テスト**といいます。

ここでは結合テストの方法のひとつとして、E2Eテストを述べます。

#### E2Eテスト

下記サイトから引用すると、E2Eテストは次のようなものです。

- [【E2Eテスト実践ガイド】E2Eテストの実施方法からおすすめツール、現場での実践ポイントまで詳しく解説](https://www.praha-inc.com/lab/posts/e2e-testing)

> E2Eテストとは、ソフトウェアで利用されるコンポーネントやレイヤーを全て結合した状態で、
> レイヤーの初めから終わりまで（End to End）テストすることを指します。
> システムテストなどの呼び方をする場合もあります。
> ユーザーが実際に利用する環境を再現するため、
> Webの場合はブラウザ、モバイルの場合はモバイルデバイスを用いてテストを行います。

うんうん。

「ユーザーが実際に操作するシチュエーション」を再現するために、「実際の環境」でテストを行うんですね！
そして最初から最後までの動作をテストするから、End to Endテストと言う。なるほど！

あれ？

> ユーザーが実際に利用する環境を再現するため、Webの場合はブラウザ……

と書いてありましたが、あれ、実際のブラウザを使うんじゃ自動テストできないんじゃ！？

いえ、大丈夫です。
最近の技術はすさまじく、現在はChrome・Firefox・Webkitといった主要Webブラウザを、プログラム内部で自動操作できちゃうんです！

TODO

今回使用する、目に見えるウィンドウを持たないブラウザを、ヘッドレスブラウザと言います。

ヘッドレス Chrome ことはじめ

用法として、以下のCLIがわかりやすいかもしれません。

PDF を作成する--print-to-pdf フラグを使うと、ページの PDF を作成します。$ chrome --headless --disable-gpu --print-to-pdf https://www.chromestatus.com/

ヘッドレスブラウザ（Chrome）を用いたテストコードを、書いてみましょう。

const timeout = 5000;

describe(
  '/ (Home Page)',
  () => {
    let page;
    beforeAll(async () => {
      page = await global.__BROWSER__.newPage();
      await page.goto('https://google.com');
    }, timeout);

    it('should load without error', async () => {
      const text = await page.evaluate(() => document.body.textContent);
      expect(text).toContain('google');
    });
  },
  timeout,
);

引用元: puppeteer を使用する

以下の点など、特徴的ですね。

const timeoutを設定している

本当に「ユーザーが実際に利用する環境の再現」なので、タイムアウトが発生し得ます ※

await page.goto('https://google.com');

本当に実在するページにアクセスしてそう

なんだかとっても、「実際の環境」という感じがします！



自動テストでのタイムアウトの扱い

E2Eテストではタイムアウトが発生し得ます。
では単体テストや結合テストでは、タイムアウトは発生しないのでしょうか？
必ず実行が完遂されるのでしょうか？（停止するのでしょうか）

筆者の答えとしては「タイムアウトは発生させるな！ 自動テストは必ず完遂できろ！ 絶対だぞ！」です。

というのも、開発者によってはTest Driven Development（TDD）ないし・それに近いことを行っている場合があり、テスト全体の実行時間は短ければ短いほどよいからです。
（ここでTDDの解説は本稿の対象外なので、割愛します。）

単体テストは停止するようにしよう！
（無限ループとか超こわいね！）



詳細

puppeteer を使用する

Nuxt.js に後から E2E テスト (puppeteer,  jest-puppeteer) を入れる - Qiita





Advanced

以下、より高度なテスト技法についての参考リンクです。

興味のある方は、ぜひ読んでみてください！

境界値テスト

テストケースを作成する手法「境界値分析」について解説｜ソフトウェアテストのSHIFT

Property Based Testing

fast-check で Property Based Testing を試してみる

dubzzz/fast-check: Property based testing framework for JavaScript (like QuickCheck) written in TypeScript

doctest

doctestの使い方メモ - Qiita

「python標準ライブラリです」と書かれていますが、実際は多くの言語に文化として取り入れられています



終わり

おつかれさまでした！

あなたは本稿で、以下のことを学びました。

テストの種類

手動テストの概要

自動テストの概要

手動テスト

Storybook

自動テスト

単体テスト

関数の単体テスト

コンポーネントの単体テスト

ビジュアルリグレッションテスト

E2Eテスト

それぞれの詳細を理解する場合は、本稿に載せた詳細リンク・公式サイト・他の解説サイトを巡ってみて、実際に使ってみるのがおすすめです！

それでは！

- - - - -

[^validation]: 例えばお問い合わせ画面（フォーム）のうち、名前などの必須項目が入力されているかどうか、などの確認のことです。
[^ui-component]: UIコンポーネント。あるひとつのUI部品・要素のこと。だいたいの場合、略してコンポーネントと言います。Vue.jsでは.vueファイル・React.jsでは.tsxファイルの、一つ一つがコンポーネントになります。
