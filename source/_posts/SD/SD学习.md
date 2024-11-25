---
title: SD学习
tags: AI绘画
category: AI绘画
abbrlink: e5cb238c
cover: 'https://s2.loli.net/2024/11/25/KLGtlVmyhuOXzgv.png'
date: 2024-11-02 18:13:10
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


<img src="..\..\img\SD\进阶提示词语法.png" width="100%" height="100%" align="middle">

### 反向提示词
标准化提示词-抄作业
```
NSFW, (worst quality:2), (low quality:2), (normal quality:2), lowers, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, (ugly:1.331), (duplicate:1.331), (morbid:1.21), (mutilated:1.21), (tranny:1.331), mutated hands, (poorly drawn hands:1.5), blurry, (bad anatomy:1.21), (bad proportions:1.331), extra limbs, (disfigured:1.331), (missing arms:1.331), (extra legs:1.331), (fused fingers:1.61051), (too many fingers:1.61051), (unclear eyes:1.331), lowers, bad hands, missing fingers, extra digit, bad hands, missing fingers, (((extra arms and legs))),
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
- 存储地址: `根目录\models\Lora`
- 使用: 在提示词框里用 `<lora:lora模型名称>` 启用
- 一般要确定权重
- 类似彩页传单，比如要生成佩奇，它直接写明了佩奇是什么，有什么特点，怎么样呈现出佩奇
- 一般有一些触发词
## 导入方式
- 直接放入文件夹中
    - 存储地址: `根目录\models\Lora`
    - 使用`<lora:lora模型名称>` 启用
- 使用addtional network选项(额外附加网络 / 扩展模型)
    - 一个红色按钮
- 使用`Addtional Network`扩展
    - 在折叠选单`Addtional Networks`中添加
    - 存储地址: `根目录\extensions\sd-webui-addtional-networks\models\Lora`
    - 可以改为webui默认地址，将两个lora路径统一
    - 记得勾选启用
## 使用方法
### 人物角色形象
- 例子：制作赛博coser
    - 真实系大模型+lora
- 有类lora会提供某类特定人物形象展示
    - 特定方向上的角色美化
    - 可以当作调味剂，权重0.2-0.3
    - 例子：
        - Fashion girl 作者是使用很多个人审美的时尚女性制作的lora
        - Asian male 使用亚洲男性

### 画风或风格
- 替代Hypernetwork
- 如吉卜力画风lora
- 画风类lora比人物类lora影响更大，可以适当降低一点画风类lora的权重

## 概念图
- Gacha splash LORA (立绘lora)
- Anime Tarot Card Art Style LoRA (塔罗牌)
- zyd232's Stasis Pod/Chamber (密封舱)
- mugshot lora (档案照片)
- 一定要看作者的readme

## 服饰
- 权重一定不要太高，如果太高容易没有人只有衣服(这是由训练时只用衣服导致的)
- 绘制特殊、具体的服饰 (比如你就要某个角色穿的衣服)
- 如果你想要强调某一个特质，比如机甲，可以用多个lora叠加强调
- 机甲衣服
    - 机甲的英文(Mecha)
    - 推荐的checkpoint：
        - 二次元: Cetus-Mix
        - 真实系: Experience
    - 机甲的lora
    - 可以加一些提示词：cyberpunk、futuristic
    - 如果要机甲与人紧密结合：robotic arms/legs、mechanical parts(机械义体)
<img src="..\..\img\SD\机甲提示词.png" width="100%" height="100%" align="middle">

- 推荐：
    - hanfu 汉服
    - holographic clothing 镭射服装


## 物体/特定元素
- 产品设计
- 可以使用局部重绘的方法将这些物品引入到作品内
    - 先画全图(不开LoRA)、再画小的(重绘)(打开LoRA)
- [使用局部重绘引入头盔](https://www.bilibili.com/video/BV1nL411B7XT?spm_id_from=333.788) 22:10


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
<img src="..\..\img\SD\inPaint_Sketch.png" width="100%" height="100%" align="middle">

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

# ControlNet
## 基本原理
类似图生图
<img src="..\..\img\SD\ControlNet结构.png" width="70%" height="70%" align="middle">

## 安装和下载模型
- 需要下载模型
- 下载模型 `.pth` 和 对应的 `.yaml`
- 模型地址(yaml文件也要放进去): `根目录\extensions\sd-webui-controlnet\models`

## 基本使用方法
- 先在ControlNet栏中导入一张图片(目标姿势图片)
- 再选择预处理器(Annotator)
    - 从图片中提取姿势的叫`Openpose`
- 再选择和当前预处理器匹配的**控制模型**

- PS: 
    - 如果是图生图使用，则如果空着`ControlNet`栏的图片，则使用原图姿势
    - 如果导入的是骨骼图，则在预处理器中选**无**, 然后使用生成骨骼图时对应的模型
## 参数设置 (控制效果的强弱)
- 低显存模型 (如果cuda out of memory时用, 代价是速度慢一些)
- Pixel Perfect (推荐选中)
    - 自动计算预处理器产出图像的最合适分辨率, 避免因为尺寸不合导致的图像模糊变形
    - 不用手动设置预处理分辨率了
- Allow Preview 打开小的预处理器窗口
- 可以通过预处理器旁边的 **爆炸** 按钮来生成图片的**骨骼图**

- Control Weight 控制权重 (一般默认1)
- Starting Control Step 控制ControlNet什么时候(迭代过程)生效
- Ending Control Step 控制ControlNet什么时候(迭代过程)失效
- Control Mode 控制ControlNet和提示词哪个更重要 (一般选blance)
<img src="..\..\img\SD\ControlNet_ControlMode.png" width="70%" height="70%" align="middle">

- lookback 默认关闭
- 其余一些参数是不同预处理器对应的，一般维持默认即可
## ControlNet五大模型
### openpose
- openpose_face
- openpose_faceonly (适合大头照)
- openpose_hand 
- openpose_full (包含所有细节)

### depth
用来对场景的还原
<img src="..\..\img\SD\ControlNet_depth.png" width="70%" height="70%" align="middle">

- 生成具有空间感的图片

- 可以解决肢体交叉问题, 比如手在头前还是头后 (但人体形状会非常固定)
<img src="..\..\img\SD\ControlNet_Depth解决肢体交叠问题.png" width="70%" height="70%" align="middle">


### canny
- 使用边缘 精准的还原
- 如果输入图片是线稿, 用`inverse`的预处理器交换黑白
### softedge / HED
- 主要关注轮廓
<img src="..\..\img\SD\ControlNet_HED和Canny的区别.png" width="70%" height="70%" align="middle">

### scribble(涂鸦乱画)
- 可以用乌龟图片生成乌龟形太空飞船

## 多重ControlNet应用

- 在配置里`ControlNet`选单中, 通过`MultiControlNet(多重ControlNet)`来设置最大模型数量

- 经典的组合 OpenPose和depth
[组合解决手臂挡脸问题](https://www.bilibili.com/video/BV1Ds4y1e7ZB?spm_id_from=333.788) 25:15


