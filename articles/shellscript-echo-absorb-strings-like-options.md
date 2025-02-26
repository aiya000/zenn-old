# 【シェルスクリプト】`$ echo '-eq'`みたいにオプションっぽい文字列を表示しようとすると、無が表示される

単純だけど、そういうことです。

```shell-session
$ echo '-eq'
（無）
```

おそらく`echo`コマンドの`-eq`オプションを実行しようとしているんだと思う。
おまえ`-eq`オプションなんてないだろうが！！！

解決策は`printf`です。
`printf`しか勝たん。

```shell-session
$ printf '%s\n' '-eq'
-eq
```

ちなみに以下でもムダムダムダァです。
全部吸収するじゃん…そりゃそうなんだけど。

```shell-session
$ x=$(printf '%s\n' '-eq')
$ echo "$x"
（無）
```

こいつなんでも食いやがる…。
なんも信じられん。

ちなみになんでこんな異常行動をしようと思ったかというと、Bash/Zsh向けの[Jest](https://jestjs.io/ja/docs/expect)みたいなものを作ろうと思ったからです。
こういうやつ。

```shell-session
$ x=10
$ expects "$x" to_be 10 && echo Success
Success
```

```shell-session
$ x=42
$ expects "$x" to_be 10
FAIL: expected {actual} to_be 10, but {actual} is: 42
```

需要あるのか？

- [bin/expects - aiya000/bash-toys](https://github.com/aiya000/bash-toys/blob/main/bin/expects)

あと、なんで修正方法聞いたのに、全く教えてくれなかったんだよ、GitHub Copilot Chatたん～～～！！！
