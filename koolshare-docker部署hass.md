# LEDE x86 koolshare Docker CE Portainer 部署 hass hassio homeassistant home-assistant
## 成果
避免浪费大家时间先说成果吧，我试了很多个基于lean大编译的固件包括自己拉源码编译，都没有在docker上跑成功。大概学习了一下好像是dbus的原因，这部分实在没搞懂。更新：问题已解决，见[另一帖](https://www.jcat.cn/archives/lean-lede-docker-hass)
最后在koolshare上尝试成功了，虽然很多专业人士不看好koolshare，但是个人使用下来还是很稳定的。

非常感谢https://bbs.hassbian.com/thread-8844-1-1.html 帖子，让我找到着手的地方。
## 背景
之前的hass一直部署在树莓派2b上，稳定的用了很长时间，但是最近不知道是天气热还是什么原因频繁死机。
手头上有个nas上淘汰下来很久的j1900，早就配好了pcie网卡准备当软路由。但是一直也没折腾，主要是现在用的linksys 1900ac还很够用。趁着这个机会想换上软路由然后用docker跑起hass。
## 步骤
### 安装koolshare
- koolshare官网下载最新固件
- u盘安装winpe，下载好physdiskwrite.exe备用
- 进入winpe使用命令行刷写固件,输完命令后选择硬盘，注意容量和型号，一般选0回车即可
```
physdiskwrite.exe -u koolshare.img
```
- 若刷写失败，使用DiskGenius删除该硬盘所有分区后重新执行上面的步骤
- 刷写完毕后重启
### koolshare配置
- 重启后192.168.1.1进入后台，密码为koolshare
- 建议修改密码及LAN ip，然后ssh连接后执行以下命令解除敏感词设定
```
sed -i 's/\tdetect_package/\t# detect_package/g' /koolshare/scripts/ks_tar_install.sh
```
- 在酷软中心下载硬盘助手，按照提示将剩余的空间挂载。比如我这边挂载在/mnt/sda3
### docker安装
- 在酷软中心安装docker
- 打开docker配置界面勾选开启服务然后提交，如果成功可忽略后面的步骤
- 第二步基本上是不可能成功的，需要手动下载
```
https://download.docker.com/linux/static/stable/x86_64/docker-18.09.6.tgz
```
- 将下载的包解压
```
#解压
tar -xzvf docker-18.09.6.tgz
mkdir /mnt/sda3/docker/bin
#移动docker执行文件至新建的bin目录
mv docker/* docker/bin
```
- 回到界面，修改docker安装目录为/mnt/sda3，再勾选开启服务直接点提交应该就可以了。
- 安装docker图形界面Portainer, 可通过http://192.168.1.1:9000 管理容器
```
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --restart=always --name prtainer portainer/portainer
```
### 部署hass
- hass_dns
```
docker run -v /data/hassio/supervisor/dns:/config --restart=always --name hassio_dns homeassistant/amd64-hassio-dns
```
- hass_supervisor
```
docker run -d --privileged=true -v /var/run/dbus:/var/run/dbus -v /var/run/docker.sock:/var/run/docker.sock -v /data/hassio/supervisor:/data -e HOMEASSISTANT_REPOSITORY=homeassistant/qemux86-64-homeassistant -e SUPERVISOR_SHARE=/data/hassio/supervisor -e SUPERVISOR_NAME=hassio_supervisor --restart=always --name hassio_supervisor homeassistant/amd64-hassio-supervisor
```
- 成功启动后会拉起其他需要的容器，最后访问ip:8123即可访问


https://github.com/coolsnowwolf/lede/issues/1760
