# JavaScript语言编程核心（一）--- 对象

##对象

这是系列关于js语言核心思想的一些理解，包括对象、继承、原型链、构造函数、执行上下文堆栈、执行上下文、变量对象、活动对象、作用域链、闭包以及this。

本文参考自《JavaScript权威指南》、《JavaScript语言精粹》以及前端早读君的文章《javaScript核心》。

## 对象
在JavaScript中，对象是指可变的键控集合。属性名为字符串，值可以为任何类型的javascript值，每个属性都是一个名/值对。除了可以拥有属性外，对象还可以从一个称为原型的对象中继承属性。这就是JavaScript中的 **“原型式继承”**。

#### 1、属性特性：
在ES5中，属性还包括了一些标识可写、可枚举和可配置的特性。可以通过这些API给原型对象添加方法，并设置成不可枚举，让它们更像内置方法，也可以给对象定义不能修改或删除的属性，“锁定”该对象。
获取和定义可使用 **getOwnPropertyDescriptor 和 defineProperty ** 方法：

	var o = {};
	Object.defineProperty(o,"x",{
		value:2,
		writable:true,   // 可写性；
		enumerable:false,   // 可枚举性；
		configurable:true     //  可配置性;
	});
	console.log(Object.getOwnPropertyDescriptor(o,"x"));
	console.log(o.x);
	
> * 可写，表明是否可以设置改属性的值；
> * 可枚举，表明是否可以通过for/in循环返回；
> * 可配置，表明是否可以删除或修改该属性；

#### 2、对象特性：
> * 对象的原型属性（prototype）通过隐含的`__proto__`属性指向另一个对象，可以继承原型对象的属性；

	Object.getPrototypeOf( )   // 可查询对象的原型；
	p.isPrototypeOf(o)   // 来检测p是否是o的原型；
	
> * 对象的类（class）是一个表示对象类型的字符串，用以表示对象的类型信息，一般用toString( )来获取：[Object *class*]。
> 下面是封装好的 classof函数，用来检测对象类型；

	function classof(o){
		if(o === null){return "Null"};
		if(o === undefined){return "Undefined"};
		return Object.ptototype.toString.call(o).slice(8,-1);
	}

> * 对象的扩展标记（extensible flag）指明了是否可以向该对象添加新属性；ES5中，所有的内置对象和自定义对象都是显式可扩展的，宿主对象的可扩展性是依据引擎所定义的；

	Object.esExtensible( );   // 判断对象是否可扩展；
	Object.preventExtensions( ); // 将对象设置为不可拓展并且无法再转换回可拓展；但同样可以继承属性；
	Object.seal( );   // 封闭对象；将对象设置为不可拓展，而且将所有自有属性设置为不可配置；
	Object.isSealed( );  // 检测对象是否被封闭；
	Object.freeze( );   // 更严格的冻结对象，不可拓展，配置，且只读；
	Object.isFrozen( );   // 检测对象是否被冻结；

#### 3、对象的分类：
> * 内置对象：JS内部定义的对象或类，如数组（Array）、函数（Function）、日期（Date）和正则表达式（RegExp）等；
> * 宿主对象：由js解释器所嵌入的宿主环境中定义的对象，如浏览器客户端；
> * 自定义对象：在JS代码中创建的对象；

#### 4、对象属性的分类：
>* 自有属性：在对象内部定义的属性；
>* 继承属性：从原型对象中继承得到的属性；

#### 5、 对象的创建：
>* 对象字面量法：
>一个对象字面量就是包围在一对花括号中的若干个名/值对组成的映射表，名与值之间用`:`分隔，名值对间用`,`分隔。该方法的`__proto__`指向Object.prototype.

	// 注意，最后一个名/值对后不再添加逗号；
	var student = {
		"name":"Tayde",
		"age":26
	}
>* new操作符+构造函数：
>通过new操作符后跟一个函数，初始化创建出一个新的对象，该方法的`__proto__`指向构造函数.prototype。
	
	var o = new Object();  // 与 var o = { }效果一样；

>* ES5中的Object.create( ):
>该方法可填两个参数，第一个填对象原型，第二选填自定义的属性；`__proto__`指向函数中填入的对象。

	var o1 = Object.create({x:1,y:2});  // 则o1对象已经继承了x,y属性；

#### 6、对象属性的查询和设置：
>* `.`运算符，当属性名为一个简单的标识符时，一个静态值时使用；优先考虑，因为它有更好的可读性；
>* `[ ]`运算符，该方法更全面，当属性名为变量或动态值时使用；
	
	var name = student.name; 
	var age = student["age"];
	
> Tips: 
> 1、当属性不存在时，会返回 undefined，并不会报错；
> 2、但当查询属性的对象不存在时，则会报错；解决方法：

	var len = book && book.subtitle && book.subtitle.length;
	
> 3、可以用 `||`运算符来填充默认值；

	var sex = student.sex || "unknown";

#### 7、对象的引用：
对象通过引用来传递，他们永远不会被复制。
对象并没有被复制，而是仅仅将x,y指向同一个对象的引用而已。

	var x = {a:1};
	var y = x;
	y.a = 2;
	console.log(x.a);   // 2;
	
#### 8、对象属性的删除：

	delete student.name;
	delete student["age"];

> Tips：
> delete操作符只能删除自身属性，无法删除原型上的继承属性；
> delete操作符值是断开属性和宿主对象的联系，并不是删除属性，因此外部的引用仍会存在；

	var a = {x:{y:1}};
	var b = a.x;
	delete a.x;
	console.log(b);    // {y:1};
	console.log(a.x);  // undefined;

#### 9、检测属性与对象的从属关系：

> 1、in运算符：

	var o ={x:1};
	console.log("y" in o);   // false;

> 2、hasOwnProperty( )：检测是否为自有属性，如果是继承属性，则返回false；

	var o = {x:1};
	console.log(o.hasOwnProperty("x"));  // true;
	console.log(o.hasOwnProperty("toString"));  // false;

> 3、propertyIsEnumerable( ): 属于hasOwnProperty的增强版，只有当属性为自有属性且可枚举性时，才返回true;

	var o = {x:1};
	console.log(o.propertyIsEnumerable("x"));  // true;
	console.log(o.hasOwnProperty("y"));  // false;
	console.log(o.hasOwnProperty("toString"));  // false;

> 4、直接与undefined比较；

	var o = {x:1};
	o.x !== undefined;    // true;
	o.y !== undefined;    // false;

####  10、对象属性的getter和setter:
在ES5中，对象的属性值可以用一个或两个方法替代，他们就是getter和setter。由这两个方法定义的属性称作“存取器属性”。

	var serialnum = {
		"$n":0,
		get next(){
			return this.$n++;
		},
		set next(){
			if(n>=this.$n){this.$n = n;}else{throw "新值必须大于当前值。"}
		}
	}

####  11、序列化对象：
对象序列化是指将对象的状态转换成字符串，也可以将字符串转换为对象。使用ES5中的 JSON.stringify( ) 和 JSON.parse( )。

> 1、NaN、Infinity和-Infinity序列化后的结果是Null;
> 2、函数、RegExp、Error对象和undefined值不能序列化和还原；
> 3、JSON.stringify( ) 只能序列化对象可枚举的自有属性；

#### 12、对象方法：

> 1、toString( )：无需参数，返回一个该对象的字符串。
> 2、toLocaleString( )：返回一个表示这个对象的本地化字符串。
> 3、toJSON( )：
> 4、valueOf( ):
