# HTML学习笔记

## 一、发展史：

## 二、文档：
	<!DOCTYPE html>  // 以H5解析文档；
	<html>
		<head></head>
		<body></body>
	</html>

### 文档头部内容：
	<head>
		// 定义文档编码类型；
		<meta charset = "utf-8" /> 
		<title>网页标题</title>
		// 描述网页的基本信息，有利于SEO；
		<meta name = "keywords" concent = "..." />
		<meta name = "description" content = "..." />
		// 移动端使用，定义viewport;
		<meta name = "viewport" content = "width = device-width" />
		// 定义标题前的图标，网页的icon；
		<link rel = "shortCut icon" href = "***.ico" />
		// 外联CSS样式；
		<link rel = "stylesheet" href = "***.css" />
		// 内联样式；
		<style></style>
	</head>

###  文档主体：
####  语法： 小写、属性双引号、嵌套缩进；
#### 常用属性：id/class/style/title （每个标签都有）；

##  三、标签：

### 1、文档章节：
	<body>/<header>/<nav>/<aside>/<article>/<section>/<footer>/<hx>

### 2、文本内容：

#### 超链接：
  `<a href = "http://...." target = "_self/_blank/inner"></a>`
  
* href中直接填入文件，则为下载；
* href中填入“#pay”，则为跳转到id = ”pay“的页面锚点；
* href中填入 “mailto: emailAdress@qq.com” ，则可以直接链接到email；
* href中填入“tel:111111111”，可链接到tel；

#### 强调： `<em>斜体</em> <strong>粗体</strong>`

#### 区分样式： `<span>span</span>`

#### 换行： `<br/>`

#### 引用： `<cite>引用</cite> <q>引用</q> <blockquote>块状引用</blockquote>`

#### 代码： `<code>代码</code>`

#### 格式化： `<b></b> <i></i>`

#### 块： `<div>块</div>`

#### 段落：`<p>段落</p>`

#### 列表：
	// 无序列表； 
	<ul>
		<li>苹果</li>
		<li>西瓜</li>
		<li>梨</li>
	</ul>
	// 有序列表； 
	<ol type = "a" start = "2">
		<li>冠军</li>
		<li>亚军</li>
		<li>季军</li>
	</ol>
	// 定义列表；
	<dl>
		<dt>标题</dt>
		<dd></dd>
		<dd></dd>
	</dl>

#### 将特殊符号转义成实体字符：`<pre>></pre>`

#### 图片：
	<img src = "" alt = "" />

#### 外部资源嵌入：
	// 可嵌入多种对象；
	<object type = "">
		<param name = "" value = "" />
	</object>
	
	// 嵌入多种多媒体内容；
	<embed type = " " src = " " width="" height="" />

#### 视频：
	//    自动播放 循环   控制器  封面
	<video auto   loop  controls poster="">
		// 视频资源
		<source src=" " type="" />
		<source src=" " type="" />
		// 引入字幕；
		<track kind="subtitles" src="" srclang="cn" label="cn" />
	</video>

#### 音频：（与video一致）
	<audio>
		<source src="">
	</audio>

#### 图形图像：`<canvas> <svg>`

#### 热点区域：
	<img src="" alt="" width="" height="" usemap="#Map">
	// map标签中的name属性与img标签中的usemap一致；
	<map name="Map" data-ganame="" data-gapoint="">
		// 热点区域   形状     坐标      链接     打开方式
		<area shape="rect" coords="" href="" target="" />
	</map>

#### 表格：
	<table>
		<caption>标题</caption>
		// 表头
		<thead>
			//         跨2列
			<tr><th  colspan="2">表头</th></tr>
		</thead>
		<tbody>
			<tr>
				//     跨2行
				<th rowspan="2">行头</th>
				<td>单元格</td>
			</tr>
		</tbody>
		<tfoot>
			<tr><td></td></tr>
		</tfoot>
	</table>

#### 表单：
	//       提交到文件           提交方式
	<form action="/login.php" method="post">
		// 分区
		<fieldset>
			<legend>区域标题</legend>
			<label for="" />
		</fieldset>
		// 上传文件
		<input type="file" name="file" />
		// 单选框
		<input type="checkbox" name="size" value="5" />
		// 多选框
		<input type="radio" name="material" value="5" />
		// 文本输入
		<input type="text" name="description" placeholder="提示"  value="初始化内容" />
		// 提交，重置按钮
		<div>
			<button type="submit">提交</button>
			<button type="reset">重置</button>
		</div>
		// 下拉选择框
		<label for="delivery">配送方式:</label>
		<select id="delivery">
			// 选项组
			<optgroup label="组标题">
								// 默认选中
				<option value="0" selected>快递</option>
				<option value="1">面取</option>
			</optgroup>
		</select>
		// 多行文本框
		<textarea name="text" rows="4" id=""></textarea>

#### HTML5新命令：

input标签的type属性 = email/url/number/tel/search/range/color/data picker....

#### 实体字符转义：

> 空格 ===>` &nbsp;`
>` "`       ===> `&quot;`
>`>` ===> `&gt;`
>`<` ===> `&lt;`
> `@`===> `&copy;`
> `&`===> `&amp;`
				

