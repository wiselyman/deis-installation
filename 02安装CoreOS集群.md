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
## 安装CoreOS
### 准备apache存储image和image签名
### 准备cloud-config.yaml
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