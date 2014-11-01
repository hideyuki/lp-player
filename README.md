# LP Player with RaspberryPi

## Installation Guide

MacOSX上での例です。

### Setup Raspberry Pi

#### イメージのダウンロード

[こちら](http://downloads.raspberrypi.org/raspbian_latest)から最新版のRaspbianをダウンロードしてくる。そしてzipを解凍。.imgファイルがでてきます。


#### SDにイメージを書き込む

お手元にSDカードをご用意ください。たしか8GB以上が推奨だったように思います。私は16GBのSDカードを使っています。

SDにイメージをさくっと書き込むためにApple-Pi Bakerを使ってみる。

[こちら](http://www.tweaking4all.com/hardware/raspberry-pi/macosx-apple-pi-baker/)からアプリケーションをダウンロード。

まず左上側のSD-Card選択を済ませて、右上側でIMG fileを選択の後Restore Backupをクリック。５分くらいでイメージを書き込むことができます。

![applepi-baker](ahttps://dl.dropboxusercontent.com/u/180053/github/lp-player/applepi_baker.png)


#### RaspberryPiを起動

RaspberryPiの設定を行うためには、HDMIディスプレイやUSBキーボードを接続する必要があります。それらを用意するのがめんどくさいので簡単な構成でセットアップしてみましょう。

LANケーブルをRaspberryPiに接続。RaspberryPiのMicroUSB端子に電源を供給し、RaspberryPiを起動しよう。

このままではRaspberryPiのIPアドレスがわからないので、Mac側からarp-scanでネットワーク内にあるRaspberryPiを探そう。`arp-scan`コマンドがない人はhomebrewなどでインストールしてね。

```
$ arp-scan 192.168.1.0/24
Interface: en0, datalink type: EN10MB (Ethernet)
Starting arp-scan 1.9 with 256 hosts (http://www.nta-monitor.com/tools/arp-scan/)
192.168.1.1	24:a2:e1:e9:47:a9	(Unknown)
192.168.1.100	b8:27:eb:6b:00:31	Raspberry Pi Foundation    # これだ！
・・以下たくさん出てくる・・
```

上記の結果から、`192.168.1.100`がRaspberryPiだということがわかります。

さっそくSSHでログインしてみましょう。デフォルトのユーザ名は`pi`、パスワードは`raspberry`です。

```
$ ssh pi@192.168.1.100
pi@192.168.1.100's password: 
Linux raspberrypi 3.12.28+ #709 PREEMPT Mon Sep 8 15:28:00 BST 2014 armv6l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Sep  9 08:09:11 2014 from 192.168.1.118

NOTICE: the software on this Raspberry Pi has not been fully configured. Please run 'sudo raspi-config'

-bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
pi@raspberrypi ~ $
```

これでRaspberryPiの起動はOK！


#### SDカードいっぱいにFSを拡張する

以下のコマンドでファイルシステムのサイズを見てみよう。

```
$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
rootfs           3023728 2370300    480116  84% /         # ここ注目
/dev/root        3023728 2370300    480116  84% /
devtmpfs          219764       0    219764   0% /dev
tmpfs              44788     208     44580   1% /run
tmpfs               5120       0      5120   0% /run/lock
tmpfs              89560       0     89560   0% /run/shm
/dev/mmcblk0p1     57288    9880     47408  18% /boot
```

上記のように、初期状態では3.2GBほどしかRaspbianの領域が確保されておらず使用率も84%となっています。

SDのサイズは16GBあるのでFSを拡張してみよう。

```
$ sudo raspi-config
```

すると以下の画像のような表示になります。

![image](ahttps://dl.dropboxusercontent.com/u/180053/github/lp-player/raspi-config.png)

`1 Expand Filesystem` でEnterキーを押してみると、黒い画面でいろいろ動いてる感がしたのちにまた表示が戻ってきます。これで完了です。Tabキーを2回押して`Finish`でEnterキーを押して終了。再起動しますかと聞かれるので、そのまま再起動してしまおう。

1分してから再度SSHでRaspberryPiにログインして、もう一度FSのサイズを確認。
(このタイミングでIPアドレスがかわってしまう可能性があるので、arp-scanでもう一度RaspberryPiのIPアドレスも確認しておこう)

```
$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
rootfs          14984668 2373452  11952132  17% /         # ここ注目
/dev/root       14984668 2373452  11952132  17% /
devtmpfs          219764       0    219764   0% /dev
tmpfs              44788     208     44580   1% /run
tmpfs               5120       0      5120   0% /run/lock
tmpfs              89560       0     89560   0% /run/shm
/dev/mmcblk0p1     57288    9880     47408  18% /boot
```

見事使用率が17%に！14984668KBと、16GB相当が割り当てられていますね。


#### いろいろアップデートする

まぁいちおう。意外に待たされるので、ここでコーヒーブレイクをお薦めします。

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo rpi-update
```

#### 音量上げる

alsamixerでalsaの音量をコントロールします。

```
$ alsamixer
```

すると以下のような表示になるので、上(↑)キーを押して音量を100にしてからESCキーで設定終了します。

![alsamixer](ahttps://dl.dropboxusercontent.com/u/180053/github/lp-player/alsamixer.png)

音量設定を保存するために、以下のコマンドをば。

```
$ sudo alsactl store
```

これで再起動しても音量が保持されます。


#### 無線LANで接続できるように

LANケーブルは邪魔です。無線LANを使いましょう。

[USB無線ＬＡＮアダプタGW-USNANO2A](http://www.amazon.co.jp/gp/product/B00ESA34GA/ref=pd_lpo_sbs_dp_ss_1?pf_rd_p=466449256&pf_rd_s=lpo-top-stripe&pf_rd_t=201&pf_rd_i=B004AP8QLQ&pf_rd_m=AN1VRQENFRJN5&pf_rd_r=05BFR2RF1R2T062636FA)をRaspberryPiに接続して、以下のコマンドを実行。

