# USBメモリのマウント

`mount`コマンドを使用すれば、起動時の間はマウントしたままですが、再起動すると解除されます。以下は起動時に自動でマウントするようにセットアップするための方法です。

**!!注意!!** *USBメモリが抜かれたり・壊れたりし、認識できない状態でシステムを起動すると、ちゃんと起動しない？ssh経由での操作も出来なくなった。*

## Case1. そのままマウント

起動時に自動的にマウントされるようにするため、
```/etc/fstab```に設定を追加する。

**/etc/fstab**
```sh
proc          /proc      proc defaults         0 0
PARTUUID=???  /boot      vfat defaults         0 2
PARTUUID=???  /          ext4 defaults,noatime 0 1
UUID="??????" /mount/to/ ext4 defaults         0 2
```

UUIDについては、```lsblk```等を使用して調べる。

```sh
$ lsblk -o NAME,SIZE,UUID
NAME          SIZE UUID
sda         233.3G
mqsda1      233.3G xxxx-xxxx-xxxx-...
sdb           1.8T
mqsdb1        1.8T xxxx-xxxx
```

## Case2. RAID0の構築してマウント

RAIDは複数の記憶媒体を1つに纏め、あたかも1つの記憶媒体のように見せる技術です。

ここでは、mdadmを使用してRAIDを構築する手順を記述します。

### RAIDを構築するデバイスの特定

まずは、どのデバイスが目的のデバイスか特定する。

sdで始まるデバイス一覧を表示する
```sh
$ ls -l /dev/sd*
```
他のデバイスを表示している可能性があるので確認が必要(blkidコマンド等)


### USBデバイスのフォーマット

RAID用に記憶媒体をフォーマットする。

**!!注意!!** *間違った記憶媒体を指定するとデータが消滅するので注意！目的の媒体かしっかりと確認する事(場合によっては、目的外の媒体を外す)。*

```sh
$ fdisk /dev/???
Command (m for help): d
Command (m for help): n
Partition number (1-4, default 1): 1
First sector (2048-123437055, default 2048):
Command (m for help): t
Hex code (type L to list all codes): fd 
Command (m for help): w 
```

上記の作業はRAIDを構築する記憶媒体の数だけ実施する必要がある。

### デバイス名を固定化する

記憶媒体の名称が再起動しても変わらないようにするための作業

デバイスの情報を取得
```sh
$ udevadm info -a -n /dev/??? | grep serial
    ATTRS{serial}=="01013c..."
    ATTRS{serial}=="0000:01:00.0"
```

上記をもとに、デバイス名を設定する。
* 「99-usbmem.rules」の名前は任意
* 「KERNEL=="sd*"」や「SUBSYSTEMS=="usb"」は自身の環境に合わせて変更
* 「SYMLINK」は、記憶媒体に付けるデバイス名の命名規則

```sh
$ cd /etc/udev/rules.d
$ vi 99-usbmem.rules
KERNEL=="sd*", SUBSYSTEMS=="usb", ATTRS{serial}=="01013c...", SYMLINK+="myusb1%n"
```

設定後、OS再起動すると、```/dev/myusb1```、 ```/dev/myusb11```といったデバイスが追加される。