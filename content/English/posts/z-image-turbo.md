---
date: '2026-02-04T22:13:42+08:00'
draft: false
title: 'Z Image Turbo'
categories: ["AI"]
tags: ["Comfyui","Multimodal","Python","Workflow","Z-image-turbo"]
---

### **Content**

This article introduces the usage of Z-Image-Turbo in conjunction with ComfyUI.

Advantages of Z-Image-Turbo: 
1. **Strong Chinese prompt-following and Chinese character generation capabilities.** 
2. **Requires only 8 inference steps for image generation. With a compact 6B parameter count, it can run on consumer-grade hardware (16GB VRAM) using quantization.**

Due to network restrictions in certain regions that prevent the use of ComfyUI-Manager for automatic downloads, all file downloads are provided for manual installation.

### **Prerequisites**

Configure [ComfyUI](/posts/comfyui.md/). You will need to install ControlNet components. Additionally, you can install the `llama-cpp-vlm` extension to enable image-to-text interrogation based on Qwen3-VL. To view the generated text output, install `comfyUI-custom-scripts`.

1. ControlNet Repository: [Here](https://github.com/Fannovel16/comfyui_controlnet_aux)
2. Llama-cpp-vlm Repository: [Here](https://github.com/lihaoyun6/ComfyUI-llama-cpp_vlm)
3. ComfyUI-custom-scripts Repository: [Here](https://github.com/pythongosssss/ComfyUI-Custom-Scripts)

***Note: The download link for the `llama-cpp-python.whl` plugin required by Llama-cpp-vlm is listed below along with the Qwen3-VL model download link.***

### **Model Download Summary**

1. **Z-image-turbo Triad**: The download link includes both full-precision and quantized versions. During execution, you can select the quantized versions for `diffusion_models` and `text_encoders` to minimize VRAM usage. Place them into the ComfyUI directory as shown in the image: [**Download Link**](https://www.modelscope.cn/models/Comfy-Org/z\_image\_turbo/summary)

![directory-1](/images/z-image-turbo/directory-1.png)


2. **ControlNet Base Model**: [** Download Link**](https://modelscope.cn/models/PAI/Z-Image-Turbo-Fun-Controlnet-Union-2.1/summary)

![directory-2](/images/z-image-turbo/directory-2.png)


3. **ControlNet Human Pose Control Model**: Requires `body_pose_model.pth`, `hand_pose_model.pth`, and `facenet.pth`: [**Download Link**](https://modelscope.cn/models/soulteary/ControlNet/files)

![directory-3](/images/z-image-turbo/directory-3.png)



4. **ControlNet Depth Control Model**: [**Download Link**](https://www.modelscope.cn/models/depth-anything/Depth-Anything-V2-Large/files)

![directory-4](/images/z-image-turbo/directory-4.png)


5. **Qwen3-VL Model + Wheel files for required plugins**: [**Download Link**](https://modelscope.cn/models/Qwen/Qwen3-VL-4B-Instruct-GGUF) and [**Wheel Download**](https://github.com/JamePeng/llama-cpp-python) ***Please verify your system version and Python version before downloading.***

![directory-5](/images/z-image-turbo/directory-5.png)



### **Common Workflow Setups**

1. **Text-to-Image (txt2img)**

![workflow-1](/images/z-image-turbo/workflow-1.png)

2. **Image-to-Image (Canny Edge Detection)**

![workflow-2](/images/z-image-turbo/workflow-2.png)

3. **Image-to-Image (Human Pose Detection)**

![workflow-3](/images/z-image-turbo/workflow-3.png)

4. **Image-to-Image (Depth Detection)**

![workflow-4](/images/z-image-turbo/workflow-4.png)

5. **Image-to-Image (Inpainting / Masked Generation)**

![workflow-5](/images/z-image-turbo/workflow-5.png)

6. **Qwen3-VL Interrogation to Text-to-Image**

![workflow-6](/images/z-image-turbo/workflow-6.png)

### **Ready-to-Use Image Workflow Templates (Import directly after downloading)**
[**Click here**](/images/z-image-turbo/image-workflows.zip)
1. **Basic Version**

![image-1](/images/z-image-turbo/image-1.png)

2. **ControlNet Version**

![image-2](/images/z-image-turbo/image-2.png)

3. **Qwen3-VL Interrogation Version**

![image-3](/images/z-image-turbo/image-3.png)