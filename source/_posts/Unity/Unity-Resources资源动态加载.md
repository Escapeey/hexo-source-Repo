---
title: Unity-Resources资源动态加载
tags: C#
category: Unity
abbrlink: d86321cd
date: 2023-10-31 09:27:38
---
## 特殊文件夹
### 工程路径获取
```C#
print(Application.dataPath);
```
- 注意 该方式 获取到的路径 一般情况下 只在 编辑模式下使用
- 我们不会在实际发布游戏后 还使用该路径
- 游戏发布过后 该路径就不存在了 

### Resources 资源文件夹
- 路径获取：
    - 一般不获取
    只能使用Resources相关API进行加载
    - 如果硬要获取 可以用工程路径拼接
    ```C#
    print(Application.dataPath + "/Resources");
    ```
- 注意：
    该文件夹需要我们自己创建
- 作用：资源文件夹
    - 需要通过Resources相关API动态加载的资源需要放在其中
    - 该文件夹下所有文件都会被打包出去
    - 打包时Unity会对其压缩加密
    - 该文件夹打包后<font color=red>只读</font> 只能通过Resources相关API加载

### StreamingAssets 流动资源文件夹
- 路径获取：
```c#
print(Application.streamingAssetsPath);
```
- 注意：
    需要我们自己将创建
- 作用：流文件夹
    - 打包出去不会被压缩加密，可以任由我们摆布
    - 移动平台<font color=red>只读</font>，PC平台<font color=red>可读可写</font>
    - 可以放入一些需要自定义动态加载的初始资源

### persistentDataPath 持久数据文件夹
- 路径获取：
```C#
print(Application.persistentDataPath);
```
- 注意：
    不需要我们自己将创建
- 作用：固定数据文件夹
    - 所有平台都<font color=red>可读可写</font>
    - 一般用于放置动态下载或者动态创建的文件，游戏中创建或者获取的文件都放在其中

### Plugins 插件文件夹
- 路径获取：一般不获取
- 注意：
    需要我们自己将创建
- 作用：插件文件夹
    - 不同平台的插件相关文件放在其中比如IOS和Android平台

### Editor 编辑器文件夹
- 路径获取：
    - 一般不获取
    - 如果硬要获取 可以用工程路径拼接
    ```C#
    print(Application.dataPath + "/Editor");
    ```
- 注意：需要我们自己将创建
- 作用：编辑器文件夹
    - 开发Unity编辑器时，编辑器相关脚本放在该文件夹中
    - 该文件夹中内容不会被打包出去

### 默认资源文件夹 Standard Assets
- 路劲过去：一般不获取
- 注意：需要我们自己将创建
- 作用：默认资源文件夹
    - 一般Unity自带资源都放在这个文件夹下
    - 代码和资源优先被编译

## Resources资源动态加载
### 作用
- 通过代码动态加载**Resources**文件夹下指定路径资源
- 避免繁琐的拖曳操作
### 常用资源类型
- 预设体对象——```GameObject```
- 音效文件——```AudioClip```
- 文本文件——```TextAsset```
- 图片文件——```Texture```
- 其它类型——需要什么用什么类型
- 注意：
    预设体对象加载需要实例化
    其它资源加载一般直接用

## Resources资源同步加载
### 同步加载(普通方法)
使用```Resources.Load()```方法加载预设体的资源文件<font color="red">(本质上 是加载 配置数据 到内存中)</font>
#### 预设体对象
- <font color="red">需要实例化</font>
```C#
Object obj = Resources.Load("Cube");
Instantiate(obj);
```

#### 音效资源
```C#
AudioSource audioS;
audioS.clip = Resource.Load("Music/BKMusic") as AudioClip;
audioS.Play();
```

#### 文本资源
支持的格式<font color='blue'>.txt, .xml, .bytes, .json, .html, .csv, ...</font>
```C#
TextAsset ta = Resouces.Load("Txt/Test") as TextAsset;
//文本内容
print(ta.text);
//字节数据组
print(ta.bytes);
```

#### 图片
```C#
Texture tex;
tex = Resources.Load("Tex/TestJPG") as Texture;
```

#### 问题
若资源同名时怎么办，因为同一个文件夹中不同格式的文件可以同名
- 方法一：加载指定类型的资源
```C#
tex = Resources.Load("Tex/TestJPG", typeof(TextAsset)) as TextAsset;
```
- 方法二：加载指定名字的所有资源
```C#
Object[] objs = Resources.LoadAll("Tex/TestJPG");
foreach (Object item in objs){
    if (item is Texture){
    }
    else if(item is TextAsset){
    }
}
```

### 同步加载(泛型方法)
```C#
TextAsset ta = Resources.Load<TextAsset>("Test");
Texture tex = Resouces.Load<Texture>("Test");
```

## Resources资源异步加载
### 什么是Resources异步加载
同步加载过大的资源可能会造成程序卡顿//
异步加载是内部新开一个线程进行资源加载 不会造成主线程卡顿
### Resources异步加载方法
**使用```Resources.LoadAsync()```方法**
异步加载 不能马上得到加载的资源 **至少要<font color='red'>等一帧</font>**
#### 完成事件监听异步加载
```C#
ResourceRequest rq = Resources.LoadAsync<Texture>("Tex/TestJPG");
//进行一个 资源下载结束 的一个事件函数监听(当资源加载结束后自动调用事件)
rq.completed += LoadOver;

private void LoadOver(AsyncOperation rq)
{
    print("加载结束");
    //asset 是资源对象 加载完毕过后 就能够得到它
    tex = (rq as ResourceRequest).asset as Texture;
}
```
<img src="..\img\Unity\AsyncOperation类.png" width="50%" height="50%">
<img src="..\img\Unity\ResourceRequest类.png" width="50%" height="50%">

#### 协程异步加载
##### 协程回顾
- 两大关键
    - 迭代器函数 (自己写)
    - 协程协调器 (Unity自带)
- **迭代器函数**当遇到```yield return```时就会停止执行之后的代码;
然后**协程协调器**通过得到**返回的值**去判断下一次执行后面的步骤**将会是何时**
##### 协程异步加载
1. 使用```yield return rq;```时Unity 会自己判断 该资源是否加载完毕了; 加载完毕过后 才会继续执行后面的代码.
```C#
StartCoroutine(Load1());

IEnumerator Load1()
{
    ResourceRequest rq = Resources.LoadAsync("Tex/TestJPG");

    yield return rq;
    tex = rq.asset as Texture;
}
```
2. 使用```rq.isDone```判断是否加载结束
```C#
StartCoroutine(Load2());

IEnumerator Load2()
{
    //加载两个资源
    ResourceRequest rq1 = Resources.LoadAsync("Tex/TestJPG1");
    ResourceRequest rq2 = Resources.LoadAsync("Tex/TestJPG2");
    //判断资源是否加载结束
    while(!rq1.isDone || !rq2.isDone)
    {
        //打印当前的 加载进度 
        //该进度 不会特别准确 过渡也不是特别明显
        print(rq1.progress);
        print(rq2.progress);
        yield return null;
    }
    tex1 = rq1.asset as Texture;
    tex2 = rq2.asset as Texture;
}
```
### 总结
#### 完成事件监听异步加载
- <font color='red'>我们在只加载一个资源时使用</font>
- 好处：写法简单
- 坏处：只能在资源加载结束后 进行处理
- “线性加载”

#### 协程异步加载
- <font color='red'>我们在加载多个资源时使用</font>
- 好处：可以在协程中处理复杂逻辑，比如**同时加载多个资源**，比如进度条更新
- 坏处：写法稍麻烦
- “并行加载”

#### 注意：
- 理解为什么异步加载不能马上加载结束，为什么至少要等1帧
- 理解协程异步加载的原理

## Resources资源卸载
### Resources重复加载资源会浪费内存吗？
- 在Resources加载一次资源过后该资源就一直存放在内存中作为缓存，第二次加载时发现缓存中存在该资源，会直接取出来进行使用，所以**多次重复加载不会浪费内存**
- 但是 **会浪费性能**(每次加载都会去查找取出，始终伴随一些性能消耗)
### 手动释放掉缓存中的资源
#### 卸载指定资源
```Resources.UnloadAsset```方法
注意：
- 该方法 不能释放 ```GameObject```对象 因为它会用于实例化对象
- 它只能用于一些 **不需要实例化的内容** 比如 图片 和 音效 文本等等
- **一般情况下 我们很少单独使用它**
``` C# 
Resources.UnloadAsset(tex);
```
#### 卸载未使用的资源
- 注意：**一般在过场景时和GC一起使用**
```C#
Resources.UnloadUnusedAssets();
GC.Collect();
```  