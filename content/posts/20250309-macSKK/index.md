---
title: "20250309 yaskkserv2を導入してmacSKKと連携する"
date: 2025-03-09T16:18:08+09:00
---

## 概要

yasskkserv2を導入したのでその記録です。yasskkserv2はGoogle日本語入力のために導入します。

### yasskkserv2とyaskkserv2_make_dictionaryを使えるようにする

https://github.com/wachikun/yaskkserv2

> $ cargo build --release
$ cp -av target/release/yaskkserv2 /YOUR-SBIN-PATH/
$ cp -av target/release/yaskkserv2_make_dictionary /YOUR-BIN-PATH/

README.mdにはこのように書いてありますが、GitHub Releaseにも置いてあるようなので、そちらから取得するようにしました。

以下のようにしました。バイナリは`~/.yaskkserv2/bin`に配置します。

```shell
mkdir -p ~/.yaskkserv2/bin
YASKKSERV2_VERSION="0.1.7"
YASKKSERV2_ARCHIVE="yaskkserv2-$YASKKSERV2_VERSION-x86_64-apple-darwin.tar.gz"

if [ ! -z "$YASKKSERV2_ARCHIVE" ]; then
  curl -L -o /tmp/$YASKKSERV2_ARCHIVE "https://github.com/wachikun/yaskkserv2/releases/download/$YASKKSERV2_VERSION/$YASKKSERV2_ARCHIVE"
  tar -xzf /tmp/$YASKKSERV2_ARCHIVE -C ~/.yaskkserv2 --strip-components=1
  mv ~/.yaskkserv2/yaskkserv2 ~/.yaskkserv2/bin/yaskkserv2
  mv ~/.yaskkserv2/yaskkserv2_make_dictionary ~/.yaskkserv2/bin/yaskkserv2_make_dictionary
  cp ./macSKK/dictionary.yaskkserv2 ~/.yaskkserv2/dictionary.yaskkserv2
  rm /tmp/$YASKKSERV2_ARCHIVE
fi
```


`.zshrc` で以下を追記して、PATHを通します。

```shell
export PATH=$HOME/.yaskkserv2/bin:$PATH
```

### yasskkserv2の辞書を作成

既にmacSKK自体で辞書を管理しているので、yasskkserv2ではGoogle日本語入力のみ管理したいです。


(macSKKでは優先的にファイル辞書を読み込むとのことなので、望んだ挙動になりそうです)

https://github.com/mtgto/macSKK?tab=readme-ov-file#skkserv%E3%82%92%E8%BE%9E%E6%9B%B8%E3%81%A8%E3%81%97%E3%81%A6%E4%BD%BF%E3%81%86

> 常にファイル辞書よりも変換候補は後に出るようにしています。
並び替えのUIで迷ったために先送り。将来並び替えできるようにすると思います。


しかし、yasskkserv2を立ち上げるには、必ず辞書を指定する必要があるみたいでした。そのため、空の辞書を読み込むようにしました。


以下の内容で`skk-jisyo-dummy.utf8`を作成します。

```
;; -*- mode: fundamental; coding: utf-8 -*-
;; okuri-ari entries.
```

そのファイルを指定して、`yaskkserv2_make_dictionary`を実行します。

```shell
yaskkserv2_make_dictionary --dictionary-filename=dictionary.yaskkserv2 skk-jisyo-dummy.utf8
```

`dictionary.yaskkserv2` が生成されるはずです。

### yasskkserv2を立ち上げる

あとは、生成された辞書を指定して`yasskkserv2`を立ち上げるだけです。

```shell
yaskkserv2 --google-suggest --google-cache-filename=/tmp/yaskkserv2.cache ~/.yaskkserv2/dictionary.yaskkserv2
```

これで、macSKKでSKKservを有効にしてあげればOKです。

本当はMacを起動したときに、自動で実行するようにしたいのですが、まあ普段はsleepで運用しているので、都度実行でもいいかなと思っています。


## おわりに
素晴らしいソフトウェアを制作して頂き感謝！
