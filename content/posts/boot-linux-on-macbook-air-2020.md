+++
banner = ""
categories = ["Development", "Operating System"]
date = 2022-04-09
description = "MacBook Air 2020 で Ubuntu をデュアルブートする"
images = []
menu = ""
tags = ["Linux", "Ubuntu", "macOS"]
title = "Boot Linux on MacBook Air 2020"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

MacBook Air 2020 (**Intel**) で Linux (Ubuntu) をデュアルブートできるようにしたのでそのメモ書きです.

**万が一のために, ディスクのデータはすべてバックアップをとっておいてください.**

# Prepare Devices

Linux をデュアルブートするために, いくつかハードウェアが必要になります.

## USB メモリ

Linux のインストーラを書き込むために必要です. 8 GB 以上あれば十分です.
インストーラを書き込むと既存のデータが消えてしまうので, バックアップが必要です
(そんなに高価なものではないので新しく購入するほうがいいかもしれません).

## 外付けキーボード

搭載キーボードは, **デフォルトでは機能しない**ので必要になります.

## マウス

こちらも, MacBook Air のトラックパッドは機能しないので, 操作を快適にするためにはあったほうがいいかもしれません.
ただ, 最低限, 外付けキーボードがあれば必要な操作はできます.

# Download Image and Create Installer

[Ubuntu ディスクイメージ](https://ubuntu.com/download/desktop/thank-you?version=20.04.4&architecture=amd64#download)をダウンロードします (ちなみに, 2022 / 04 / 09 時点での LTS は 20.04, 最新バージョンが 21.10 です. 今回は, LTS を使いました). また, ディスクイメージをもとに USB メモリにインストーラを作成するために, [balenaEtcher](https://www.balena.io/etcher/) を使います (`dd` コマンドを使う場合は不要です).

事前に, USB メモリを Disk Utility を使って, **FAT**・**GUID Partition Map** でフォーマットしておきます.

![USB メモリ フォーマット](https://user-images.githubusercontent.com/4006693/162565377-800dc8d0-e684-4f30-8149-b2b6282eb1f6.gif)

あとは, balenaEtcher のナビゲーションにしたがって, ディスクイメージを選択して, 書き込み先の USB メモリを指定して, 書き込むだけです.

![インストーラの書き込み](https://user-images.githubusercontent.com/4006693/162566236-6fc3b43b-6a08-48a0-99a3-b1a9ae801577.gif)

# Enable to boot from USB

## SIP (System Integrity Protection

wip

macOS のデフォルトの設定だと, USB メモリから Ubuntu を起動することはできないので, macOS Recovery を起動して設定を変更します.
まず, 電源を ON にしたときに (再起動時に), **command + R** を押し続けると, macOS Recovery が起動します.

![macOS Recovery の起動](https://user-images.githubusercontent.com/4006693/162568106-e0d58420-584b-40eb-93c0-81eb1cf3ea9c.gif)

ユーザーを選択して, パスワードを入力したら, 左上のメニューバーにある, Startup Security Utility を起動します.

![Startup Security Utility](https://user-images.githubusercontent.com/4006693/162568190-aa3d9e24-d0da-4463-b5dd-71e2f0c36d20.png)

以下のように設定を変更します

<dl>
  <dt>Secure Boot</dt>
  <dd>No Security</dd>
  <dt>Allowed Boot Media</dt>
  <dd>Allow booting from external or removable media</dd>
</dt>

![Startup Security Utility の設定変更](https://user-images.githubusercontent.com/4006693/162568298-f865f202-6331-4cba-85b0-c32861503b61.png)

最後に, 同じように左上のメニューバーから, Terminal を起動して, `csrutil disable` と入力します.
これは, SIP (**S**ystem **I**ntegrity **P**rotection) を (一時的に) 無効にしています (Linux インストールが完了すればもとに戻します).

![Terminal](https://user-images.githubusercontent.com/4006693/162568621-11134909-5c83-4eb6-9d6b-7d832a90b46e.png)

![SIP の無効化](https://user-images.githubusercontent.com/4006693/162568597-03a4005b-cbc1-4664-ba81-61727bdca697.png)

これで, ようやく, USB メモリに書き込んだインストーラを起動する準備ができました !

# Create Partition for Linux

Linux をインストールするためのパーティションを作成します.
Disk Utility を使って, **パーティション**を作成します (**ボリュームではないので注意してください**).
割り当てたいディスクサイズを選択して, **FAT** でフォーマットします.
Name は任意です (ただ, あとから判別しやすいようなパーティション名にしておくほうがよいでしょう).

![パーティションの作成](https://user-images.githubusercontent.com/4006693/162567106-2da33cae-205c-4dad-a1f6-f6f7c21a867c.gif)

もし, 割り当てサイズを変更したい場合などは, 以下のように 1 度パーティションを削除して, 再度割り当て直します.

![パーティションの削除](https://user-images.githubusercontent.com/4006693/162566700-bd088037-1833-4a01-baa5-bb083589279a.gif)

# Install Linux

電源を ON にする前に, インストーラを書き込んだ USB メモリと, 外付けキーボードを接続しておきます.
起動時に, **option** キーを押し続けると, 起動するディスクを選択することができるので, 最も右側の USB メモリのようなアイコンの **EFIBoot** を選択すると, Ubuntu が起動します.

![起動ディスクの選択](https://user-images.githubusercontent.com/4006693/162569662-5dac2d8c-d721-47ad-a886-236607fd0137.gif)

「Try Ubuntu」を選択して, ターミナルを起動します (**ここからは, 外付けキーボード, マウスで入力してください**).
そして, `sudo ubiquity -b` でインストーラを起動します (`-b` は, ブートローダーはインストールしないオプションです).

ブートローダーである GRUB (**GR**and **U**nified **B**ootloader) は, あとから別途インストールします.

![Try Ubuntu](https://user-images.githubusercontent.com/4006693/162569785-612ffb5e-9e79-48bd-aa71-768dcdbed6df.gif)

![ターミナルの起動](https://user-images.githubusercontent.com/4006693/162570784-2210e39e-4e7a-432b-bdbb-eff8ab21d728.gif)

![インストーラの起動 (ブートローダーなし)](https://user-images.githubusercontent.com/4006693/162572347-e43a16b7-6143-4d88-b5a1-5683fc90371b.gif)

言語や, キーボードレイアウトは自由に選択してください.

インストールでは「通常のインストール」を選択します (オプションはインストール後でも対応可能なのでどちらでも問題ありません).
インストールの種類では「それ以外」を選択して, パーティションの割り当てやフォーマットを手動で実行します.

![インストール・インストールの種類](https://user-images.githubusercontent.com/4006693/162572576-dd133ace-0062-4c33-bc4e-5f948345b5a9.gif)
パーティションの編集では, FAT (fat32) でフォーマットしたパーティション (先に, Linux 用に割り当てたパーティション. おそらく, `/dev/nvme0n1p3` になっているかと思います. 割り当てたディスク容量などと合わせると確実です) を選択します.

<dl>
  <dt>利用方法</dt>
  <dd>
    ext4 ジャーナリングファイルシステム<br />
    (ちなみに, extN は Linux の標準的なファイルシステムです)
  </dd>
  <dt>パーティションの初期化</dt>
  <dd>チェックをつける</dd>
  <dt>マウントポイント</dt>
  <dd><code>/</code></dd>
</dl>

上記の設定で, パーティションの編集をします.

![パーティションの編集](https://user-images.githubusercontent.com/4006693/162572787-9b44fc43-6a08-4e59-90dc-836506b0479b.gif)

あとは, タイムゾーンやユーザー名, パスワードなどを入力してインストールを開始します.

## Install GRUB

先ほどまでの手順では, Ubuntu をディスクにインストールしただけなので, **起動することはできません**.
Linux を起動するための, ブートローダーである GRUB をインストールします.

まず, 同じ手順で, 再度, USB メモリから Ubuntu を起動して, 「Try Ubuntu」を選択して, ターミナルを起動します.

ターミナルを起動したら以下のコマンドを実行します.

```bash
$ sudo -s
$ mount /dev/nvme0n1p3 /mnt # Ubuntu をインストールしたパーティションをマウント
$ mount /dev/nvme0n1p1 /mnt/boot/efi  # EFI (Extensible Firmware Interface) のパーティションをマウント
$ for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt${i}; done
$ sudo chroot /mnt
$ grub-install --force /dev/nvme0n1
$ update-grub
```

![マウント](https://user-images.githubusercontent.com/4006693/162574981-f969a856-f3ca-48a1-b84f-4f8f7bd96cb6.jpeg)

`/dev/nvme0n1pX` の数値の部分は, もしかしたら異なる可能性もあるので, 以下のコマンドで確認しておくと確実です.

```bash
$ lsblk
```

![lsblk の出力](https://user-images.githubusercontent.com/4006693/162574953-52f3b563-e872-472a-97c7-56fc8a98f8f4.jpeg)

`grub-install` では警告は発生しますが, エラーが発生していなければ問題ありません.
`update-grub` で 「cannot find a GRUB drive for`/dev/sda1`.」 とエラーが発生するようであれば, USB メモリを取り外して, 再度 `update-grub` を実行してみてください.

![`grub-install`](https://user-images.githubusercontent.com/4006693/162575019-c35ed63d-2360-46ca-aab9-411ca727a907.jpeg)

![`update-grub` でエラー](https://user-images.githubusercontent.com/4006693/162575059-a5e50f84-20fb-4b6f-9b4f-89c217269f42.jpeg)

![USB メモリを外して, `update-grub` を実行](https://user-images.githubusercontent.com/4006693/162575078-c2c6ab13-0d27-485c-96af-f3a6db3ec0fc.jpeg)

ここまで完了したら, 再起動します.

# Boot Linux

USB メモリを外し, option キーを押しながら電源を ON にして, 今度は, ディスクのようなアイコンの **EFIBoot** を選択して起動すると, GRUB のターミナルが起動します.

```bash
grub> ls
```

と入力し, `(hdX,gptY)` と表示されているデバイスに対して, しらみつぶし (といっても, 数としては多くないですが ...) に,
以下のコマンドを入力すると, いずれかで Ubuntu が起動します.

```bash
grub> configfile (hd0,gpt1)/boot/grub/grub.cfg
```

![GRUB ターミナル](https://user-images.githubusercontent.com/4006693/162574093-f84fe44d-5599-45aa-8c74-f9e981f5f064.gif)

![GRUB `ls`](https://user-images.githubusercontent.com/4006693/176887035-a4c8afe5-6ac0-4b8b-8ec6-63b093e3ebce.png)

![GRUB `configfile (hdX,gptY)/boot/grub/grub.cfg`](https://user-images.githubusercontent.com/4006693/176887060-494fe0f1-6b34-4c52-9fe0-a6185c654a55.png)

![Ubuntu を MacBook Air でブート](https://user-images.githubusercontent.com/4006693/162573998-e93ce08f-109c-4a8d-8c06-c5728404c688.gif)

![Ubuntu デスクトップ画面](https://user-images.githubusercontent.com/4006693/162574153-9c42a9d0-5bab-4f9d-8d4d-d89cfc261e34.png)

Ubuntu がディスクから起動できたら, 再度, macOS Recovery を起動して, SIP の設定をもとに戻しておきます.

```bash
$ csrutil enable
```

また, Macintosh HD を選択すれば, macOS が起動できることも確認しておきましょう.

![macOS を起動](https://user-images.githubusercontent.com/4006693/162607249-f0a2a610-bb24-4942-bf15-f38209f2d59a.gif)

## Enable Devices

2016 年以降の Mac には, T2 セキュリティチップがあり, 搭載のキーボードやトラックパッド, オーディオデバイス, WiFi はそのままでは動作しません. さいわい, ドライバーや Linux カーネルとパッチをインストールすれば動作します.

ドライバーやパッチは, ネットワーク経由で取得するので, インターネットに接続している必要があります.
初期状態では WiFi が動作しないので, スマートフォンの有線テザリングなどを利用してインターネットに接続します.

![有線テザリングでネットワークに接続](https://user-images.githubusercontent.com/4006693/167605272-5ca1262f-9cf0-48eb-8dae-c88cd62f66d4.jpeg)

### Install Drivers

キーボード, トラックパッド, オーディオデバイスのドライバーをインストールします.

[DKMS](https://wiki.t2linux.org/guides/dkms/) のインストール.

```bash
$ sudo apt -y install dkms
```

[BCE](https://wiki.t2linux.org/guides/dkms/#installing-modules) (**B**uffer **C**opy **E**ngine) のインストール

```bash
$ sudo git clone https://github.com/t2linux/apple-bce-drv /usr/src/apple-bce-0.2
$ cd /usr/src/apple-bce-0.2
$ sudo vi dkms.conf
```

dkms.conf に以下の設定を記述します.

```
PACKAGE_NAME="apple-bce"
PACKAGE_VERSION="0.2"
MAKE[0]="make KVERSION=$kernelver"
CLEAN="make clean"
BUILT_MODULE_NAME[0]="apple-bce"
DEST_MODULE_LOCATION[0]="/kernel/drivers/misc"
AUTOINSTALL="yes"
```

dkms.conf に設定を書き込んだら, 以下のコマンドを実行します.

```bash
$ sudo dkms install -m apple-bce -v 0.2
```

Linux カーネルにモジュールをロードします.

```bash
$ sudo modprobe apple_bce
```

以上で, キーボード, トラックパッド, オーディオデバイスが動作します.

あとは, 起動時にモジュールをロードするように設定を追記します.

```bash
$ sudo -s
$ echo apple-bce >> /etc/modules-load.d/t2.conf
```
