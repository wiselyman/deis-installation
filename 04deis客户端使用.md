# 1. deis客户端安装
- 工作机在非root用户下
- `cd /home/wisely/workspace/deis/client`
- `make build`
- 放置在`/usr/local/bin`下
# 2. DNS配置
工作机上安装bind
- `yum -y install bind bind-utils `
- `vi /etc/named.conf`
  增加
  ```
        zone "wisely.priv" IN {
                type master;
                file "wisely.priv.lan";
                allow-update { none; };
        };
  ```
- `vi /var/named/wisely.priv.lan`
  增加
```
$ORIGIN .
$TTL 7D
wisely.priv    IN SOA  ns.wisely.priv. admin.wisely.priv. (
        2014112701      ; Serial
        8H        ; Refresh
        2H        ; Retry
        4W        ; Expire
        1D)       ; Minimum TTL

        NS  ns.wisely.priv.

$ORIGIN wisely.priv.
deis1            A   192.168.1.107
deis2            A   192.168.1.108
deis3            A   192.168.1.109

*                 A   192.168.1.107
*                 A   192.168.1.108
*                 A   192.168.1.109
```
# 3. deis客户端使用