+++
banner = ""
categories = ["Operating System"]
date = 2022-04-10
description = "macOS に UNIX V6 (UNIX 6th edition) のシミュレータをインストールする"
images = []
menu = ""
tags = ["UNIX", "UNIX V6", "PDP-11", "PDP-11/40"]
title = "Install UNIX V6 simulator"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# UNIX V6 (UNIX 6th edition)

**UNIX V6** (**UNIX 6th edition**) は, 現在では実用途で使わていない OS ですが, 現在使われている OS の基本となる処理がほとんど実装されています. 言い換えると, UNIX V6 を発展させていった OS が, 現在使われている OS でもあるので, UNIX V6 を学ぶことで, 最新の OS を理解することにもつながります.

ちなみに, 現在使われている OS で, UNIX V6 には実装されていない機能としては,

- スレッド
- ネットワーク
- GUI
- マルチコア対応
- 仮想化

しかし, これらの機能も根底には, UNIX V6 で実装されているアイデアがあります.

また, UNIX V6 は聡明期の OS なのでコードリーディングしやすく, カーネルのソースコードはおよそ **10,000 行** です. ちなみに, 人間が 1 つのシステムで理解できるソースコード量の限界が 10,000 行と言われているので, UNIX V6 は OS 全体を把握するのに最適と言えます.

また, インターネット上での資料や論文, そして, シミュレータ (エミュレータ) があるので, 学ぶ材料も豊富にあります.

そこで, この記事では, UNIX V6 のシミュレータをインストールして動作できる手順までを解説します.

# Install UNIX V6 simulator

まずは, 歴史的なシミュレータである, `simh` をインストールします.

```bash
$ brew install simh
```

これによって, のちほど使う `pdp11` コマンドが利用可能になります.

次に, UNIX V6 のディスクイメージを取得して解凍します (ディレクトリにまとまっていないのであらかじめ作業ディレクトリを作成しておくほうがよいでしょう).

```bash
$ mkdir unix-v6  # unzip 時にディレクトリにまとまっておらず, ファイルがバラけてしまうのであらかじめ作業ディレクトリを作成
$ wget http://simh.trailing-edge.com/kits/uv6swre.zip
$ unzip uv6swre.zip
$ ls -lA  # ファイル一覧
total 18408
-rw-rw-rw-  1 ikedatomohiro  staff    12299  1 24  2002 AncientUnix.pdf
-rw-rw-rw-  1 ikedatomohiro  staff      263 11 25  1996 README.txt
-rw-rw-rw-  1 ikedatomohiro  staff  2077696  9  8  1998 unix0_v6_rk.dsk
-rw-rw-rw-  1 ikedatomohiro  staff  2440704  9  8  1998 unix1_v6_rk.dsk
-rw-rw-rw-  1 ikedatomohiro  staff  2440704  9  8  1998 unix2_v6_rk.dsk
-rw-rw-rw-  1 ikedatomohiro  staff  2440704  9  8  1998 unix3_v6_rk.dsk
```

エディタを起動して, 設定ファイルを以下の内容で編集します.

```bash
$ vim pdp11.ini  # ファイル名は任意. エディタはお使いのもので
```

```bash
$ cat pdp11.ini
set cpu 11/40
set cpu u18
att rk0 unix0_v6_rk.dsk
att rk1 unix1_v6_rk.dsk
att rk2 unix2_v6_rk.dsk
att rk3 unix3_v6_rk.dsk
boot rk0
```

# Start Simulator

上記の手順が完了していれば, 以下のコマンドで UNIX V6 のシミュレータを起動することができます.

```bash
$ pdp11 pdp11.ini  # ファイル名は先ほど編集したファイルに合わせる
```

以下のような出力になって, プロンプト (`@`) が表示されるので,

```bash
PDP-11 simulator V3.11-1
Disabling XQ
@
```

`unix` を入力, ログインユーザーは `root` を入力します.

```bash
PDP-11 simulator V3.11-1
Disabling XQ
@unix

login: root
#
```

プロンプトが `#` に変わり, コマンド入力できるようになります.

```bash
# pwd
/
# ls -lA
total 182
drwxr-xr-x  2 bin      1040 Jan  1  1970 bin
drwxr-xr-x  2 bin       352 Jan  1  1970 dev
drwxr-xr-x  2 bin       304 Aug 20 12:19 etc
drwxr-xr-x  2 bin       336 Jan  1  1970 lib
drwxr-xr-x 17 bin       272 Jan  1  1970 mnt
drwxr-xr-x  2 bin        32 Jan  1  1970 mnt2
-rw-rw-rw-  1 root    28472 Aug 20 12:01 rkunix
-rwxr-xr-x  1 bin     28636 Aug 20 11:38 rkunix.40
drwxrwxrwx  2 bin       144 Aug 20 12:14 tmp
-rwxr-xr-x  1 bin     28472 Aug 20 12:01 unix
drwxr-xr-x 13 bin       224 Aug 20 12:22 usr
drwxr-xr-x  2 bin        32 Jan  1  1970 usr2
#
```

UNIX V6 シミュレータでいくつかのコマンドを実行した例です.

# End Simulator

UNIX V6 シミュレータを終了するには, `sync` を 3 回入力したあと, `<Ctrl-E>` を入力します.

```bash
# sync
# sync
# sync
#
Simulation stopped, PC: 021630 (MOV (SP)+,177776)
sim>
```

最後に, シミュレータを終了します.

```bash
sim> quit # `exit` でも可能
Goodbye
```

![UNIX V6 simulator 起動から終了まで](https://user-images.githubusercontent.com/4006693/162605591-e2b1f3b6-9a4d-4cf8-8ece-5ae03fcdd5c9.gif)
# References

- [はじめてのOSコードリーディング ――UNIX V6で学ぶカーネルのしくみ](https://gihyo.jp/book/2013/978-4-7741-5464-0)
- [UNIX V6 on PDP-11をMacで動かす](https://qiita.com/morinokami/items/4538f610f72779be8aef)
