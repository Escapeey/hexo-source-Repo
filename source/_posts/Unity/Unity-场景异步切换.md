---
title: Unity-场景异步切换
date: 2023-12-25 15:58:08
tags: C#
category: Unity
---
## 场景同步切换
```C#
SceneManger.LoadScene("TestScene");
```
场景同步切换的**缺点**：
- 因为先删除当前场景中所有对象，且去加载新场景的相关信息，会很耗时，造成卡顿

## 场景异步切换
和**资源异步加载**几乎一致，有两种方法
### 通过事件回调函数 异步加载
```C#
AsyncOperation ao = SceneManger.LoadSceneAsync("TestScene");
ao.completed += (a) =>
{
    print("加载结束");
}
```

### 通过协程 异步加载
- 需要注意的是 加载场景会把当前场景上**没有特别处理**的对象**都删除了**，所以**协程中的部分逻辑执行不了**
- 解决思路：
让处理场景加载的**脚本依附的对象** 过场景时**不被移除**
```C#
DontDestroyOnLoad(this.gameObject);
StartCoroutine(LoadScene("TestScene"));
IEnumerator LoadScene(string name)
{
    //第一步
    //异步加载场景
    AsyncOperation ao = SceneManager.LoadSceneAsync(name);
    yield return ao;
}
```
#### 使用协程的好处 
- 可以在异步**加载场景的同时**做一些别的逻辑，比如**更新进度条**
##### 方法一
```C#
//利用 场景异步加载的进度去更新 不是特别准确
IEnumerator LoadScene(string name)
{
    //第一步
    //异步加载场景
    AsyncOperation ao = SceneManager.LoadSceneAsync(name);
    while(!ao.isDone)
    {
        print(ao.progress);
        yield return null;
    }
}
//离开循环后 就会认为场景加载结束
//可以把进度条顶满 然后 隐藏进度条
```
##### 方法二
根据游戏规则**自定义**进度条变化的条件,例如：
- 场景加载结束 更新20%进度
- 动态加载怪物 再更新20%进度
- 动态加载场景模型 认为加载结束 进度条顶满 
- 隐藏进度条

## 总结
### 事件回调函数
- 优点：写法简单，逻辑清晰
- 缺点：只能**加载完场景**做一些事情 不能再加载过程中处理逻辑
### 协程异步加载
- 优点：可以在**加载过程中**处理逻辑，比如进度条更新等
- 缺点：写法较为麻烦，要通过协程