#编译deisctl
这步可以放在CoreOS集群安装之前，若你采用的是deis1.0.2我已将编译好的![](https://github.com/wiselyman/deis-installation/blob/master/01resources/deisctl)放在此处。
##下载deis1.0.2
- 地址：https://github.com/deis/deis/archive/v1.0.2.zip
- 安装go语言并配置`export GOPATH=/root/workspace`
- 将文件解压在`/root/workspace/src/github.com/deis`目录下，注意此deis为deis用户不是deis平台，解压后是`deis/deis`
- 安装godep，在国内无法安装godep，将编译好的[godep](https://github.com/wiselyman/deis-installation/blob/master/01resources/godep)放再此下载，放置在`/usr/local/bin`下
- `cd /root/workspace/src/github.com/deis/deis`
- `make -C deisctl build`
-  这时候编译好的deisctl在`/root/workspace/src/github.com/deis/deisctl`目录下

#安装deis
#安装deis客户端
##编译deis客户端
##配置DNS