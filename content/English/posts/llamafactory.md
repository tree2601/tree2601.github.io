---
date: '2026-01-21T23:06:53+08:00'
draft: false
title: 'Llamafactory Distributed Training'
categories: ["AI"]
tags: ["AI","Finetuning", "SFT", "Training", "Llamafactory"]
---

### Content

Conduct SFT training using the llamafactory framework on L20*8 servers with Ubuntu 22.04. Utilize both single-node multi-GPU and multi-node multi-GPU modes. Selected base model: Qwen3-32B.

### **Environment Configuration**

1.  Clone the code repository, set up a new conda environment, and install dependencies.

```Plain
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
conda activate llamafactory_env
pip install -e .
pip install -r requirements/metrics.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

2.  Prepare SFT data, place it in the `data` folder, and register it in `dataset_info.json`.

```Plain
cd ./data
Open dataset_info.json
and add the dataset, for example:

"my_example": {
    "file_name": "my_example.json"
  },

# Use Alpaca or ShareGPT format for SFT data. Alpaca format example is used here.

# Alpaca format: (where `instruction` and `input` are automatically concatenated with `\n`)
[{
"instruction": "Human instruction (required)",
"input": "Human input (optional)",
"output": "Model response (required)",
"system": "System prompt (optional)",
"history":
[
["First round instruction (optional)", "First round response (optional)"],
["Second round instruction (optional)", "Second round response (optional)"]
]
}]
```

### **Single-Node Multi-GPU Training**

```Plain
# Prepare a yaml file by referring to existing templates in `./examples` and run it.

# If using deepspeed for multi-GPU training, specify the number of GPUs via CUDA_VISIBLE_DEVICES.

CUDA_VISIBLE_DEVICES=0,1 FORCE_TORCHRUN=1 llamafactory-cli train examples/train_lora/qwen3_30b_lora_sft.yaml

# After training, the parameters can be found in the path specified by `output_dir` in the yaml file.
```

After training, you can invoke the LoRA adapter using vLLM. Here is a docker compose template.

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

### **Multi-Node Multi-GPU Training**

Configure identical environments on each server, place the same data, and use the same yaml file. **Ensure consistent torch, nccl, cuda, and deepspeed versions across all servers.**

Launch the command on each server respectively:

```JavaScript
# Assuming the main host IP is 10.128.7.1
Main Host (manager):
FORCE_TORCHRUN=1 NNODES=2 NODE_RANK=0 MASTER_ADDR=10.128.7.1 MASTER_PORT=20000 llamafactory-cli train examples/train_lora/qwen3_30b_lora_sft.yaml

# Assuming the main host IP is 10.128.6.1
Worker Node (worker):
FORCE_TORCHRUN=1 NNODES=2 NODE_RANK=1 MASTER_ADDR=10.128.7.1​ ​MASTER_PORT=20000 llamafactory-cli train examples/train_lora/qwen3_30b_lora_sft.yaml

```