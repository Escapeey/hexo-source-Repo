---
title: Unity-物理系统之范围检测
tags: C#
category: Unity
abbrlink: 43ff44cd
date: 2023-12-26 10:41:26
---
## 什么是范围检测
游戏中**瞬时**的攻击范围判断一般会使用范围检测
举例：
- 玩家在前方5m处释放一个地刺魔法，在此处范围内的对象将受到地刺伤害
- 玩家攻击，在前方1米圆形范围内对象都受到害

类似这种并**没有实体物体** 只想要检测在指定**某一范围**是否让敌方受到伤害时 便可以使用范围判断

## 范围检测
**必备条件**：想要**被**范围检测到的对象必须**具备碰撞器**
注意点：
- 范围检测相关API **只有当执行该句代码时** 进行一次范围检测 它是<font color='red'>瞬时</font>的
- 范围检测相关API 并<font color='red'>不会真正产生一个碰撞器</font> 只是**碰撞判断计算**而已

### 盒状范围检测API
<font color='green'>Physics.OverlapBox()</font>

- 参数一：立方体中心点
- 参数二：立方体三边大小
- 参数三：立方体角度
- 参数四：检测指定层级（不填检测所有层）
- 参数五：是否忽略触发器 ```UseGlobal```-使用全局设置 ```Collide```-检测触发器 ```Ignore```-忽略触发器 不填使用```UseGlobal```
- 返回值：在该范围内的碰撞器（得到了对象碰撞器就可以得到对象的所有信息）
```C#
Collider[] colliders = Physics.OverlapBox(Vector3.zero, Vector3.one, 
                    Quaternion.AngleAxis(45, Vector.Up), 
                    1 << LayerMask.NameToLayer("UI") | 
                    1 << LayerMask.NameToLayer("Default"), 
                    QueryTriggerInteraction.UseGlobal);
for (int i = 0; i < colliders.Length; i++)
{
    print(colliders[i].gameObject.name);
}
```
另一个API <font color='green'>Physics.OverlapBoxNonAlloc()</font>
- 返回值：碰撞到的碰撞器**数量**
- 参数：**传入一个数组**进行存储
```C#
if(Physics.OverlapBoxNonAlloc(Vector3.zero, Vector3.one, colliders) != 0){
}
```

### 球形范围检测API
<font color='green'>Physics.OverlapSphere()</font>

- 参数一：中心点
- 参数二：球半径
- 参数三：检测指定层级（不填检测所有层）
- 参数四：是否忽略触发器 
- 返回值：在该范围内的触发器
```C#
colliders = Physics.OverlapSphere(Vector3.zero, 5, 
            1 << LayerMask.NameToLayer("Default"));
```
另一个API <font color='green'>Physics.OverlapSphereNonAlloc()</font>
- 返回值：碰撞到的碰撞器**数量**
- 参数：传入一个**数组**进行存储
```C#
if( Physics.OverlapSphereNonAlloc(Vector3.zero, 5, colliders) != 0 ){
}
```
### 胶囊范围检测API
<font color='green'>Physics.OverlapCapsule()</font>

- 参数一：半圆一中心点
- 参数二：半圆二中心点
- 参数三：半圆半径
- 参数四：检测指定层级（不填检测所有层）
- 参数五：是否忽略触发器
- 返回值：在该范围内的触发器
```C#
colliders = Physics.OverlapCapsule(Vector3.zero, Vector3.up, 1, 
            1 << LayerMask.NameToLayer("UI"), 
            QueryTriggerInteraction.UseGlobal);
```
另一个API <font color='green'>Physics.OverlapCapsuleNonAlloc()</font>

- 返回值：碰撞到的碰撞器数量
- 参数：传入一个数组进行存储
```C#
if ( Physics.OverlapCapsuleNonAlloc(Vector3.zero, Vector3.up, 1, colliders ) != 0 ){
}
```
## 总结
- 范围检测主要用于**瞬时**的**碰撞范围**检测
- 主要掌握 **Physics类中的静态方法：球形 盒装 胶囊三种API的使用**即可
