---
title: SD学习
date: 2024-11-02 18:13:10
tags:
---
**参考课程**: [SD绘画](https://space.bilibili.com/1814756990/channel/collectiondetail?sid=1285674)
**提示词+参数+模型**
# 提示词
- tips: **多多益善**
- 提示词工具网站
[提示词工具箱](http://www.atoolbox.net/Tool.php?Id=1101)
- 提示词格式
    - 词组为单位
    - 分隔符: ", " 
## 内容提示词
### 一些提示词
- xxx in the background 可以更精确的将东西定义到"背景"的范围里
- depth of field 景深 
### 人物提示词
- 服装穿搭
- 发型发色
- 五官特点
- 面部表情
- 肢体动作
### 场景提示词
- 室内、室外(**重要**)
    - indoor / outdoor
- 大场景
    - forest, city, street
- 小细节
    -  tree, bush, white flower
### 环境光照
- 白天黑夜
    - day / night
- 特定时段
    - morning, sunset
- 光环境
    - sunlight, bright, dark
- 天空
    - blue sky, starry sky
### 画幅视角
- 距离
    - close-up, distant
- 人物比例
    - full body, upper body
- 观察视角
    - from above, view of back
- 镜头类型
    - wide angle, Sony A7 3
## 标准化提示词
### 画质提示词
- 通用高画质
    - best quality, ultra-detailed, masterpiece, hires, 8k
- 特定高分辨率类型
    - extremely detailed CG unity 8k wallpaper, unreal engine rendered
### 画风提示词
- 插画风
    - illustration, painting, paintbrush
- 二次元
    - anime, comic, game CG
- 写实系
    - photorealistic, realistic, photograph

## 提示词权重
- tips:
    - 尽量保持权重在**1上下0.5内**
    - 如果想让个别词条很突出，则建议使用多个词条，避免单一词条权重过大
- 加括号
    - 圆括号 **权重*1.1**
        - (white flower)
    - 大括号 **权重*1.05**
        - {white flower}
    - 方括号 **权重*0.9**
        - [white flower]
- 括号加数字权重
    - (white flower:1.5)


<img src="..\img\SD\进阶提示词语法.png" width="100%" height="100%" align="middle">

### 反向提示词
标准化提示词-抄作业
```
NSFW, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, (ugly:1.331), (duplicate:1.331), (morbid:1.21), (mutilated:1.21), (tranny:1.331), mutated hands, (poorly drawn hands:1.5), blurry, (bad anatomy:1.21), (bad proportions:1.331), extra limbs, (disfigured:1.331), (missing arms:1.331), (extra legs:1.331), (fused fingers:1.61051), (too many fingers:1.61051), (unclear eyes:1.331), lowers, bad hands, missing fingers, extra digit, bad hands, missing fingers, (((extra arms and legs))),
```

# 出图参数
- 采样迭代步数(step)
    - 一般设为20
- 采样方法
    - 用最下面带有加号的
    - 模型一般有提示最适合自己的算法
- 分辨率
    - 过大显存不够
    - 过大容易出现多人、多手(一般模型训练时用的小图片, 分辨率过大会让AI认为是多个图片拼接)
        - **避免该问题**：使用低分辨率先绘制，然后用高清修复放大
- 面部修复：勾选上(类似美图P脸)
- 平铺：不要勾选
- 提示词相关性：类型权重
    - 7-12 较安全
- 生成批次：建议多批次一起
- 每批数量：1

# 图生图
原理：提取图像特征，并输入到网络中
- tips: 
    - 一般生成的图片建议和原始图片保持相同的缩放比例
    - 如果要不同的比例，建议用修图软件裁剪后再导入 
## 参数
- 重绘幅度
- 缩放模式
    - 直接缩放(放大潜变量)：不建议使用

# 模型
## 概述
- 存储地址: `根目录\models\Stable-diffusion\`
- 模型类型
    - `.ckpt`
    - `.safetensor`
        - 占用空间小
## 风格分类
### 二次元
- 搜索标签：illustration, painting, sketch, drawing, painting, comic, anime, catoon
- 推荐:
    - Anything V5
    - Counterfeit V2.5
        - 精致，类似紫罗兰永恒花园
    - Dreamlike Diffusion
        - 幻想系，超现实
### 真实系
- 搜索标签：photography, photo, realistic, photorealistic, RAW photo
- 推荐:
    - Deliberate
    - Realistic Vision
    - L.O.F.I
    
### 2.5D
- 搜索标签：3D, render, chibi, digital art, concept art, {realistic}
- 推荐:
    - Never Ending Dream (NED)
    - Protogen (Realistic)
    - 国风3 (GuoFeng3)
## VAE
- 可以理解为AI绘画的一种**调色滤镜**
- 大多数模型都将其融入大模型内了
## Hypernetwork
- 存储地址: `根目录\models\hypernetworks\`
- 使用: 在设置里找到 **附加网络** 选项，将对应超网络添加到提示词
- 类似lora, 一般用于改变画风, 小区别(类似梵高和莫奈)
## Embeddings
- 存储地址: `根目录\embeddings\`
- **WebUI中embeddings不用特别调用，只需要用特定提示词去召唤**
- 后缀 **.pt**
- 通常很小
- 下载模型时标签为：**textual inversion**
- 不包含信息, 只是一个标记, 包含特征信息去哪里找, 类似书签
- 例子：
    - 比如Dva的embeddings 样例中用 `corneo_dva` 召唤
    - 可以用特定的embeddings画角色图
    - 解决画错手的问题(它是在一系列画错的上面学到的，放在负面提示词)
        - Deep Negative (真人模型)
        - Easy Negative (二次元)
## LoRA
- 存储地址: `根目录\models\Lora\`
- 使用: 在提示词框里用 `<lora:lora模型名称>` 启用
- 一般要确定权重
- 类似彩页传单，比如要生成佩奇，它直接写明了佩奇是什么，有什么特点，怎么样呈现出佩奇

# 高清修复、细节优化、无损放大
## 高清修复(文生图)
- **打回重画，再来一幅**
- 别名：高分辨率修复、超分辨率(超分)
- **tips**:
    - 先在低分辨率下反复抽卡，抽到合适的后再固定种子，进行高清修复
### 参数
- 放大倍率
- 高清修复采样次数(高清修复需要在生成的图片基础上进行重绘)
    - 保持默认0, 表示沿用原始迭代次数
- 重绘幅度：类似图生图(0.3-0.5)
- 重绘算法：
    - 无脑选  R-ESRGAN 4x+
    - 二次元选R-ESRGAN 4x+ Anime6B
    - 可以用作者推荐的
## SD放大
upscale放大脚本
- 为上采样过程
- 将图片分成几块分别重绘，并拼在一起, 使用重叠的像素作为缓冲带
- 重叠的像素: 为缓冲带
- 开启SD放大后, **最终宽高 = (设置的宽高-重叠像素) x 放大倍率**

## 附加功能放大(无损放大)
- 重绘幅度为0的高清修复
- **不涉及再扩散**
- 类似老照片修复

# 局部重绘
- 使用蒙版
## 参数：
- 蒙版模式:
    - 重绘涂黑的 / 重绘未涂黑的
- 蒙版蒙住的内容：
    - 一般用原图
- 重绘区域: 
    - 全图: 重绘全图，然后把蒙版部分拼回去(一般用这个)
    - 仅蒙板: 只重绘蒙版部分(针对性比较强时用这个)
- 蒙版模糊:
    - 类似羽化
    - 使重绘区域拼接回去时更加自然(保持10以下)
## inPaint Sketch(绘制/局部重绘(手涂蒙版))
- 这里用画笔涂的就不是蒙版选区了，是真正的图形内容
<img src="..\img\SD\inPaint_Sketch.png" width="100%" height="100%" align="middle">

- 蒙版透明度: 绘制的颜色印在画面上的显著程度(一般维持默认0,完全不透明)
## inPaint Upload(上传蒙版)
[使用PS制作蒙版并上传](https://www.bilibili.com/video/BV1uL411e7Uk/?spm_id_from=333.788) 12:34

# 扩展插件
- 图库浏览器 image browser
- LLC (Local Latent Couple)
    - 将部分区域变得更加精致
    - LLUL权重
- CutOff 
    - 解决提示词间的互相干预