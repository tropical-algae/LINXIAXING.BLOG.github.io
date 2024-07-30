---
title: 服务器快速迁移
published: 2022-10-14
description: 新服务器一键安装库，数据备份与迁移，定时任务等，仅适个人使用。
tags: [Linux]
category: Records
draft: false
---



### 服务安装

```
apt-get update
apt-get install -y vim wget openssh-server git zsh

# ssh配置
sed -i -e '$a\PermitRootLogin yes' -e '$a\PasswordAuthentication yes' /etc/ssh/sshd_config
service ssh restart
sudo systemctl enable ssh

# 安装anaconda
wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh
chmod 777 Anaconda3-2024.02-1-Linux-x86_64.sh
./Anaconda3-5.3.0-Linux-x86_64.sh

# 配置zsh
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O - | sh
git clone https://github.com/zsh-users/zsh-autosuggestions /root/.oh-my-zsh/custom/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting /root/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
chsh -s /bin/zsh
sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting)/' ~/.zshrc

# 安装docker
sudo curl -fsSL https://github.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun
sed -i '$a\{\
    "registry-mirrors": [\
        "https://docker.m.daocloud.io",\
        "https://docker.1panel.live",\
        "https://hub.rat.dev"\
    ]\
}' /etc/docker/daemon.json
systemctl enable docker
systemctl daemon-reload 
systemctl restart docker
```



### 数据迁移

```

```
