---
title: Unity-LineRenderer组件
tags: C#
category: Unity
abbrlink: af0e4a0a
date: 2023-12-25 21:48:47
---
## LineRenderer是什么
```LineRenderer```是Unity提供的一个**用于画线的组件**
使用它我们可以在场景中绘制线段,一般可以用于
- 绘制攻击范围
- 武器红外线
- 辅助功能
- 其它画线功能

## LineRender参数相关
- ```Corner Vertices```(角顶点，圆角)
此属性指示在一条线中绘制角时使用了多少个额外的顶点
增加此值，使**线角看起来更圆**
- ```End Cap Vertices```(终端顶点，圆角)
终点圆角
- 其余重点参数在代码部分介绍
## LineRender代码相关
- 动态添加一个线段
```C#
GameObject line = new GameObject();
LineRenderer lineR = line.AddComponent<LineRenderer>;
```
- 首尾相连
```C#
lineR.loop = true;
```
- 开始结束宽
```C#
lineR.startWidth = 0.02f;
lineR.endWidth = 0.02f;
```
- 开始结束颜色
```C#
lineR.startColor = Color.white;
lineR.endColor = Color.red;
```
- 设置材质
```C#
m = Resources.Load<Material>("M");
lineR.material = m;
```
- 设置点
一定注意设置点要**先设置点的个数**
接着就设置 对应每个点的位置
当设置的位置数少于先前设置点的个数时，剩余的点坐标都默认为原点
```C#
lineR.positionCount = 4;
lineR.SetPositions(new Vector3[] { new Vector3(0,0,0),
                                   new Vector3(0,0,5),
                                   new Vector3(5,0,5)});
lineR.SetPosition(3, new Vector3(5, 0, 0));
```
- 是否使用世界坐标系
决定了 是否随对象移动而移动
```C#
lineR.useWorldSpace = false;
```
- 让线段受光影响 会接受光数据 进行着色器计算
```C#
lineR.generateLightingData = true;
```

## 例子
### 练习一
**请写一个方法，传入一个中心点，传入一个半径，用LineRender画一个圆出来**
```C#
public void DrawLineRenderer(Vector3 centerPos, float r, int pointNum)
{
    //动态创建 画线对象
    GameObject obj = new GameObject();
    obj.name = "R";
    LineRenderer line = obj.AddComponent<LineRenderer>();
    //设置有多少个点
    line.positionCount = pointNum;
    //让其首尾相连
    line.loop = true;
    //得到每个点之间 间隔的度数
    float angle = 360f / pointNum;
    //准备得到每一个点
    for (int i = 0; i < pointNum; i++)
    {
        //知识点
        //1.点加向量 相当于平移点
        //2.四元数 * 向量 相当于在 旋转向量
        line.SetPosition(i, centerPos + Quaternion.AngleAxis(angle * i, Vector3.up) * Vector3.forward * r);
    }
}
```
### 练习二
**在Game窗口长按鼠标用LineRenderer画出鼠标移动的轨迹**
- 重点是 如何得到鼠标转世界坐标的 对应点 
- 知识点
    - 得到鼠标位置 ```Input.mousePosition```
    - 把鼠标 转世界坐标 ```Camera.main.ScreenToWorldPoint(Input.mousePosition);```
```C#
private LineRenderer line2;
void Start()
{
    line2 = this.gameObject.AddComponent<LineRenderer>();
    line2.loop = false;
    line2.startWidth = 0.5f;
    line2.endWidth = 0.5f;
    line2.positionCount = 0;
}
private Vector3 nowPos;
private void Update()
{
    if( Input.GetMouseButton(0) )
    {
        line2.positionCount += 1;
        nowPos = Input.mousePosition;
        //设置z轴为横截面深度
        nowPos.z = 10;
        line2.SetPosition(line2.positionCount - 1, Camera.main.ScreenToWorldPoint(nowPos));
    }
}
```