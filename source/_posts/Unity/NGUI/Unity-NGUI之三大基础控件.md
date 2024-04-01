---
title: Unity-NGUI之基础及组合控件
date: 2024-01-05 19:49:55
tags: C#
category: Unity
---

<img src="../../../img/Unity/NGUI_Root_缩放模式设置.bmp" width="70%" height="50%">

# 基础控件
## Sprite
### Sprite作用
NGUI中所有中小尺寸图片显示都用Sprite显示
使用它来显示图集中的单个图片资源

### 创建Sprite
- Scene窗口红框内右键 
- 上方工具栏**NGUI**中创建
### Sprite参数
略
### 代码设置图片
```C#
//1.改变为当前图集中选择的图片
sprite.spriteName = "bk";

//2.改变为其它图集中的图片
//先加载图集
NGUIAtlas atlas = Resources.Load<NGUIAtlas>("Atlas/login");
sprite.atlas = atlas;
//再设置图片
sprite.spriteName = "ui_DL_anniuxiao_01";
```

## Label
- 文本(支持富文本)
## Texture
- 一般图片，不能放到atlas 中的图片

# 组合控件
## 