
# Wan视频生成项目

**作者**: 杨东勋  
**更新时间**: 2026年3月29日

## 项目简介

本项目基于 **ComfyUI** 平台，集成了 **Wan 2.1 / 2.2** 视频生成模型，提供了从文本到视频、图像到视频、运动控制以及摄像机角度控制等多种视频生成工作流。项目旨在通过直观的图形化界面，让用户能够轻松创建高质量的AI生成视频。

## 技术栈

- **ComfyUI**: 强大且模块化的视觉AI引擎，提供节点式工作流界面
- **Wan 2.1 / 2.2**: 先进的视频生成模型，支持多种生成模式
- **Python 3.10**: 项目运行环境
- **PyTorch + CUDA 13.0**: 深度学习框架与GPU加速
- **Hugging Face**: 模型资源平台

## 使用了什么、为什么这样选

### 1. ComfyUI
- **选择原因**: 提供强大的节点式工作流，支持复杂的AI生成管道设计，无需编码即可实验
- **优势**: 模块化、可扩展、支持多种模型，智能内存管理，支持低显存GPU运行

### 2. Wan 2.1 / 2.2 模型
- **选择原因**: 目前最先进的开源视频生成模型之一，生成质量高
- **优势**:
  - 支持文生视频、图生视频多种模式
  - 提供运动控制和相机控制功能
  - 模型规模适中（5B / 14B参数）
  - 有FP8量化版本，降低显存需求

### 3. Wan2.2-Lightning LoRA
- **选择原因**: 加速推理速度，减少采样步数
- **优势**: 可将采样步数从30+步减少到4步，显著提升生成速度

## 环境配置

### 环境要求
- Miniconda / Anaconda
- Python 3.10
- NVIDIA GPU（推荐）

### 安装步骤

```bash
# 1. 创建conda环境
conda create -n video_wan python=3.10 -y
conda activate video_wan

# 2. 安装PyTorch（CUDA 13.0）
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130

# 3. 安装git和huggingface相关依赖
pip install git huggingface_hub

# 4. 进入ComfyUI目录并安装依赖
cd ComfyUI
pip install -r requirements.txt
```

## 配置教程

### 下载模型文件

项目所需的模型文件需要下载到对应文件夹：

#### 1. VAE模型
```bash
cd ComfyUI/models/vae
wget https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files/vae/wan2.2_vae.safetensors
```

#### 2. 文本编码器
```bash
cd ComfyUI/models/text_encoders
wget https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors
```

#### 3. Diffusion模型
```bash
cd ComfyUI/models/diffusion_models

# 文生视频/图生视频5B模型
wget https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files/diffusion_models/wan2.2_ti2v_5B_fp16.safetensors

# 运动控制高噪声14B模型
wget https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files/diffusion_models/wan2.2_fun_control_high_noise_14B_fp8_scaled.safetensors

# 运动控制低噪声14B模型
wget https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files/diffusion_models/wan2.2_fun_control_low_noise_14B_fp8_scaled.safetensors

# 相机控制1.3B模型
wget https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/diffusion_models/wan2.1_fun_camera_v1.1_1.3B_bf16.safetensors
```

#### 4. Wan2.2-Lightning LoRA（加速推理）
```bash
cd ComfyUI/models/loras

# 高噪声LoRA
wget https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files/loras/wan2.2_i2v_lightx2v_4steps_lora_v1_high_noise.safetensors

# 低噪声LoRA
wget https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files/loras/wan2.2_i2v_lightx2v_4steps_lora_v1_low_noise.safetensors
```

#### 5. CLIP Vision模型（用于相机控制工作流）
```bash
cd ComfyUI/models/clip_vision
wget https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/clip_vision/clip_vision_h.safetensors
```

## 运行方式

### 启动ComfyUI
```bash
cd ComfyUI
python main.py
```

启动后，在浏览器中访问 `http://127.0.0.1:8188` 即可打开ComfyUI界面。

### 加载工作流
在ComfyUI界面中，点击 `Load` 按钮，选择 `/root/workspace/workflows/` 目录下的任意工作流JSON文件即可加载。

## 功能介绍

项目提供4个预配置工作流：

### 1. 文生视频 (text_to_video_wan22_5B.json)
- **功能**: 直接从文本描述生成视频
- **模型**: wan2.2_ti2v_5B_fp16
- **特点**: 无需输入图像，完全基于文本生成

### 2. 图生视频 (image_to_video_wan22_5B.json)
- **功能**: 基于输入图像和文本描述生成视频
- **模型**: wan2.2_ti2v_5B_fp16
- **特点**: 保持输入图像的风格和内容，添加动态效果

### 3. 运动控制 (video_wan2_2_14B_fun_control.json)
- **功能**: 基于参考视频的运动轨迹生成新视频
- **模型**: wan2.2_fun_control_14B_fp8_scaled
- **特点**: 可以精确控制视频中的运动，支持Canny边缘检测控制

### 4. 摄像机角度控制 (wan2.1_fun_camera_1.3B.json)
- **功能**: 基于输入图像，模拟摄像机运动生成视频
- **模型**: wan2.1_fun_camera_v1.1_1.3B_bf16
- **特点**: 支持多种摄像机运动模式（Zoom In、Zoom Out、Pan Left、Pan Right等）

## 核心参数说明

### Wan22ImageToVideoLatent节点
| 参数 | 说明 |
|------|------|
| **width/height** | 视频的分辨率（宽×高），推荐1280x704 |
| **length** | 视频的帧数，直接决定视频时长 |
| **batch_size** | 批量生成的视频数量 |

### SaveAnimatedWEBP节点
| 参数 | 说明 |
|------|------|
| **fps** | 每秒帧数，常见值：24（电影感）、30（常规）、60（高流畅） |
| **lossless** | 是否无损压缩（一般关闭以减小体积） |
| **quality** | 图像质量（仅对有损压缩生效） |
| **method** | 压缩算法（一般保持默认） |

### KSampler节点
| 参数 | 说明 |
|------|------|
| **steps** | 采样步数，影响生成速度和画质，通常20-30步 |
| **cfg** | 引导系数，控制文本对齐程度，通常4-8 |
| **sampler_name** | 采样器类型，推荐uni_pc或euler |

### WanCameraEmbedding节点（相机控制）
| 参数 | 说明 |
|------|------|
| **camera_type** | 摄像机运动类型（Zoom In/Out, Pan Left/Right, Tilt Up/Down等） |
| **width/height** | 输出视频分辨率 |
| **length** | 视频帧数 |
| **motion_scale** | 运动幅度参数 |

## 能力边界

### 支持的功能
- ✅ 文本到视频生成
- ✅ 图像到视频生成
- ✅ 基于参考视频的运动控制
- ✅ 摄像机角度控制（推拉摇移等）
- ✅ 多种输出格式（WebP、WebM、视频文件）
- ✅ LoRA加速推理
- ✅ 灵活的参数调整

### 限制
- ❌ 视频时长有限（通常几秒到十几秒）
- ❌ 长视频可能存在连贯性问题
- ❌ 需要较高的GPU显存（推荐16GB+）
- ❌ 生成速度较慢（尤其是14B模型）
- ❌ 复杂场景可能存在细节失真

## 已知问题

1. **文生视频存在一些失真，色彩丢失**
   - 原因：模型在生成过程中对色彩的还原存在局限性
   - 建议：可以在提示词中强调色彩描述，或使用图生视频模式

2. **模型速度较慢**
   - 原因：模型参数量大（5B/14B），采样步数多
   - 后续优化方向：
     - 结合FastVideo等加速方案
     - 使用Wan2.2-Lightning LoRA减少采样步数到4步
     - 使用FP8量化版本降低计算量

3. **显存占用高**
   - 建议：使用FP8量化模型，或降低分辨率/帧数

## 常见问题

### Q: 需要多少显存才能运行？
A: 推荐至少16GB显存，使用FP8量化模型可以在12GB显存上运行。如果显存不足，可以降低分辨率和帧数。

### Q: 如何加速生成？
A: 
1. 使用Wan2.2-Lightning LoRA，将steps减少到4步
2. 使用FP8量化模型
3. 降低分辨率和帧数
4. 使用euler采样器

### Q: 支持哪些输出格式？
A: 支持WebP（动图）、WebM、以及标准视频格式（MP4等）。

## 项目结构

```
/comfyUI_ai_MVP
├── ComfyUI/                    # ComfyUI主程序
│   ├── models/                 # 模型存放目录
│   │   ├── diffusion_models/   # Diffusion模型
│   │   ├── vae/               # VAE模型
│   │   ├── text_encoders/     # 文本编码器
│   │   ├── loras/             # LoRA模型
│   │   └── clip_vision/       # CLIP Vision模型
│   └── custom_nodes/          # 自定义节点（已安装）
├── workflows/                  # 预配置工作流
│   ├── text_to_video_wan22_5B.json
│   ├── image_to_video_wan22_5B.json
│   ├── video_wan2_2_14B_fun_control.json
│   └── wan2.1_fun_camera_1.3B.json
└── Readme.md                   # 本文档
```

## 参考链接

- [ComfyUI官方文档](https://docs.comfy.org/)
- [Wan 2.1/2.2 ComfyUI示例](https://comfyanonymous.github.io/ComfyUI_examples/wan/)
- [Wan模型Hugging Face页面](https://huggingface.co/Comfy-Org)

---

