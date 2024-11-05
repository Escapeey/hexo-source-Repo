---
title: SD相关推荐
date: 2024-11-04 16:02:38
tags: AI绘画
category: AI绘画
---
# 大模型
- TMND-Mix 二次元-细节惊人
# 扩展插件
## ADetailer (**S**)
- 自动识别面部等重点区域进行强调
- 模型目录: `根目录\models\adetailer`
- 提供针对脸部的提示词窗口
    - `detailed face, close-up, portrait`
- 识别不到脸, 就尽量下降检测阈值
- 能感知到大概明确偏向哪边就用下面的X、Y轴偏移来进行调整
- 支持单独对面部的 `ControlNet`(推荐使用inPaint模型)

## Tiled Diffusion
- 由`TiledDiffusion`和`TiledVAE`组成
- `TiledDiffusion`负责扩散生成图像
- `TiledVAE`负责编码与解码(将图像打入/捞出潜空间)

### 参数
- 方案: 用`MultiDiffusion`
- 重绘幅度: 低一点(建议0.3)
- 放大设置：
    <img src="..\..\img\SD\TiledDiffusion放大方案一.png" width="100%" height="100%" align="middle">
    <img src="..\..\img\SD\TiledDiffusion放大方案二.png" width="100%" height="100%" align="middle">

### Tiled VAE
- 是降低显存的关键
- 一般**维持默认参数不变**
- 只有在两种情况下改参数：
    - **爆显存**: 降低编码器分块大小
    - 当使用的Tile太小且图片变得**灰暗不清晰**时: 启用`快速编码器颜色修复`

### Tiled Diffusion
- 实现大尺寸下"体面"重绘并还原细节
<img src="..\..\img\SD\TiledDiffusion块设置.png" width="100%" height="100%" align="middle">

### 放大实践
[6K超高清分辨率放大实践](https://www.bilibili.com/video/BV1Su4y1d7Dp/?spm_id_from=333.788)

# embeddings

# LoRA
 
