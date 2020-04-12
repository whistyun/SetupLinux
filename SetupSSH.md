# SSHの設定

## SSHの有効化

デフォルトだと、sshが無効化されているので、```raspi-config```を使用して有効化する。


## IPアドレス固定

端末が起動する度にアドレスが変わったら困るので、
固定のアドレスとなるように設定する。有線で設定。

**/etc/network/interfaces**
```sh
allow-hotplug eth0
iface eth0 inet static
address 192.168.???.???
netmask 255.255.255.0
gateway 192.168.???.1
```

##　なんちゃってセキュリティ設定

### SSHのポート変更

乗っ取られにくくするため、設定ファイルを編集し、ポート番号と許可ユーザーを設定（ポート番号をwell-knownなものでなくし、また、ログイン可能なユーザーも特定のIPアドレスからのみログイン可能とする）。


**/etc/ssh/sshd_config**
```sh

Port ??????
AllowUsers hoge@???.???.???.???
```


### iptablesによるファイアウォール的設定追加

特定のIPアドレス以外からはリクエストを受け付けないように設定する。

raspberypiだと設定ファイルがなく、```iptables-persistent```をインストールする必要がある。インストール後、 ```/etc/iptable/rule.v4``` および ```/etc/iptables/rule.v6```を編集し、設定を行う。

**/etc/iptable/rule.v4**
```sh
*filter
# ポリシーの設定
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [873:243693]
:myAccept - [0:0]
:myDrop - [0:0]
# 接続許可： ループバック
-A INPUT -i lo -j myAccept
# 接続許可： 確立済みの接続
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# 接続許可： ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j myAccept
# 接続許可： sshのポート番号。内部ネットワークからのみアクセスを許可。
-A INPUT -s xxx.xxx.xxx.xxx/xx -p tcp -m tcp --dport ??? -j myAccept
# 接続許可： sambaのポート番号
-A INPUT -p udp --dport 137 -j myAccept
-A INPUT -p udp --dport 138 -j myAccept
-A INPUT -p tcp --dport 139 -j myAccept
-A INPUT -p tcp --dport 445 -j myAccept

# 接続拒否： 上記以外
-A INPUT -j myDrop

# 接続が確立したさにはログ出力
-A myAccept -j LOG --log-prefix "IPTABLES-ACPT: "
-A myAccept -j ACCEPT
# 接続拒否した場合はログ出力
-A myDrop -j LOG --log-prefix "IPTABLES-DROP: "
-A myDrop -j DROP
COMMIT
```

上記の設定の中では、
```sh
-A INPUT -s xxx.xxx.xxx.xxx/xx -p tcp -m tcp --dport ??? -j myAccept
```
で、アクセスを許可する通信を設定する。

例えば、
```sh
-A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 80 -j myAccept
```
とした場合、「192.168.1.*」のIPアドレスを持つクライアントからの、ポート80番へのアクセスが許可される。