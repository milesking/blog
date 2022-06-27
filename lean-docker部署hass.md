# LEDE x86 Lean Docker CE Portainer 部署 hass hassio homeassistant home-assistant
## 成果
最终在Lean lede x86 上用docker跑起了hass。

## 背景
最开始尝试在Lean lede上用docker安装hass，碰到的问题是docker无法访问外网。于是转到koolshare，安装没有问题，但是重启之后没办法自动启动docker，再加上很多软件比如IPSEC出现配置环境失败，于是还是折腾Lean lede。既然不能访问外网，就从解决这个问题开始。

## 步骤
- 自行编译Lean lede x86，或者下载别人编译好的，我这边使用的是efi引导的固件。
- 使用win pe + 写盘工具，我用的是DiskImage，也可以使用纯命令行的physdiskwrite。
- 重启之后修改管理员密码，ssh连入。
- 在服务tab下找到docker ce，有个无脑配置文档，配置完成之后开始修复docker不能联网的问题。
- 我参照的是这个[github issue](https://github.com/coolsnowwolf/lede/issues/1760), vim /etc/sysctl.conf，添加下列内容：
```/etc/caddy/sites/blog
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
```
Luci > 网络 > 防火墙 > 转发：接受
Luci > 状态 > 防火墙 > 重启防火墙
ssh 执行 service dockerd restart
- 重启之后可以开始安装hass相关容器。
- hass_dns, ssh执行以下命令, 在[hassbian论坛](https://bbs.hassbian.com/thread-8844-1-1.html)下载corfile.rar，解压后将两个文件放入/opt/docker/volumes/hassio/supervisor/dns。
```
docker run -v /opt/docker/volumes/hassio/supervisor/dns:/config --restart=always --name hassio_dns homeassistant/amd64-hassio-dns
```
- hass_supervisor，ssh执行以下命令，成功后supervisor会拉取最新的hass相关image并创建容器，观察hass_supervisor开始拉取image一般就没有问题了。
```
docker run -d --privileged=true -v /var/run/dbus:/var/run/dbus -v /var/run/docker.sock:/var/run/docker.sock -v /opt/docker/volumes/hassio/supervisor:/data -e HOMEASSISTANT_REPOSITORY=homeassistant/qemux86-64-homeassistant -e SUPERVISOR_SHARE=/opt/docker/volumes/hassio/supervisor -e SUPERVISOR_NAME=hassio_supervisor --restart=always --name hassio_supervisor homeassistant/amd64-hassio-supervisor
```
- 访问软路由地址:8123即可访问，执行上面的命令之后会先拉取homeassistant:landingpage的image，这时可以访问但是没有功能，后台会拉取homeassistant的最新镜像。

如有问题欢迎留言。