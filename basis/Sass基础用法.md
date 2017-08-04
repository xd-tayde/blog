#Sass基础用法

### 1、变量：

Sass中的变量需要以 `$`开头进行命名，如 `$blue`;如果需要插入在字符串中，则需使用`#{ }`进行包裹；如：`border-#{$de}-color`;

### 2、计算：

Sass允许在语句中直接使用算式记性计算，如：`right: $var * 10;`

### 3、嵌套：

Sass可以使用语法的嵌套，可以使代码的可读性更强。如：

(1) 标签嵌套	

	div{
		h1{ };
	}

(2)属性嵌套：

	p{
		border:{
			color:red;
		}
	}

> TIPS：
> 在嵌套的代码中，可以使用`&`来引用父级，如
		
	a{
		&:hover{ }
	}

### 4、注释：

`/* 注释 */`：普通注释，会保留到编译后的文件中，如果使用压缩模式编译，则会省略；

`// 注释`：只保留在Sass源文件中，编译后则省略；

`/*! 重要注释 */`：压缩模式下编译也会保留；

### 5、继承：

使用 `@extend` 可实现继承：

	.class1{ };
	.class2{
		@extend class1;
		font-size:14px;
	}

### 6、重用代码块：

使用 `@mixin`进行代码块的定义，使用`@include`进行已定义的代码块的调用；

	@mixin left{
		float:left;
		margin-left:10px;
	}
	
	div{
		@include left;
	}

在代码块的定义中，也可以传入参数和默认值：

	@mixin left($value:10px){
		float:left；
		margin-right:$value;
	}
	// 调用；
	div{
		@include left(20px);
	}

### 7、插入文件：

可以使用 `@import`进行文件的引入：

	@import "path/filename.scss";

### 8、条件判断：

使用 `@if/@else`写条件语句；

	p{
		@if 1+1 == 2{
			width:10px;
		}@else{
			width:20px;
		}
	}

### 9、循环语句：

for循环：

	@for $i from 1 to 10{
		.border-#{$i}{
			border:#{$i}px solid blue;
		}
	}

while循环：
	
	$i:6;
	@while $i>0{
		.item-#{$i}{
			width:2em * $i;
		}
		$i:$i-1;
	}

each语句：

	@each $menber in a,b,c,d{
		.#{$menber}{
			background:blue;
		}
	}

### 10、自定义函数：

	@function double($n){
		@return $n * 2;
	}

	#sidebar{
		width:double(5px);
	}

