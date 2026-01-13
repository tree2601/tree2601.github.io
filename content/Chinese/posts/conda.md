---
date: '2026-01-07T09:18:13+08:00'
draft: false
title: 'Conda 使用流程'
categories: ["通用"]
tags: ["Conda","DevOps","Ubuntu","Python"]
---
### 内容

在Ubuntu 22.04 上安装 conda，修改conda路径至数据盘，配置使用国内镜像源，以及打包环境至离线环境方法。

### 安装包

使用wget或者直接点击下载[安装包](https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh)

### 将conda添加至PATH以持久化使用

```Plain
# 查看 conda 命令的位置
which conda

# 查看 conda 的安装根目录
conda info --base

# 打开 ~/.bashrc并在最下面加入 
export PATH="/home/ubuntu/miniconda3/bin:$PATH"

#运行
source ~/.bashrc
conda init

#重开一下窗口
```

### 配置conda的env和package的存储路径

```Plain
# 打开 ~/.condarc并加入

envs_dirs:
  - /your/path/to/conda/envs
pkgs_dirs:
  - /your/path/to/conda/pkgs
```

### 配置conda默认使用国内镜像源

```Plain
# 使用阿里源为例
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/main/
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/free/
conda config --add channels https://mirrors.aliyun.com/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.aliyun.com/anaconda/cloud/bioconda/

#查看结果
conda config --show channels
```

### 创建conda环境

```Plain
# 创造一个conda环境（指定路径或者使用默认路径）
conda create --prefix /your/path/to/conda/envs/my_env python=3.12
conda create -n my_env python=3.12

#激活 新环境
conda activate my_env
```

### 查看已存在conda环境

```Plain
conda info --envs
```

### conda内使用pip/uv示例

```Plain
#使用pip
pip install tqdm -i https://mirrors.aliyun.com/pypi/simple/

#使用uv可以加速
uv pip install tqdm -i https://mirrors.aliyun.com/pypi/simple/
```

### 打包conda环境

```Plain
# 打包一个已经存在的conda环境:
conda pack -n my_env

# 将压缩包传至目标服务器，解压打包好的conda环境
tar -xvzf my_env.tar.gz -C /your/conda/envs/my_env
```

### 将本地conda环境打包成docker镜像以实现容器化部署 
Dockerfile 示例

```Markdown
FROM miniconda3:25.3.1

# 复制压缩的虚拟环境
COPY my_env.tar.gz /tmp

# 复制你的脚本或文件夹
#COPY /your/python/script.py /workspace

# 创建目标目录
RUN mkdir -p /opt/conda/envs/my_env

# 解压虚拟环境
RUN tar -xzvf /tmp/my_env.tar.gz -C /opt/conda/envs/my_env && rm /tmp/my_env.tar.gz

# 清理 Conda 缓存
RUN conda clean --all --yes

# 设置默认 Shell 为 Bash
SHELL ["/bin/bash", "-c"]

# 激活环境
RUN echo "conda activate my_env" >> ~/.bashrc

# 设定默认工作目录
WORKDIR /workspace

# 默认启动 Bash
CMD ["/bin/bash"]
```