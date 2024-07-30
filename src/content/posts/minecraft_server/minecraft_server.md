---
title: Minecraft-Bedrock服务器快速搭建
published: 2024-05-10
description: '构建Docker镜像来部署基岩版我的世界服务器，在不影响本地存档的条件下实现版本迭代。'
image: ''
tags: [Game, Docker]
category: 'TechTalk'
draft: false 
---



## Download Package ##

服务器需要始终保持最新，落后的版本可能影响玩家连接。

下载地址：[[Click Here]](https://www.minecraft.net/zh-hans/download/server/bedrock)



## 开服脚本 ##

> 适用于Linux服务器，参考环境：`Ubuntu 22.04 LTS` `Docker 24.0.4`
>
> 宿主机映射配置文件与存档至容器，后续维护工作只需修改宿主机下的配置文件并重启容器。

运行 `bash <file_name>.sh` 启动服务器，执行前请先修改配置。

```shell
# basic config
server_pkg="/workspace/game/builder/bedrock-server-1.20.71.01.zip" #"https://minecraft.azureedge.net/bin-linux/bedrock-server-1.20.71.01.zip"
sv_name="YOURCRAFT"
sv_version="1.20.71"
sv_ipv4_port="11451"
sv_ipv6_port="11452"
sv_default_world_file="/workspace/game/builder/world.zip"
sv_default_world_name="world"
sv_data_root="/data/game/minecraft"

config_file="server.properties"
builder_env="minecraft_server_builder"

# clean history works
echo "Try to clean docker builder environment..."
rm -rf $builder_env || true
echo "Try to remove docker contain..."
docker rm -f minecraft_bedrock_server 2>/dev/null || true
echo "Try to remove docker image..."
docker rmi -f minecraft_bedrock:$sv_version 2>/dev/null || true

# initialize work environment
echo "Create work envirment"
mkdir -p $sv_data_root/worlds || true
mkdir $builder_env
cd $builder_env

# get server package
pkg_name=$(basename $server_pkg)
if [[ $server_pkg =~ ^https?:// ]]; then
    echo "Downloading server package..."
        wget $server_pkg
else
        cp $server_pkg $pkg_name
fi
echo "Start unzip $pkg_name"
unzip $pkg_name

# change config
if [[ -f "$sv_data_root/$config_file" ]]; then
		cp -f $sv_data_root/$config_file ./$config_file
fi
sed -i -e "s/server-name=.*/server-name=$sv_name/g" ./$config_file
sed -i -e "s/server-port=.*/server-port=$sv_ipv4_port/g" ./$config_file
sed -i -e "s/server-portv6=.*/server-portv6=$sv_ipv6_port/g" ./$config_file
sed -i -e "s/level-name=.*/level-name=$sv_default_world_name/g" ./$config_file

cp -f ./$config_file $sv_data_root/$config_file

# unzip default world file, skip steps when the target world already exists
if [[ -z "$sv_default_world_file" ]]; then
    echo "World file not configured."
elif [[ -d "$sv_data_root/worlds/$sv_default_world_name" ]]; then
    echo "World already exists."
else
    unzip -n -d $sv_data_root/worlds $sv_default_world_file
fi

tee ./launch.sh <<- EOF
LD_LIBRARY_PATH=. ./bedrock_server
EOF

tee ./Dockerfile <<- EOF
FROM ubuntu:22.04
LABEL maintainer="Tropical Algae<tropicalalgae@gmail.com>" version="1.0"

WORKDIR /game
RUN apt-get update && apt-get install -y libcurl4-openssl-dev screen
COPY . .

EXPOSE $sv_ipv4_port
ENTRYPOINT ["sh", "./launch.sh"]
# ENTRYPOINT ["screen", "-S", "minecraft", "-s", "'./launch.sh'"]
EOF

docker build -t minecraft_bedrock:$sv_version .
docker run -itd -p $sv_ipv4_port:$sv_ipv4_port/udp -p $sv_ipv6_port:$sv_ipv6_port/udp --name minecraft_bedrock_server --restart=always -v $sv_data_root/worlds:/game/worlds -v $sv_data_root/server.properties:/game/server.properties minecraft_bedrock:$sv_version
docker logs -f minecraft_bedrock_server
```

| Variable                | Value                                                     | Describe                                                 |
| ----------------------- | --------------------------------------------------------- | -------------------------------------------------------- |
| `server_pkg`            | 官方提供的安装包，可以是URL或本地文件路径                 | 使用绝对路径                                             |
| `sv_name`               | 服务器名称                                                |                                                          |
| `sv_version`            | 服务器游戏版本                                            |                                                          |
| `sv_ipv4_port`          | 服务器连接IPV4端口                                        |                                                          |
| `sv_ipv6_port`          | 服务器连接IPV6端口                                        | [2024.3.4]暂时没使用到                                   |
| `sv_default_world_file` | 服务器默认加载的World压缩包本地文件路径，空值跳过解压环节 | 使用绝对路径                                             |
| `sv_default_world_name` | 服务器默认加载的World名称                                 | 与压缩包解压后的文件夹名称一致，当文件存在时跳过解压环节 |
| `sv_data_root`          | 宿主机存储配置文件与地图文件的本地路径                    | 使用绝对路径                                             |
| `config_file`           | 服务器配置文件                                            | 官方默认 `server.properties` 勿动                        |
| `builder_env`           | 构建Docker的环境，工作都将在该文件夹下执行                | 文件夹名称，随便命名                                     |

