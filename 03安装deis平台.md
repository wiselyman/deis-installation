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

#安装deis平台
## pull deis docker images
用工作机pull deis平台的镜像，以备以后内网使用,关于如何建立docker私有registry和将docker image转成tar包方便以后复制到非互联网机器请参照
《[打包docker镜像并使用文件导入](http://wiselyman.iteye.com/blog/2153202) 》
《[建立docker私有库(docker registry](http://wiselyman.iteye.com/blog/2153083)) 》。
下面是需要pull的镜像列表，除了dies/builder需要自己编译，其余可使用官方版本。
```
deis/logger      	docker pull deis/logger:v1.0.2

deis/logspout    	docker pull deis/logspout:v1.0.2

deis/controller         docker pull deis/controller:v1.0.2

deis/builder  		docker pull deis/builder:v1.0.2  自己编译

deis/data  		docker pull deis/data

deis/base 		docker pull deis/base

deis/router		docker pull deis/router:v1.0.2

deis/publisher 		docker pull deis/publisher:v1.0.2

deis/cache 		docker pull deis/cache:v1.0.2

deis/registry 		docker pull deis/registry:v1.0.2

deis/database   	docker pull deis/database:v1.0.2

deis/store-metadata 	docker pull deis/store-metadata:v1.0.2

deis/store-gateway   	docker pull deis/store-gateway:v1.0.2  

deis/store-monitor  	docker pull deis/store-monitor:v1.0.2  

deis/store-daemon 	docker pull deis/store-daemon:v1.0.2  

ubuntu-debootstrap 

progrium/cedarish 
```
##将这些镜像push到本地docker registry
```
docker tag img_id localhost:5000/deis/base

docker push localhost:5000/deis/base
```
##编译deis/builder
###修改dies/builder的Dockerfile
- 修改安装etcdctl路径为内网地址`RUN curl -sSL -o /usr/local/bin/etcdctl http://192.168.1.103/opdemand/etcdctl-v0.4.6`
- 修改安装confd路径为内网`RUN curl -sSL -o /usr/local/bin/confd http://192.168.1.103/opdemand/confd-v0.5.0-json`
###修改slugbuilder

#安装deis客户端
##编译deis客户端
- `cd /root/workspace/src/github.com/deis/deis/client`
- `make build`
##配置DNS