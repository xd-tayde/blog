### 三、图片的合成

图片的合成原理其实类似于`photoshop`的理念，通过图层的叠加，最后合成并导出， 相比于裁剪和缩放，其实基本原理是一致的，但是它涉及了更多的计算和比较复杂的流程，我们先一起来梳理下合成的整个逻辑。

相信大家对 `photoshop`都是较为了解的，我们可以借鉴它的思维方式:

- 新建 `psd` 文件, 设置宽高;
- 从底部到顶部一层层添加所需要的图层；
- 最后直接将整个文件导出成一张图片；

以需要合成下图为例：

<div align='center'>
	<img src="./images/mcanvas/ear.jpeg" width = "300" align=center /><br/>
</div>

1、首先我们需要创建一个与原图一样大小的画布；

2、加载背景图并添加背景图层，也就是这个美女啦~

3、加载猫耳朵图并添加美女头上的猫耳朵图层

> Tips: 2/3顺序不可逆，否则耳朵会被美女盖在下面哦~因此图片的加载控制十分重要；

4、将整个画布导出成一张图片；

合成部分，主要以封装的插件为栗子哈。这样能尽可能的完整，避免遗漏点。在开始之前，我们先来简单了解下图片的加载的细节。

#### 3、队列系统；

图片的加载时间是异步且未知的，而图片的合成需要严格保证绘制的顺序，越后绘制的图片会置于越顶层，因此我们需要一套严格机制来控制图片的加载与绘制，否则我们将无法避免的写出回调地狱，这里我使用到了简单的队列系统；

队列系统的原理其实也很简单，主要是为了我们能确保图层从底到顶一层一层的绘制；

- 我们需要一个函数队列栈: `let queue = []`;
- 将各个图层的绘制逻辑存入队列中，但不执行, 最后通过一个启动函数开启队列的运转；
- 每层图层绘制完成后调用 `_next`方法，表示该图层已绘制完毕，可执行下一图层的绘制；

我期望的使用方式如下：

```js
let queue = []
// 创建画布；
let mc = new MCanvas();

// 将合成逻辑存入队列queue中；
mc.add(image-1).add(image-2);

// 启动图片的绘制，并导出图片；
mc.draw();
```
因此，我们需要在 `add`函数逻辑的末尾调用`_next`，这样便可以确保队列的顺序调用；

```js
MCanvas.prototype.add = function(){
	// 绘制逻辑，之后详解；
	...
	
	this._next();
}

MCanvas.prototype._next = function(){
    if(this.queue.length > 0){
        this.queue.shift()();
    }else{
        this.fn.success();
    }
};

MCanvas.prototype.draw = function(){
	// 导出逻辑；
	...
		
	this.fn.success = () => {
		 // 使用 setTimeout 能略微提升性能表现；
		 // 且队列函数中都为真正的异步，因此此处不会影响逻辑；
        setTimeout(()=>{
            b64 = this.canvas.toDataURL(`image/jpeg}`, 0.9);
            ...
        },0);
   };
	this._next();
}
```

此时，`queue`、`_next`与`draw`便组成了一整套队列系统，可确保图片的顺序加载和绘制，准备好素材和队列后，我们便可以开始真正的合成图片咯~~

#### 4、创建画布并绘制背景图；

这里的创建与缩放的创建其实是一样的，创建一个跟原图一样大小的画布；

```js
MCanvas.prototype._init = function(){
    this.canvas = document.createElement('canvas');
    this.ctx = this.canvas.getContext('2d');
};
```
设置画布大小并绘制美女背景图；

```js
MCanvas.prototype.background = function(image, bgOps){
	this.queue.push(() => {
        loadImage(bgOps.image, img =>{
            let { iw, ih } = this._getSize(img);
			    // 图片与canvas的长宽比；
			    let iRatio = iw / ih;
			    // 背景绘制参数；
			    let dx,dy,dwidth,dheight;
				 this.canvas.width = iw;
	           this.canvas.height = ih;
	           dx = dy = 0;
	           dwidth = this.canvas.width;
	           dheight = this.canvas.height;
				 this.ctx.drawImage(img,dx,dy,dwidth,dheight);
			     this._next();
        },this.fn.error);
    });
    return this;
};
```
