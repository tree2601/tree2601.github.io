---
date: '2026-01-21T23:06:53+08:00'
draft: false
title: 'Llamafactory分布式训练'
categories: ["人工智能"]
tags: ["AI","Finetuning", "SFT", "Training", "Llamafactory"]
---

### 内容

在L20\* 8 服务器, Ubuntu 22.04 系统上 使用llamafactory框架 进行 sft 训练。分别使用单机多卡，和多机多卡模式。

### **环境配置**

1. 下载代码仓库,配置一个新的conda环境，并安装依赖

```Plain
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
conda activate llamafactory_env
pip install -e . 
pip install -r requirements/metrics.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

2. 准备sft数据，放在data文件夹，并记录在dataset\_info.json中

```Plain
cd ./data
打开 dataset_info.json
并加入数据集,例如

"my_example": {
    "file_name": "my_example.json"
  },
  
# SFT数据使用Alpaca 或ShareGPT 格式，此处用Alpaca示例

# Alpaca格式：（其中instruction会和input自动用\n拼接）
[{    
"instruction": "人类指令（必填）",    
"input": "人类输入（选填）",    
"output": "模型回答（必填）",    
"system": "系统提示词（选填）",    
"history": 
[
["第一轮指令（选填）", "第一轮回答（选填）"],
["第二轮指令（选填）", "第二轮回答（选填）"]
] 
}]
```

### **单机多卡训练**

```Plain
# 参考./examples里存在的模版准备一个yaml文件并运行即可。

# 如果使用deepspeed 来执行多卡训练，通过 CUDA_VISIBLE_DEVICES 来指定用多少张gpu

CUDA_VISIBLE_DEVICES=0,1 FORCE_TORCHRUN=1 llamafactory-cli train examples/train_full/qwen3_30b_lora_sft.yaml

#训练完以后的参数可以在 yaml 文件里的 output_dir 路径找到
```

训练完后可以用vllm调用lora框架，这里是一个docker compose 的模板

```SQL
qwen3-32b-lora:
    image: vllm/vllm-openai:v0.11.0
    container_name: qwen3-32b-lora
    environment:
      - VLLM_ALLOW_RUNTIME_LORA_UPDATING=False
    deploy:
        resources:
          reservations:
            devices:
              - driver: nvidia
                capabilities: [gpu]
                device_ids: ['4','5','6','7']
    volumes:
      - /your/path/to/qwen3-32b:/root/models
      - /your/path/to/lora_output/checkpoint-xxx:/root/loras
    ports:
      - "10010:8000"
    shm_size: '8g'
    command: >
      --model /root/models
      --host 0.0.0.0
      --port 8000
      --trust-remote-code
      --served-model-name qwen3-32b-lora
      --gpu-memory-utilization 0.8
      --max-model-len 36000
      --tensor-parallel-size 4
      --enable-lora 
      --lora-modules lora1=/root/loras
```

### 多机多卡**训练**

每台服务器上配置完全相同的环境，放入相同的数据，配置相同的yaml文件。**确保每台服务器的torch版本，nccl版本，cuda版本，deepspeed版本一致。**

在每台服务器分别启动命令：

```JavaScript
# 假设主机ip是10.128.7.1
主机(manager):
FORCE_TORCHRUN=1 NNODES=2 NODE_RANK=0 MASTER_ADDR=10.128.7.1 MASTER_PORT=20000 llamafactory-cli train examples/train_lora/qwen3_30b_lora_sft.yaml

# 假设主机ip是10.128.6.1
子节点（worker):
FORCE_TORCHRUN=1 NNODES=2 NODE_RANK=1 MASTER_ADDR=10.128.7.1​ ​MASTER_PORT=20000 llamafactory-cli train examples/train_lora/qwen3_30b_lora_sft.yaml

```

