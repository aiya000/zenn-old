---
title: "【WSL2 × ClamAV】なぜか動かない？その原因と解決策を完全解説！"
emoji: "🐕"
type: "tech"
topics: [Windows, Linux, WSL2, ClamAV, アンチウイルス]
published: true
---
## はじめに

WSL2にClamAVを導入しようとして、「情報が見つからない」「Ubuntuの手順通りにやったのに動かない」という壁にぶつかりませんでしたか？

私もまさにその状況に陥り、試行錯誤の末に動作させることができました。
この記事では、その過程を余すことなく共有します！

「ClamAVを入れたのに動かない！」と悩んでいる方は、ぜひ最後までご覧ください。

## ClamAVのインストール

まずは通常の手順でClamAVをインストールします。

```shell-session
$ sudo apt install clamav clamav-daemon
$ sudo systemctl start clamav-daemon.service
$ sudo systemctl status clamav-daemon.service
```

ここまではスムーズに進むはず……なのですが、WSL2では次のエラーが発生することがあります。

```shell-session
ERROR: Failed to open log file /var/log/clamav/freshclam.log: Permission denied
```

## 発生するエラーとその原因

WSL2環境では、以下の問題が発生することがあります。

1. `clamd`との接続エラー (`Can't connect to clamd`)
2. `freshclam`のログファイルへの書き込み権限エラー (`Permission denied`)
3. `freshclam`のプロセスがロックされてしまう (`Failed to lock the log file`)

これらの問題の解決策を、順を追って解説していきます。

## 解決策 ① ログファイルの権限修正

まず、`freshclam.log`のパーミッションを変更します。

```shell-session
$ sudo chmod 666 /var/log/clamav/freshclam.log
$ ls -l /var/log/clamav/freshclam.log
-rw-rw-rw- 1 clamav clamav 1610 Feb 24 01:19 /var/log/clamav/freshclam.log
```

「666って危険じゃない？」と思うかもしれませんが、ログファイルなので今回は許容しました。

## 解決策 ② ロックされたプロセスを強制終了

次に、ロックされている可能性のある`freshclam`プロセスを全て終了させます。

```shell-session
$ sudo pkill freshclam
```

または、`killall`を使ってもOKです。

## 解決策 ③ WSL2向け設定を適用

WSL2の環境に合わせて、不要なディレクトリをスキャン対象から除外すると動作が安定します。

```shell-session
$ sudoedit /etc/clamav/clamd.conf
```

以下の設定を追記しましょう。

```shell-session
# WSL2向け設定（追加）
ExcludePath ^/proc
ExcludePath ^/sys
ExcludePath ^/run
ExcludePath ^/dev
ExcludePath ^/snap
ExcludePath ^/mnt
```

特に、`/mnt`を除外することでWindows側のファイルスキャンを回避できます。

## 動作確認

ここまで設定すれば、ClamAVがWSL2で正常に動作するはずです。

```shell-session
$ sudo freshclam
Mon Feb 24 01:30:38 2025 -> ClamAV update process started at Mon Feb 24 01:30:38 2025
```

```shell-session
$ clamscan install.sh
----------- SCAN SUMMARY -----------
Known viruses: 8704520
Engine version: 0.103.12
Scanned files: 1
Infected files: 0
```

これでWSL2上でClamAVが動作するようになりました！

## まとめ

WSL2でClamAVがうまく動作しない場合の主なポイントは以下の3点です。

1. ログファイルのパーミッションを適切に設定する。
2. `freshclam`のロック問題を解決する。
3. 不要なディレクトリをスキャン対象から除外する。

WSL2の利用が増える中、Linux環境でのセキュリティ対策はますます重要になっています。
ClamAVを活用して、安全な開発環境を整えましょう！

それでは、よいWSL2ライフを！🎉

## 参考リンク

- [Ubuntu20.04 で ClamAV を使ってウイルススキャン](https://qiita.com/kannkyo/items/1cc32231afad88c11d8e)
- [Clamavをやってみた。そして動かねぇ。](https://www.blog.slow-fire.net/2021/12/13/clamav%E3%82%92%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%81%9F%E3%80%82%E3%81%9D%E3%81%97%E3%81%A6%E5%8B%95%E3%81%8B%E3%81%AD%E3%81%87%E3%80%82/)
