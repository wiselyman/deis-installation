#准备安装CoreOS
###准备CoreOS安装文件
1. ISO:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_iso_image.iso
2. image:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2
3. image签名:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2.sig

# 安装CoreOS
## 在XenServer上安装CoreOS
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/xenserver1.jpg)
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/xenserver2.jpg)
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/xenserver3.jpg)
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/xenserver4.jpg)
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/xenserver5.jpg)
启动成功后，是一个运行在光盘里的系统，我们需要把它安装到硬盘里。
## 安装CoreOS
### 准备apache存储image和image签名
因为CoreOS安装时候会在线下载image和image签名，故将这两个文件下载放置在工作机（192.168.1.103）上。
在CoreOS的ISO里找到coreos-install脚本，修改下载位置为本地:
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/coreos-install1.jpg)
![](https://raw.githubusercontent.com/wiselyman/deis-installation/master/01resources/coreos-install2.jpg)
### 准备cloud-config.yaml
CoreOS的配置都是通过cloud-config.yaml来配置的，这里不作示例，在下面的集群配置里会专门贴出安装deis所需要的配置。
### 配置静态网络
因我所在的内网没有dhcp，所以需要对当前机器配置静态IP才能访问apache下载安装所需的文件。
`sudo vi static.network  `
```
[Match]
Name=eth0 #网卡名

[Network]
Address=192.168.1.107/24
Gateway=192.168.1.254
```
保存退出，执行一下命令生效。
`sudo systemctl restart systemd-networkd`
### 安装
- 获得cloud config:wget http://192.168.1.103/107deis.yaml
- 获得coreos-install:wget http://192.168.1.103/coreos-install
- 为coreos-install赋权限`chmod +x coreos-install`
- `sudo ./coreos-install -d /dev/xvda -C alpha -c 107deis.yaml`

## XenServer对CoreOS的特殊配置
XenServer不支持CoreOS的双系统启动，在安装完成后作一下修改：
```
sudo -s

mount LABEL=EFI-SYSTEM /mnt

echo "DEFAULT coreos.A" > /mnt/syslinux/default.cfg

umount /mnt
```
eject ISO，重启安装成功。

# 安装CoreOS集群
安装集群的方式是在三台服务器分别重复上述步骤，下面是3台cloud config
- [`107deis.yaml`](https://github.com/wiselyman/deis-installation/blob/master/01resources/107deis.yaml)
- [`108deis.yaml`](https://github.com/wiselyman/deis-installation/blob/master/01resources/108deis.yaml)
- [`109deis.yaml`](https://github.com/wiselyman/deis-installation/blob/master/01resources/109deis.yaml)

##配置说明
对于install-deisctl.service，可自行编译好deisctl放置在apache上让安装过程下载。
我暂且将编译好的[deisctl](https://github.com/wiselyman/deis-installation/blob/master/01resources/deisctl)放在这里。在讲述《[安装deis平台](https://github.com/wiselyman/deis-installation/blob/master/03%E5%AE%89%E8%A3%85deis%E5%B9%B3%E5%8F%B0.md)》时，我会讲述如何编译deisctl。