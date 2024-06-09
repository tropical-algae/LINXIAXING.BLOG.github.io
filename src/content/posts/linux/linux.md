---
title: linux基础
published: 2019-09-21
description: Linux基础命令与操作.
tags: [Linux]
category: Records
draft: false
---



### 重置root密码

```
sudo passwd root
```



### 配置SSH

```
sudo passwd root # 改root密码
apt-get update
apt-get install -y openssh-server
vim /etc/ssh/sshd_config
```

修改以下配置：

```
PermitRootLogin yes #PermitRootLogin prohibit-password
PasswordAuthentication yes
```

重启SSH服务，开启自启：

```
service ssh restart
systemctl enable ssh
```

连接ssh：

```
ssh-keygen  # 产生公私钥对
ssh-copy-id -i ~/.ssh/id_rsa.pub root@<目标ip>  # 复制公钥 ssh连接免密登录
```



### 常用

按关键字kill进程

```
ps -ef | grep keyword | awk '{print $2}' | xargs kill -9
```

内网传输文件

```
scp -r <文件地址> -P <目的端口> root@<目的IP>:<目的地址>
```

查看目录磁盘占用

```
du -ah --max-depth=1
```



### 基本查询

```
# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

| uname -a                | 查看内核/操作系统/CPU信息 |
| ----------------------- | ------------------------- |
| head -n 1 /etc/issue    | 查看操作系统版本          |
| cat /proc/cpuinfo       | 查看CPU信息               |
| hostname                | 查看计算机名              |
| lspci -tv               | 列出所有PCI设备           |
| lsusb -tv               | 列出所有USB设备           |
| lsmod                   | 列出加载的内核模块        |
| env                     | 查看环境变量              |
| lspci \| grep -i nvidia | 查看NVIDIA显卡            |



### 资源查询

| 命令                        | 含义                           |
| :-------------------------- | :----------------------------- |
| free -m                     | 查看内存使用量和交换区使用量   |
| df -h                       | 查看各分区使用情况             |
| du -sh <目录名>             | 查看指定目录的大小             |
| grep MemTotal /proc/meminfo | 查看内存总量                   |
| grep MemFree /proc/meminfo  | 查看空闲内存量                 |
| uptime                      | 查看系统运行时间、用户数、负载 |
| cat /proc/loadavg           | 查看系统负载                   |



### 磁盘和分区查询

| 命令               | 含义                          |                           |
| :----------------- | :---------------------------- | :------------------------ |
| mount              | column -t                     | 查看挂接的分区状态        |
| fdisk -l           | 查看所有分区                  |                           |
| swapon -s          | 查看所有交换分区              |                           |
| hdparm -i /dev/hda | 查看磁盘参数(仅适用于IDE设备) |                           |
| dmesg              | grep IDE                      | 查看启动时IDE设备检测状况 |



### 网络查询

| 命令          | 含义                   |
| :------------ | :--------------------- |
| ifconfig      | 查看所有网络接口的属性 |
| iptables -L   | 查看防火墙设置         |
| route -n      | 查看路由表             |
| netstat -lntp | 查看所有监听端口       |
| netstat -antp | 查看所有已经建立的连接 |
| netstat -s    | 查看网络统计信息       |



### 用户查询

| 命令                    | 含义                   |
| :---------------------- | :--------------------- |
| w                       | 查看活动用户           |
| id <用户名>             | 查看指定用户信息       |
| last                    | 查看用户登录日志       |
| cut -d: -f1 /etc/passwd | 查看系统所有用户       |
| cut -d: -f1 /etc/group  | 查看系统所有组         |
| crontab -l              | 查看当前用户的计划任务 |



### 服务查询

| 命令                       | 含义                   |
| :------------------------- | :--------------------- |
| chkconfig –list            | 列出所有系统服务       |
| chkconfig –list \| grep on | 列出所有启动的系统服务 |



### 查看CPU信息（型号）

- CPU型号
  `cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c`
- 几颗核心
  `cat /proc/cpuinfo | grep physical | uniq -c `
- 查看CPU模式
  `getconf LONG_BIT`
- 查看CPU运算flags
  `cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l`
- 完整看cpu详细信息
  `dmidecode | grep 'Processor Information`
- 查看内存信息
  `cat /proc/meminfo`
- 查看当前操作系统内核信息
  `uname -a`
- 查看当前操作系统发行版信息
  `cat /etc/issue | grep Linux`
- 查看机器型号
  `dmidecode | grep "Product Name`
- 查看网卡信息
  `dmesg | grep -i eth`



### GPU相关命令

- 查看显卡信息
  `lspci | grep -i vga`
- 若使用NVIDIA显卡
  `lspci | grep -i nvidia`
- 查看显卡详情
  `lspci -v -s 00:0f.0`
- 查看显存使用情况
  `nvidia-smi`
- 周期性输出显卡使用情况
  `watch -n 10 nvidia-smi`
- 查看cuda版本
  `cat /usr/local/cuda/version.txt`
- 查看cudnn版本
  `cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2`
