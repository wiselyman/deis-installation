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

## 3.1 注册用户
`deis register http://deis.wisely.priv` 第一个注册用户为管理员账号。

## 3.2 上传ssh公钥(在使用buildpack发布程序时必须)
- 将deis的公钥上传工作机的/home/user/.ssh下
- `deis keys:add` 按提示操作
- ``eval `ssh-agent -s` ``
- `ssh-add ~/.ssh/deis`

## 3.3 使用hero-buildpacks-java发布spring boot jar程序
参见：https://github.com/wiselyman/deis-spring-boot

## 3.4 使用docker image发布spring boot jar程序

### 3.4.1 Dockerfile
使用下面Dockerfile编译docker image
```
FROM ingensi/oracle-jdk:centos6-7u65

MAINTAINER wiselyman

ADD platform-0.0.1-SNAPSHOT.jar /app/

WORKDIR /app/

EXPOSE  8888

CMD ["java","-jar","platform-0.0.1-SNAPSHOT.jar"]
```

### 3.4.2 编译docker image
```
docker build -t localhost:5000/platform .
docker push localhost:5000/platform
```

### 3.4.3 发布
- `mkdir -p /tmp/platform && cd /tmp/platform`
- `deis create`
- `deis pull 192.168.1.103:5000/platform`

## 3.5 应用自动扩展
- 对于buildpack发布的程序`deis scale web=2`
- 对于docker image发布的程序`deis scale cmd=2`

## 3.6 常用命令
- 登录deis`deis login http://deis.wisely.priv   `
- 登出deis `deis logout`
- app列表`deis apps:list`
- 查看单个app信息`deis apps:info --app=platform`
- 删除app`deis destroy --app=platform`
