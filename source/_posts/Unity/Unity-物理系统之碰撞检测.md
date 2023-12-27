---
title: Unity-物理系统之碰撞检测
tags: C#
category: Unity
abbrlink: 2b4a076a
date: 2023-12-10 16:35:16
---
**碰撞**和**触发**响应函数 属于 特殊的<font color="red">生命周期函数</font> 也是**通过反射调用**

## 知识点回顾
- 如何让两个游戏物体之间**产生碰撞** -- <font color="blue">至少1个刚体 和 两个碰撞器</font>
- 如何让两个物体之间**碰撞时表现出不同效果** -- <font color="blue">物理材质</font>
- **触发器**的作用是什么 -- <font color="blue">让两个物体碰撞没有物理效果，只进行碰撞处理</font>

## 物理碰撞检测响应函数

```Collision```类型的 参数 包含了 **碰到自己的对象**的相关信息
- 碰撞对象 的碰撞器 ```collision.collider```
- 碰撞对象 的依附对象 ```collision.gameObject```
- 碰撞对象 的依附对象的位置 ```collision.transform```
- 触碰点 个数 ```collision.contactCount```
- 接触点 具体坐标 ```ContactPoint[] pos = collision.contacts;```
- 只要得到了 碰撞到的对象的 任意一个信息 就可以得到它的所有信息

### 碰撞 接触时 自动执行
<font color="green">OncollisionEnter(Collision collision)</font>

```C#
private void OncollisionEnter(Collision collision)
{
    print(this.name + "被" + collision.gameObject.name + "撞到了");
}
```

### 碰撞 结束分离时 自动执行
<font color="green">OnCollisionExit(Collision collision)</font>

```C#
private void OnCollisionExit(Collision collision)
{
    print(this.name + "被" + collision.gameObject.name + "结束碰撞了");
}
```

### 碰撞 相互接触时 自动不停执行
<font color="green">OnCollisionStay(Collision collision)</font>

```C#
private void OnCollisionStay(Collision collision)
{
    print(this.name + "一直在和" + collision.gameObject.name + "接触");
}
```

## 触发器检测响应函数
### 触发 开始时 自动调用
<font color="green">OnTriggerEnter(Collider other)</font>

```C#
private void OnTriggerEnter(Collider other)
{
    print(this.name + "被" + other.gameObject.name + "触发了");
}
```

### 触发 结束时 自动调用
<font color="green">OnTriggerExit(Collider other)</font>

```C#
private void OnTriggerExit(Collider other)
{
    print(this.name + "被" + other.gameObject.name + "结束相融的状态了");
}
```

### 触发 相融时 自动不停调用
<font color="green">OnTriggerStay(Collider other)</font>

```C#
private void OnTriggerStay(Collider other)
{
    print(this.name + "和" + other.gameObject.name + "正在相融");
}
```

## 要明确什么时候会响应函数
- <font color="red">！！！</font>碰撞和触发都是**相互的**，不是说只有isTrigger为True的对象才能触发，和这个触发的对象也会被触发
- 只要挂载的对象 能和别的物体产生碰撞或者触发 那么对应的这6个函数 就能够被响应
- 6个函数根据需求来进行选择书写
- 如果是一个异形物体，刚体在父对象上，如果你想通过子对象上挂脚本检测碰撞是不行的 必须**挂载到这个刚体父对象上**才行
- 要明确 物理碰撞和触发器响应的区别

## 6个函数都可以写成虚函数
一般会把想要重写的 碰撞和触发函数 写成**保护类型**的 没有必要写成public 因为不会自己手动调用 都是Unity通过反射帮助我们自动调用的