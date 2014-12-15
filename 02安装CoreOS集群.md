#准备安装CoreOS
###准备CoreOS安装文件
1. ISO:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_iso_image.iso
2. image:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2
3. image签名:http://alpha.release.core-os.net/amd64-usr/current/coreos_production_image.bin.bz2.sig
#安装CoreOS
##在XenServer上安装CoreOS
##XenServer对CoreOS的特殊配置
XenServer不支持CoreOS的双系统启动，在安装完成后作一下修改：

#安装CoreOS集群