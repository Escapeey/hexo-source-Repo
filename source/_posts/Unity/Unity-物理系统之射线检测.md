---
title: Unity-物理系统之射线检测
tags: C#
category: Unity
abbrlink: 4d19935
date: 2023-12-26 16:34:42
---
## 什么是射线检测
解决以下问题：
- 鼠标选择场景上一物体
- FPS射击游戏（无弹道-不产生实际的子弹对象进行移动）
- 等需要判断一条线和物体的碰撞情况
它可以在**指定点**发射一个**指定方向**的射线
判断该射线与哪些**碰撞器**相交，得到对应对象

## 射线对象
单独的射线对于我们来说没有实际的意义,我们需要用它**结合物理系统**进行**射线碰撞判断**
### 3D世界中的射线

- 参数一：起点
- 参数二：方向
```C#
Ray r = new Ray(Vector3.right, Vector3.forward);
```
### 摄像机发射出的射线
**屏幕位置** 为起点
**摄像机视口方向** 为方向
```C#
Ray r2 = Camera.main.ScreenPointToRay(Input.mousePosition);
```
## 碰撞检测 API
射线检测也是**瞬时**的
执行代码时进行**一次射线检测**
### 最原始的射线检测
准备一条射线
```C#
Ray r = new Ray(Vector3.zero, Vector3.forward);
```
<font color='green'>Physics.Raycast()</font>

- 参数一：射线
- 参数二: 检测的最大距离 超出这个距离不检测
- 参数三：检测指定层级（不填检测所有层）
- 参数四：是否忽略触发器 ```UseGlobal```-使用全局设置 ```Collide```-检测触发器 
                       ```Ignore```-忽略触发器  不填使用```UseGlobal```
- 返回值：bool 当碰撞到对象时 返回 true 没有 返回false
```C#
if (Physics.Raycast(r, 1000, 1 << LayerMask.NameToLayer("Monster"), 
    QueryTriggerInteraction.UseGlobal))
{
    print("碰撞到了对象");
}
```
还有一种重载 不用传入 射线 直接传入起点 和 方向 也可以用于判断
```C#
if (Physics.Raycast(Vector3.zero, Vector3.forward, 1000, 
    1 << LayerMask.NameToLayer("Monster"),QueryTriggerInteraction.UseGlobal))
{
    print("碰撞到了对象");
}
```

### 获取相交的单个物体信息
物体信息类
通过<font color='green'>RaycastHit</font>类 我们得到得到 **碰撞到的对象信息**
还可以得到一些 碰撞的点 距离 法线等等的信息
```C#
RaycastHit hitInfo;
```
- 参数一：射线
- 参数二：```RaycastHit```是结构体 是**值类型** Unity会通过**out** 
         在函数内部处理后 得到碰撞数据后返回到该参数中
- 参数三：距离
- 参数四：检测指定层级
- 参数五：是否忽略触发器 
```C#
if( Physics.Raycast(r, out hitInfo, 1000, 1<<LayerMask.NameToLayer("Monster"), 
    QueryTriggerInteraction.UseGlobal) )
{
    //碰撞器信息
    print("碰撞到物体的名字" + hitInfo.collider.gameObject.name);
    //碰撞到的点
    print(hitInfo.point);
    //法线信息
    print(hitInfo.normal);
    //得到碰撞到对象的位置
    print(hitInfo.transform.position);
    //得到碰撞到对象 离自己的距离
    print(hitInfo.distance);
}
```
同理 还有一种重载 不用传入 射线 直接传入起点 和 方向 也可以用于判断

### 获取相交的多个物体
<font color='green'>Physics.RaycastAll()</font>

可以得到**碰撞到的多个对象**，如果没有 就是容量为0的数组
- 参数一：射线
- 参数二：距离
- 参数三：检测指定层级
- 参数四：是否忽略触发器
```C#
RaycastHit[] hits = Physics.RaycastAll(r, 1000, 1 << LayerMask.NameToLayer("Monster"),
                                       QueryTriggerInteraction.UseGlobal);
for (int i = 0; i < hits.Length; i++)
{
    print("碰到的所有物体 名字分别是" + hits[i].collider.gameObject.name);
}
```
还有一种NonAlloc型的函数 返回的碰撞的数量 通过out得到数据
<font color='green'>Physics.RaycastNonAlloc()</font>

```C#
RaycastHit[] hits;
if(Physics.RaycastNonAlloc(r, hits, 1000, 1 << LayerMask.NameToLayer("Monster"), 
                           QueryTriggerInteraction.UseGlobal) > 0 ){
}
```

### 使用时注意的问题
**注意**：
**距离**、**层级**两个参数 都是**int类型**
当我们传入参数时 一定要明确传入的参数代表的是距离还是层级
