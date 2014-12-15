# 1. 编译deisctl
这步可以放在CoreOS集群安装之前，若你采用的是deis1.0.2我已将编译好的![](https://github.com/wiselyman/deis-installation/blob/master/01resources/deisctl)放在此处。
## 1.1 下载deis1.0.2
- 地址：https://github.com/deis/deis/archive/v1.0.2.zip
- 安装go语言并配置`export GOPATH=/root/workspace`
- 将文件解压在`/root/workspace/src/github.com/deis`目录下，注意此deis为deis用户不是deis平台，解压后是`deis/deis`
- 安装godep，在国内无法安装godep，将编译好的[godep](https://github.com/wiselyman/deis-installation/blob/master/01resources/godep)放再此下载，放置在`/usr/local/bin`下
- `cd /root/workspace/src/github.com/deis/deis`
- `make -C deisctl build`
-  这时候编译好的deisctl在`/root/workspace/src/github.com/deis/deisctl`目录下

# 2. 安装deis平台
## 2.1 pull deis docker images
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

ubuntu-debootstrap  docker pull ubuntu-debootstrap

progrium/cedarish docker pull progrium/cedarish 
```
## 2.2 将这些镜像push到本地docker registry
举下面一例，所有都要操作
```
docker tag img_id localhost:5000/deis/base

docker push localhost:5000/deis/base
```
## 2.3 编译deis/builder
### 2.3.1 修改dies/builder的Dockerfile
- 修改安装![](https://github.com/wiselyman/deis-installation/blob/master/01resources/etcdctl-v0.4.6)路径为内网地址`RUN curl -sSL -o /usr/local/bin/etcdctl http://192.168.1.103/opdemand/etcdctl-v0.4.6`
- 修改安装[confd](https://github.com/wiselyman/deis-installation/blob/master/01resources/confd-v0.5.0-json)路径为内网`RUN curl -sSL -o /usr/local/bin/confd http://192.168.1.103/opdemand/confd-v0.5.0-json`
- 注释下载progrium_cedarish_2014_10_01.tar 
```
# HACK: import progrium/cedarish as a tarball
# see https://github.com/deis/deis/issues/1027
#RUN curl -#SL -o /progrium_cedarish.tar \
#    http://192.168.1.103/opdemand/progrium_cedarish_2014_10_01.tar 
```
### 2.3.2 修改slugrunner的Dockerfile
第一句修改为`FROM 192.168.1.103:5000/progrium/cedarish:latest`
### 2.3.3 修改slugbuilder的Dockerfile
第一句修改为`FROM 192.168.1.103:5000/progrium/cedarish:latest`
### 2.3.4 修改slugbuilder/builder/install-buildpacks
这个文件里会下载外网的git，所以需要你将github上的git克隆到本地的git server上，本文使用gitblit，修改如下：
```
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-multi.git         9350571
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-ruby.git           v127
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-nodejs.git         v60
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-java.git           658ecd2
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-gradle.git         743f73c
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-grails.git         1ef927d
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-play.git           ceede86
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-python.git         v52
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-php.git            v43
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-clojure.git        bc2bfd8
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-scala.git          v41
download_buildpack http://admin@192.168.1.110:8080/r/heroku-buildpack-go.git             b261aab
```
#### 2.3.4.1 针对java的修改
对于heroku-buildpack-java，当使用buildpack发布java程序的时候会从外网下载JDK和maven等，这时候需要下载jdk和maven放在内网，我们这时候也需要修改hero-buildpack-java的代码。
- /bin/common:` mavenUrl="http://192.168.1.103/maven/maven-${mavenVersion}.tar.gz"`
- /bin/compile 
 - 下载http://lang-jvm.s3.amazonaws.com/jvm-buildpack-common-v7.tar.gz
 - 用7zip打开并修改内如如图：
   ![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/heroku-buildpack-java.jpg)
 - `JVM_COMMON_BUILDPACK=http://192.168.1.103/jvm-buildpack-common-v7.tar.gz`
- 此时java的git版本是658ecd2，如上。

### 2.3.5 deis/builder
- `make build`
- push到本地docker registry

### 2.3.6 安装deis平台

#### 2.3.6.1 修改deisctl/units下的服务
- 修改所有涉及ubuntu-debootstrap为`192.168.1.103:5000/ubuntu-debootstrap`。
- 其余镜像相关的无须修改，因我在每台cloud config里的get_image脚本中已修改。

#### 2.3.6.2 上传units目录到/home/core/.deis目录

#### 2.3.6.3 上传deis.pub到/home/core/.ssh目录

#### 2.3.6.4开始安装
- `chmod 0600 /home/core/.ssh/deis  `
- `eval `ssh-agent -s`  `
- `ssh-add ~/.ssh/deis `
- `export DEISCTL_TUNNEL=192.168.1.107`
- `deisctl config platform set sshPrivateKey=~/.ssh/deis  `
- `deisctl config platform set domain=wisely.priv`
- `deisctl install platform  `
- `deisctl start platform  `
- 成功后效果：
  ![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/deis-success.jpg)
