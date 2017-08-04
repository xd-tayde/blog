# JavaScript语言编程核心（二）--- 代码的复用模式

在大多数编程语言中，代码复用都是很重要的一块内容，而代码的继承性，是代码复用的一种重要形式，可以显著地减少开发成本，提高开发效率。继承的方式可以大致的分成两类：1、传统的类式继承；2、现代化继承方式：包括原型继承、复制属性继承、混入、借用、借用与绑定。

## 构造函数

在JavaScript中，构造函数其实也是一类普通函数，但它有几个特点：

> 1、函数名首字母大写，以区分普通函数；
> 2、通过this创建新的属性和方法；
> 3、配合new操作符，可创建出一个新对象；

#### new操作符

new操作可以配合函数，返回一个对象，实际上它是做了三件事情：

> 1、创建一个新对象； `var _obj = {}`；
> 2、使`_obj`继承该函数的原型；`obj.__proto__ = CO.prototype`;
> 3、执行函数中的语句，为this指向添加属性、方法；
> 4、返回该新对象；
> 
> **Tips：**
> 如果函数中有主动return一个对象，则返回该对象，否则隐式返回`_obj`;
> 如果忘记使用new，则会造成在全局中添加属性和方法，这是很糟糕的。因此不要忘记new！
> 为了避免当忘记new时，不会出现意外，便出现了一种模式，称为 **自调用构造函数**，它可以使得  `var a = new CO()`与`var a = CO()`效果一致；

	function Person(){
		if(!(this instanceof Person)){
			return new Person();
		}
		this.name = "dong";
	}
	Person.prototype.sayName = function(){
		alert(this.name);
	}


## 类式继承

实现类式继承的目标，是通过构造函数Child( )来获取一个父类构造函数Parent( )的属性，然后再由new Child( )实例化对象，实现继承。因此，关键的步骤就是 `inherit(Child,Parent)`;

这里，有多种模式可以完成这样的继承函数：

### 1、默认模式：
	
	function inherit(c,p){
		c.prototype = new P();
	}

> Tips:
> 对象指向原型，使用对象内部的`__proto__`属性，指向一个实例对象。但该属性不符合W3C标准，不能使用，但可以让我们更好的理解JS的原型继承的概念；
> 构造函数指向原型，使用`prototype`属性，也是指向一个实例对象；
> 通过默认模式完成的继承，属性与方法并不在自身上，只是通过原型链查找到的，仅仅是获取到引用，而且无法通过子构造函数传参。

### 2、借用构造函数：
	
	function Child(a,b,c,d){
		Parent.apply(this,arguments);
	}

该方法是只可以继承父级构造函数中添加到this的属性，原型中的属性并不能继承。但相比于第一种默认方式，它继承的属性都变成了自身属性，即hasOwnProperty为true，而不是从原型链上查找到的属性。

> 通过该模式，可以实现多重继承，即同时继承多个父级函数的属性；

	function Child(){
		Father.apply(this,arguments);
		Mother.apply(this,arguments);
	}

> 该模式的优点是，继承后的子级是父级函数的真实副本，操作子级函数并不会影响或覆盖父级函数。
> 而缺点，便是无法从原型中继承属性，而是为每个实例都初始化各自的方法属性。

### 借用与设置原型

为了解决借用模式的缺点，我们可以为其设置原型来解决。

	function Child(){
		Parent.apply(this,arguments);
	}
	Child.prototype = new Person();

这样既能得到父级函数中属性的真是副本，又可以继承到原型上的方法。但仍然是有缺点，便是该模式调用了两次`Parent`构造函数，属性被继承了两次，导致了效率的低下。


### 共享原型

该模式有点类似于第一种默认方式，但是不同点在于，将子函数的原型直接指向父级构造函数的原型。

	function inherit(c,p){
		c.prototype = p.prototype;
	}

只要将要复用的属性，方法都放在父级的原型里，即可以达到继承的目的。但是该模式的缺点在于，当修改了继承的子级的原型时，便直接影响到了该类中的所有对象。

### 代理构造函数模式

通过创建一个代理构造函数，链接父级函数的原型与子级函数，从而避免了共享原型所带来的弊端，而且可以利用原型链的优势。

	function inherit(c,p){
		var F = function(){};
		F.prototype = p.prototype;
		c.prototype = new F();
	}

> **Tips：**
> 该模式不会继承父级构造函数上this的属性，只会继承父级原型上的属性。
> 更近一步，可以将父级构造函数和其原型的存为子级构造函数的一个指针，以备不时之需；
	
	c.uber = p.prototype;
	c.prototype.constructor = c;

### 综上可以整理出最后，也是JS中较为完美的类式继承模式，也称为 圣杯模式；

最后一点优化，便是使用即时执行函数和闭包，优化每次都需要创建一个新的代理函数的缺点。

### 圣杯模式

	var inherit = (function(c,p){
		var F = function(){};
		return function(c,p){
			F.prototype = p.prototype;
			c.prototype = new F();
			c.uber = p.prototype;
			c.prototype.constructor = c;
		}
	})();

### Klass语法糖

	var klass = function(Parent,props){
		var Child,F,i;
		// 1.
		// 新构造函数；
		Child = function(){
			if(Child.uber && Child.uber.hasOwnProperty("__construct")){
				Child.uber.__construct.apply(this,arguments);
			}
			if(Child.prototype.hasOwnProperty("__construct")){
				Child.prototype.__construct.apply(this,arguments);
			}
		}

		// 2.
		// 继承;
		Parent = Parent || Object;
		F = function(){};
		F.prototype = Parent.prototype;
		Child.prototype = new F();
		Child.uber = Parent.prototype;
		Child.prototype.constructor = Child;

		// 3.
		// 添加实现方法：
		for(i in props){
			if(props.hasOwnProperty(i)){
				Child.prototype[i] = props[i];
			}
		}

		// 4.
		// 返回该Child;
		return Child;
	}

 
## 现代化继承方式

### 1、原型继承：

对象可以通过为它指定原型对象而完成继承，这样的继承称为**委托继承**，也可以称为**原型继承**。原型对象也是一个简单的对象，此时也可以继续为它指定它的原型对象而完成继承。此时，由 实例对象 ---> 原型对象 ---> 原型对象...，这样构建出来的一条线，便称为原型链。

> **原型链**：是由原型对象组成，是一个用来实现继承和共享属性的有限的对象链。

> **属性查找机制**：当查找对象的属性时，如果实例对象自身不存在该属性，则沿着原型链往上一级查找，找到时则输出，不存在时，则继续沿着原型链往上一级查找，直至最顶级的原型对象Object.prototype，如还是没找到，则输出undefined；

> **属性修改机制**：只会修改实例对象本身的属性，如果不存在，则进行添加该属性，如果需要修改原型的属性时，则可以用：`b.prototype.x = 2`；但是这样会造成所有继承于该对象的实例的属性发生改变。

1、通过`__proto__`来实现原型继承，从测试代码可以看出，b对象继承了a对象的x属性，同时a对象继承于顶级的原型对象 Object.prototype。因此则组成了一条原型链：   b --->  a ---> Object.prototype;
	
	// 
	var a = {x:1};
	var b = {};
	b.__proto__ = a;
	console.log(b.x);   // 1；
	console.log(a.__proto__ == Object.prototype);   // true;
	console.log(b.__proto__ == Object.prototype);   // false;

2、第二种方式可以使用ES5中的Object.create( )方法来实现原型的继承：
也就是可以通过给一个函数F，直接通过`prototype`指定一个实例对象作为原型，完成继承。
	
	// 低版本浏览器的兼容；
	if(!Object.create){
		Object.create = function(o){
			if(arguments.length>1){
				throw new Error("只接受第一个参数");
			}
			function F(){};   // 创建一个新对象；
			F.prototype = o;   // 将新对象的原型对象指向传入对象；
			return new F();   // 返回该对象的实例； 
		}
	}

### 2、通过复制属性实现继承

这种模式中，对象将从另一个对象中以直接复制的形式继承属性，可以由一个extend( )实现复制继承。
	
**浅复制**：如果对象中的属性是对象，那只是简单的进行引用的复制。会导致修改子对象的对象属性，则父对象的属性也会被修改；
	
	function extend(parent,child){
		var i;
		child = child || {};
		// 遍历父对象的属性，判断是否为自身属性；
		for(i in parent){
			if(parent.hasOwnProperty(i)){
				child[i] = parent[i];
			}
		}
		return child;
	}

**深复制**：以浅复制的形式，增加判断，当遇到属性是对象时，则递归进行更深一步的复制；

	function extendDeep(parent,child){
		var i,
			toStr = Object.prototype.toString;
		child = child || {};
		// 遍历父对象属性，先判断是否为自身属性，后属性值判断是否为对象；
		for(i in parent){
			if(parent.hasOwnProperty(i)){
				if(typeof parent[i] === "object"){
					child[i] = (toStr.call(parent[i]) === "[object Array]"?)[]:{};
					extendDeep(parent[i],child[i]);
				}else{
					child[i] = parent[i]''
				}
			}
		}
		return child;
	}

### 3、混入模式

所谓的混入模式，就是将多个对象进行合并，组成一个新对象，该对象拥有对象的所有属性，可以建立一个`mix`函数进行混入，也仅为浅复制模式；

	function mix(){
		var i,
			prop,
			child={};
		for(i=0;i<arguments.length;i++){
			for(prop in arguments[i]){
				if(arguments[i].hasOwnProperty(prop)){
					child[prop] = arguments[i][prop];
				}
			}
		}
		return child;
	}

### 4、借用方法

有时，并不需要达到继承，只是需要借用一两个方法。该情况下，可以使用apply( )与call( )方法。这两个函数可以改变函数中的this指向，两者的区别在于传入的第二个参数。
	
	parent.doSomething.call(child,p1,p2,p3);
	parent.doSomething.apply(child,[p1,p2,p3]);

### 5、借用和绑定

有时仅仅使用借用方法，会导致一些意想不到的错误，如：

	var sayName = one.say;
	sayName('ho!')；  // 由于this的指向全局对象，会导致出错；

此时，便需要有个绑定的方法来将一个对象与一个方法绑定：

	function bind(o,m){
		return function(){
			// slice方法不传参数时，将伪数组转换成数组；
			return m.apply(o,[].slice.call(arguments));
		};
	}

> 该方法的代价便是额外的闭包开销；

