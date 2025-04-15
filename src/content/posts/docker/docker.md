---
title: Docker
published: 2023-10-01
image: "./cover.jpg"
description: Summary of Docker commands.
tags: [Docker]
category: Documents
draft: false
---

## Install ##

### 安装Docker ###

安装之前，先卸载已安装的Docker：

```shell
sudo apt-get remove docker docker-engine docker.io
```

使用脚本，通过阿里云镜像安装：

```
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun

/* -------------------------------------------------------- */
// 2024.6.17更新 关于国内无法访问Docker的问题，使用如下代理安装
sudo curl -fsSL https://github.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun

# 备用
sudo curl -fsSL https://gitee.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun
```

修改/创建`daemon.json`文件，添加镜像站：

```
vim /etc/docker/daemon.json
```

```
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1panel.live",
        "https://hub.rat.dev"
    ]
}
```

重启docker，配置开机自启：

```
sudo systemctl enable docker
systemctl daemon-reload 
systemctl restart docker
```



### 安装Docker compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose version
```





## Basic Command

```
docker version
docker info

docker search <镜像名> #查找镜像
docker search --filter=STARS=9000 mysql  #搜索 STARS=9000的 mysql 镜像
docker system df -v  # 查看docker磁盘占用
```



## Image

```shell
# 查看全部docker镜像
docker images

# 拉取tag版本镜像   https://hub.docker.com/search?type=image&q=
docker pull <镜像名>:<tag>  

# 删除镜像 (后接多个可批量删除,空格隔开
docker rmi -f <镜像名/镜像ID>

# 保存镜像
docker save -o <镜像文件.tar> <镜像名>:<tag>

# 加载镜像
docker load -i <镜像地址>

# 删除悬虚镜像
docker image prune

# 构建镜像 ( By Dockerfile
docker build -t <image_name> .

# 查看镜像详细信息
docker inspect <镜像ID>

# 删除悬虚镜像
docker image prune
```



## Container

```shell
# 查看全部镜像
docker ps -a

# 基于镜像创建容器
docker run <相关参数> <镜像 ID> <初始命令>
docker run -i -t -v /root/software/:/mnt/software/ 9f38484d220f /bin/bash
#	-i：表示以“交互模式”运行容器
#	-t：表示容器启动后会进入其命令行
#	-v：表示需要将本地哪个目录挂载到容器中，格式：-v <宿主机目录>:<容器目录>

# 存为镜像
docker commit -a "tropical_algae" -m "test" <container id> <image name>:<tag>
#	-a 提交人的姓名
#	-m 提交内容  现有容器ID

# 容器重命名
docker rename <old> <new>  

# 启动容器
docker start <容器ID>

# 停止容器
docker stop <容器ID>

# 查看容器配置
docker inspect <容器ID>

# 查看容器状态
docker stats <容器ID>

# 删除容器
docker rm <容器id>  

# 运行容器
docker exec -ti --user root <container id> /bin/bash

# 查看容器日志
docker logs -f <container id>

# 进入容器交互
docker attach <container id>
```



## 我的常用

```shell
# 深度学习镜像启动
docker run -itd \
-p 5444:22 --name <container name> \
--gpus all -e NVIDIA_VISIBLE_DEVICES=all \
-m 128g --cpus=32 --shm-size=64g \
--restart=always \
--device /dev/nvidiactl:/dev/nvidiactl --device /dev/nvidia-uvm:/dev/nvidia-uvm \
--device /dev/nvidia0:/dev/nvidia0 --device /dev/nvidia1:/dev/nvidia1 --device /dev/nvidia2:/dev/nvidia2 --device /dev/nvidia3:/dev/nvidia3 \
-v /datasets/dataset:/dataset:ro \
<image name>:<tag>

# 删除全部镜像  -a 显示全部, -q 只显示ID
docker rmi -f $(docker images -aq)

# 查看镜像实际版本
docker image inspect <images>:latest | grep -i version

# 启动全部容器
docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)  

```

