---
date: '2026-01-07T09:18:13+08:00'
draft: false
title: 'Conda Guide'
categories: ["General"]
tags: ["Conda","DevOps","Ubuntu","Python"]
---

### Overview

This guide covers installing Conda on Ubuntu 22.04, migrating the Conda path to a data disk, configuring mirror sources (for regions with internet restrictions), and methods for packing environments for offline deployment.

### Installation Package

Use `wget` or click to download the [Installation Package](https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh) directly.

### Add Conda to PATH for Persistence

```bash
# Locate the conda command
which conda

# Check the conda root installation directory
conda info --base

# Open ~/.bashrc and add the following line at the bottom
export PATH="/home/ubuntu/miniconda3/bin:$PATH"

# Apply changes
source ~/.bashrc
conda init

# Restart your terminal session
```

### Configure Storage Paths for Environments and Packages

```yaml
# Open ~/.condarc and add the following lines:

envs_dirs:
  - /your/path/to/conda/envs
pkgs_dirs:
  - /your/path/to/conda/pkgs
```

### Configure Conda Mirror Sources (China)

```bash
# Using Aliyun mirrors as an example
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/main/
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/free/
conda config --add channels https://mirrors.aliyun.com/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.aliyun.com/anaconda/cloud/bioconda/

# Verify the configuration
conda config --show channels
```

### Create Conda Environments

```bash
# Create an environment (at a specific path or using the default path)
conda create --prefix /your/path/to/conda/envs/my_env python=3.12
conda create -n my_env python=3.12

# Activate the new environment
conda activate my_env
```

### List Existing Environments

```bash
conda info --envs
```

### Examples: Using pip/uv within Conda

```bash
# Using pip
pip install tqdm -i https://mirrors.aliyun.com/pypi/simple/

# Using uv for faster installation
uv pip install tqdm -i https://mirrors.aliyun.com/pypi/simple/
```

### Pack Conda Environments

```bash
# Pack an existing conda environment:
# (Note: Requires conda-pack to be installed)
conda pack -n my_env

# Transfer the archive to the target server and extract it
tar -xvzf my_env.tar.gz -C /your/conda/envs/my_env
```

### Dockerizing Local Conda Environments

Example `Dockerfile`:

```dockerfile
FROM miniconda3:25.3.1

# Copy the compressed virtual environment
COPY my_env.tar.gz /tmp

# Copy your scripts or folders
# COPY /your/python/script.py /workspace

# Create the target directory
RUN mkdir -p /opt/conda/envs/my_env

# Extract the virtual environment
RUN tar -xzvf /tmp/my_env.tar.gz -C /opt/conda/envs/my_env && rm /tmp/my_env.tar.gz

# Clean Conda cache
RUN conda clean --all --yes

# Set default Shell to Bash
SHELL ["/bin/bash", "-c"]

# Activate the environment in .bashrc
RUN echo "conda activate my_env" >> ~/.bashrc

# Set the working directory
WORKDIR /workspace

# Start Bash by default
CMD ["/bin/bash"]
```