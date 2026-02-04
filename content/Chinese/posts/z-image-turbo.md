---
date: '2026-02-04T22:13:42+08:00'
draft: false
title: 'Z Image Turbo'
categories: ["人工智能"]
tags: ["Comfyui","Multimodal","Python","Workflow","Z-image-turbo"]
---

### **内容**

本文介绍使用Z-Image-Turbo配合ComfyUI的使用方法。

Z-Image-Turbo的优点：1. 较强的中文提示词跟随能力和汉字生成能力 2. 仅需8步推理即可生图，且参数量仅6B，配合量化可以在家用主机上运行(16G显存)。

竞争对手: Flux-2系列

考虑到某些地区无法使用ComfyUI-manager自动下载的网络限制，文件下载均以手动的方式。

### 前置准备工作

配置[ComfyUI](/posts/comfyui.md/), 需要安装controlnet组件，可以额外安装llama-cpp-vlm文件来实现基于Qwen3-VL的图生文反推。如果需要查看反推的文本，可以安装comfyUI-custom-scripts。

1. Controlnet 代码仓库:     [这里](https://github.com/Fannovel16/comfyui_controlnet_aux)
2. Llama-cpp-vlm代码仓库：   [这里](https://github.com/lihaoyun6/ComfyUI-llama-cpp_vlm)
3. ComfyUI-custom-scripts代码仓库:    [这里](https://github.com/pythongosssss/ComfyUI-Custom-Scripts)

***注: Llama-cpp-vlm中需要的一个llama-cpp-python.whl插件的下载地址和下面和Qwen3-VL模型下载链接一并列出***

### **需要下载的模型汇总**

1. Z-image-turbo三件套，下载链接里包含满血版和量化版，运行的时候diffusion\_models和text\_encoders可以选择量化版以节省显存开销。下载后以图中的方式放入comfyui的目录中: [**模型下载链接**](https://www.modelscope.cn/models/Comfy-Org/z\_image\_turbo/summary)

![directory-1](/images/z-image-turbo/directory-1.png)


2. ControlNet基础模型: [**模型下载链接**](https://modelscope.cn/models/PAI/Z-Image-Turbo-Fun-Controlnet-Union-2.1/summary)

![directory-2](/images/z-image-turbo/directory-2.png)


3. ControlNet 人物动作控制模型，需要body\_pose\_model.pth,hand\_pose\_model.pth和facenet.pth:  [**模型下载链接**](https://modelscope.cn/models/soulteary/ControlNet/files)

![directory-3](/images/z-image-turbo/directory-3.png)



4. ControlNet 深度控制模型: [**模型下载链接**](https://www.modelscope.cn/models/depth-anything/Depth-Anything-V2-Large/files)

![directory-4](/images/z-image-turbo/directory-4.png)


5. Qwen3-VL 模型 + 相关插件所需wheel文件: [**模型下载链接**](https://modelscope.cn/models/Qwen/Qwen3-VL-4B-Instruct-GGUF) 以及 [**wheel下载**](https://github.com/JamePeng/llama-cpp-python) ***下载前请核对系统版本和python版本***

![directory-5](/images/z-image-turbo/directory-5.png)



### 常用的工作流搭建：

1. 文生图

![workflow-1](/images/z-image-turbo/workflow-1.png)

2. 图生图(边缘检测)

![workflow-2](/images/z-image-turbo/workflow-2.png)

3. 图生图(人物姿势检测)

![workflow-3](/images/z-image-turbo/workflow-3.png)

4. 图生图(深度检测)

![workflow-4](/images/z-image-turbo/workflow-4.png)

5. 图生图(蒙版局域绘图)

![workflow-5](/images/z-image-turbo/workflow-5.png)

6. Qwen3-VL反推后文生图

![workflow-6](/images/z-image-turbo/workflow-6.png)

### 可以直接使用的图片工作流示例(下载后直接导入): 
[**点击这里下载**](/images/z-image-turbo/image-workflows.zip)

1. 基础版本

![image-1](/images/z-image-turbo/image-1.png)

2. controlnet版本

![image-2](/images/z-image-turbo/image-2.png)

3. qwen3-vl反推版本

![image-3](/images/z-image-turbo/image-3.png)
