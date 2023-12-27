---
title: Unity-物理系统之刚体加力
date: 2023-12-27 21:39:19
tags: C#
category: Unity
---
## 刚体自带添加力的方法
给刚体加力的目标是 让其有一个**速度** 朝向**某一个方向移动**

### 获取刚体组件
```C#
Rigidbody rigidBody = this.GetComponent<Rigidbody>();
```

### 添加力
相对世界坐标 <font color="green">rigidBody.AddForce()</font>
```C#
rigidBody.AddForce(Vector3.forward * 10);
```

相对本地坐标 <font color="green">rigidBody.AddRelativeForce()</font>
```C#
rigidBody.AddRelativeForce(Vector3.forward * 10);
```

### 添加扭矩力
相对世界坐标 <font color="green">rigidBody.AddTorque()</font>
```C#
rigidBody.AddTorque(Vector3.up * 10);
```

相对本地坐标 <font color="green">rigidBody.AddRelativeTorque()</font>
```C#
rigidBody.AddRelativeTorque(Vector3.up * 10);
```

### 直接改变速度
<font color="green">rigidBody.velocity</font>

这个速度方向 是相对于 世界坐标系的 
```C# 
rigidBody.velocity = Vector3.forward * 5;
```

### 模拟爆炸效果
<font color="green">rigidBody.AddExplosionForce()</font>

- 模拟爆炸的力 一定是 **所有希望产生爆炸效果影响的对象都需要得到他们的刚体** 来执行这个方法 才能都有效果
```C#
rigidBody.AddExplosionForce(100, Vector3.zero, 10);
```

## 力的几种模式
- 第二个参数 力的模式 主要的作用 就是 计算方式不同 
- 由于4中计算方式的不同 最终的移动速度就会不同
```C#
rigidBody.AddForce(Vector3.forward * 10, ForceMode.Acceleration);
```

### Acceleration
- 给物体增加一个持续的加速度，忽略其质量
- v = Ft/m
- F:(0,0,10)
- t:0.02s
- m:默认为1
- v = 10*0.02/ 1 = 0.2m/s
- 每物理帧移动0.2m/s*0.02 = 0.004m

### Force
- 给物体添加一个持续的力，与物体的质量有关
- v = Ft/m
- F:(0,0,10)
- t:0.02s
- m:2kg
- v = 10*0.02/ 2 = 0.1m/s
- 每物理帧移动0.1m/s*0.02 = 0.002m

### Impulse
- 给物体添加一个瞬间的力，与物体的质量有关,忽略时间 默认为1
- v = Ft/m
- F:(0,0,10)
- t:默认为1
- m:2kg
- v = 10*1/ 2 = 5m/s
- 每物理帧移动5m/s*0.02 = 0.1m

### VelocityChange
- 给物体添加一个瞬时速度，忽略质量，忽略时间
- v = Ft/m
- F:(0,0,10)
- t:默认为1
- m:默认为1
- v = 10*1/ 1 = 10m/s
- 每物理帧移动10m/s*0.02 = 0.2m

## 立场脚本
```Constant Force```组件
添加恒定力

## 补充 刚体的休眠
Unity为了**节约性能**，会对一些最近没改变的刚体进行休眠，可能会影响后续操作
获取刚体是否处于休眠状态
```C#
if (rigidBody.IsSleeping())
{
    //就唤醒它
    rigidBody.WakeUp();
}
``` 
