# H5的图片合成与处理

## 简介：

由于在项目上有很多图片处理方面的需求，例如添加边框，添加文案，合成贴纸等功能，这些功能经常需要对尺寸，位置等进行大量的计算，当业务较复杂时，这些计算也会变得麻烦。因此我们梳理了合成逻辑与计算，封装了一个小插件，使用简单易用的`API`, 让`H5`在基础图片处理变得十分的简单快捷，并且该插件已经使用于多个线上项目中。

希望能让大家更好更快速地完成项目上的需求，有图像处理方面兴趣的童鞋，欢迎与我讨论和提`issue`哈。~~

**[Example](http://f2er.meitu.com/gxd/mcanvas/example/index.html)**

**[中文文档](https://github.com/xd-tayde/mcanvas/blob/master/README_ZH.md)**

**[Git](https://github.com/xd-tayde/mcanvas)**



## 实现：

该插件基于`canvas`，将图片的合成逻辑简单地梳理成类似于`photoshop`中的图层系统，以链式调用的方式得到合成结果图，基本的流程是：

- 创建画布；
- 添加图层；
- 合成导出；

### 创建画布,并设置背景图片

```js
let mc = MCanvas({
	width: 1000,
	height: 1000,
});
	
mc.background(image, options);
```

### 在画布上绘制添加图层

有多个`API`可以提供添加各式各样的图层类型；

- `mc.add(image, options)`: 在画布上添加图片图层，支持缩放/旋转/裁剪；

- `mc.watermark(image, options)` : 基于 `add` 封装，专门用于添加水印；

- `mc.rect(options)` : 在画布上绘制矩形；

- `mc.circle(options)` : 在画布上绘制圆形；

- `mc.text(context, options)` : 在画布上添加文字，支持自定义文字样式，包含渐变/阴影/描边等功能；

### 合成导出图片

在链式末尾调用`mc.draw()`，在成功回调中便可得到 `base64` 格式的合成图；

```js
mc.draw({
    // 导出图片格式： png/jpg/jpeg/webp;
    // default : png;
    type: 'png',

    //  图片质量，对 png 格式无效； 0~1；
    // default: .9;
    quality: 1,

    // 成功回调；
    success(b64){
    	console.log(b64);
    },

    // 错误回调；
    error(err){
    	console.log(err);
    }
})
```
