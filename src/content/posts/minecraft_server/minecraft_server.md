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



## 使用方法 ##

> 适用于Linux服务器，参考环境：`Ubuntu 22.04 LTS` `Docker 24.0.4`
>
> 宿主机映射配置文件与存档至容器，后续维护工作只需修改宿主机下的配置文件并重启容器。

环境变量`.env`，**变量值及注意事项见下表**
```shell
server_pkg="/workspace/temp/mc/bedrock-server-1.21.73.01.zip"
sv_name="YOURCRAFT"
sv_ipv4_port="11451"
sv_ipv6_port="11452"
sv_default_world_file="/workspace/temp/mc/world.zip"
sv_root="/data/minecraft/bedrock"
```

运行 `bash <file_name>.sh` 启动服务器，执行前请先修改配置。

```shell
RED='\033[31m'
GREEN='\033[32m'
YELLOW='\033[33m'
BLUE='\033[34m'
RESET='\033[0m'

if [ -f .env ]; then
    source .env
else
    echo -e "${RED}❌ .env dose not exised! ${RESET}" >&2
    exit 1
fi
world_name="world"
config_file="server.properties"
workspace="minecraft_server_builder"
sv_world_path=${sv_root}/worlds/${world_name}
sv_version=$(echo "${server_pkg}" | grep -oP '\d+\.\d+\.\d+\.\d+')

# clean history works
echo -e "${YELLOW}Try to clean docker builder environment... ${RESET}"
rm -rf $workspace || true
echo -e "${YELLOW}Try to remove docker contain... ${RESET}"
docker rm -f minecraft_bedrock_server 2>/dev/null || true
echo -e "${YELLOW}Try to remove docker image... ${RESET}"
image_ids=$(docker images | grep minecraft_bedrock | awk '{print $3}')

if [[ -z "$image_ids" ]]; then
    echo -e "${YELLOW}⚠️  There is no image named minecraft_cedrock. ${RESET}"
else
    echo -e "${BLUE}Get images named minecraft_bedrock: ${RESET}"
    docker images | grep minecraft_bedrock
    read -p "$(echo -e "${BLUE}Did you wanna delete them? (y/n): ${RESET}")" confirm
    if [[ "${confirm}" == "y" || "${confirm}" == "Y" ]]; then
        echo -e "${BLUE}Deleting images: ${RESET}"
        echo "$image_ids"
        echo "$image_ids" | xargs docker rmi -f
        echo -e "${GREEN}✅ Delete completed! ${RESET}"
    else
        echo -e "${GREEN}🟢 The deletion operation has been canceled. ${RESET}"
    fi
fi

# initialize work environment
echo -e "${GREEN}Create work envirment -> ${workspace} ${RESET}"
mkdir -p ${workspace}
mkdir -p ${sv_root}
cd ${workspace}

# get server package
pkg_name=$(basename ${server_pkg})
if [[ ${server_pkg} =~ ^https?:// ]]; then
    echo -e "${BLUE}Downloading server package... ${RESET}"
    wget ${server_pkg}
else
    if [[ -f "${server_pkg}" ]]; then
        echo -e "${BLUE}Preparing local service files. ${RESET}"
        cp ${server_pkg} ${pkg_name}
    else
        echo -e "${RED}❌ Local service ${server_pkg} does not existed. ${RESET}" >&2
        exit 1
    fi
fi
echo -e "${BLUE}Unzip ${pkg_name}... ${RESET}"
unzip -q ${pkg_name}
rm -f ${pkg_name}

# change config
if [[ -f "${sv_root}/${config_file}" ]]; then
    cp -f ${sv_root}/${config_file} ./${config_file}
fi
sed -i -e "s/^server-name=.*/server-name=${sv_name}/" \
    -e "s/^server-port=.*/server-port=${sv_ipv4_port}/" \
    -e "s/^server-portv6=.*/server-portv6=${sv_ipv6_port}/" \
    -e "s/^level-name=.*/level-name=${world_name}/" \
    "${config_file}"

cp -f ./${config_file} ${sv_root}

# unzip default world file, skip steps when the target world already exists
if [[ -d "${sv_world_path}" ]]; then
    echo -e "${YELLOW}World already exists. ${RESET}"
elif [[ -z "${sv_default_world_file}" ]]; then
    echo -e "${RED}❌ World file not configured. ${RESET}" >&2
    exit 1
elif [[ ! -f "${sv_default_world_file}" ]]; then
    echo -e "${RED}❌ World file '${sv_default_world_file}' does not existed. ${RESET}" >&2
    exit 1
else
    echo -e "${BLUE}Copying world data... ${RESET}"
    mkdir -p ${sv_world_path} || true
    mkdir -p ./temp
    unzip -q ${sv_default_world_file} -d ./temp
    mv ./temp/*/* ${sv_world_path}
    rm -rf ./temp
fi

tee ./launch.sh <<- EOF
LD_LIBRARY_PATH=. ./bedrock_server
EOF

echo -e "${BLUE}Build Dockerfile for service: ${RESET}"

tee ./Dockerfile <<- EOF
FROM debian:bookworm-slim

LABEL maintainer="Tropical Algae<tropicalalgae@gmail.com>" version="1.0"

WORKDIR /game

RUN apt-get update && apt-get install -y --no-install-recommends libcurl4 screen ca-certificates unzip && rm -rf /var/lib/apt/lists/*

COPY . .

EXPOSE ${sv_ipv4_port}

# RUN useradd -m mcroot && chown -R mcroot /game

# USER mcroot

ENTRYPOINT ["sh", "./launch.sh"]
EOF

docker build -t minecraft_bedrock:${sv_version} .

echo -e "${GREEN}✅ Done! Minecraft startup! ${RESET}"

docker run -itd --name minecraft_bedrock_server --restart=always \
    -p ${sv_ipv4_port}:${sv_ipv4_port}/udp -p ${sv_ipv6_port}:${sv_ipv6_port}/udp \
    -v ${sv_root}/worlds:/game/worlds \
    -v ${sv_root}/server.properties:/game/server.properties \
    minecraft_bedrock:$sv_version

docker logs -f minecraft_bedrock_server
```

| Variable                | Value                                                     | Describe                                                 |
| ----------------------- | --------------------------------------------------------- | -------------------------------------------------------- |
| `server_pkg`            | 官方提供的安装包，可以是URL或本地文件路径                 | 使用绝对路径                                             |
| `sv_name`               | 服务器名称                                                |                                                          |
| `sv_ipv4_port`          | 服务器连接IPV4端口                                        |                                                          |
| `sv_ipv6_port`          | 服务器连接IPV6端口                                        | [2024.3.4]暂时没使用到                                   |
| `sv_default_world_file` | 服务器默认加载的World压缩包本地文件路径，空值跳过解压环节 | 使用绝对路径                                             |
| `sv_root`          | 宿主机存储配置文件与地图文件的本地路径                    | 使用绝对路径                                             |


## 版本更新 ##

修改`server_pkg`为最新的service包，重跑脚本即可完成更新

**注意**: 存档文件夹默认为world，更新将不会对该文件夹做任何修改。若希望修改服务器存档，请手动修改配置文件，并将存档文件解压到`sv_root`下。
