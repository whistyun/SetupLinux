# 共有フォルダの設定

sambaをインストール後、設定ファイルをいじる。

下記の設定で、"/path/to/dir”を基底とする"catpocket"という名称の共有フォルダが作られる(アクセスできるのはhogeユーザのみ)。

**/etc/samba/smb.conf**
```sh

[global]
workgroup = WORKGROUP
dos charset = CP932
unix charset = utf-8
unix extensions = no
hosts allow = 192.168.1.0/255.255.255.0

...

[catpocket]
path = /path/to/dir
read only = No
guest ok = No
valid users = hoge
```

(linux上の)ファイルアクセス権限がhogeユーザに無いと、共有フォルダを開くことはできても、ファイルを読み込めない・書き込めないになるので注意。

`chown`や``chmod``を使用して、基底のフォルダの所有者、権限を変更する必要があります。

Sambaで使用できるユーザーは別途登録が必要です。
```sh
smbpasswd -a hoge
```
