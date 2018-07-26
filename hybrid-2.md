# Hybrid App技术解析 -- 实战篇

## 引言

上一篇原理篇，我们已经详细地阐述了 Hybrid App 的理论原理，了解了 Native端 和 H5端 是如何通信的，还有 bridge 的设计和接入。而本篇文章将开始把这些理论进行进一步的实践，真正地去实现一套完整且稳定的 Hybrid 方案。如果对原理还有疑问的小伙伴，请移步[Hybrid App技术解析 -- 原理篇](https://github.com/xd-tayde/blog/blob/master/hybrid-1.md)，只有在理解了理论的基础上，进一步与实践相结合，才能真正地去深入一项技术。

有什么建议或疑问，大家可以到 [github.com/xd-tayde](https://github.com/xd-tayde/blog/blob/master/hybrid-2.md) 上与我进行讨论哈！

## 摩天大楼展示

说了那么一大堆理论知识，可能有小伙伴会说：“ 你是不是吹流弊啊。”。😅。那先简单介绍下我们已经使用这套方案落地的项目吧。

<div align='center'>
	<img src="./images/hybrid/hybrid-5.jpg" width = "400" align=center /><br/>
</div>

这是一个完全内置在 App 里的线上 Hybrid 模块，由 Native 与 H5 深度合作完成，总共有4个页面，其中首页和制作页由 H5 制作，而相机页和保存页是复用 Native页面。

项目上线一年累积使用次数已经超过10亿次。这套方案经受住了考验，并在过程中逐步优化和拓展。

这套实现方案是基于以下几点考虑：

- 整个模块的风格是多变的，新推一套新妆容时，就经常需要搭配相应UI风格；
- 项目的流程逻辑的可变性很大，需要H5的热更新能力，及时应对数据的变化，快速的试错和纠正；
- 拍摄页与保存页是客户端已经有的模块，可以略微定制后直接复用；
- 需要由客户端协助接入算法SDK进行图像的复杂处理。

简单看完项目，我们接下来开始 bridge.js 的构建。由于本系列文章主要面向前端童鞋的，我们主要展开 H5 的部分，即会注入到每个页面头部的 bridge.js 的实现。

## 挖地基 --- Bridge.js 架构

基于上篇文章阐述的结构，我们进一步去完善细节部分，先整理成下面这样的流程结构图，nativeCall 与 postMessage 这两个主体 API 桥接了 Native端 和 H5端，大家先看下图，有个大致的概念：

<div align='center'>
	<img src="./images/hybrid/hybrid-6.png" width = "600" align=center /><br/>
</div>

接下来我们会细看里面各个部分的代码实现。

### (一) 业务方使用姿势

下面是在 JS 中获取网络状态的协议使用方法:

<div align='center'>
	<img src="./images/hybrid/hybrid-7.png" width = "600" align=center /><br/>
</div>

### (二) H5 --> Native

我们先来直接来看 nativeCall 的内部实现：

<div align='center'>
	<img src="./images/hybrid/hybrid-8.png" width = "600" align=center /><br/>
</div>

我们可以将里面分成这4个步骤:

1. 生成唯一 handler 标识，从 0 开始累加；
2. 将参数按 handler 值的规则存入参数池 _paramsStore 中；
3. 以 handler 注册自定义事件，绑定 callback，并将 callback也存入 _callbackStore 池中，`addEvent()`;
4. 以 iframe 的形式发送协议，并携带唯一标识 handler，`send()`；

<div align='center'>
	<img src="./images/hybrid/hybrid-9.png" width = "600" align=center /><br/>
</div>

Native:

- 客户端接收到请求后，会使用 handler 调用 getParams 从参数池中获取对应的参数；

<div align='center'>
	<img src="./images/hybrid/hybrid-10.png" width = "600" align=center /><br/>
</div>

- 执行协议对应的功能；

### (二) Native -> H5

Native：

- Native 调用 `Bridge.postMessage(handler, data)`；

<div align='center'>
	<img src="./images/hybrid/hybrid-11.png" width = "600" align=center /><br/>
</div>


H5：
- 通过唯一标识初始化自定义事件，挂载数据后触发，这里涉及的就是 `fireEvent` 这个函数:

<div align='center'>
	<img src="./images/hybrid/hybrid-12.png" width = "600" align=center /><br/>
</div>

我们现在已经根据整体的架构图梳理出了整个 bridge.js 的核心代码了，包含了：

- 最重要的开放API: `nativeCall` 与 `postMessage` ；
- 客户端获取参数函数: `getParam` ；
- 事件回调系统中的 `addEvent` 和 `fireEvent` ；
- 用于发送协议的 `send` 。

现在看下来，是不是觉得炒鸡简单？。分分钟能写100个。😂。没错！其实核心的原理就是这么的简单，但这只是一个最基础的地基而已，而基于地基之上，我们就可以开始一层一层建造我们的大楼了！

### 安卓兼容性:

如果看过上一篇原理篇的童鞋，这时可能会有个疑问：在 Android 4.4以下时，使用的 `loadUrl` 进行 js 函数的调用，而此时是无法获取函数的返回值的，也就是说4.4- 时，安卓并无法通过 `getParam` 这个函数来获取到协议的参数，因此这里客户端的童鞋可以用到一个曲线救国的骚操作，使用到的原理就是上一篇文章中有提到的一种 JS -> Native 的方案：

**WebView 中的 prompt 拦截**

方案如下:

- 当安卓接受到协议，并拿到 handler 值；
- 执行 js：`Bridge.getParam(handler)` ，直接将返回值通过 js 中的 `prompt` 发出；

<div align='center'>
	<img src="./images/hybrid/hybrid-13.png" width = "600" align=center /><br/>
</div>

- 通过重写 `onJsPrompt` 这个方法，拦截上一步发出的 prompt 的内容，并解析出相应的参数；

<div align='center'>
	<img src="./images/hybrid/hybrid-14.png" width = "600" align=center /><br/>
</div>

通过这样的方式，安卓全平台都可以完成参数的获取，并且方式统一，不需要做兼容处理，这就非常的skrskr啦。~~🤘🏻🤘🏻

## 建造大楼 --- 协议的定制

在完成最基础的架构后，我们就可以开始来进一步完成一些上层建筑了，制定一系列真正开放给业务方使用的协议 API。

首先我们可以将这些协议分成 功能协议 和 业务协议；

### 功能协议

这类协议是指 用于完善整套方案的基础功能 的一些通用协议，我们现在大概有这几条：

### 1.初始化机制

上篇文章我们有提到由于bridge.js注入的异步性，我们需要由客户端在注入完成后通知H5。

这里我们可以约定一个通用的初始化事件，这里我们约定为 `_init_`，因此前端就可以进行入口的监听, 类似于我们的 `DOMContentLoaded`:

<div align='center'>
	<img src="./images/hybrid/hybrid-16.png" width = "600" align=center /><br/>
</div>

我们通过约定，在该事件中，客户端会注入给H5很多的系统信息便于使用，以我们的🌰：




### 2.包更新机制

Hybrid模块的其中一种方式是将前端代码打包后内置于 App 本地，以便拥有最快的启动性能和离线访问能力。而这种方式最大的麻烦点，就是代码的更新，我们不可能每次有修改时就手动重新打包给客户端童鞋，而且这样也失去了我们的热更新机制。

因此这里就需要一套新的热更新机制，这套机制需要由客户端/前端/服务端 三端的童鞋提供对应的资源，共同协作完成整套流程。

资源：

- H5: 每个代码包都有一个唯一且递增的版本号；
- Native: 提供包下载且解压到对应目录的服务；
- 服务端: 提供一个接口，可以获取线上最新代码包的版本号和下载地址

流程：

- 前端代码包按版本号上传至指定的服务器上；
- 每次打开页面时，H5请求接口，获取线上最新代码包版本号，并与本地包进行版本号比对，当线上的版本号 大于 本地包版本号时，发起包下载协议：

<div align='center'>
	<img src="./images/hybrid/hybrid-15.png" width = "600" align=center /><br/>
</div>

- 客户端接受到协议后，直接去线上地址下载最新的代码包，并解压替换到当前目录文件；

拥有这样的机制后，H5在开发后，就可以直接打包将包上传到对应的服务器上，这样在 App 中打开页面后，即可以实时的热更新。


### 业务协议




## 大楼装修 --- 方案优化实践




