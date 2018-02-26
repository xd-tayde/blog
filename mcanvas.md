# H5的图片处理与合成

## 引言：

图片处理现在已经成为了我们生活中的刚需，想必大家也经常有这方面的需求。实际前端业务中，也经常会有很多的项目需要用到图片加工和处理。但是前端的图片处理能力在`js`和设备性能的限制下，其实表现并不突出。由于公司的业务需求，过去一段时间我对这方面有了些粗浅的了解和尝试，希望能总结出来与大家分享，通过本文，应该能对图片处理方面有一定程度的理解，有感兴趣的童鞋可以与我深入讨论，希望本文能达到抛砖引玉的效果，有不足之处希望谅解。

通常我们所说的图片处理一般可以分成两种：

### 基础类型的图片处理

我们常用的图片裁剪，图片添加边框，多张图片的合成等业务都属于基础类型的图片处理。这类型的图片处理在实际项目中有着大量的使用场景，其区分点在于无需从像素级别，而是通过计算来改变图片的尺寸及位置来改造图片，例如图片的合成 / 拼接 / 裁剪 / 缩放 / 旋转等，本文将会对这类型的图片处理展开进行详细的阐述；

### 算法类型的图片处理

这类型的图片处理复杂度较高，需要从像素级别对图片进行改造，例如我们使用`photshop`或者美图秀秀等工具对图片进行的 美颜 / 滤镜 / 黑白 / 抠图 / 模糊等美化，这类型的重点主要在于算法层面。在前端，由于`js`及设备性能的限制，通常表现并不理想，在真正的线上业务中，为了追求更好的用户体验，通常会把该类型的处理由服务端来进行，通过`ajax`发送原图给服务端再接收处理完成后的结果图。当然，比较简单算法处理，例如黑白，模糊等，也是可以直接将算法跑在浏览器中，由前端单独完成的，例如：

[图片模糊效果(手机体验)](http://api.meitu.com/front_end/xiuxiu/online_mapp/makeup/index.html?makeupType=251&pic=http://mtapplet.meitudata.com/57ea433108c45eb2b166.jpg)

#### 效果图如下：

<div align='center'>
	<img src="./images/mcanvas/blur-border.jpg" width = "400" align=center /><br/>
</div>

这个小应用通过服务端的美妆算法将图片添加木偶妆容后，再由前端算法进行边框及模糊背景图的添加，这类处理的基本原理是通过`canvas`的`getImageData`来获取图片的所有像素点，通过对每个像素点的`rgba`通道值的改变，达到图片处理的效果，但复杂的算法处理需要大量的像素点操作，需要很高的性能要求，例如抠图/美颜等，由于前端由于现有设备性能、`js`自身及算法安全性的限制，暂时并无法达到大面积使用的程度，通常这种算法处理会放在服务端实现，由前端传递原图，服务端处理完后返回结果图的形式。

由于该类型更重要的是算法，我对算法的理解也比较肤浅，因此本文不在此做深入的研究及探讨。

## 实现：

本文中，我将基础类型的图片处理大致的分成以下几种类型，这些类型基本能覆盖日常所有业务场景：

- 图片的缩放；
- 图片的裁剪；
- 图片的合成；
	- 图片与图片的合成，例如贴纸，边框，水印等；
	- 为图片添加文字；
	- 为图片添加基础几何图形；

> Tips: 我已将该类型的图片处理场景封装成了一个插件，基本上能应付所有这类型图片处理的需求，**[GIT地址](https://github.com/xd-tayde/mcanvas)** (欢迎探讨);

### 一、图片的缩放

图片的缩放通常是用来做图片的压缩。在保证图片清晰的前提下通过合理地缩小图片尺寸，能大大的降低图片的大小，在实际应用场景中，有着广泛的用途。例如图片上传时，用户自主上传的图片可能是一张非常大的尺寸，例如现在手机所拍摄的照片尺寸经常能达到`1829*2560`的尺寸,大小可能超过5M。而在项目中，我们可能并不需要用到这么大的尺寸，因此对图片的压缩能大大的优化加载速度。方法是：

#### 1、新建一个`canvas`画布，将宽高设置为需要的尺寸;

该画布既为图片缩放后的尺寸，此处有的重点是需要保证图片的比例不变, 因此需要通过计算得出画布的宽与高：

```js
let imgRatio = img.naturalWidth / img.naturalHeight;
let cvs = document.createElement('canvas');
let ctx = cvs.getContext('2d');
cvs.width = 1000;
cvs.height = cvs.width / imgRatio;
```

#### 2、将图片画入后再导出成`base64`;

这里使用2个最常用的方法:

- `ctx.drawImage(image, x, y, dw, dh)`: 这个方法其实最多可以接收9个参数, 实现压缩，只需要使用其中的5个参数即可, 其余参数在其它部分使用到时再做详解；

	- image : 需要绘制的图片源，需要接收已经 **加载完成** 的HTMLImageElement，HTMLCanvasElement或者HTMLVideoElement；
	- x / y : 相对于画布左上角的绘制起始点坐标；
	- dw / dh : 绘制的宽度和高度，宽高比例并不锁定，可使图片变形；

- `cvs.toDataURL(type, quality)`: 该方法用于将画布上的内容导出成 `base64` 格式的图片，可配置2个参数； 

	- type: 图片格式, 一般可以使用 `image/png` 或者 `image/jpeg`, 当图片不包含透明时，建议使用`jpeg`，可使导出的图片大小减小很多；
	
	> Tips: 此处有个坑, 想导出`jpg`格式的图片必须用`image/jpeg`，不能使用`image/jpg`；
	
	- quality: 图片质量，可使用0~1之间的任意值；经过测试，该值设置成0.9时较为合适，可以有效减小图片文件大小且基本不影响图片清晰度，导出后的 `base64` 既为压缩后的图片；

```js
// 将原图等比例绘制到缩放后的画布上；
ctx.drawImage(image, 0, 0, cvs.width, cvs.height);

// 将绘制后的图导出成 base64 的格式；
let b64 = cvs.toDataURL('image/jpeg', 0.9);
```

### 二、图片的裁剪

在实际项目中，由于图片的宽高比例各式各样，而展示和使用一般需要一个较为固定的比例，此时便需要将图片裁剪成我们需要的宽高比例，使用到的方式其实和图片的缩放基本一致，主要是通过调整 `drawImage` 的`x, y`参数来实现，此处以需要将一张`600*800`的长方形图竖直居中裁剪为`600*600`的正方形图为例, 简单封装成一个功能函数:

```js
// 使用方式：
let b64 = cropImage(img, {
    width : 600,
    height : 600,
});

// 居中裁剪
function cropImage(img, ops){
	// 图片原始尺寸；
	let imgOriginWidth = img.naturalWidth,
        imgOriginHeight = img.naturalHeight;
        
    // 图片长宽比，保证图片不变形；
    let imgRatio = imgOriginWidth / imgOriginHeight;
    
    // 图片裁剪后的宽高， 默认值为原图宽高；
	let imgCropedWidth = ops.width || imgOriginWidth,
        imgCropedHeight = ops.height || imgOriginHeight;
        
    // 计算得出起始坐标点的偏移量, 由于是居中裁剪，因此等于 前后差值 / 2；
	let dx = (imgCropedWidth - imgOriginWidth) / 2,
		dy = (imgCropedHeight - imgOriginHeight) / 2;

    // 创建画布，并将画布设置为裁剪后的宽高；
	let cvs = document.createElement('canvas');
	let ctx = cvs.getContext('2d');
	cvs.width = imgCropedWidth;
	cvs.height = imgCropedHeight;
	
    // 绘制并导出图片；
	ctx.drawImage(img, dx, dy, imgCropedWidth, imgCropedWidth / imgRatio);
	return cvs.toDataURL('image/jpeg', 0.9);
}
```

原理其实是，将`drawImage`的绘制起始点`(dx, dy)`向上偏移，此时由于`canvas`已被我们设置成期望裁剪后的尺寸，而超出画布的部分不会绘制，从而达到裁剪的目的；通过灵活的设置值，基本可以完成各种图片裁剪需求，

简单示例图(黑色框代表创建的画布的尺寸): 
<div align='center'>
	<img src="./images/mcanvas/crop.png" width = "400" align=center /><br/>
</div>

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

#### 1、图片的跨域

首先，图片加载并绘制涉及了图片的跨域问题，因此如果是一张在线的图片，需要在图片服务器上设置跨域，并且加载之前将`crossOrigin`设置为`*`，否则绘制的时候会报跨域的错误。

> Tips: 这里积累了一些小坑，可以跟大家提下：
> 
> 1、`crossOrigin`需要严格设置，既只有是线上图片时，才设置，而本地路径或者`base64`时，则一定不能设置，否则在某些系统下会报错；
> 
> 2、当项目会本地包时，例如内置于 `App`中时，`crossOrigin`值无效，无论该值设置与否，都会报跨域的错误；解决办法是：需要将所有图片转换成`base64`再进行绘制；
> 
> 3、`crossOrigin`值一定要在为`img`赋值`src`之前进行设置，否则无效；

#### 2、图片的加载

由于`canvas`的绘制需要的是已经加载完成的图片，而图片的合成一般需要加载很多张图片，例如上面的美女图与猫耳朵图；

```js
function loadImage(image, loader, error){
	// 创建 image 对象加载图片；
	let img = new Image();
	
	// 当为线上图片时，需要设置 crossOrigin 属性；
	if(image.indexOf('http') == 0)img.crossOrigin = '*';
	img.onload = () => {
	    loaded(img);
	    
	    // 使用完后清空该对象，释放内存；
	    setTimeout(()=>{
	        img = null;
	    },1000);
	};
	img.onerror = () => {
	    error('img load error');
	};
	img.src = image;
}
```

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

























