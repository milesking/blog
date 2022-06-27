# 部署OceanBase-CE

## 环境准备

### 添加新用户

所有操作在新用户下进行

```shell
useradd admin
passwd admin
usermod -G wheel admin
```

### 修改目录权限

```shell
chown -R admin.admin /data /redo
```

### 机器规划

| 角色     | 机器           | 备注                  |
| -------  | ------------- | -------------------- |
| obd      | 192.21.10.9 | OceanBase自动化部署工具 |
| observer | 192.21.10.9 | zone1监听2881/2882端口 |
| obproxy  | 192.21.10.9 | (可选)监听2883/2884端口 |

### 修改系统环境

修改会话变量设置

```plain
vi /etc/security/limits.conf
* soft nofile 655360
* hard nofile 655360
* soft nproc 655360
* hard nproc 655360
* soft core unlimited
* hard core unlimited
* soft stack unlimited
* hard stack unlimited
```

## 安装

### 下载配置文件

下载

```plain
部署单节点 observer 进程：https://github.com/oceanbase/obdeploy/blob/master/example/mini-single-example.yaml
部署单节点 observer 和 obproxy 进程：https://github.com/oceanbase/obdeploy/blob/master/example/mini-single-with-obproxy-example.yaml
```

### 修改配置文件(单节点observer)

```yaml
## Only need to configure when remote login is required
user:
  username: root
  password: rootroot
  # key_file: your ssh-key file path if need
  # port: your ssh port, default 22
  # timeout: ssh connection timeout (second), default 30
oceanbase-ce:
  servers:
    # Please don't use hostname, only IP can be supported
    - 192.21.10.9
  global:
    #  The working directory for OceanBase Database. OceanBase Database is started under this directory. This is a required field.
    home_path: /home/admin/observer
    # The directory for data storage. The default value is $home_path/store.
    data_dir: /data
    # The directory for clog, ilog, and slog. The default value is the same as the data_dir value.
    redo_dir: /redo
    # Please set devname as the network adaptor's name whose ip is  in the setting of severs.
    # if set severs as "127.0.0.1", please set devname as "lo"
    # if current ip is 192.168.1.10, and the ip's network adaptor's name is "eth0", please use "eth0"
    devname: eth0
    mysql_port: 2881 # External port for OceanBase Database. The default value is 2881. DO NOT change this value after the cluster is started.
    rpc_port: 2882 # Internal port for OceanBase Database. The default value is 2882. DO NOT change this value after the cluster is started.
    zone: zone1
    cluster_id: 1
    # please set memory limit to a suitable value which is matching resource.
    memory_limit: 8G # The maximum running memory for an observer
    system_memory: 4G # The reserved system memory. system_memory is reserved for general tenants. The default value is 30G.
    stack_size: 512K
    cpu_count: 16
    cache_wash_threshold: 1G
    __min_full_resource_pool_memory: 268435456
    workers_per_cpu_quota: 10
    schema_history_expire_time: 1d
    # The value of net_thread_count had better be same as cpu's core number.
    net_thread_count: 4
    major_freeze_duty_time: Disable
    minor_freeze_times: 10
    enable_separate_sys_clog: 0
    enable_merge_by_turn: FALSE
    datafile_disk_percentage: 20 # The percentage of the data_dir space to the total disk space. This value takes effect only when datafile_size is 0. The default value is 90.
    syslog_level: WARN # System log level. The default value is INFO.
    enable_syslog_wf: false # Print system logs whose levels are higher than WARNING to a separate log file. The default value is true.
    enable_syslog_recycle: true # Enable auto system log recycling or not. The default value is false.
    max_syslog_file_count: 4 # The maximum number of reserved log files before enabling auto recycling. The default value is 0.
    root_password: rootroot
```

### 下载离线安装包

```plain
wget https://mirrors.aliyun.com/oceanbase/community/stable/el/7/x86_64/ob-deploy-1.1.2-1.el7.x86_64.rpm
wget https://mirrors.aliyun.com/oceanbase/community/stable/el/7/x86_64/oceanbase-ce-3.1.1-4.el7.x86_64.rpm
wget https://mirrors.aliyun.com/oceanbase/community/stable/el/7/x86_64/oceanbase-ce-libs-3.1.1-4.el7.x86_64.rpm
wget https://mirrors.aliyun.com/oceanbase/community/stable/el/7/x86_64/obclient-2.0.0-2.el7.x86_64.rpm
wget https://mirrors.aliyun.com/oceanbase/community/stable/el/7/x86_64/libobclient-2.0.0-2.el7.x86_64.rpm
wget https://mirrors.aliyun.com/oceanbase/community/stable/el/7/x86_64/obproxy-3.2.0-1.el7.x86_64.rpm
```

### 安装obddeploy

```plain
rpm -ivh ob-deploy-1.1.2-1.el7.x86_64.rpm
source /etc/profile.d/obd.sh
```

### 配置obddeploy

删除远程仓库

```plain
rm -rf ~/.obd/mirror/remote/OceanBase.repo
```

将下载的文件添加到本地仓库

```plain
obd mirror clone *.rpm
```

查看本地仓库

```plain
obd mirror list local
```

### 执行安装命令

```plain
obd cluster deploy obce-single -c /home/miles/mini-single.yaml
```

### 启动集群

```plain
obd cluster start obce-single
```

## 连接

使用mysql驱动连接，端口2881，用户名密码root/配置文件中的密码
