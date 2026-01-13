---
date: '2026-01-07T09:18:13+08:00'
draft: false
title: 'Docker Guide'
categories: ["General"]
tags: ["Docker","DevOps","Ubuntu"]
---

### Overview

This guide provides a comprehensive walkthrough for installing Docker on Ubuntu 22.04, migrating the Docker root directory to a data disk, configuring registry mirrors (for regions with internet restrictions), and setting up the NVIDIA Container Toolkit for GPU acceleration.

### 1. Install Dependencies

```bash
sudo apt update && sudo apt install -y ca-certificates curl gnupg lsb-release
```

### 2. Import Docker GPG Key (Aliyun Mirror)

```bash
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/aliyun-docker.gpg
```

### 3. Register the Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/aliyun-docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 4. Install Docker Engine

```bash
sudo apt update
sudo apt install -y docker-ce

# Docker starts automatically on Ubuntu. For WSL, use:
sudo service docker start
```

### 5. Verify Installation

```bash
sudo docker info
```

### 6. Manage Docker as a Non-Root User

```bash
# Create the docker group if it doesn't exist
sudo groupadd docker

# Add your user to the group
sudo usermod -aG docker $USER

# Apply group changes without logging out
newgrp docker
```

### 7. Operational Cheat Sheet

```bash
# List all containers (including stopped ones)
docker ps -a 

# List all local images
docker images

# Debug the Docker daemon (useful if the service fails to start)
dockerd 

# Retag an image
docker tag <image-name-1>:<tag-1> <image-name-2>:<tag-2> 

# Remove an image
docker rmi <image-id-or-name>

# Follow container logs (Ctrl+C to exit)
docker logs -f <container-id>

# Inspect container metadata
docker inspect <container-id>

# Force remove a container
docker rm -f <container-id>
```

### 8. Migrating the Docker Root Directory

To prevent the OS drive from filling up, migrate the storage path to a data disk:

```bash
# Sync existing data to the new path
sudo rsync -aP /var/lib/docker/ /mnt/data0/docker

# Edit the configuration file
sudo nano /etc/docker/daemon.json 

# Update the data-root (ensure valid JSON syntax)
{
  "data-root": "/your/new/path"
}

# Reload and restart the service
sudo systemctl daemon-reload
sudo systemctl restart docker

# Verify the new storage location
docker info | grep "Docker Root Dir"
```

### 9. Configuring Registry Mirrors

```bash
# Edit /etc/docker/daemon.json and add mirrors:
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.hlmirror.com"
  ]
}

# Apply changes
sudo systemctl daemon-reload
sudo systemctl restart docker

# Verification
docker info

# Tip: Use a specific mirror for a one-time pull:
docker pull docker.1ms.run/<image>:<tag>
```

### 10. Offline Deployment (Export/Import)

```bash
# Export an image to a tarball
docker save -o my-docker-image.tar my-docker-image:tag

# Transfer and load the image on a target server
docker load -i my-docker-image.tar
```

### 11. Configuring NVIDIA Container Toolkit

Check if NVIDIA drivers are installed:
```bash
nvidia-smi
```

#### Online Installation

```bash
# Import GPG key
curl -fsSL https://mirrors.ustc.edu.cn/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Add the repository (using USTC mirror)
curl -s -L https://mirrors.ustc.edu.cn/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install the toolkit
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

#### Offline Installation
Download the following `.deb` packages from an online environment and install them in order:
1. `nvidia-container-toolkit-base.deb`
2. `libnvidia-container1.deb`
3. `libnvidia-container-tools.deb`
4. `nvidia-container-toolkit.deb`

#### Docker Integration

```bash
# Configure the Docker runtime
sudo nvidia-ctk runtime configure --runtime=docker

# Restart Docker
sudo systemctl restart docker

# Enable Persistence Mode to reduce nvidia-smi latency
sudo nvidia-smi -pm 1
```