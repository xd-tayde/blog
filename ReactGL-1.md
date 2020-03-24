# React 实践揭秘之旅，中高级前端必备(上)

> 感谢大家又被我的标题党骗进来了😂。这是我最近几个月亲身探寻过的一趟旅途，感受颇深。途中也遇到许多困难，但坚持到底，相信最终一定会让各位小伙伴受益匪浅，不虚此行！
> 
> 同时也感恩大家对之前一个系列文章的喜欢和修正😘，提了不少的建议和问题，我也会用心写我的每一系列文章，与大家共同成长！！
>
> 跪求点赞、关注、Star！[更多文章猛戳 ->](https://github.com/xd-tayde/blog)
>
> 下篇已经新鲜出炉: **[React 实践揭秘之旅，中高级前端必备(下) ->](https://github.com/xd-tayde/blog/blob/master/ReactGL-2.md)**

## 引言

之前面试三部曲简明地梳理了前端知识结构体系，浅尝辄止。这个系列则要进一步研究和领会 **内在的奥妙**。今天打算以一个比较新颖的角度切入，深入地梳理下 `React` 的内部实现。

- **1. 有利于大家在 React 日常业务使用中更加得心应手**；

- **2. 也可将领会到的思想融会贯通，拓展到其它领域**；

那如何深入研究一个玩具呢？最好的方式便是: **拆解 - 重组装**。

众所周知，`React` 是一款非常神奇的 `Web UI` 框架，得益于强大的架构，**逻辑层** 与 **视图层** 的解耦，使其思想及开发模式可以很好地移植到其它平台，例如 `React-Native`。所以今天，我打算跳出常规的 `Web-DOM`，目标是 **以类 React 的模式来进行 Web 游戏开发**。其实就是我们需要实现一套 `React` 上层，并把底层对接 `WebGL API`。期待这样不同的视角，能给大家带来新的启发与帮助。😄

> Tips:
> 
> `WebGL` 不是本文关注的重点，使用到的 API 也非常有限，并不需要你具备相关的知识储备。

## 第一站: 穿越之门 - JSX

作为一名前端，我们需要实现一个个展示给用户的页面。因此视图层便是我们的工作之始。前端领域不断地快速发展，最终都是为了解答 **如何更高效地开发更完美的页面**。而 `React` 就是答案之一，其中 `JSX` 便是重要的第一站。

那什么是 `JSX` 呢？

**`JSX` 就是在 `JS` 环境中约定的一种类 `HTML` 或 `XML` 的 动态模板语法，有着极高的 可读性 与 可拓展性，目的是 为了能更便捷地使用 JS 搭建视图结构与布局。**

```js
// 这就是 JSX
const jsx = <JSX>Hello World</JSX>
```

这是一种新全新的 `JS` 语法，不属于标准，成功地把类 `HTML` 的标签型模板语法引入 JS 中，创造了一种全新高效的开发模式。但即使最新版的 V8 引擎也无法支持，那怎么执行呢？钥匙就是 **预编译**！得益于 `Babel` 的强大，我们可以通过 **预编译，将代码编译成浏览器看得懂的 `JS`**。

### babel-plugin-transform-jsx

这是 `Babel` 的一款插件，主要的功能就是编译 `JSX`，直接配置即可 (对编译感兴趣的童鞋，可以继续深入下了解下 `Babel` 的编译原理，这里就不作展开了)。

```js
// 打包工具中的 babel loader 配置
use: {
    loader: 'babel-loader',
    options: {
        plugins: [["transform-jsx", { 
            "function": "ReactWebGL.createElement",
            "useVariables": true
        }]],
    }
}
```

配置完毕，我们先来写段 `JSX` 试试。由于我们不使用 `DOM` 了，视图层被直接绘制于 `canvas` 上，就自然不能使用常规的 `HTML` 标签了。我们先来定个容器标签 (`<Container>`)。

```js
const jsx = (
    <Container name="parent">
        ReactWebGL Hello World
        <Container name="child">
            Child
        </Container>
    </Container>
)
```

咦。秒报错。先不管，我们来看下编译后的文件:

```js
var jsx = ReactGL.createElement({
    elementName: Container,
    attributes: {
        name: 'parent'
    },
    children: [
        "ReactWebGL Hello World", 
        ReactWebGL.createElement({
            elementName: Container,
            attributes: {
                name: "child"
            },
            children: ["Child"]
        }),
    ]
});
```

原来如此，其实 `Babel` 做的，就是把上面的 `JSX` 模板代码, 解析并提取出标签的信息后，转换成常规的函数形式。这个函数就是我们配置中指定的 `ReactWebGL.createElement`。

接下来我们来看下报错。很明显，是因为在上下文中有一些变量没有进行定义，那接下来我们先定义上:

```js
// 由于编译是直接 变量传递
// 因此标签名作为一个变量需要先定义为 string；
const Container = 'Container'

// 匹配配置中的 `ReactWebGL.createElement`
const ReactWebGL = {
    createElement(tag) {
        console.log(tag)
    }
}
```

## 第二站: 天空之城 - Virtual DOM

来到第二站: 什么是 **虚拟DDM** 呢？

顾名思义，**它并不是真正的视图元素，而是将真实的视图元素抽象为一个 Javascript 对象，包含完整描述了一个真实元素的所有信息，但并不具备渲染功能**。同时由于 `DOM` 本身便是 **树形结构**，因此使用 `Javascript` 对象便能很好对整个页面结构进行描述。

刚才 `JSX` 编译后，参数便是一个最简单的 **虚拟DOM** 对象(我们称为 `VNode`):

```js
// 一个最简单的 VNode
{
    // 类型
    type: 'Container'
    // 属性
    props: { 
        name: 'parent'
    }
    // 子级列表
    children: [...]
}
```

这其实只是一个普通的 `Javascript` 对象，那为什么要设计它呢？可能很多人都会有这种观念: **虚拟DOM 快啊，diff 算法非常厉害，直接操作真实的 DOM 消耗很大。**

掐指一算，事情并没有那么简单，听我细细道来🧐。想象下，当出现以下场景时:

- **初次渲染**: 

	- 解析 `JSX`，生成 **虚拟DOM 树**，然后经过各种计算，最终调用 `DOM` 绘制一个个视图元素；
	
	- 很明显，我们通过 `HTML` 或 `innerHTML` 直接创建元素会更快，且白屏时间更短，多了上层的 **计算消耗与内存消耗**，反而是一种 **性能损耗**；
	
- **极小更新**: 

	- 需要修改一个标题文案，调用 `setState` 触发更新。此时，`React` 并不知道新的 `state` 会引起多大的变动，需要经过全局逐一的 `diff` 确定变动的元素再触发更新。
	
	- 相较而言，获取对应标题元素修改，省去 `diff`，会更直接高效。那这么说来，**虚拟DOM** 反而变慢了，那为什么还要用它呢？

如果你的场景是 大量且分散 地更新页面中的元素，那 **虚拟DOM** 能大大减少其业务逻辑的复杂度，达到一个比较高的消耗性价比。直接操作 `DOM` 元素，一来逻辑复杂，代码健壮性弱；二来错误的操作反而可能导致更差的性能。

所以快慢并不是衡量 **虚拟DOM** 价值的因素，其价值更多在于:

- **虚拟DOM** 其实是一种 **牺牲最小性能与空间，换取 架构优化 的方式**，能较大提升项目的可拓展性与可维护性；

- 对 `DOM` 的操作进行 **集中化管理**，更加安全稳定且高效；

- **渲染** 与 **逻辑** 的解耦，高度的 **组件化** 与 **模块化**，提升了代码复用性及开发效率；

- **虚拟DOM** 是一种抽象化的对象，可以对接不同的渲染层，完成 **跨平台渲染**。例如 `RN / SSR` 等方案；

`JSX` 仅仅是一种 **独立的动态模板语法**，通过编译转化为 **虚拟DOM**，跟 `React` 并无耦合。因此可以将其 **接入到任意环境或者框架**，例如我们也可以在 `Vue` 中使用 `JSX`。

聊完 **虚拟DOM**，我们回归主线。需要先来设计一个最简单的 `VNode` 结构，以刚才 `Babel` 编译出的结构为基础，加上额外的值，方便渲染。

```js
// VNode 定义
interface VNode {
    // 标签类型
    type: any,
    // 标签属性
    props: { [key: string]: any },
    // 子级列表
    children: VNode[],
    // 唯一标识
    key: string
    // 获取视图元素
    ref: any
    // 视图元素
    elm: any
    // 文本内容
    text: string | number | undefined
}

// VNode 生产函数
function createVNode(type, props, ref, key, children, elm, text) {
    return {
        type, 
        props, 
        children,
        ref, 
        key,
        elm,
        text,
    }
}
```

现在定义了 `VNode` 后，我们就可以开始编写对应的生成函数(`createElement`)了。

## 第三站: 转换之桥 - createElement

这个函数就是传说中的 `h` 函数，用于 **模板 到 虚拟DOM 之间的桥梁**。在现在的大多数主流 **虚拟DOM** 库中，都拥有该函数。在 `React` 中，它是通过 `Babel` 将 `JSX` 编译成 `h` 函数，即 `React.createElement`。而在 `Vue` 中，则是通过 `vue-loader` 将 `<template>` 编译成其 `h` 函数。该函数主要功能是: 

**加工生成完整的 虚拟DOM树**，用于后面的渲染。

```js
function createElement(tag) {
    const { elementName: type, attributes: data, children } = tag
    const { key, ref, ...props } = data
    
    // 处理文本内容
    let text
    
    // 处理子级列表中的 string or number
    // 同样转换为 VNode
    if (children && children.length) {
        let i, l = children.length
        for (i = 0; i < l; ++i) {
            const child = children[i]
            if (['string', 'number'].includes(typeof child)) {
                if (type === 'Text') {
                    // 基于 WebGL 的需要，区别于 DOM
                    // 这里新增一个 <Text> 标签，vnode.type === 'Text'
                    // 需特殊处理
                    if (text === undefined) text = ''
                    text += String(child)
                } else {
                    // 非标签的文字节点， vnode.type === undefined
                    // 例如 <Container>Text</Container>
                    // 中间的 Text 其实是同样需要一个文字元素
                    children[i] = vnode(
                    	undefined, 
                    	{}, 
                    	undefined, 
                    	undefined, 
                    	undefined, 
                    	undefined, 
                    	String(children[i])
                    )
                }
            }
        }
    }
    return vnode(type, props, ref, key, children, undefined, text)
}
```

执行，Perfect！打印下 `JSX`，可以看到一棵转换后的 `VNode Tree`，并且包含我们模板标签中的所有完整信息。

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd034c84e81ad?w=357&h=588&f=jpeg&s=23458)

<p style="text-align: center; font-weight: bold;">图1. VNode Tree</p>

### WebGL API

为了更好地 **解耦视图层**，我们把需要用到的一些与 `WebGL` 相关的 `API` 简单包裹下:

```js
const Api = {
    // 根据 标签类型 创建 视图元素
    createElement(vnode) {
        return new PIXI[vnode.type]()
    },
    // 创建、设置 文本元素
    createTextElement(vnode) {
        const { text: content = '', style } = vnode
        return new PIXI.Text(content, style)
    },
    setTextContent(elm, content) {
        if (elm && ['string', 'number'].includes(typeof content)) {
            elm.text = content
        }
    },
    // 获取父级
    parentNode(elm) {
        return elm && elm.parent
    },
    // 添加、删除子级
    appendChild(parent, child) {
        parent.addChild(child)
    },
    removeChild(parent, child) {
        if (child && child.parent) {
            parent.removeChild(child)
        }
    },
    // 获取下一个兄弟元素
    nextSibling(elm) {
        const parent = Api.parentNode(elm)
        if (parent) {
            const index = parent.children.indexof(elm)
            return parent.children[index + 1]
        } else {
            return undefined
        }
    },
    // 插入到指定元素之前
    insertBefore(parentElm, newElm, referenceElm) {
        if (referenceElm) {
            const refIndex = parentElm.children.indexOf(referenceElm)
            parentElm.addChildAt(newElm, refIndex)
        } else {
            Api.appendChild(parentElm, newElm)
        }
    },

}
```

这一部分可以称为 **对接层**，可以对接到各个平台，如果使用原生 `DOM` 的 `API`，则就是 `Web` 渲染，有点类似于 `react-dom` 所完成的事。

为了演示方便，我们就使用 `pixi.js` 来作为 **渲染接口**。这里与 `WebGL` 库无关，可以对接到 **任意渲染框架**。

## 第四站: 创造之柱 - Render

有了 **接口层 API** 与 `VNode` 后，可以开始 **创建真实视图元素**，并同时根据 `vnode.props` **同步属性并绑定事件**。最后 **递归创建子级**:

```js
// 根据 vnode 创建 视图元素
function createElm(vnode) {
    const { children, type, text, props } = vnode
    
  	 // vnode 的 type 为字符串时，表示其为 元素节点  
  	 // 依照 type 创建 视图元素
    if (type && typeof type === 'string') {
        // 调用接口    	 
        vnode.elm = Api.createElement(vnode) 
        
        // 递归创建子级元素，并添加到父元素中
        if (Array.isArray(children)) {
            // 创建子级
            let i, l = children.length
            for (i = 0; i < l; ++i) {
                Api.appendChild(vnode.elm, createElm(children[i]))
            }
        }
    } else if (type === undefined && text) {
        // 被非 <Text> 包裹的文字节点
        vnode.elm = Api.createTextElement(vnode)
    }
    
    // 元素创建成功时，执行设置属性
    if (vnode.elm) setProps(vnode.elm, props)
    
    return vnode.elm
}

// 属性处理
// 将 虚拟DOM 上的属性同步设置到 上面创建的真实元素 elm 上
function setProps(elm, props) {
    if (elm && typeof props === 'object') {
        const keys = Object.keys(props)
        let l = keys.length, i
        for (i = 0; i < l; i++) {
            const key = keys[i]
            const value = props[key]

            if (key.startsWith('on')) {
                // 事件绑定
                if (typeof value === 'function') {
                    const evName = key.substring(2).toLowerCase()
                    elm.on(evName, value)
                }
            } else {
                // 属性设置
                elm[key] = value
            }
        }
    }
}
```

最后，我们可以开始渲染 `VNode`：

```js
function render(vnode, parent) {
	 // 根据 vnode 创建出对应的元素
    const elm = createElm(vnode)
    
	 // 并添加到容器中即可    
    Api.appendChild(parent, elm)
    
    return elm
}
```

大功告成，到这里我们已经成功完成把 `JSX` 的初次渲染了。那下一步的需求就是: **如何更新视图呢？** 这里就是传说中的 `Diff` 算法的用武之地了！

## 第五站: 时空之匙 - Diff

说到 **虚拟DOM** 就马上能提到其核心的 `diff` 算法。当我们通过 `setState` 去更新组件时，是重新生成一棵全新的完整 **虚拟DOM树**，此时就需要对比 **新旧两棵树的差异点**，再针对性更新。也就是一种 **计算得出两个对象差异** 的算法。

这种 `diff` 算法其实使用的场景很多，例如我们很熟悉的代码版本控制。前后两份提交的代码需要比对出差异点，然后进行更新保存。只不过这里的 **代码文件** 变成了 **虚拟DOM**。由于 **虚拟DOM** 相当复杂，包含非常多的属性，并且可能拥有非常深的层级，因此如果用常规的循环递归去比较，时间复杂度为 `O(n^3)`，这性能是无法接受的。

因此天才工程师们基于 Web 视图渲染的一些特征，在比对上通过 **制定规则，选择性取舍**，大大优化了算法的效率。包含以下三种优化策略:

### 1. 同层比对策略

由于在大部分 Web 视图渲染中，我们很少会去跨层级移动元素，移动元素通常出现在同层级的列表中，因此这里可以有一个优化策略: 

**只做同层级的比对，忽略跨层级的元素移动**。

传统的 `diff` 算法需要两层循环，每两个节点之间都需要进行对比。而制定了同层比对后，节点只需要跟同一层级的节点进行比对，如下图所示。

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd046b8a7cfcd?w=1068&h=622&f=jpeg&s=37416)

<p style="text-align: center; font-weight: bold;">图2. diff 比对策略</p>

此时，性能已经大大的提升了，时间复杂度优化成 `O(n^2)`。另外，**如果真出现跨层级移动时，会直接将旧元素删除，在新的位置重新创建，也能保证更新的准确行。但可能会导致状态的丢失。**

### 2. 唯一标识策略

虽然我们做了同层比对的优化，但此时有一个问题: 

例如图2，当 `C1` / `C2` 交换位置，我们在循环比对时，由于它们均属于相同类型的节点，单单通过 type 并无法正确区分，无法识别出位置的移动。只能做到把 `C1` 修改成 `C2`，把 `C2` 修改成 `C1`。这样不仅会 **损耗性能**，而且可能导致 **状态丢失**。

最优的方式应该就是: 把 `C1` 与 `C2` 正确交换位置。关键点就在于: **如何正确识别与区分节点**。因此这里便引入了 `key` 作为唯一标识，用 `type` + `key` 便可确切地识别出节点的准确位置，从而将时间复杂度优化成了 `O(n)`。

### 3. 组件模式策略

在复杂度方面，已经有了有效的优化。接下来，便是从逻辑层进行优化。

首先一个最大的损耗就是:  无法准确定位目标节点，需要 **树遍历** 寻找更新的目标节点。

如图2，我们只要更新 **`D`节点**，却必须从最根级的 **`A`节点** 逐层 `diff`，完整比对新旧两棵 虚拟树，这里有明显的无谓损耗。如果将  **`D`节点** 抽离成一个独立的模块，则可以只调用 **`D`节点** 自身的 `diff`。 因此便引入了 **组件模式**，能够 **碎片化 虚拟DOM**。

如果我们确实需要同时更新 **`A` / `D`** 节点呢？其实左边分支 `B1` 节点并不需要更新，不需要 `diff`。如果能让 `B1` 节点拥有一个标识标识自己为非更新目标，在更新流中可以 **主动打断更新流**。那就可以只 `diff` `A` -> `B2` -> `D` 这条线了。这里，便是我们熟知的 `shouldComponentUpdate`。

### Diff 的实现

首先我们先来梳理下两个 `VNode diff` 可能出现的情况:

- 非同类型节点：
	- 直接 **创建新元素** 并 **替换旧元素**；
- 同类型节点：
	- **更新属性、事件**；
	- **递归更新子级列表**；

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd04a1c761185?w=1450&h=1042&f=jpeg&s=46707)

<p style="text-align: center;font-weight: bold;">图3. diff 流程图</p>

根据这个流程图，我们可以先从入口开始实现。

```js
function diff(oldVNode, newVNode) {
    if (isSameVNode(oldVNode, newVNode)) {
        // 开始 diff
        // 	diffVNode
        ...
    } else {
        // 新节点替换旧节点
        // replaceVNode
        ...
    }
}

// 根据 type || key 判断是否为同类型节点
function isSameVNode(oldVNode, newVNode) {
    return oldVNode.key === newVNode.key && oldVNode.type === newVNode.type
}
```

### 1. 当新旧节点不同时，直接替换 (`replaceVNode`)

```js
function replaceVNode(oldVNode, newVNode) {
    // 移除旧元素
    const { elm: oldElm } = oldVNode
    const parent = Api.parentNode(oldElm)
    Api.removeChild(parent, oldElm)

    // 创建新元素，并添加到父级中
    const newElm = createElm(newVNode)
    Api.appendChild(parent, newElm)
}
```

### 2. 当新旧节点为同一节点时，开始正式的比对 (`diffVNode`)

根据上面的流程图，我们也能明白这里我们要做以下这些事:

- **比对属性与事件** (`diffProps`)；

- **递归比对子级列表**: 这里也有三种情况；

	- 新旧节点 **均有子级列表**，则进入 **列表比对** (`diffChildren`)；
	
	- **旧节点没有** 子级，**新节点有** 子级，则 直接 **新增** 新子级 (`addVNodes`)；
	
	- **旧节点有** 子级，**新节点没有** 子级，则 直接 **删除** 旧子级 (`removeVNodes`)； 

根据以上梳理，我们先来实现 `diffVNode` 这个函数。

```js
function diffVNode(oldVNode, newVNode) {
    const { elm, children: oldChild, text: oldText, props: oldProps } = oldVNode
    const { children: newChild, text: newText, props: newProps } = newVNode

    if (oldVNode === newVNode || !elm) return

    // 已判断为同一节点，目的为 更新元素
    // 因此直接复用旧元素
    newVNode.elm = elm
    
        
    // 比对属性与事件
    diffProps(elm, oldProps, newProps)

    const hasOldChild = !!(oldChild && oldChild.length)
    const hasNewChild = !!(newChild && newChild.length)
     
    // 判断为 元素节点 或者 文字节点
    if (newText === undefined) {
        // 元素节点
        
        // 判断如何更新子级
        if (hasOldChild && hasNewChild) { 
            // 新旧节点均存在子级列表时，直接 diff 列表
            if (oldChild !== newChild) {
                // diff 列表
                diffChildren(elm, oldChild, newChild)
            }
        } else if (hasNewChild) {
            // 旧节点 不包含子级，而新节点包含子级
            // 则直接新增新子级
            addVNodes(elm, null, newChild, 0, newChild.length - 1)
        } else if (hasOldChild) {
            // 新子级不包含元素，而旧节点包含子级
            // 则需要删除旧子级
            removeVNodes(elm, oldChild, 0, oldChild.length - 1)
        } else if (oldText !== undefined) {
            // 当新旧均无子级
            // 这里有可能存在 <Text> 标签，且新内容为空
            // 因此直接清空旧元素文字
            Api.setTextContent(elm, '')
        }
    } else if (oldText !== newText) {
        // 文字节点
        // 当新旧文字内容不同时，直接修改内容
        Api.setTextContent(elm, newText)
    }
}

// 更新属性
function diffProps(elm, oldProps, newProps) {
    if (oldProps === newProps || !elm) return
    if (typeof oldProps === 'object' && typeof newProps === 'object') {
        let keys = Object.keys(oldProps), i, l = keys.length
        
        // 重置被删除的旧属性
        for (i = 0; i < l; i++) {
            const key = keys[i]
            const oldValue = oldProps[key], newValue = newProps[key]

            if (key.startsWith('on')) {
                /*
                 * 当存在旧事件，且新旧值不一致时
                 * 事件解绑
                 */
                if (typeof oldValue === 'function' && oldValue !== newValue) {
                    const evName = key.substring(2).toLowerCase()
                    elm.off(evName, oldValue)
                }
            } else {
                /* 
                 * 属性被赋值时会被自动重置
                 * 只需要重置被删除的属性即可
                 */
                if (newValue === undefined) {
                    // 元素属性默认值
                    elm[key] = DEFAULT_PROPS[key]
                }
            }
        }
        // 设置新属性
        setProps(elm, newProps)
    }
}
```

## 比对子级列表 (`diffChildren`)

其实比对属性、事件都是相对简单的，而 **子级列表的比对，才是整个 diff 算法中最核心且最考验性能的部分**，因此这里的列表比对算法决定了整个更新渲染的性能。在 **虚拟DOM** 刚出现时，使用的是比较简单的 **深度优先(DFS) + 排序比对** 的方式。后来出现了更为高效且沿用至今的 **两端比对算法 + Key值比对**，直接把 `diff` 的效率提高了一个层级，且更好理解，有三种优先级不同的比对策略:

- 优先从新旧列表的 **两端** 的 **四个节点** 开始进行 **两两比对**；

- 如果均不匹配，则尝试 **key 值比对**；

	- 如 **key 值** 匹配上，则移动并更新节点；
	
	- 如 未匹配上，则在对应的位置上 **新增新节点**；
	
- 最后全部比对完后，列表中 **剩余的节点** 执行 **删除或新增**；

这里这么说大家是不是一脸懵🤣。没事，先稍微理解下就行。我们接下来直接用动画来更直观地看下两端比较算法的具体过程。这里就以刚才 图2 的子级列表为例，即: 

- oldChildren 包含 5 个子级节点: `[A, B, C, D, E]`；

- newChildren 的子级修改为: `[F, C, B, D, A]`；

> Tips: 
> 
> 如新旧列表中的 A，代表这两个节点为 **同类型节点**，即节点的 `type / key` 均相等；

### 第一轮比对: 

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd04d40973064?w=750&h=389&f=gif&s=941267)

<p style="text-align: center; font-weight: bold;">图4. diff 第一轮循环</p>

- **1.** 优先从新旧列表的两端正向开始，**不相同**: `A !== F` 且 `E !== A`；

- **2.** 两端交叉比对，发现 **旧列表的第一项与新列表的末项相同** (`isSameVNode`):

	- 把旧列表中的第一项 **移动** 到最后一项；
	
	- 继续递归 diff 新旧 `A`；

### 第二轮比对:

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd0505762e643?w=585&h=350&f=gif&s=724488)

<p style="text-align: center; font-weight: bold;">图5. diff 第二轮循环</p>

- **1.** 同样，优先从两端正向比对，`B !== F & E !== D`；

- **2.** 两端交叉比对，`B !== D & E !== F`；

- **3.** 进入 **key 比对**:

	- 固定取 **新列表首项 (`F`)**；
	
	- 循环与旧列表 **key列表** 逐项比对，均 **无法匹配**；
	
	- 因此为 **新节点**，直接 **创建并添加到旧列表首项**；

### 第三轮比对:

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd0542e834ceb?w=750&h=410&f=gif&s=692139)

<p style="text-align: center; font-weight: bold;">图6. diff 第三轮循环</p>

- **1.** 两端正向、交叉比对，不匹配；

- **2.** 进入 **key 比对**；

	- 固定取 **新列表首项 (`C`)**；
	
	- 循环与旧列表逐项比对，**匹配** 到旧列表第二项；
	
	- 则把旧列表中的第二项 **移动到首项**，并继续 递归继续 diff 新旧 `C`；

### 第四、五轮比对:

![](https://user-gold-cdn.xitu.io/2020/3/9/170bd057514a585e?w=750&h=420&f=gif&s=814595)

<p style="text-align: center; font-weight: bold;">图7. diff 第四五轮循环</p>

- **1.** 首项比对，匹配成功；

	- 递归继续 diff 新旧 `B`；
	
	- 移动下标，进入下一轮比对；
	
- **2.** 同样首项比对匹配；

	- 递归继续 diff 新旧 `D`；
	
	- 移动下标，进入下一轮比对；
	
- **3.** 新列表循环已结束，**删除** 旧列表中的剩余节点；

经过了五轮的比对，旧列表已经被成功更新。为了包含我们上面解释的三种策略，举例时用的是复杂度较高，较少出现的场景。在日常业务中，大部分都会更简单，性能表现会更好。接下来，我们把这个算法用代码实现下，采用 **双列表游标 + while 循环** 的方式:

```js
function diffChildren(parentElm, oldChild, newChild) {

    /**
     * 更新子级列表
     * 双列表游标 + while
    */
	 
    // 初始化游标
    let oldStartIdx = 0, newStartIdx = 0
    let oldEndIdx = oldChild.length - 1, newEndIdx = newChild.length - 1
    
    // 列表首尾节点
    let oldStartVNode = oldChild[0], oldEndVNode = oldChild[oldEndIdx]
    let newStartVNode = newChild[0], newEndVNode = newChild[newEndIdx]

    let oldKeyToIdx, idxInOld, elmToMove, before

    /**
     * 当起始游标 < 终止游标时，
     * 表示列表中仍有未 diff 的节点
     * 进入循环
    */
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        /**
         * 1. 排除非有效的节点
         * 剔除列表中包含的 undefined || false || null
        */
        if (oldStartVNode == null) {
            oldStartVNode = oldChild[++oldStartIdx]
        } else if (oldEndVNode == null) {
            oldEndVNode = oldChild[--oldEndIdx]
        } else if (newStartVNode == null) {
            newStartVNode = newChild[++newStartIdx]
        } else if (newEndVNode == null) {
            newEndVNode = newChild[--newEndIdx]
        } else if (isSameVNode(oldStartVNode, newStartVNode)) {
            /**
             * 2. 正反向两两比对列表首项与末项匹配成功
             * 移动游标，递归 diff 两个节点
             * 均未匹配上，则进入 3. key 值比对
            */
            diff(oldStartVNode, newStartVNode)
            
            oldStartVNode = oldChild[++oldStartIdx]
            newStartVNode = newChild[++newStartIdx]
        } else if (isSameVNode(oldEndVNode, newEndVNode)) {
        
            diff(oldEndVNode, newEndVNode)
            
            oldEndVNode = oldChild[--oldEndIdx]
            newEndVNode = newChild[--newEndIdx]
        } else if (isSameVNode(oldStartVNode, newEndVNode)) {
        
            Api.insertBefore(parentElm, oldStartVNode.elm, Api.nextSibling(oldEndVNode.elm)) 
            diff(oldStartVNode, newEndVNode)
            
            oldStartVNode = oldChild[++oldStartIdx]
            newEndVNode = newChild[--newEndIdx]
        } else if (isSameVNode(oldEndVNode, newStartVNode)) {
        
            Api.insertBefore(parent, oldEndVNode.elm, oldStartVNode.elm)
            diff(oldEndVNode, newStartVNode)
            
            oldEndVNode = oldChild[--oldEndIdx]
            newStartVNode = newChild[++newStartIdx]
        } else {
            /**
             * 3. 两端比对均不匹配
             * 进入 key 值比对
             */
            // 根据剩余的旧列表创建 key list
            if (!oldKeyToIdx) {
                oldKeyToIdx = createKeyList(oldChild, oldStartIdx, oldEndIdx)
            }
            
            // 判断新列表项的 key值 是否存在
            idxInOld = oldKeyToIdx[newStartVNode.key || '']
            if (!idxInOld) {
                /* 
                 * 4. 新 key 值在旧列表中不存在
                 * 直接将该节点插入
                */
                Api.insertBefore(parentElm, createElm(newStartVNode), oldStartVNode.elm)
                newStartVNode = newChild[++newStartIdx]
            } else {
                /* 
                * 5. 新 key 在旧列表中存在时
                * 继续判断是否为同类型节点
                */
                elmToMove = oldChild[idxInOld]
                if (isSameVNode(elmToMove, newStartVNode)) {
                    /* 
                    * 6. 新旧节点类型一致
                    * key 有效，直接移动并 diff
                    */
                    Api.insertBefore(parentElm, elmToMove.elm, oldStartVNode.elm)    
                    diff(elmToMove, newStartVNode)
                    
                    // 清空旧列表项
                    // 后续的比对可以直接跳过
                    oldChild[idxInOld] = undefined
                } else {
                    /* 
                     * 7. 新旧节点类型不一致
                     * key 效，直接创建元素并插入
                    */
                    Api.insertBefore(parentElm, createElm(newStartVNode), oldStartVNode.elm)
                } 
                newStartVNode = newChild[++newStartIdx]
            }
        }
    }

    /* 
     * 8. 当有游标列表为空时，则结束循环，进入策略3
     * 当 旧列表为空 时，则创建并插入新列表中的剩余节点
     * 当 新列表为空 时，则删除旧列表中的剩余节点
    */
    if (oldStartIdx <= oldEndIdx || newStartIdx <= newEndIdx) {
        if (oldStartIdx > oldEndIdx) {
            // 新增节点
            const vnode = newChild[newEndIdx + 1]
            before = vnode ? vnode.elm : null
            addVNodes(parentElm, before, newChild, newStartIdx, newEndIdx)
        } else {
            // 删除节点
            removeVNodes(parentElm, oldChild, oldStartIdx, oldEndIdx)
        }
    }
}
```

恭喜童鞋们~ 我们完成了传说中的 `两端比对算法` 咯🥳。其实只要按比对的优先级把逻辑理清楚，逐一执行判断，思路还是比较清晰的。

### 实践建议

通过上面真正的代码编写以及了解实现原理后，我们其实能从中找到一些好的实践方式，从而优化我们的代码，提高性能。

#### 1.`props`的传递

`JSX` 中标签可以传递属性，最简单的方式就是 **值传递**:

```js
const tmp = <Container text="text" data={{ a: 1 }}>Text</Container>
```

由于在比对的时候，在判断 `props` 是否发生变化时，采用 **全等** 的比较。因此，当出现值是 **引用对象** 时，如果直接像上面这样, `data` 写在 `JSX` 中，则每次 `diff` 时 `data` 的值都是全新的对象，不会相等。即使对象属性全部一致，但每次均需要循环比对每一项属性。

因此建议:

- 当属性值为 **引用对象** 时，如 `Object`、`Array`、`Function`等，直接使用 **引用传递**:

```js
const data = { a: 1 }
const tmp = <Container text="text" data={data}>Text</Container>
```

- 同理，当数据需要修改时，不要直接修改源对象，而是应该 **遵循数据不可变原则**，生成一个新对象。

这样的建议可以有效降低 `diff` 时的性能损耗，当场景复杂时，收益就比较可观了。

#### 2. 渲染树结构稳定

`diff` 算法采用的 **同层比对** 的策略，因此如果是跨层级的移动，就会 重新创建新节点并删除原来的节点，并不是真正的移动。所以保证 **渲染树结构稳定** 可以有效提高性能。

- 尽量避免节点的 **跨层级移动**；

- 如无法避免，则需要考虑 **状态同步** 的问题；

- 同层移动同样也会在 `diff` 时需要额外的循环比对，应该减少不必要的频繁移动；

#### 3. key 的使用

当需要列表中 `VNode` 的 **同层移动** 时，加上唯一标识 `key` 能有效提高 `diff` 性能，避免元素的 **重渲染**。

- 注意确保 `VNode.key` 在 `diff` 前后的 **一致**，这样才可有效提升性能，避免使用 `index`、 `Math.random`、时间戳等值；

- 即使在非循环列表渲染时，给标签添加 `key` 值同样会生效，不同的 `key` 会导致节点被判定为非同类节点，从而进行替换；

#### 4. 组件化

- **复用性高** 且需要 **频繁更新** 的节点抽离成 **组件**，会使 `VNode Tree` 碎片化，从而能更有效地进行 **局部更新**，减少触发 `diff` 的节点数量，提高性能且提高代码复用率；

- 但由于组件的创建和 `diff` 相比普通节点来说更为 **复杂**，需要执行例如生命周期，组件比对 等，所以需要 **合理规划**，避免 **过分组件化** 导致 **内存的浪费和影响性能**，一些 **复用率低的静态元素** 直接使用元素节点更为合理；

## 第六站: 休憩之地 - 总结归纳

受篇幅所限，本文暂且完成到这里。先来总结回顾下我们完成的部分:

- **1.** `JSX` 是一种 **动态模板语法**，通过 `Babel` 编译为 `createElement` 函数；

- **2.** `createElement` 将 `JSX` 转换为 **虚拟Dom(VNode)**，包含完整的标签信息 (类型、属性、子级列表)；

- **3.** **虚拟DOM** 是一种 **牺牲最小性能与空间，换取 架构优化** 的方式，其 **组件化** 以及 **解耦** 的思想，提高了项目的拓展性、复用性与可维护性，同时为 **跨平台渲染** 奠定基础；

- **4.** 通过实现 `createElm` 与 `render`，完成 `VNode` 的 **初次渲染**；

- **5.** `Diff` 是一种 **计算得出两个 VNode 差异** 的算法，使用 **同层比对、唯一标识、组件模式** 优化了算法比对性能，将时间复杂度降低为 `O(n)`；

- **6.** 列表比对 (`diffChildren`) 采用 **两端比对算法 + Key值比对** 算法，大大提高了 `Diff` 效率；

- **7.** 实现 `diffVNode`、`diffProps`、`diffChildren`，完成 `VNode` 的 **动态更新**；

我们完成了框架最核心的 **`JSX` - `Render` - `Diff`** 的渲染更新的主机制，奠定了最底层的基础。在下一篇文章中，我们将继续基于此完成 **组件化(`Component`)**、**组件更新(`setState`)** 以及 **生命周期**。希望能帮助大家更了解 React，掌握一些优秀的编程思维。

**[React 实践揭秘之旅，中高级前端必备(下) ->](https://github.com/xd-tayde/blog/blob/master/ReactGL-2.md)**

这篇文章所完成的类 `React` 框架是过去几个月我自己在 `Web` 游戏方面的尝试和探索，目的是期望能 **降低传统前端工程师开发游戏的门槛** 和 **引入前端开发模式提升开发效率**，使 前端开发 与 游戏开发 形成一定程度的优势互补。当然这个目标很大，也并不简单，规划中仍然有许多工作要做。有兴趣的小伙伴移步 github 查看完整代码:

**[react-webgl.js ->](https://github.com/xd-tayde/amoy.js)**

最后，我们去年下半年在厦门创办了一家技术公司，主要是在 **游戏领域** 和 **K12 STEAM 教育** 方向上的努力。非常欢迎 有兴趣想深入了解 或者 想跟我探讨 的小伙伴直接联系我!🥳~~

> Tips:
> 
> 看博主写得这么辛苦下，跪求点赞、关注、Star！[更多文章猛戳 ->](https://github.com/xd-tayde/blog) 
> 
> 邮箱: 159042708@qq.com  微信/QQ: 159042708

**[祝福#感恩#武汉加油##RIP KOBE#]()**