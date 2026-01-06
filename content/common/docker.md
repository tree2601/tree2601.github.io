---
date: '2026-01-05T14:51:30+08:00'
draft: false
title: 'Docker 部署流程'
weight: 3
---


### 内容

在国内的网络环境下，在Ubuntu 22.04服务器上安装 docker，修改docker镜像存放位置至数据盘，配置使用国内镜像源，以及配置英伟达驱动使用方法。

1. ### 安装必要工具

```Plain
sudo apt update && sudo apt install -y ca-certificates curl gnupg lsb-release
```

2. ### 添加阿里源 Docker 镜像仓库证书证书

```Plain
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/aliyun-docker.gpg
```

3. ### 添加仓库

```Bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/aliyun-docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. ### 安装docker

```Plain
sudo apt update
sudo apt install -y docker-ce


# Ubuntu系统安装完以后自动启动，WSL可能需要额外一行命令：
sudo service docker start
```

5. ### 验证docker启动

```Plain
sudo docker info
```

6. ### 当前用户加入docker用户组

```Plain
#如果没有docker group
sudo groupadd docker

sudo usermod -aG docker $USER
newgrp docker
```

7. ### 运维相关

```Plain
#查看所有的container示例
docker ps -a 

#查看所有的镜像
docker images

#查看docker运行日志，在启动失败时可用于排查问题
dockerd 

#用于修改docker镜像的名称,请自行修改<image-name-1>,<tag-1>,<image-name-2>, <tag-2>的内容
docker tag <image-name-1>:<tag-1> <image-name-2>:<tag-2> 

#移除docker镜像名 / 移除docker镜像
docker rmi A1:tag1 

#查看一个容器的日志,使用ctrl+C 退出
docker logs -f <container-1>

#查看一个容器的信息
docker inspect <container-1>

#删除一个容器
docker rm -f <container-1>
```

8. ### 镜像储存位置修改

```Plain
#docker 镜像 迁移存储路径
sudo rsync -aP /var/lib/docker/ /your/new/path

#修改docker配置
打开 /etc/docker/daemon.json 

#加入新位置,,维持daemon.json的语法
{
["data-root":"/your/new/path"]
}


#重启docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 确认docker文件位置
docker info | grep "Docker Root Dir"
```

9. ### 换国内源

```Plain
打开 /etc/docker/daemon.json

加入源地址,维持daemon.json的语法
  {
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.hlmirror.com",
    "your-other-source.com"
  ]
  }

# 加载并重启docker
sudo systemctl daemon-reload
sudo systemctl restart docker


#确认换源成功
docker info
--------------------------------------------------

#或者可以一次性使用源：
docker pull docker.1ms.run/<image>:<tag>
```

10. ### 配置Nvidia-Container-Toolkit

```Plain
# 确认 Nvidia 驱动已经安装：
nvidia-smi


...未完待续
```
