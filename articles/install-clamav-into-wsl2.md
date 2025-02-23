---
title: "【WSL2アンチウイルス】WSL2にClamAV（アンチウイルスソフト）を入れる"
emoji: "🐕"
type: "tech"
topics: [Windows, Linux, WSL2, ClamAV, アンチウイルス]
published: true
---
## 問題

記事の初めから「問題」というのは恐縮ですが、WSL2にClamAVを入れる方法がなかなか見つからず、また通常のUbuntu向け情報の通りだとうまくいかなかったので、ここに記録しておきます。

## 手順

まずは通常通り、ClamAVをインストールします。 [^apt-fast]

Ubuntu（Linux, systemd）を運用しなれていない人向けに説明すると、`systemctl`はサービスの起動、停止、再起動、状態確認などを行うコマンドです。

```shell-session
$ sudo apt install clamav clamav-daemon
... # 省略
$ sudo systemctl start clamav-daemon.service
... # 省略
$ sudo systemctl status clamav-daemon.service
● clamav-daemon.service - Clam AntiVirus userspace daemon
     Loaded: loaded (/lib/systemd/system/clamav-daemon.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/clamav-daemon.service.d
             └─extend.conf
     Active: active (running) since Mon 2025-02-24 01:19:09 JST; 4s ago
       Docs: man:clamd(8)
             man:clamd.conf(5)
             https://docs.clamav.net/
    Process: 558290 ExecStartPre=/bin/mkdir -p /run/clamav (code=exited, status=0/SUCCESS)
    Process: 558291 ExecStartPre=/bin/chown clamav /run/clamav (code=exited, status=0/SUCCESS)
   Main PID: 558292 (clamd)
      Tasks: 1 (limit: 38432)
     Memory: 772.1M
     CGroup: /system.slice/clamav-daemon.service
             └─558292 /usr/sbin/clamd --foreground=true

$ sudo systemctl start clamav-freshclam.service
... # 省略
$ sudo systemctl status clamav-freshclam.service
● clamav-freshclam.service - ClamAV virus database updater
     Loaded: loaded (/lib/systemd/system/clamav-freshclam.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-02-24 01:18:40 JST; 1min 8s ago
       Docs: man:freshclam(1)
             man:freshclam.conf(5)
             https://docs.clamav.net/
   Main PID: 557725 (freshclam)
      Tasks: 1 (limit: 38432)
     Memory: 237.5M
     CGroup: /system.slice/clamav-freshclam.service
             └─557725 /usr/bin/freshclam -d --foreground=true
... # 省略
Feb 24 01:19:01 Kanako freshclam[557725]: Mon Feb 24 01:19:01 2025 -> ^Clamd was NOT notified: Can't connect to clamd through /var/run/cla>
```

ここで問題が起きているようです。
`Can't connect to clamd through /var/run/clamav/clamd.ctl`というエラーが出ています。
これは、ClamAVのデーモンが起動できていないこと示しています。

試しに`freshclam`コマンド（データベースの更新）を実行してみても、エラーが出てます。

```shell-session
$ sudo freshclam
ERROR: Failed to open log file /var/log/clamav/freshclam.log: Permission denied
ERROR: Problem with internal logger (UpdateLogFile = /var/log/clamav/freshclam.log).
ERROR: initialize: libfreshclam init failed.
ERROR: Initialization error!
```

どうやら、`/var/log/clamav/freshclam.log`のパーミッションが間違っているようです。

```shell-session
$ ls -l /var/log/clamav/freshclam.log
-rw-r----- 1 clamav clamav 1610 Feb 24 01:19 /var/log/clamav/freshclam.log
```

`/var/log/clamav/freshclam.log`のパーミッションを変更してみます。

```shell-session
$ sudo chmod 666 /var/log/clamav/freshclam.log
$ ls -l /var/log/clamav/freshclam.log
-rw-rw-rw- 1 clamav clamav 1610 Feb 24 01:19 /var/log/clamav/freshclam.log
```

666は危険かと思うかもしれませんが、ログファイルなので今回は許容しました。
次もまだエラーが出ます。

```shell-session
$ sudo freshclam
ERROR: Failed to lock the log file /var/log/clamav/freshclam.log: Resource temporarily unavailable
ERROR: Problem with internal logger (UpdateLogFile = /var/log/clamav/freshclam.log).
ERROR: initialize: libfreshclam init failed.
ERROR: Initialization error!
```

ファイルのロックが取得できていない雰囲気があるので、怪しいプロセスを全員殺します。
イヤーーーーッッッ！！！！！！！

```shell-session
$ sudo pkill freshclam
```

ここでは`pkill`を使いましたが、`killall`でもよいでしょう。

:::details $ sudo chown clamav:clamav /var/log/clamav/freshclam.log

これもしましたが、そもそもオーナーはclamav:clamavだったので、不要かもしれません。
一応メモとして残しておきます。

```shell-session
$ sudo chown clamav:clamav /var/log/clamav/freshclam.log
```


:::

次は設定を変更します。
`/etc/clamav/freshclam.conf`を編集して、`NotifyClamd /etc/clamav/clamd.conf`をコメントアウトします。

今回はdiffを見せやすいように、/etc/clamav/freshclam.conf-newというファイルを作成して変更点を見せていますが、通常の作業では直接`sudoedit`してかまいません。

```shell-session
$ sudo cp /etc/clamav/freshclam.conf{,-new}
$ sudoedit /etc/clamav/freshclam.conf-new  # `NotifyClamd /etc/clamav/clamd.conf`をコメントアウトする
$ diff /etc/clamav/freshclam.conf{,-new}
27c27,28
< NotifyClamd /etc/clamav/clamd.conf
---
> # below is commented out by aiya000
> # NotifyClamd /etc/clamav/clamd.conf
$ sudo mv -f /etc/clamav/freshclam.conf{-new,}
```

ちなみにシェルの`{a,b}`という表現は`a b`の略記です。
例えば`$ sudo cp /etc/clamav/freshclam.conf{,-new}`は`$ sudo cp /etc/clamav/freshclam.conf /etc/clamav/freshclam.conf-new`と同じで、`$ sudo mv -f /etc/clamav/freshclam.conf{-new,}`は`$ sudo mv -f /etc/clamav/freshclam.conf-new /etc/clamav/freshclam.conf`と同じです。

これで無事WSL2上で、ClamAVが動くようになりました。

```shell-session
$ sudo freshclam
Mon Feb 24 01:30:38 2025 -> ClamAV update process started at Mon Feb 24 01:30:38 2025
Mon Feb 24 01:30:38 2025 -> daily.cvd database is up-to-date (version: 27558, sigs: 2073020, f-level: 90, builder: raynman)
Mon Feb 24 01:30:38 2025 -> main.cvd database is up-to-date (version: 62, sigs: 6647427, f-level: 90, builder: sigmgr)
Mon Feb 24 01:30:38 2025 -> bytecode.cvd database is up-to-date (version: 335, sigs: 86, f-level: 90, builder: raynman)
```

```shell-session
$ clamscan install.sh
/tmp/release/install.sh: OK

----------- SCAN SUMMARY -----------
Known viruses: 8704520
Engine version: 0.103.12
Scanned directories: 0
Scanned files: 1
Infected files: 0
Data scanned: 0.01 MB
Data read: 0.00 MB (ratio 2.00:1)
Time: 15.215 sec (0 m 15 s)
Start Date: 2025:02:24 01:31:51
End Date:   2025:02:24 01:32:06

# ※ clamscanではなくclamdscan
$ clamdscan --fdpass /
...
```

## WSL2用設定

ついでにWSL2向けの設定もしておきましょう。

これはチェックしたくないディレクトリを除外する設定です。
好みで設定してください。

Windows側のファイルシステムをWindows Defenderに任せたい場合は、`/mnt/c`は除外しておくといいと思います。

```shell-session
$ sudoedit /etc/clamav/clamd.conf
$ sudo diff /etc/clamav/clamd.conf{,-new}
87a88,95
>
> # Below section is edited by me
> ExcludePath ^/proc
> ExcludePath ^/sys
> ExcludePath ^/run
> ExcludePath ^/dev
> ExcludePath ^/snap
> ExcludePath ^/mnt
```

## おわりに

いかかだったでしょうか？

今まで「Linuxは一般ユーザーが少ないため、アンチウイルスソフトを入れる必要性は少ない」と言われてきましたが、WindowsにWSLが入ってきた以上、そうは言っていられない時代なのではないでしょうか。
そこでClamAVを選ぶ選択肢は賢いと思います！

cronを設定して、clamdscanを定期的に動かすのもよいでしょう。

ClamAVを活用して、安全なWSL2ライフを送りましょう！
それでは、よいWSL2ライフを！:tada: :sparkles:

## 参考URL

- [Ubuntu20.04 で ClamAV を使ってウイルススキャン #ubuntu20.04 - Qiita](https://qiita.com/kannkyo/items/1cc32231afad88c11d8e)
- [Clamavをやってみた。そして動かねぇ。 - Slow-fire hacking next](https://www.blog.slow-fire.net/2021/12/13/clamav%E3%82%92%E3%82%84%E3%81%A3%E3%81%A6%E3%81%BF%E3%81%9F%E3%80%82%E3%81%9D%E3%81%97%E3%81%A6%E5%8B%95%E3%81%8B%E3%81%AD%E3%81%87%E3%80%82/)

[^apt-fast]: 蛇足ですが、`apt install`を使うときは、[apt-fast](https://zenn.dev/ryuu/articles/fast-aptcommand)を使うとダウンロードが速いです。その場合は、以降の`apt install`を`apt-fast install`に置き換えてください。
