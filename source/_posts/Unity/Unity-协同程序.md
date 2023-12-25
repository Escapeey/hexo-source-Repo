---
title: Unity-协同程序
tags: C#
category: Unity
katex: true
abbrlink: 2befdcf2
date: 2023-10-26 17:25:37
---
## Unity是否支持多线程？
- Unity是支持多线程的, 只是新开线程<font color="red">无法访问Unity相关对象的内容</font>
- 注意：Unity中的多线程要记住关闭，否则即使停止运行主线程新开线程也会继续运行 

## 协同程序是什么？
- 协同程序简称协程，它是“假”的多线程，它不是多线它的主要作用
- 将代码分时执行，不卡主线程
    - 简单理解，是把可能会让主线程卡顿的耗时的分步执
- 主要使用场景
    - 异步加载文件
    - 异步下载文件
    - 场景异步加载
    - 批量创建时防止卡顿

## 协同程序和线程的区别
- 新开一个线程是独立的一个管道，和主线程并行执行
- 新开一个协程是在原线程之上开启，进行逻辑分时分步执行

## 协程的使用
- 继承MonoBehavior的类 都可以开启 协程函数
### 第一步：申明协程函数
- 协程函数2个关键点
    - 1-1返回值为IEnumerator类型及其子类
    - 1-2函数中通过 yield return 返回值; 
```C#
//关键点一： 协同程序（协程）函数 返回值 必须是 IEnumerator或者继承它的类型 
IEnumerator MyCoroutine(int i, string str)
{
    print(i);
    //关键点二： 协程函数当中 必须使用 yield return 进行返回
    yield return null;
    print(str);
    yield return new WaitForSeconds(1f);
    print("2");
    yield return new WaitForFixedUpdate();
    print("3");
    //主要会用来截图时会使用，保证画面完整正常
    yield return new WaitForEndOfFrame();
    //可以写死循环，不会卡死主线程
    while(true)
    {
        print("5");
        yield return new WaitForSeconds(5f);
    }
}
```

### 第二步：开启协程函数
- 这样执行没有任何效果
```C#
MyCoroutine(1, "123");
```
- 常用开启方式
```C#
IEnumerator ie = MyCoroutine(1, "123");
StartCoroutine(ie);
```

### 第三步：关闭协程
- 关闭所有协程
```C#
StopAllCoroutines();
```
- 关闭指定协程
```C#
Coroutine c1 = StartCoroutine(MyCoroutine(1,"123"));
StopCoroutine(c1);
```

## yield return 不同内容的含义
### 下一帧执行
```C#
yield return 数字;
yield return null;
```
- 在Update和LateUpdate之间执行
### 等待指定秒后执行
```C#
yield return new WaitForSeconds(second);
```
- 在Update和LateUpdate之间执行
### 等待下一个固定物理帧更新时执行
```C#
yield return new WaitForFixedUpdate();
```
- 在FixedUpdate和碰撞检测相关函数之后执行
### 等待摄像机和GUI渲染完成后执行
```C#
yield return new WaitForEndOfFrame();
```
- 在LateUpdate之后的渲染相关处理完毕后之执行
### 一些特殊类型的对象 比如异步加载相关函对象
- 之后讲解：异步加载资源、异步加载场景、网络加载时讲解
- 一般在Update和LateUpdate之间执
### 跳出协程
```C#
yield break;
```

## 协程受对象和组件失活销毁的影响
- 协程开启后
    - 组件和物体销毁，协程<font color="red">不执行</font>
    - 物体失活协程<font color="red">不执行</font>，组件失活协程<font color="red">执行</font>
- 与延迟函数相比 延迟函数：
    - 组件和物体销毁，延迟函数<font color="red">不执行</font>
    - 物体失活延迟函数<font color="red">执行</font>，组件失活延迟函数<font color="red">执行</font>

## 协程的本质
- 协程可以分成两部分
    - 协程函数本体
        协程本体就是一个能够中间暂停返回的函数,本质上就是一个 C#的迭代器方法
    - 协程调度器
        协程调度器是Unity内部实现的，会在对应的时机帮助我们继续执行协程函数
- Unity只实现了协程调度部分

### 理解
- C#看到迭代器函数和yield return 语法糖就会把原本是一个的 函数变成"几部分"
我们可以通过迭代器  从上到下遍历这 "几部分"进行执行  就达到了将一个函数中的逻辑分时执行的目的

- 而协程调度器就是 利用迭代器函数返回的内容来进行之后的处理
比如Unity中的协程调度器 根据yield return 返回的内容 决定了下一次在何时继续执行迭代器函数中的"下一部分"

- 理论上来说 我们可以利用迭代器函数的特点 自己实现协程调度器来取代Unity自带的调度器


## 例子：使用协程避免批量加载物体时卡顿
问题：请在场景中创建100000各随机位置的立方体，让其不会明显卡顿
分析：如果在同一帧中同时创建100000个物体，会超出1帧的时间，让玩家感到明显卡顿；  
此时可以使用协程分不同帧分批次创建，例如每帧创建1000个，可减少卡顿。
```C#
IEnumerator CreateCube(int num)
{
    for(int i=0; i<num; i++)
    {
        GameObject obj = GameObject.CreatePrimitive(Primitive.Cube);
        obj.transform.position = new Vector3(Random.Range(-100, 100),Random.Range(-100, 100),Random.Range(-100, 100));
        if(i % 1000 == 0)
            yield return null;
    }
}
```

## 总结
1. Unity支持多线程，只是新开线程无法访问主线程中Unity相关内容
    - 一般主要用于进行复杂逻辑运算或者网络消息接收等等
    - 注意：Unity中的多线程一定记住关闭
2. 协同程序不是多线程，它是将线程中逻辑进行分时执行，避免卡顿
3. 继承MonoBehavior的类都可以使用协程
4. 开启协程方法、关闭协程方法
5. yield return 返回的内容对于我们的意义
6. 协程只有当组件单独失活时不受影响，其它情况协程会停止