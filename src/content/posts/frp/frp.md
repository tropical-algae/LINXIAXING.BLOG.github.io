---
title: Frp内网穿透&VPN部署
published: 2024-05-09
description: ''
image: ''
tags: [Frp]
category: 'Documents'
draft: false 
---

教程参考版本为 `0.38.0` ，注意新版本的frp部分字段和规范与旧版本有差异。

## FRP 部署 ##

### 服务端frps（公网IP） ###

> 方法：Docker运行服务，并将配置文件映射出来

编写配置文件 `frps.ini`，位置随意。

```
[common]
# 服务器开放给客户端的frp端口
bind_port = 20003
# http端口，访问此端口即可
vhost_http_port = 20000
# https端口
vhost_https_port = 20001
#dashboard_addr = 0.0.0.0
# 控制面板端口,用户名,密码
dashboard_port = 20002
dashboard_user = admin
dashboard_pwd = 1234567890
# 自己用于frp服务的域名，可不填
# subdomain_host = tropicalalgae.cn
# frps服务token,建议是数字组合。拥有此token的客户端才允许连接
token = a0f57a51-af16-4d91-9e75-b08bb5df325b
```

拉取 `0.38.0` 版本的 frp docker 镜像并启动：

```
docker pull snowdreamtech/frps:0.38.0
docker run --restart=always --network host -d -v /<the path of you frps.ini>/frps.ini:/etc/frp/frps.ini --name frps snowdreamtech/frps:0.38.0
```

配置中启用了控制面板时，可访问 `<IP>:<port>` 查看代理状态。

此外记得放开端口的防火墙。

### 客户端frpc（内网IP） ###

> 直接从[github](https://github.com/fatedier/frp/releases)下载包进行配置（实测0.49.0版本可用），以amd64 linux为例：

```
wget https://github.com/fatedier/frp/releases/download/v0.49.0/frp_0.49.0_linux_amd64.tar.gz
tar zxvf frp_0.49.0_linux_amd64.tar.gz 
cd frp_0.49.0_linux_amd64 
vim ./frpc.ini
```

`frpc.ini`的配置内容如下：

```
[common]
# frps服务器IP地址，可用域名
server_addr = 47.96.175.7
# 与上面frps.ini配置中的bind_port一致
server_port = 20003
# frps服务token，与server配置中的token一致
token = a0f57a51-af16-4d91-9e75-b08bb5df325b

[web]
type = http
# 客户机需要开放的本地服务端口
local_port = 1234
# 服务端域名/IP
custom_domains = 47.96.175.7
```

启动服务，访问frps中 `vhost_http_port` 端口即可访问内网：

```
./frpc -c frpc.ini
```

### 将frpc安装为系统服务 ###

上面Client的服务启动方法在SSH链接断开后就会停止。一种更好的方式是装为systemd后台服务：

```
vim /lib/systemd/system/frpc.service
```

```
[Unit]  
Description=FRP Client Daemon  
After=network.target  
Wants=network.target  
  
[Service]  
Type=simple  
# 注意修改frpc和frpc.ini的路径
ExecStart=/workspace/frp/frp_0.49.0_linux_amd64/frpc -c /workspace/frp/frp_0.49.0_linux_amd64/frpc.ini  
Restart=always  
RestartSec=20s  
User=nobody  
  
[Install]  
WantedBy=multi-user.target  
```

重载并启动：

```
systemctl daemon-reload  # 重载命令
systemctl enable frpc  # 添加开机启动
systemctl start frpc  # 启动frpc
systemctl status frpc  # 查看 frpc 状态
```

需要关闭后台服务时执行：

```
systemctl stop frpc
```



## VPN 部署 ##

> VPN部署通过Frp socket5通信实现，部署流程与上面一致，修改配置文件即可

### 服务端frps配置 ###

```
[common]
# 服务器开放给客户端的frp端口
bind_port = 22003
token = 92d40cae-745f-4243-afa5-167bb6808ad2
#dashboard_addr = 0.0.0.0
dashboard_port = 22002
dashboard_user = admin
dashboard_pwd = 1206669327
```

### 客户端frpc配置

```
[common]
server_addr = 47.96.175.7
server_port = 22003
token = 92d40cae-745f-4243-afa5-167bb6808ad2

[socks5]
type = tcp
remote_port = 1234
plugin = socks5
plugin_user = root
plugin_passwd = 4fc4-a855-7287dbe38d0e
use_encryption = true
use_compression = true
```

