# CSS学习笔记

### 引入：
* 外部样式：

		<link rel="stylesheet" href="style.css" />

* 内部样式表：
	
		<head>
			<style>...<style>
		</head>

* 内嵌样式：

		<p style="color:red;margin-left:20px"></p>

### 注释：   

>`/* ... */`

### 浏览器私有属性：
>chrome、safari:  -webkit-;
>firefox:  -moz-;
>IE:-ms-;
>opera: -o-;

### 语法规则：

	margin:[<length>|<percentage>|auto]{1,4}
	/*
		<length>:基本元素；
		`|`:组合符号；
		{1,4}:数量符号；
	*/

基本元素：
1、关键字： auto、solid、bold……
2、组合符号：
> 空格：表示必须出现，且顺去固定；

> `&&`：必须出现，但顺序不固定；

>` ||`：至少出现一个，但顺序随意；

> `|`：只能出现一个；

> `[]`：分组；

>` +`：可出现一次或多次；

> `？`：可出现也可不出现；

> `{}`：可以出现的次数；{2，4}：最少出现2次，最多出现4此；

> `*`：可出现0次，1次或多次；

> `#`：需出现1次或多次，需要用`,`隔开；

### 规则语法：
> @media：响应式布局时监听屏幕尺寸；

> @keyframs：描述CSS动画帧；

> @font-face：引入外部字体；

> @import/@charset/@name space/@page/@supports/@document

### 选择器：
#### 简单选择器：
	标签选择器： tag{ }
	类选择器：.className{ }
	id选择器： #id{ }
	通配符： *{ }
	属性选择器： [disabled]{ }/[type=button]{ }
			   [class~="sports"]{ }   // 类名中包含sports,且以空格隔开；
			   [lang|="en"]{ }        // 类名中包含en，且以“-”隔开；
			   [href^="#"]{ }         // href属性值以 # 开头的元素；
			   [href$="pdf"]{ }       // href属性值以 pdf 结尾的元素；
			   [here*="lady"]{ }      // href属性值包含lady的元素；

#### 伪类选择器：
	a:link{ }                 // 选中具有href的a标签样式；
	a:visited{ }              // 已访问过的链接的样式；
	a:hover{ }                // 当鼠标移上时的样式；
	a:active{ }               //  当鼠标点击时的样式；
	
	:enabled{ }  :disabled{ }   :chedcked{ }

	li:first-child{ }  li:last-child{ }       // 元素首项、末项
	li:nth-child(even){ }                     // 选中偶数项
	li:nth-child(3n+1){ }              // 选中正序第1、4、7。。。项
	li:nth-last-child(3n+1){ }         // 选中倒序第1、4、7。。。项

	li:only-child{ }                 // 选中作为唯一子元素的的li标签
	dd:first-of-type{ }              // 选中第一个dd类型的元素
	dd:last-of-type{ }               // 选中最后一个；
	dd:nth-of-type{ } 
	span:only-of-type{ }             // 选中同级元素只有一个span的<span>

	:empty{ }   :root{ }   :not(){ }  
	:target(){ }                // 选择器会突出显示当前活动的HTML锚;

#### 组合选择器：
>可以将简单选择器进行组合，如 img[src$="jpg"]{ };

#### 伪元素选择器：
	::first-letter{ };               /* 选择头字母；*/
	::first-line{ };                 /* 选择首行；*/
	/* 插入内容；*/
	::before{content:"before";}
	::after{content:"after";}	
	/* 应用于被用户选中的内容 */
	::selection{ }

#### 后代选择器：
	<div class="main">
		<h2>111</h2>
		<div>
			<h2>222</h2>
			<p>333</p>
		</div>
	</div>

	.main h2{ }                     // 选中111、222;
	.main>h2{ }                     // 选中111

#### 兄弟选择器：
	<div class="main">
		<h2>111</h2>
		<p>222</p>
		<p>333</p>
	</div>

	h2+p{ }               // 选中与h2相邻的p：222；
	h2~p{ }               // 选中h2同级之后的所有p:222,333;
	
#### 选择器分组：
> 可以使用 `,`讲选择器合并：

		h1,h2,h3{ };

#### CSS选择器优先级：
>| 样式类型         |   权重     | 
>| :--------      | --------:      | 
>| a：行内样式  | 1000  | 
>| b：id选择器    |   100   | 
>| c：类、伪类和属性选择器  |    10     |
>| d：标签和伪元素选择器    |   1   |
> ##### value值 = a* 1000 + b* 100 + c* 10+ d ;
>1、当value不等时，value值越大，则优先权更大；不同属性则合并，相同属性覆盖；
>2、当value相等时，相同属性，后面的覆盖前面；不同属性合并；

>> 改变CSS优先级：
>1、通过改变CSS里的前后顺序；
>2、提升选择器的value值；
>3、加入 !important. （优先使用1、2）；

### 文本与颜色：
#### 字体大小
	
	font-size:<length>|<percentage>|<absolute-size>|<relative-size>    
> `<length>`: 例如14px;
> `<percentage>`: 例如200%/2em(2em/200% ：表示为父级字体大小的2倍)；
> `<absolute-size>`:例如small、large;
> `<relative-size>`: 例如smaller、larger;

#### 字体

	font-family:[<family-name>|<generic-family>]#

字体族名称(family-name)：是具体的字体名称, "Microsoft YaHei"
 
 类族名称(generic-family)：是字体类型名称而具体的字体则由浏览器决定,serif/sans-serif/cursive/fantasy/monospace等

>* font-family中设置的字体不会被浏览器自动下载, 字体是否可用则完全依靠用户电脑是否已安装该字体库而已；
>* 属性值必须以类族名称(generic-family)结尾确保字体会以正确的形式解析渲染；
>* 类族名称(generic-family)后的字体族名称(family-name)不会生效;

#### 字体粗细；

	font-weight:normal|bold|bolder|lighter|100|200...
#### 斜体；

	font-style:normal|italic|oblique
####  行高；
 
	line-height:normal|<number>|<length>|<percentage>

#### 设置linge-height:300%与设置line-height:3的区别：
> line-height：两行基线的间隔；
> 设置为300%时，直接以父元素的字体大小计算后继承，不管子元素自身的字体大小；既为父元素字体大小的3倍；（先计算后继承，为静态值）
> 设置为3时，则是动态的根据子元素自身的字体大小乘以3；（先继承后计算，为动态值，会自适应）；

####  利用line-height让多行文本垂直居中：
在文本的后面添加个空白元素，设置其line-height和父元素高度一致，并在文本上设置vertical-align:middle;

#### 字体组合样式：
	// font:italic bolid 20px/1.5 arial,serif;
	font:[<font-style>||<font-weight>]?<font-size>[/<line-height>]?<font-family>

#### 字体颜色：
	color:red;
	color:#ff0000;
	color:rgb(255,0,0);
	color:rgba(255.0.0.1)

### 对齐：
#### 水平对齐：左对齐/右对齐/居中/两端对齐 
	
	text-align:left|right|center|justify

#### 垂直对齐：
	
	vertical-align:baseline|sub|super|top|text-top|middle|bottom|text-bottom|<percentage>|<length>

#### 文本缩进：
	
	text-indent:<length>|<pencentage>
	
### 换行，空格，自动换行：

	white-space:normal|nowrap|pre|pre-wrap|pre-line
> normal： 换行符、空格合并，自动换行；
> nowrap：换行符、空格合并，不自动换行；
> pre：换行符、空格保留，不自动换行；
> pre-wrap：换行符、空格保留，自动换行；
> pre-line：换行符保留、空格/tab合并，自动换行；

### 单词换行：
	word-wrap:normal|break-word;
	word-break:normal|keep-all|break-all;

### 文字阴影：
	
	text-shadow:none|[<length>{2,3}&&<color>?]#;
	text-decoration:none|[underline||overline||line-through];

> 隐藏文本，且用省略号显示：
	
	text-overflow:ellipsis;
	overflow:hidden;
	white-space:nowrap;

### 定义鼠标形状：
	// 自定义|自适应|鼠标|消失|疑问|点击|放大|缩小|移动
	cursor:[<url>]*,[auto|default|none|help|pointer|zoom-in|zoom-out|move];

### 强制继承： inherit;

### 盒模型：
	// min-width,max-width用法一致；
	width:<length>|<percentage>|auto|inherit;
	// min-width,max-width用法一致；
	height:<length>|<percentage>|auto|inherit;
	
	// 顺序为 上，右，下，左
	padding:[<length>|<percentage>]{1,4};
	margin:[<length>|<percentage>|auto]{1,4};

	
#### 边框：
	border:[<border-width>||<border-style>||<border-color>];
	border-width:[<length>]{1,4};
	//             实线|虚线|点
	border-style:[solid|dashed|dotted]{1,4};
	border-color:[<color>|transparent]{1,4};

#### 圆角边框：
	border-radius:[<length>|<percentage>]{1,4}[/[<length>|<pencentage>]{1,4}]?
	//                  水平半径     /     垂直半径
	border-radius: 0px 5px 10px 15px/20px 15px 10px 5px;
	border-top-left-radius:5px;

### # 设置超出部分的样式：	
	//   overflow-x|overflow-y;
	//         可见|隐藏|出现滚动条|自动显隐滚动条；
	overflow:visible|hidden|scroll|auto;
#### 切换盒子模型模式：
	box-sizing:content-box|border-box|inherit;

#### 盒阴影：
	box-shadow:none|<shadow>[,<shadow>]*
	<shadow>:inset?&&<length>{2,4}&&<color>?
	//水平偏移 垂直偏移 模糊半径 阴影大小 阴影颜色；
	box-shadow:4px 6px 3px 3px red;
	//  内阴影
	box-shadow:inset 0px 0px 5px red;

#### 轮廓：
	outline:[<out-width>||<outline-style>||<outline-color>];
	outline-width:<length>;
	outline-style:solid|dashed|dotted;
	outline-color:<color>|invert;
> 与border值较为相似，但有两点区别：
> 1、border占据空间，而outline并不占据空间；
> 2、outline在border的外围；

### 背景(位于盒模型的最底层)：
	background-color:<color>；
	background-image:<bg-image>[,<bg-image>]*
	<bg-image> :<image>|none;
	
> Tips:
> 1、绝对路径：“http://....”;
> 2、相对路径，相对于根目录，CSS文件是相对于CSS文件，JS文件是相对于HTML的文件，`..`代表上一级："../images/x.png"；

#### 背景平铺方式：
	background-repeat:<repeat-style>[,>repeat-style>]*
	//             X方向重复|Y方向重复|重复|留空隙平铺|伸缩铺满
	<repeat-style>:repeat-x|repeat-y|[repeat|space|round|no-repeat]{1,2};
	//  x轴不重复，y轴重复
	background-repeat:no-repeat repeat;

#### 背景固定方式：
	background-attachment:<attachment>[,<attachment>]*
	//           背景随滚动而滚动|背景固定|继承
	<attachment>:scroll|fixed|inherit;

#### 背景图片位置：
	background-position:<position>[,<position>]*
	<position>:left|center|right|top|bottom|<percentage>|<length>

> background-position:10px 20px;   表示x轴偏移10px,y轴偏移20px；
> background-position:20% 50%;  表示相对于父元素的定位；
> background-position: center center;   图片居中；
> background-position:right;  水平靠右，垂直居中；
> background-position:right 10px top 20px;   与右边偏10px ,与上边偏20px；

#### 常用的背景属性：
	background:url("x.png") no-repeat 10px 10px;

#### 设置背景参照盒子：
	background-origin:<box>[,<box>]*
	<box>:border-box|padding-box|concent-box;

#### 设置背景平铺的范围：
	background-clip:<box>[,<box>]*
	<box>:border-box|padding-box|concent-box;

#### 改变背景图片的大小：
	background-size:<bg-size>[,<bg-size>]*;
	<bg-size>:[<length>|<percentage>|auto]{1,2}|cover|contain;

#### 组合：
	background:[<bg-image>||<position>[/<bg-size>]?||<repeat-style>||<attachment>||<box>||<box>]*<bg-layer>||<background-color>

	background:url("red.png") 0 0/20px 20px no-repeat,url("blue.png") 50% 50%/contain no-repeat content-box green;

#### 背景的线性渐变：
	linear-gradient( );
		[[<angle>|to <side-or-corner>],]?<color-stop>[,<color-stop>]+;
		<side-or-corner>:[left|right]||[top|bottom];
		<color-stop>:<color>[<percentage>|<length>]?
	
	background-image:linear-gradient(red,blue);
	background-image:linear-gradient(to top right,red,blue);
	background-image:linear-gradient(45deg,red,blue);
	background-image:linear-gradient(red,green 20%,blue);

#### 背景的径向渐变：
	background-image:radial-gradient(closest-side,red,blue);
	background-image:radial-gradient(circle,red,blue);
	background-image:radial-gradient(circle 100px,red,blue);
	background-image:radial-gradient(red,blue);
	background-image:radial-gradient(100px 50px,red,blue);
	
> 重复渐变：
> 
> background-image:repeating-linear-gradient( );
> 
> background-image:repeating-radial-gradient( );


### 布局：
### display:
	display:block|inline|inline-block|none;

> * #### display : block;
> 块级元素，包括`<div><p><h1>..<h6><ol><ul><dl><table><form>`...
> 特点：
> 1、占据整行；
> 2、 默认宽度为父元素宽度；
> 3、可设置宽高；

> * #### display:inline;
> 行内元素，包括`<span><a><i><em><label><cite>`...
> 1、宽度为内容宽度；
> 2、不可设置宽高；
> 3、同行显示；

> * #### display:inline-block;
> 内联块级元素，包括`<input><img><textarea><select><button>`...
> 1、默认宽度为内容宽度；
> 2、可设置宽高；
> 3、同行显示；
> 4、整块换行；

> * #### display: none;
> 元素不显示，且不占据空间；

> * #### display: hidden;
> 元素隐藏，但占据空间；

小技巧：

	// 块级元素水平居中：
	.div{margin:0 auto; width:900px;}

### position
	//     默认定位| 相对定位 | 绝对定位| 固定定位
	positon:static|relative|absolute|fixed；
	// 配合属性：
	top/right/bottom/left/z-index:

Tip：
1、如果同时设置了top与bottom，且没有设置高度时，则元素会被强制拉伸；如果设置了高，则不拉伸，显示为设定的高度；
2、z-index：设置层级，层级高的在顶层，会覆盖住层级低的元素。同时需要考虑父级元素的层级。

> * #### position:relative;
> 其主要的用途是元素设置后作为子元素绝对定位的参照物；
> 1、仍处于文档流中；
> 2、参照物为元素本身；

> * #### position:absolute;
> 绝对定位；
> 1、默认宽度为内容宽度；
> 2、脱离文档流；
> 3、参照物为父级第一个定位元素或者根元素；
> 用途：轮播图，定位....

> * #### position:fixed;
> 固定定位；
> 1、默认宽度为内容宽度；
> 2、脱离文档流；
> 3、参照物为窗口；
> 用途：固定顶栏，遮罩...

#### 三行自适应布局：
	.head{positon:absolute;top:0;left:0;width:100%;height:100px;}
	.body{positon:absolute;top:100px;left:0;bottom:100px;right:0;overflow:auto;}
	.foot{positon:absolute;bottom:0;left:0;width:100%;height:100px;}

### float:
	float:left|right|none|inherit;
> 1、默认宽度为内容宽度；
> 2、半脱离文档流；脱离元素文档流，但出于内容文档流中；
> 3、想指定方向一直移动；
> 4、设置了float属性的元素在同一文档流中；

#### 清楚浮动的影响：
1、空白元素：在浮动元素的后面添加一个空白元素；

	<br class="cf">
	.cf{clear:both;height:0;overflow:hidden;visibility:hidden;}
	
2、claearfix：为设置float属性的元素的父元素添加类名“clearfix”;
	
	.clearfix:after{content:".";display:block;clear:both;height:0;overflow:hidden;visibility:hidden;}
	.clearfix{zoom:1}
	

### 弹性flex布局
1、创建弹性容器flex container：(父级)
	
	.container{
		display:flex;
	}

2、为容器设置成弹性容器后，处于文档流中的直接子元素自动变成弹性元素flex item。弹性元素的横向排列方向为`main axis`，纵向为`cross axis`。

排列方向：（弹性容器）

	//           横正向|   横逆向   |纵正向 |     纵逆向
	flex-direction:row|row-recerse|column|column-reverse;

是否换行：（弹性容器）
	
	flex-wrap:nowrap|wrap|wrap-reverse；

> 当设置为不换行时，会对弹性元素进行弹性收缩。

上面两个属性可以合成为：（弹性容器）

	flex-flow:<flex-derection> || <flex-wrap>;

定义弹性元素的排列顺序，`order`值，大的排在后面；（弹性元素）

	order:<interger>;    // 整数，初始值为0；

#### 弹性属性：

设置弹性元素在main-axis上的宽度和高度；（弹性元素）

	flex-basis:main-size|<width>;

设置剩余空间的分配比例，如item1的 `flex-grow:1`；item2的`flex-grow:2`;则剩余的空间则按1/3,2/3分配给item1和item2；（弹性元素）

	flex-grow:<number>;    // 初始值为0；

设置当空间不足时，按比例收缩弹性元素，与`flex-grow`相对应；（弹性元素）

	flew-shrink:<number>;   // 初始值为1;

组合起来，可以设置为：（弹性元素）

	// 初始值为   0   1   main-size
	flew:<flew-grow>||<flex-shrink>||<flex-basis>

#### 对齐属性：
	
设置在主轴main-axis方向上的对齐方式。（弹性容器）

	justify-content:flex-start|flex-end|center|space-between|space-around;
	
> `flex-start`：左对齐；
> `flex-end`：右对齐；
> `center`：居中；
> `space-between`：中间间隔平分；
> `space-around`：首尾也分到间隔（当都设置宽度，且有剩余空间时）；

设置辅轴cross-axis上的对齐（但弹性元素高度不一的时候）：（弹性容器）

	align-items:flex-start|flex-end|center|baseline|strech;

> `flex-start`：上对齐；
> `flex-end`：下对齐；
> `center`：居中；
> `baseline`：内容对齐；
> `strech`：拉伸对齐；

设置单个弹性元素在cross-axis上的对齐属性：（弹性元素）

	align-self:auto|flex-start|flex-end|center|space-between|space-around|stretch;
	
设置cross-axis方向上行对齐的方式：（弹性容器）

	align-content:auto|flex-start|flex-end|center|space-between|space-around|stretch;
	


	

