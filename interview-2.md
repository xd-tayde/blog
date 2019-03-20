# (下)中高级前端大厂面试秘籍，寒冬中为您保驾护航，直通大厂

> 感恩!~~没想到上篇文章能这么受大家的喜欢，激动不已。🤩。但是却也是诚惶诚恐，这也意味着责任。下篇许多知识点都需要比较深入的研究和理解，博主也是水平有限，担心自己无法承担大家的期待。不过终究还是需要摆正心态，放下情绪，一字一字用心专注，不负自己，也不负社区。与各位小伙伴相互学习，共同成长，以此共勉！
> 
> 最近业务繁忙，精力有限，虽然我尽量严谨和反复修订，但文章也定有疏漏。上篇文章中，许多小伙伴们指出了不少的问题，为此我也是深表抱歉，我也会虚心接受和纠正错误。也非常感激那么多通过微信或公众号与我探讨的小伙伴，感谢大家的支持和鼓励。

## 引言

本篇文章会继续沿着上篇的脚步，继续梳理前端领域一些比较主流的技术知识点。同样也继续坚持之前的理念，并不去展开深挖，而是能让大家在横向层面有个大概的了解和概念，在面试时有限的时间，更有利于去快速 focus 重点和与面试官交流。

下篇的知识点都属于比较进阶的，属于加分项。如果能在面试时从容侃侃而谈，想必面试官会记忆深刻，为你折服的~🤤

另外，在上篇文章有许多童鞋提到: 面试造火箭，实践全不会，对这种应试策略表达一些担忧。其实我是觉得 面试或者这些知识点，也仅仅是个 **开始**。能帮助在初期的快速成长，但这种策略并没办法让你达到更高的水平，只有后续不断地真正实践和深入研究，才能突破自己的瓶颈，继续成长。进入公司，不也只是一个开始而已嘛。~😋

建议各位小伙从基础入手，先看 [面试上篇](https://github.com/xd-tayde/blog/blob/master/interview-1.md)。

> Tips: 组里面试官小伙伴说: 这你都出秘籍了，看来我得改变下策略了~hahahahah。。🤪。

## 进阶知识

## 框架: React

React 也是现如今最流行的前端框架，也是很多大厂面试必备。React 与 Vue 虽有不同，但同样作为一款  UI 框架，在一些理念上还是有相似的，可能实现可能不一样，例如数据驱动、组件化、虚拟 dom 等。这里就主要列举一些 React 中独有的概念。

### 1. Fiber

React 的核心流程可以分为两个部分: 

- reconciliation (**调度算法**，也可称为 render):
	- 更新 state 与 props；
	- 调用生命周期钩子；
	- 生成 virtual dom；
	- 通过新旧 vdom 进行 diff 算法，获取 vdom change；
	- 确定是否需要重新渲染
- commit:
	- 如需要，则操作 dom 节点更新；

要了解 Fiber，我们首先来看为什么需要它？

- **问题**: 随着应用变得越来越庞大，整个更新渲染的过程开始变得吃力，大量的组件渲染会导致主进程长时间被占用，导致一些动画或高频操作出现卡顿和掉帧的情况。而关键点，便是 **同步阻塞**。在之前的调度算法中，React 需要实例化每个类组件，生成一颗组件树，使用 **同步递归** 的方式进行遍历渲染，而这个过程最大的问题就是无法 **暂停和恢复**。

- **解决方案**: 解决同步阻塞的方法，通常有两种: **异步** 与 **任务分割**。而 React Fiber 便是为了实现任务分割而诞生的。

- 简述: 在 React V16 将调度算法进行了重构， 将之前的 stack reconciler 重构成新版的 fiber reconciler，变成了具有链表和指针的 **单链表树遍历算法**。通过指针映射，每个单元都记录着遍历当下的上一步与下一步，从而使遍历变得可以被暂停和重启，这里我理解为是一种 **任务分割调度算法**，主要是将原先同步更新渲染的任务分割成一个个独立的 **小任务单位**，根据不同的优先级，将小任务分散到浏览器的空闲时间执行，充分利用主进程的事件循环机制。

- 核心: 
	- Fiber 这里可以具象为一个 **数据结构**:
	
	```js
	class Fiber {
		constructor(instance) {
			this.instance = instance
			// 指向第一个 child 节点
			this.child = child
			// 指向父节点
			this.return = parent
			// 指向第一个兄弟节点
			this.sibling = previous
		}	
	}
	```
	
	- **链表树遍历算法**: 通过 **节点保存与映射**，便能够随时地进行 停止和重启，这样便能达到实现任务分割的基本前提；
		- 1、首先通过不断遍历子节点，到树末尾；
		- 2、开始通过 sibling 遍历兄弟节点；
		- 3、return 返回父节点，继续执行2；
		- 4、直到 root 节点后，跳出遍历；
	 
	- **任务分割**，React 中的渲染更新可以分成两个阶段:
		- reconciliation 阶段: vdom 的数据对比，是个适合拆分的阶段，比如对比一部分树后，先暂停执行个动画调用，待完成后再回来继续比对。
		- Commit 阶段: 将 change list 更新到 dom 上，不适合拆分，因为使用 vdom 的意义就是为了节省传说中最耗时的 dom 操作，把所有操作一次性更新，如果在这里又拆分，那不是又懵了么。🙃

	- **分散执行**: 任务分割后，就可以把小任务单元分散到浏览器的空闲期间去排队执行，而实现的关键是两个新API: `requestIdleCallback` 与 `requestAnimationFrame`
		- 低优先级的任务交给`requestIdleCallback`处理，这是个浏览器提供的事件循环空闲期的回调函数，需要 pollyfill，而且拥有 deadline 参数，限制执行事件，以继续切分任务；
		- 高优先级的任务交给`requestAnimationFrame`处理；

	```js
	// 类似于这样的方式
	requestIdleCallback((deadline) => {
	    // 当有空闲时间时，我们执行一个组件渲染；
	    // 把任务塞到一个个碎片时间中去；
	    while ((deadline.timeRemaining() > 0 || deadline.didTimeout) && nextComponent) {
	        nextComponent = performWork(nextComponent);
	    }
	});
	```

	- 优先级策略: 文本框输入 > 本次调度结束需完成的任务 > 动画过渡 > 交互反馈 > 数据更新 > 不会显示但以防将来会显示的任务 

> Tips: Fiber 其实可以算是一种编程思想，在其它语言中也有许多应用(Ruby Fiber)。当遇到进程阻塞的问题时，**任务分割**、**异步调用** 和 **缓存策略** 是三个显著的解决思路。

### 2. 生命周期

在新版本中，React 官方对生命周期有了新的 **变动建议**:

- 使用`getDerivedStateFromProps` 替换`componentWillMount`；
- 使用`getSnapshotBeforeUpdate `替换`componentWillUpdate`；
- 避免使用`componentWillReceiveProps`；

其实该变动的原因，正是由于上述提到的 Fiber。首先，从上面我们知道 React 可以分成 reconciliation 与 commit 两个阶段，对应的生命周期如下:

- **reconciliation**:
	- `componentWillMount`
	- `componentWillReceiveProps`
	- `shouldComponentUpdate`
	- `componentWillUpdate`

- **commit**:
	- `componentDidMount`
	- `componentDidUpdate`
	- `componentWillUnmount`

在 Fiber 中，reconciliation 阶段进行了任务分割，涉及到 暂停 和 重启，因此可能会导致 reconciliation 中的生命周期函数在一次更新渲染循环中被 **多次调用** 的情况，产生一些意外错误。

新版的建议生命周期如下:

```js
class Component extends React.Component {
  // 替换 `componentWillReceiveProps` ，
  // 初始化和 update 时被调用
  // 静态函数，无法使用 this
  static getDerivedStateFromProps(nextProps, prevState) {}
  
  // 判断是否需要更新组件
  // 可以用于组件性能优化
  shouldComponentUpdate(nextProps, nextState) {}
  
  // 组件被挂载后触发
  componentDidMount() {}
  
  // 替换 componentWillUpdate
  // 可以在更新之前获取最新 dom 数据
  getSnapshotBeforeUpdate() {}
  
  // 组件即将销毁
  componentWillUnmount() {}
  
  // 组件已销毁
  componentDidUnMount() {}
  
  // 组件更新后调用
  componentDidUpdate() {}
}
```

- **使用建议**:
	- 在`constructor`初始化 state；
	- 在`componentDidMount`中进行事件监听，并在`componentWillUnmount`中解绑事件；
	- 在`componentDidMount`中进行数据的请求，而不是在`componentWillMount`；
	- 需要根据 props 更新 state 时，使用`getDerivedStateFromProps(nextProps, prevState)`，需要使用一个 state 来保存旧 props 以便比较，例如:
	
	```js
	public static getDerivedStateFromProps(nextProps, prevState) {
		// 当新 props 中的 data 发生变化时，同步更新到 state 上
		if (nextProps.data !== prevState.data) {
			return {
				data: nextProps.data
			}
		} else {
			return null1
		}
    }
	```
	
	- 可以在`componentDidUpdate`监听 props 或者 state 的变化，例如:

	```js
	componentDidUpdate(prevProps) {
		// 当 id 发生变化时，重新获取数据
		if (this.props.id !== prevProps.id) {
			this.fetchData(this.props.id);
		}
	}
	```
	
	- 在`componentDidUpdate`使用`setState`时，必须加条件，否则将进入死循环；
	- `getSnapshotBeforeUpdate(prevProps, prevState)`可以在更新之前获取最新的渲染数据，它的调用是在 render 之后， mounted 之前；
	- `shouldComponentUpdate`: 默认每次调用`setState`，一定会最终走到 diff 阶段，但可以通过`shouldComponentUpdate`的生命钩子返回`false`来直接阻止后面的逻辑执行，通常是用于做条件渲染，优化渲染的性能。

### 3. setState

React 中用于修改状态，更新视图层。具有以下特点:

> **事务** (Transaction): 是 React 中的一个调用结构，用于包装一个方法，结构为: initialize - perform(method) - close。通过事务，可以统一管理一个方法的开始与结束；处于事务流中，表示进程正在执行一些操作；

- **异步与同步**: `setState`并不是单纯的异步或同步，这其实与调用时的环境相关:
	- 在 **合成事件** 和 **生命周期钩子(除 componentDidUpdate)** 中，`setState`是"异步"的；
		- **原因**: 因为在`setState`的实现中，有一个判断: 当更新策略正在事务流的执行中时，该组件更新会被推入`dirtyComponents`队列中等待执行；否则，开始执行`batchedUpdates`；
			- 在生命周期钩子调用中，更新策略都处于更新之前，组件仍处于事务流中，而`componentDidUpdate`是在更新之后，此时组件已经不在事务流中了，因此则会同步执行；
			- 在合成事件中，React 是基于 **事务流完成的事件委托机制** 实现，也是处于事务流中；
		- **问题**: 无法在`setState`后马上从`this.state`上获取更新后的值。
		- **解决**: 如果需要马上同步去获取新值，`setState`其实是可以传入第二个参数的。`setState(updater, callback)`，在回调中即可获取最新值；
	- 在 **原生事件** 和 **setTimeout** 中，`setState`是同步的，可以马上获取更新后的值；
		- 原因: 原生事件是浏览器本身的实现，与事务流无关，自然是同步；而`setTimeout`是放置于定时器线程中延后执行，此时事务流已结束，因此也是同步；

- **批量更新**: 在 **合成事件** 和 **生命周期钩子** 中，`setState`更新队列时，存储的是 **合并状态**(`Object.assign`)。因此前面设置的 key 值会被后面所覆盖，最终只会执行一次更新；

- **函数式**: 由于 Fiber 及 合并 的问题，官方推荐可以传入 **函数** 的形式。`setState(fn)`，在`fn`中返回新的`state`对象即可，例如`this.state((state, props) => newState)；`
	- 使用函数式，可以用于避免`setState`的批量更新的逻辑，传入的函数将会被 **顺序调用**；

- **注意事项**:
	- setState 合并，在 合成事件 和 生命周期钩子 中多次连续调用会被优化为一次；
	- 当组件已被销毁，如果再次调用`setState`，React 会报错警告，通常有两种解决办法:
		- 将数据挂载到外部，通过 props 传入，如放到 Redux 或 父级中；
		- 在组件内部维护一个状态量 (isUnmounted)，`componentWillUnmount`中标记为 true，在`setState`前进行判断；

### 4. HOC(高阶组件)

HOC(Higher Order Componennt) 是在 React 机制下社区形成的一种组件模式，在很多第三方开源库中表现强大。

- **简述**:
	- 高阶组件不是组件，是 **增强函数**，可以输入一个元组件，返回出一个新的增强组件；
	- 高阶组件的主要作用是 **代码复用**，**操作** 状态和参数；

- **用法**: 
	- **属性代理 (Props Proxy)**: 返回出一个组件，它基于被包裹组件进行 **功能增强**；
		- **默认参数**: 可以为组件包裹一层默认参数；
		
		```js
		function proxyHoc(Comp) {
			return class extends React.Component {
				render() {
					const newProps = {
						name: 'tayde',
						age: 1,
					}
					return <Comp {...this.props} {...newProps} />
				}
			}
		}
		```
		
		- **提取状态**: 可以通过 props 将被包裹组件中的 state 依赖外层，例如用于转换受控组件:

		```js
		function withOnChange(Comp) {
			return class extends React.Component {
				constructor(props) {
					super(props)
					this.state = {
						name: '',
					}
				}
				onChangeName = () => {
					this.setState({
						name: 'dongdong',
					})
				}
				render() {
					const newProps = {
						value: this.state.name,
						onChange: this.onChangeName,
					}
					return <Comp {...this.props} {...newProps} />
				}
			}
		}
		```
		
		使用姿势如下，这样就能非常快速的将一个 `Input` 组件转化成受控组件。 
		
		```js
		const NameInput = props => (<input name="name" {...props} />)
		export default withOnChange(NameInput)
		```
		
		- **包裹组件**: 可以为被包裹元素进行一层包装，

		```js
		function withMask(Comp) {
			return class extends React.Component {
				render() {
					return (
						<div>
							<Comp {...this.props} />
							<div style={{
								width: '100%',
								height: '100%',
								backgroundColor: 'rgba(0, 0, 0, .6)',
							}} 
						</div>
					)
				}
			}
		}
		```
		
	- **反向继承** (Inheritance Inversion): 返回出一个组件，**继承于被包裹组件**，常用于以下操作:
		
		```js
		function IIHoc(Comp) {
		    return class extends Comp {
		        render() {
		            return super.render();
		        }
		    };
		}
		```
		
		- **渲染劫持** (Render Highjacking)
			- **条件渲染**: 根据条件，渲染不同的组件

			```js
			function withLoading(Comp) {
			    return class extends Comp {
			        render() {
			            if(this.props.isLoading) {
			                return <Loading />
			            } else {
			                return super.render()
			            }
			        }
			    };
			}
			```
			
			- 可以直接修改被包裹组件渲染出的 React 元素树
			
		- **操作状态** (Operate State): 可以直接通过 `this.state` 获取到被包裹组件的状态，并进行操作。但这样的操作容易使 state 变得难以追踪，不易维护，谨慎使用。
		
- **应用场景**:

	- **权限控制**，通过抽象逻辑，统一对页面进行权限判断，按不同的条件进行页面渲染:

	```js
	function withAdminAuth(WrappedComponent) {
	    return class extends React.Component {
			constructor(props){
				super(props)
				this.state = {
			    	isAdmin: false,
				}
			} 
			async componentWillMount() {
			    const currentRole = await getCurrentUserRole();
			    this.setState({
			        isAdmin: currentRole === 'Admin',
			    });
			}
			render() {
			    if (this.state.isAdmin) {
			        return <Comp {...this.props} />;
			    } else {
			        return (<div>您没有权限查看该页面，请联系管理员！</div>);
			    }
			}
	    };
	}
	```
	
	- **性能监控**，包裹组件的生命周期，进行统一埋点:

	```js
	function withTiming(Comp) {
	    return class extends Comp {
	        constructor(props) {
	            super(props);
	            this.start = Date.now();
	            this.end = 0;
	        }
	        componentDidMount() {
	            super.componentDidMount && super.componentDidMount();
	            this.end = Date.now();
	            console.log(`${WrappedComponent.name} 组件渲染时间为 ${this.end - this.start} ms`);
	        }
	        render() {
	            return super.render();
	        }
	    };
	}
	```
	
	- **代码复用**，可以将重复的逻辑进行抽象。

- 使用注意:
	- 1. **纯函数**: 增强函数应为纯函数，避免侵入修改元组件；
	- 2. **避免用法污染**: 理想状态下，应透传元组件的无关参数与事件，尽量保证用法不变；
	- 3. **命名空间**: 为 HOC 增加特异性的组件名称，这样能便于开发调试和查找问题；
	- 4. **引用传递**: 如果需要传递元组件的 refs 引用，可以使用`React.forwardRef`；
	- 5. **静态方法**: 元组件上的静态方法并无法被自动传出，会导致业务层无法调用；解决:
		- 函数导出
		- 静态方法赋值
	- 6. **重新渲染**: 由于增强函数每次调用是返回一个新组件，因此如果在 Render 中使用增强函数，就会导致每次都重新渲染整个HOC，而且之前的状态会丢失；

### 5. Redux

Redux 是一个 **数据管理中心**，可以把它理解为一个全局的 data store 实例。它通过一定的使用规则和限制，保证着数据的健壮性、可追溯和可预测性。它与 React 无关，可以独立运行于任何 JavaScript 环境中，从而也为同构应用提供了更好的数据同步通道。

- **核心理念**:
	- **单一数据源**: 整个应用只有唯一的状态树，也就是所有 state 最终维护在一个根级 Store 中；
	- **状态只读**: 为了保证状态的可控性，最好的方式就是监控状态的变化。那这里就两个必要条件：
		- Redux Store 中的数据无法被直接修改；
		- 严格控制修改的执行；
	- **纯函数**: 规定只能通过一个纯函数 (Reducer) 来描述修改；

- **理念实现**:
	- **Store**: 全局 Store 单例， 每个 Redux 应用下只有一个 store， 它具有以下方法供使用:
		- `getState`: 获取 state；
		- `dispatch`: 触发 action, 更新 state；
		- `subscribe`: 订阅数据变更，注册监听器；
	
	```js
	// 创建
	const store = createStore(Reducer, initStore)
	```
	
	- **Action**: 它作为一个行为载体，用于映射相应的 Reducer，并且它可以成为数据的载体，将数据从应用传递至 store 中，是 store 唯 **一的数据源**；

	```js
	// 一个普通的 Action
   const action = {
		type: 'ADD_LIST',
		item: 'list-item-1',
	}
	
	// 使用：
	store.dispatch(action)
	
	// 通常为了便于调用，会有一个 Action 创建函数 (action creater)
	funtion addList(item) {
		return const action = {
			type: 'ADD_LIST',
			item,
		}
	}
	
	// 调用就会变成:
	dispatch(addList('list-item-1'))
	```
		
	- **Reducer**: 用于描述如何修改数据的纯函数，Action 属于行为名称，而 Reducer 便是修改行为的实质；

	```js
	// 一个常规的 Reducer
	// @param {state}: 旧数据
	// @param {action}: Action 对象
	// @returns {any}: 新数据
	const initList = []
	function ListReducer(state = initList, action) {
		switch (action.type) {
			case 'ADD_LIST':
				return state.concat([action.item])
				break
			defalut:
				return state
		}
	}
	```
		
	> **注意**:
	>
	> 1. 遵守数据不可变，不要去直接修改 state，而是返回出一个 **新对象**，可以使用 `assign / copy / extend / 解构` 等方式创建新对象；
	> 2. 默认情况下需要 **返回原数据**，避免数据被清空；
	> 3. 最好设置 **初始值**，便于应用的初始化及数据稳定；

- **进阶**:
	- **React-Redux**: 结合 React 使用；
		- `<Provider>`: 将 store 通过 context 传入组件中；
		- `connect`: 一个高阶组件，可以方便在 React 组件中使用 Redux；
			- 1. 将`store`通过`mapStateToProps`进行筛选后使用`props`注入组件
			- 2. 根据`mapDispatchToProps`创建方法，当组件调用时使用`dispatch`触发对应的`action` 
	- **Reducer 的拆分与重构**: 
		- 随着项目越大，如果将所有状态的 reducer 全部写在一个函数中，将会 **难以维护**；
		- 可以将 reducer 进行拆分，也就是 **函数分解**，最终再使用`combineReducers()`进行重构合并；
	- **异步 Action**: 由于 Reducer 是一个严格的纯函数，因此无法在 Reducer 中进行数据的请求，需要先获取数据，再`dispatch(Action)`即可，下面是三种不同的异步实现:
		- [redex-thunk](https://github.com/reduxjs/redux-thunk)
		- [redux-saga](https://github.com/redux-saga/redux-saga)
		- [redux-observable](https://github.com/redux-observable/redux-observable)

### 6. React Hooks

React 中通常使用 **类定义** 或者 **函数定义** 创建组件:

在类定义中，我们可以使用到许多 React 特性，例如 state、 各种组件生命周期钩子等，但是在函数定义中，我们却无能为力，因此 React 16.8 版本推出了一个新功能 (React Hooks)，通过它，可以更好的在函数定义组件中使用 React 特性。

- **好处**:
	- 1、**跨组件复用**: 其实 render props / HOC 也是为了复用，相比于它们，Hooks 作为官方的底层 API，最为轻量，而且改造成本小，不会影响原来的组件层次结构和传说中的嵌套地狱；
	- 2、**类定义更为复杂**: 
		- 不同的生命周期会使逻辑变得分散且混乱，不易维护和管理；
		- 时刻需要关注`this`的指向问题；
		- 代码复用代价高，高阶组件的使用经常会使整个组件树变得臃肿；
	- 3、**状态与UI隔离**: 正是由于 Hooks 的特性，状态逻辑会变成更小的粒度，并且极容易被抽象成一个自定义 Hooks，组件中的状态和 UI 变得更为清晰和隔离。

- **注意**:
	- 避免在 循环/条件判断/嵌套函数 中调用 hooks，保证调用顺序的稳定；
	- 只有 函数定义组件 和 hooks 可以调用 hooks，避免在 类组件 或者 普通函数 中调用；
	- 不能在`useEffect`中使用`useState`，React 会报错提示；
	- 类组件不会被替换或废弃，不需要强制改造类组件，两种方式能并存；

- **重要钩子***:
	- **状态钩子** (`useState`): 用于定义组件的 State，其到类定义中`this.state`的功能；
	
	```js
	// useState 只接受一个参数: 初始状态
	// 返回的是组件名和更改该组件对应的函数
	const [flag, setFlag] = useState(true);
	// 修改状态
	setFlag(false)
		
	// 上面的代码映射到类定义中:
	this.state = {
		flag: true	
	}
	const flag = this.state.flag
	const setFlag = (bool) => {
	    this.setState({
	        flag: bool,
	    })
	}
	```

	- **生命周期钩子** (`useEffect`):
	
	类定义中有许多生命周期函数，而在 React Hooks 中也提供了一个相应的函数 (`useEffect`)，这里可以看做`componentDidMount`、`componentDidUpdate`和`componentWillUnmount`的结合。
	
	- `useEffect(callback, [source])`接受两个参数
		- `callback`: 钩子回调函数；
		- `source`: 设置触发条件，仅当 source 发生改变时才会触发；
		- `useEffect`钩子在没有传入`[source]`参数时，默认在每次 render 时都会优先调用上次保存的回调中返回的函数，后再重新调用回调；
	
	```js
	useEffect(() => {
		// 组件挂载后执行事件绑定
		console.log('on')
		addEventListener()
		
		// 组件 update 时会执行事件解绑
		return () => {
			console.log('off')
			removeEventListener()
		}
	}, [source]);
	
	
	// 每次 source 发生改变时，执行结果(以类定义的生命周期，便于大家理解):
	// --- DidMount ---
	// 'on'
	// --- DidUpdate ---
	// 'off'
	// 'on'
	// --- DidUpdate ---
	// 'off'
	// 'on'
	// --- WillUnmount --- 
	// 'off'
	```
	- 通过第二个参数，我们便可模拟出几个常用的生命周期:
		- `componentDidMount`: 传入`[]`时，就只会在初始化时调用一次；

		```js
		const useMount = (fn) => useEffect(fn, [])
		``` 
		
		- `componentWillUnmount`: 传入`[]`，回调中的返回的函数也只会被最终执行一次；

		```js
		const useUnmount = (fn) => useEffect(() => fn, [])
		```
		
		- `mounted `: 可以使用 useState 封装成一个高度可复用的 mounted 状态；

		```js
		const useMounted = () => {
		    const [mounted, setMounted] = useState(false);
		    useEffect(() => {
		        !mounted && setMounted(true);
		        return () => setMounted(false);
		    }, []);
		    return mounted;
		}
		```
		
		- `componentDidUpdate`: `useEffect`每次均会执行，其实就是排除了 DidMount 后即可；

		```js
		const mounted = useMounted() 
	    useEffect(() => {
	        mounted && fn()
	    })
		```

- **其它内置钩子**:
	- `useContext`: 获取 context 对象
	- `useReducer`: 类似于 Redux 思想的实现，但其并不足以替代 Redux，可以理解成一个组件内部的 redux:
		- 并不是持久化存储，会随着组件被销毁而销毁；
		- 属于组件内部，各个组件是相互隔离的，单纯用它并无法共享数据；
		- 配合`useContext`的全局性，可以完成一个轻量级的 Redux；([easy-peasy](https://github.com/ctrlplusb/easy-peasy))
	
	- `useCallback`: 缓存回调函数，避免传入的回调每次都是新的函数实例而导致依赖组件重新渲染，具有性能优化的效果；
	- `useMemo`: 用于缓存传入的 props，避免依赖的组件每次都重新渲染；
	- `useRef`: 获取组件的真实节点；
	- `useLayoutEffect`: 
		- DOM更新同步钩子。用法与`useEffect`类似，只是区别于执行时间点的不同。
		- `useEffect`属于异步执行，并不会等待 DOM 真正渲染后执行，而`useLayoutEffect`则会真正渲染后才触发；
		- 可以获取更新后的 state；
	
- **自定义钩子**(`useXxxxx`): 基于 Hooks 可以引用其它 Hooks 这个特性，我们可以编写自定义钩子，如上面的`useMounted`。又例如，我们需要每个页面自定义标题:

```js
function useTitle(title) {
  useEffect(
    () => {
      document.title = title;
    });
}

// 使用:
function Home() {
	const title = '我是首页'
	useTitle(title)
	
	return (
		<div>{title}</div>
	)
}
```
 
### 7. SSR

SSR，俗称 **服务端渲染** (Server Side Render)，讲人话就是: 直接在服务端层获取数据，渲染出完成的 HTML 文件，直接返回给用户浏览器访问。

- **前后端分离**: 前端与服务端隔离，前端动态获取数据，渲染页面。

- **痛点**: 

	- **首屏渲染性能瓶颈**: 
		- 空白延迟: HTML下载时间 + JS下载/执行时间 + 请求时间 + 渲染时间。在这段时间内，页面处于空白的状态。
	
	- **SEO 问题**: 由于页面初始状态为空，因此爬虫无法获取页面中任何有效数据，因此对搜索引擎不友好。
		- 虽然一直有在提动态渲染爬虫的技术，不过据我了解，大部分国内搜索引擎仍然是没有实现。

最初的服务端渲染，便没有这些问题。但我们不能返璞归真，既要保证现有的前端独立的开发模式，又要由服务端渲染，因此我们使用 React SSR。

- **原理**: 
	- Node 服务: 让前后端运行同一套代码成为可能。
	- Virtual Dom: 让前端代码脱离浏览器运行。
	
- **条件**: Node 中间层、 React / Vue 等框架。 

- **流程**: (此处以 React + Router + Redux + Koa 为例)

	- 1、在同个项目中，**搭建** 前后端部分，常规结构:
		- build
		- public
		- src
			- client
			- server 
			
	- 2、server 中使用 Koa **路由监听** 页面访问: 

	```js
	import * as Router from 'koa-router'
	
	const router = new Router()
	// 如果中间也提供 Api 层
	router.use('/api/home', async () => {
		// 返回数据
	})
	
	router.get('*', async (ctx) => {
		// 返回 HTML
	})
	```
	
	- 3、通过访问 url **匹配** 前端页面路由:

	```js
	// 前端页面路由
	import { pages } from '../../client/app'
	import { matchPath } from 'react-router-dom'
	
	// 使用 react-router 库提供的一个匹配方法
	const matchPage = matchPath(ctx.req.url, page)
	```
	
	- 4、通过页面路由的配置进行 **数据获取**。通常可以在页面路由中增加 SSR 相关的静态配置，用于抽象逻辑，可以保证服务端逻辑的通用性，如:

		```js
		class HomePage extends React.Component{
			public static ssrConfig = {
				  cache: true,
		         fetch() {
		        	  // 请求获取数据
		         }
		    }
		}
		```
		
		获取数据通常有两种情况:
		
		- 中间层也使用 **http** 获取数据，则此时 fetch 方法可前后端共享；

		```js
		const data = await matchPage.ssrConfig.fetch()
		```
		
		- 中间层并不使用 http，是通过一些 **内部调用**，例如 Rpc 或 直接读数据库 等，此时也可以直接由服务端调用对应的方法获取数据。通常，这里需要在 ssrConfig 中配置特异性的信息，用于匹配对应的数据获取方法。

		```js
		// 页面路由
		class HomePage extends React.Component{
			public static ssrConfig = {
		        fetch: {
		        	 url: '/api/home',
		        }
		    }
		}
		
		// 根据规则匹配出对应的数据获取方法
		// 这里的规则可以自由，只要能匹配出正确的方法即可
		const controller = matchController(ssrConfig.fetch.url)
		
		// 获取数据
		const data = await controller(ctx)
		``` 

	- 5、创建 Redux store，并将数据`dispatch`到里面:

	```js
	import { createStore } from 'redux'
	// 获取 Clinet层 reducer
	// 必须复用前端层的逻辑，才能保证一致性；
	import { reducers } from '../../client/store'
	
	// 创建 store
	const store = createStore(reducers)
	 
	// 获取配置好的 Action
	const action = ssrConfig.action

	// 存储数据	
	store.dispatch(createAction(action)(data))
	```
	
	- 6、注入 Store， 调用`renderToString`将 React Virtual Dom 渲染成 **字符串**: 
	
	```js
	import * as ReactDOMServer from 'react-dom/server'
	import { Provider } from 'react-redux'
	
	// 获取 Clinet 层根组件
	import { App } from '../../client/app'
	
	const AppString = ReactDOMServer.renderToString(
		<Provider store={store}>
			<StaticRouter
				location={ctx.req.url}
				context={{}}>
				<App />
			</StaticRouter>
		</Provider>
	)
	```
	
	- 7、将 AppString 包装成完整的 html 文件格式；
	
	- 8、此时，已经能生成完整的 HTML 文件。但只是个纯静态的页面，没有样式没有交互。接下来我们就是要插入 JS 与 CSS。我们可以通过访问前端打包后生成的`asset-manifest.json`文件来获取相应的文件路径，并同样注入到 Html 中引用。

	```js
	const html = `
		<!DOCTYPE html>
		<html lang="zh">
			<head></head>
			<link href="${cssPath}" rel="stylesheet" />
			<body>
				<div id="App">${AppString}</div>
				<script src="${scriptPath}"></script>
			</body>
		</html>
	`
	```

	- 9、进行 **数据脱水**: 为了把服务端获取的数据同步到前端。主要是将数据序列化后，插入到 html 中，返回给前端。

	```js
	import serialize from 'serialize-javascript'
	// 获取数据
	const initState = store.getState()
	const html = `
		<!DOCTYPE html>
		<html lang="zh">
			<head></head>
			<body>
				<div id="App"></div>
				<script type="application/json" id="SSR_HYDRATED_DATA">${serialize(initState)}</script>
			</body>
		</html>
	`
	
	ctx.status = 200
	ctx.body = html
	```
	
	> **Tips**:
	>
	> 这里比较特别的有两点:
	>
	> 1. 使用了`serialize-javascript`序列化 store， 替代了`JSON.stringify`，保证数据的安全性，避免代码注入和 XSS 攻击；
	>
	> 2. 使用 json 进行传输，可以获得更快的加载速度；
	
	- 10、Client 层 **数据吸水**: 初始化 store 时，以脱水后的数据为初始化数据，同步创建 store。

	```js
	const hydratedEl = document.getElementById('SSR_HYDRATED_DATA')
	const hydrateData = JSON.parse(hydratedEl.textContent)
	
	// 使用初始 state 创建 Redux store
	const store = createStore(reducer, hydrateData)
	```
	
### 8.函数式编程

函数式编程是一种 **编程范式**，你可以理解为一种软件架构的思维模式。它有着独立一套理论基础与边界法则，追求的是 **更简洁、可预测、高复用、易测试**。其实在现有的众多知名库中，都蕴含着丰富的函数式编程思想，如 React / Redux 等。

- **常见的编程范式**:
	- 命令式编程(过程化编程): 更关心解决问题的步骤，一步步以语言的形式告诉计算机做什么；
	- 事件驱动编程: 事件订阅与触发，被广泛用于 GUI 的编程设计中；
	- 面向对象编程: 基于类、对象与方法的设计模式，拥有三个基础概念: 封装性、继承性、多态性；
	- 函数式编程
		- 换成一种更高端的说法，面向数学编程。怕不怕~🥴

- **函数式编程的理念**:
	- **纯函数**(确定性函数): 是函数式编程的基础，可以使程序变得灵活，高度可拓展，可维护；
		- **优势**:
			- 完全独立，与外部解耦；
			- 高度可复用，在任意上下文，任意时间线上，都可执行并且保证结果稳定；
			- 可测试性极强；
			
		- **条件**:  
			- 不修改参数；
			- 不依赖、不修改任何函数外部的数据；
			- 完全可控，参数一样，返回值一定一样: 例如函数不能包含`new Date()`或者`Math.randon()`等这种不可控因素；
			- 引用透明；

		- 我们常用到的许多 API 或者工具函数，它们都具有着纯函数的特点， 如`split / join / map`；
		
	- **函数复合**: 将多个函数进行组合后调用，可以实现将一个个函数单元进行组合，达成最后的目标；
		- **扁平化嵌套**: 首先，我们一定能想到组合函数最简单的操作就是 包裹，因为在 JS 中，函数也可以当做参数:
			- `f(g(k(x)))`: 嵌套地狱，可读性低，当函数复杂后，容易让人一脸懵逼；
			- 理想的做法: `xxx(f, g, k)(x)`
		- **结果传递**: 如果想实现上面的方式，那也就是`xxx`函数要实现的便是: 执行结果在各个函数之间的执行传递；
			- 这时我们就能想到一个原生提供的数组方法: `reduce`，它可以按数组的顺序依次执行，传递执行结果；
			- 所以我们就能够实现一个方法`pipe`，用于函数组合:
			
			```js
			// ...fs: 将函数组合成数组；
			// Array.prototype.reduce 进行组合；
			// p: 初始参数；
			const pipe = (...fs) => p => fs.reduce((v, f) => f(v), p)
			```
		
		- **使用**: 实现一个 驼峰命名 转 中划线命名 的功能:
	
		```js
		// 'Guo DongDong' --> 'guo-dongdong'
		// 函数组合式写法
		const toLowerCase = str => str.toLowerCase()
		const join = curry((str, arr) => arr.join(str))
		const split = curry((splitOn, str) => str.split(splitOn));
		
		const toSlug = pipe(
			toLowerCase,	
			split(' '),
			join('_'),
			encodeURIComponent,
	    );
	    console.log(toSlug('Guo DongDong'))
		```
		
		- **好处**:
			- 隐藏中间参数，不需要临时变量，避免了这个环节的出错几率；
			- 只需关注每个纯函数单元的稳定，不再需要关注命名，传递，调用等；
			- 可复用性强，任何一个函数单元都可被任意复用和组合；
			- 可拓展性强，成本低，例如现在加个需求，要查看每个环节的输出:
			
			```js
			const log = curry((label, x) => {
				console.log(`${ label }: ${ x }`);
				return x;
			});
			
			const toSlug = pipe(
				toLowerCase,	
				log('toLowerCase output'),
				split(' '),
				log('split output'),
				join('_'),
				log('join output'),
				encodeURIComponent,
			);
			```
		
		> Tips:
		>
		> 一些工具纯函数可直接引用`lodash/fp`，例如`curry/map/split`等，并不需要像我们上面这样自己实现；
	
	- **数据不可变性**(immutable): 这是一种数据理念，也是函数式编程中的核心理念之一:
		- **倡导**: 一个对象再被创建后便不会再被修改。当需要改变值时，是返回一个全新的对象，而不是直接在原对象上修改；
		- **目的**: 保证数据的稳定性。避免依赖的数据被未知地修改，导致了自身的执行异常，能有效提高可控性与稳定性；
		- 并不等同于`const`。使用`const`创建一个对象后，它的属性仍然可以被修改；
		- 更类似于`Object.freeze`: 冻结对象，但`freeze`仍无法保证深层的属性不被串改；
		- `immutable.js`: js 中的数据不可变库，它保证了数据不可变，在 React 生态中被广泛应用，大大提升了性能与稳定性；
			- `trie`数据结构: 
				- 一种数据结构，能有效地深度冻结对象，保证其不可变；
				- **结构共享**: 可以共用不可变对象的内存引用地址，减少内存占用，提高数据操作性能；
		
	- 避免不同函数之间的 **状态共享**，数据的传递使用复制或全新对象，遵守数据不可变原则；
	- 避免从函数内部 **改变外部状态**，例如改变了全局作用域或父级作用域上的变量值，可能会导致其它单位错误；
	- 避免在单元函数内部执行一些 **副作用**，应该将这些操作抽离成更独立的工具单元；
		- 日志输出
		- 读写文件
		- 网络请求
		- 调用外部进程
		- 调用有副作用的函数 
	
- **高阶函数**: 是指 以函数为参数，返回一个新的增强函数 的一类函数，它通常用于:
	- 将逻辑行为进行 **隔离抽象**，便于快速复用，如处理数据，兼容性等；
	- **函数组合**，将一系列单元函数列表组合成功能更强大的函数；
	- **函数增强**，快速地拓展函数功能，

- **函数式编程的好处**:
	- 函数副作用小，所有函数独立存在，没有任何耦合，复用性极高；
	- 不关注执行时间，执行顺序，参数，命名等，能专注于数据的流动与处理，能有效提高稳定性与健壮性；
	- 追求单元化，粒度化，使其重构和改造成本降低，可维护、可拓展性较好；
	- 更易于做单元测试。

- **总结**: 
	- 函数式编程其实是一种编程思想，它追求更细的粒度，将应用拆分成一组组极小的单元函数，组合调用操作数据流；
	- 它提倡着 纯函数 / 函数复合 / 数据不可变， 谨慎对待函数内的 状态共享 / 依赖外部 / 副作用；

> Tips:
> 
> 其实我们很难也不需要在面试过程中去完美地阐述出整套思想，这里也只是浅尝辄止，一些个人理解而已。博主也是初级小菜鸟，停留在表面而已，只求对大家能有所帮助，轻喷🤣；
> 
> 我个人觉得: 这些编程范式之间，其实并不矛盾，各有各的优劣势；
> 
> 理解和学习它们的理念与优势，合理地设计融合，将优秀的软件编程思想用于提升我们应用；
> 
> 所有设计思想，最终的目标一定是使我们的应用更加 **解耦颗粒化、易拓展、易测试、高复用，开发更为高效和安全**；
> 
> 有一些库能让大家很快地接触和运用函数思想: `Underscore.js` / `Lodash/fp` / `Rxjs` 等。 

## Hybrid

随着 Web技术 和 移动设备 的快速发展，在各家大厂中，Hybrid 技术已经成为一种最主流最不可获取的架构方案之一。一套好的 Hybrid 架构方案能让 App 既能拥有极致的体验和性能，同时也能拥有 Web技术 灵活的开发模式、跨平台能力以及热更新机制。因此，相关的 Hybrid 领域人才也是十分的吃香，精通Hybrid 技术和相关的实战经验，也是面试中一项大大的加分项。

### 1. 混合方案简析

Hybrid App，俗称 **混合应用**，即混合了 Native技术 与 Web技术 进行开发的移动应用。现在比较流行的混合方案主要有三种，主要是在UI渲染机制上的不同: 

- **Webview UI**:
	- 通过 JSBridge 完成 H5 与 Native 的双向通讯，并 **基于 Webview** 进行页面的渲染配；
	- **优势**: 简单易用，架构门槛/成本较低，适用性与灵活性极强；
	- **劣势**: Webview 性能局限，在复杂页面中，表现远不如原生页面；

- **Native UI**:
	- 通过 JSBridge 赋予 H5 原生能力，并进一步将 JS 生成的虚拟节点树(Virtual DOM)传递至 Native 层，并使用 **原生渲染**。
	- **优势**: 用户体验基本接近原生，且能发挥 Web技术 开发灵活与易更新的特性；
	- **劣势**: 上手/改造门槛较高，最好需要掌握一定程度的客户端技术。相比于常规 Web开发，需要更高的开发调试、问题排查成本；
	
- **小程序**
	- 通过更加定制化的 JSBridge，赋予了 Web 更大的权限，并使用双 WebView 双线程的模式隔离了 JS逻辑 与 UI渲染，形成了特殊的开发模式，加强了 H5 与 Native 混合程度，属于第一种方案的优化版本；
	- **优势**: 用户体验好于常规 Webview 方案，且通常依托的平台也能提供更为友好的开发调试体验以及功能；
	- **劣势**: 需要依托于特定的平台的规范限定

### 2. Webviev

Webview 是 Native App 中内置的一款基于 Webkit内核 的浏览器，主要由两部分组成:

- **WebCore 排版引擎**；
- **JSCore 解析引擎**；

在原生开发 SDK 中 Webview 被封装成了一个组件，用于作为 Web页面 的容器。因此，作为宿主的客户端中拥有更高的权限，可以对 Webview 中的 Web页面 进行配置和开发。

Hybrid技术中双端的交互原理，便是基于 Webview 的一些 API 和特性。

### 3. 交互原理

Hybrid技术 中最核心的点就是 Native端 与 H5端 之间的 **双向通讯层**，其实这里也可以理解为我们需要一套 **跨语言通讯方案**，便是我们常听到的 JSBridge。

- JavaScript 通知 Native
	- **API注入**，Native 直接在 JS 上下文中挂载数据或者方法
		- 延迟较低，在安卓4.1以下具有安全性问题，风险较高 
	- WebView **URL Scheme** 跳转拦截
		- 兼容性好，但延迟较高，且有长度限制 
	- WebView 中的 **prompt/console/alert拦截**(通常使用 prompt)
	
- **Native 通知 Javascript**:
	- **IOS**: `stringByEvaluatingJavaScriptFromString`
	
	```js
	// Swift
	webview.stringByEvaluatingJavaScriptFromString("alert('NativeCall')")
	```
	
	- **Android**: `loadUrl` (4.4-)  

	```js
	// 调用js中的JSBridge.trigger方法
	// 该方法的弊端是无法获取函数返回值；
	webView.loadUrl("javascript:JSBridge.trigger('NativeCall')")
	```

	- **Android**: `evaluateJavascript` (4.4+)
	
	```js
	// 4.4+后使用该方法便可调用并获取函数返回值；
	mWebView.evaluateJavascript（"javascript:JSBridge.trigger('NativeCall')", 	 new ValueCallback<String>() {
	    @Override
	    public void onReceiveValue(String value) {
	        //此处为 js 返回的结果
	    }
	});
	```
	
### 4. 接入方案

整套方案需要 Web 与 Native 两部分共同来完成:

- **Native**: 负责实现URL拦截与解析、环境信息的注入、拓展功能的映射、版本更新等功能；
- **JavaScirpt**: 负责实现功能协议的拼装、协议的发送、参数的传递、回调等一系列基础功能。

**接入方式**:

- **在线H5**: 直接将项目部署于线上服务器，并由客户端在 HTML 头部注入对应的 Bridge。
	- **优势**: 接入/开发成本低，对 App 的侵入小；
	- **劣势**: 重度依赖网络，无法离线使用，首屏加载慢；
- **内置离线包**: 将代码直接内置于 App 中，即本地存储中，可由 H5 或者 客户端引用 Bridge。
	- **优势**: 首屏加载快，可离线化使用；
	- **劣势**: 开发、调试成本变高，需要多端合作，且会增加 App 包体积

### 4. 优化方案简述

- **Webview 预加载**: Webview 的初始化其实挺耗时的。我们测试过，大概在100~200ms之间，因此如果能前置做好初始化于内存中，会大大加快渲染速度。
- **更新机制**: 使用离线包的时候，便会涉及到本地离线代码的更新问题，因此需要建立一套云端下发包的机制，由客户端下载云端最新代码包 (zip包)，并解压替换本地代码。
	- **增量更新**: 由于下发包是一个下载的过程，因此包的体积越小，下载速度越快，流量损耗越低。只打包改变的文件，客户端下载后覆盖式替换，能大大减小每次更新包的体积。
	- **条件分发**: 云平台下发更新包时，可以配合客户端设置一系列的条件与规则，从而实现代码的条件更新:
		- 单 **地区** 更新: 例如一个只有中国地区才能更新的版本；
		- 按 **语言** 更新: 例如只有中文版本会更新；
		- 按 App **版本** 更新: 例如只有最新版本的 App 才会更新；
		- **灰度** 更新: 只有小比例用户会更新；
		- **AB测试**: 只有命中的用户会更新；
- **降级机制**: 当用户下载或解压代码包失败时，需要有套降级方案，通常有两种做法: 
	- **本地内置**: 随着 App 打包时内置一份线上最新完整代码包，保证本地代码文件的存在，资源加载均使用本地化路径；
	- **域名拦截**: 资源加载使用线上域名，通过拦截域名映射到本地路径。当本地不存在时，则请求线上文件，当存在时，直接加载；
- **跨平台部署**: Bridge层可以做一套浏览器适配，在一些无法适配的功能，业务方做好降级处理，从而保证代码在任何环境的可用性，一套代码可同时运行于 App内 与 普通浏览器；
- **环境系统**: 与客户端进行统一配合，搭建出正式 / 预上线 / 测试 / 开发等环境，能大大提高项目稳定性与开发效率；
- **开发模式**: 
	- 能连接PC Chrome/safari 进行代码调试；
	- 具有开发调试入口，可以使用同样的 Webview 加载开发时的本地代码；
	- 具备日志系统，可以查看 Log 信息；
	
## Webpack

### 1. 原理简述

Webpack 已经成为了现在前端工程化中最重要的一环，通过`Webpack`与`Node`的配合，前端领域完成了不可思议的进步。通过预编译，将软件编程中先进的思想和理念能够真正运用于生产，让前端开发领域告别原始的蛮荒阶段。深入理解`Webpack`，可以让你在编程思维及技术领域上产生质的成长，也能让面试官对你刮目相看。~

- **核心概念**
	- JavaScript 的 **模块打包工具** (module bundler)。通过分析模块之间的依赖，最终将所有模块打包成一份或者多份代码包 (bundler)，供 HTML 直接引用。实质上，Webpack 仅仅提供了 **打包功能** 和一套文件 **处理机制**，然后通过生态中的各种 Loader 和 Plugin 对代码进行预编译和打包。因此 Webpack 具有高度的可拓展性，能更好的发挥生态的力量。
		- **Entry**: 入口文件，Webpack 会从该文件开始进行分析与编译；
		- **Output**: 出口路径，打包后创建 bundler 的文件路径以及文件名；
		- **Module**: 模块，在 Webpack 中任何文件都可以作为一个模块，会根据配置的不同的 Loader 进行加载和打包；
		- **Chunk**: 代码块，可以根据配置，将所有模块代码合并成一个或多个代码块，以便按需加载，提高性能；
		- **Loader**: 模块加载器，进行各种文件类型的加载与转换；
		- **Plugin**: 拓展插件，可以通过 Webpack 相应的事件钩子，介入到打包过程中的任意环节，从而对代码按需修改；
 
- **工作流程** (加载 - 编译 - 输出)
	- 1、读取配置文件，按命令 **初始化** 配置参数，创建 Compiler 对象；
	- 2、调用插件的 apply 方法 **挂载插件** 监听，然后从入口文件开始执行编译；
	- 3、按文件类型，调用相应的 Loader 对模块进行 **编译**，并在合适的时机点触发对应的事件，调用 Plugin 执行，最后再根据模块 **依赖查找** 到所依赖的模块，递归执行第三步；
	- 4、将编译后的所有代码包装成一个个代码块 (Chuck)， 并按依赖和配置确定 **输出内容**。这个步骤，仍然可以通过 Plugin 进行文件的修改;
	- 5、最后，根据 Output 把文件内容一一写入到指定的文件夹中，完成整个过程；

- **模块包装**:

```js
(function(modules) {
	// 模拟 require 函数，从内存中加载模块；
	function __webpack_require__(moduleId) {
		// 缓存模块
		if (installedModules[moduleId]) {
			return installedModules[moduleId].exports;
		}
		
		var module = installedModules[moduleId] = {
			i: moduleId,
			l: false,
			exports: {}
		};
		
		// 执行代码；
		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
		
		// Flag: 标记是否加载完成；
		module.l = true;
		
		return module.exports;
	}
	
	// ...
	
	// 开始执行加载入口文件；
	return __webpack_require__(__webpack_require__.s = "./src/index.js");
 })({
 	"./src/index.js": function (module, __webpack_exports__, __webpack_require__) {
		// 使用 eval 执行编译后的代码；
		// 继续递归引用模块内部依赖；
		// 实际情况并不是使用模板字符串，这里是为了代码的可读性；
		eval(`
			__webpack_require__.r(__webpack_exports__);
			//
			var _test__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("test", ./src/test.js");
		`);
	},
	"./src/test.js": function (module, __webpack_exports__, __webpack_require__) {
		// ...
	},
 })
```

- **总结**:
	- **模块机制**: webpack 自己实现了一套模拟模块的机制，将其包裹于业务代码的外部，从而提供了一套模块机制；
	- **文件编译**: webpack 规定了一套编译规则，通过 Loader 和 Plugin，以管道的形式对文件字符串进行处理；

### 2. Loader

由于 Webpack 是基于 Node，因此 Webpack 其实是只能识别 js 模块，比如 css / html / 图片等类型的文件并无法加载，因此就需要一个对 **不同格式文件转换器**。其实 Loader 做的事，也并不难理解: **对 Webpack 传入的字符串进行按需修改**。例如:

```js
// html-loader/index.js
module.exports = function(htmlSource) {
	// 返回处理后的代码字符串
	// 删除 html 文件中的所有注释
	return htmlSource.replace(/<!--[\w\W]*?-->/g, '')
}
```

当然，实际的 Loader 不会这么简单，通常是需要将代码进行分析，构建 AST (抽象语法树)， 遍历进行定向的修改后，再重新生成新的代码字符串。如我们常用的 Babel-loader 会执行以下步骤:

- babylon 将 ES6/ES7 代码解析成 AST
- babel-traverse 对 AST 进行遍历转译，得到新的 AST
- 新 AST 通过 babel-generator 转换成 ES5

**Loader 特性**:

- **链式传递**，按照配置时相反的顺序链式执行；
- 基于 Node 环境，拥有 **较高权限**，比如文件的增删查改；
- 可同步也可异步；

**常用 Loader**:

- file-loader: 加载文件资源，如 字体 / 图片 等，具有移动/复制/命名等功能；
- url-loader: 通常用于加载图片，可以将小图片直接转换为 Date Url，减少请求；
- babel-loader: 加载 js / jsx 文件， 将 ES6 / ES7 代码转换成 ES5，抹平兼容性问题；
- ts-loader: 加载 ts / tsx 文件，编译 TypeScript；
- style-loader: 将 css 代码以`<style>`标签的形式插入到 html 中；
- css-loader: 分析`@import`和`url()`，引用 css 文件与对应的资源；
- postcss-loader: 用于 css 的兼容性处理，具有众多功能，例如添加前缀，单位转换等；
- less-loader / sass-loader: css预处理器，在 css 中新增了许多语法，提高了开发效率；

**编写原则**:

- **单一原则**: 每个 Loader 只做一件事；
- **链式调用**: Webpack 会按顺序链式调用每个 Loader；
- **统一原则**: 遵循 Webpack 制定的设计规则和结构，输入与输出均为字符串，各个 Loader 完全独立，即插即用；

### 3. Plugin

插件系统是 Webpack 成功的一个关键性因素。在编译的整个生命周期中，Webpack 会触发许多事件钩子，Plugin 可以监听这些事件，根据需求在相应的时间点对打包内容进行定向的修改。

- 一个最简单的 plugin 是这样的:

```js
class Plugin{

	constructor(options){ }
	
  	// 注册插件时，会调用 apply 方法
  	// apply 方法接收 compiler 对象
  	// 通过 compiler 上提供的 Api，可以对事件进行监听，执行相应的操作
  	apply(compiler){
  		// compilation 是监听每次编译循环
  		// 每次文件变化，都会生成新的 compilation 对象并触发该事件
    	compiler.plugin('compilation',function(compilation) {})
  	}
}
```

- **注册插件**:

```js
module.export = {
	plugins:[
		new Plugin(options),
	]
}
```

- **事件流机制**:

Webpack 就像工厂中的一条产品流水线。原材料经过 Loader 与 Plugin 的一道道处理，最后输出结果。

- 通过链式调用，按顺序串起一个个 Loader；
- 通过事件流机制，让 Plugin 可以插入到整个生产过程中的每个步骤中；

Webpack 事件流编程范式的核心是基础类 **Tapable**，是一种 **观察者模式** 的实现事件的订阅与广播：

```js
const { SyncHook } = require("tapable")

const hook = new SyncHook(['arg'])

// 订阅
hook.tap('event', (arg) => {
	// 'event-hook'
	console.log(arg)
})

// 广播
hook.call('event-hook')
```

Webpack 中两个最重要的类 Compiler 与 Compilation 便是继承于 Tapable，也拥有这样的事件流机制。

- **Compiler**: 可以简单的理解为 **Webpack 实例**，它包含了当前 Webpack 中的所有配置信息，如 options， loaders, plugins 等信息，全局唯一，只在启动时完成初始化创建，随着生命周期逐一传递；

- **Compilation**: 可以成为 **编译实例**。当监听到文件发生改变时，Webpack 会创建一个新的 Comilation 对象，开始一次新的编译。它包含了当前的输入资源，输出资源，变化的文件等，同时通过它提供的 api，可以监听每次编译过程中触发的事件钩子；

- **区别**: 
	- Compiler 全局唯一，且从启动生存到结束；
	- Compilaation 对应每次编译，每轮编译循环均会重新创建；

- **常用 Plugin**:
	- UglifyJsPlugin: 压缩、混淆代码；
	- CommonsChunkPlugin: 代码分割；
	- ProvidePlugin: 自动加载模块；
	- html-webpack-plugin: 加载 html 文件，并引入 css / js 文件；
	- extract-text-webpack-plugin / mini-css-extract-plugin: 抽离样式，生成 css 文件；
	- DefinePlugin: 定义全局变量；
	- optimize-css-assets-webpack-plugin: CSS 代码去重；
	- webpack-bundle-analyzer: 代码分析；
	- compression-webpack-plugin: 使用 gzip 压缩 js 和 css；
	- happypack: 使用多进程，加速代码构建；
	- EnvironmentPlugin: 定义环境变量；

### 4. 编译优化

- **代码优化**:
	- **无用代码消除**，是许多编程语言都具有的优化手段，这个过程称为 DCE (dead code elimination)，即 **删除不可能执行的代码**； 
		- 例如我们的 UglifyJs，它就会帮我们在生产环境中删除不可能被执行的代码，例如:
		
		```js
		var fn = function() {
			return 1;
			// 下面代码便属于 不可能执行的代码；
			// 通过 UglifyJs (Webpack4+ 已内置) 便会进行 DCE；
			var a = 1;
			return a;
		}
		```

	- **摇树优化** (Tree-shaking)，这是一种形象比喻。我们把打包后的代码比喻成一棵树，这里其实表示的就是，通过工具 "摇" 我们打包后的 js 代码，将没有使用到的无用代码 "摇" 下来 (删除)。即 消除那些被 **引用了但未被使用** 的模块代码。
		- **原理**: 由于是在编译时优化，因此最基本的前提就是语法的静态分析，**ES6 的模块机制** 提供了这种可能性。不需要运行时，便可进行代码字面上的静态分析，确定相应的依赖关系。
		- **问题**: 具有 **副作用** 的函数无法被 tree-shaking。
			- 在引用一些第三方库，需要去观察其引入的代码量是不是符合预期；
			- 尽量写纯函数，减少函数的副作用；
			- 可使用 webpack-deep-scope-plugin，可以进行作用域分析，减少之类情况的发生，但仍需要注意；
 
- **code-spliting**: **代码分割** 技术，将代码分割成多份进行懒加载或异步加载，避免打包成一份后导致体积过大，影响页面的首屏加载；
	- Webpack 中使用 SplitChunksPlugin 进行拆分；
	- 按 **页面** 拆分: 不同页面打包成不同的文件； 
	- 按 **功能** 拆分: 
		- 将类似于播放器，计算库等大模块进行拆分后再懒加载引入；
		- 提取复用的业务代码，减少冗余代码；
	- 按 **文件修改频率** 拆分: 将第三方库等不常修改的代码单独打包，而且不改变其文件 hash 值，能最大化运用浏览器的缓存；

- **scope hoisting**: **作用域提升**，将分散的模块划分到同一个作用域中，避免了代码的重复引入，有效减少打包后的代码体积和运行时的内存损耗；

- **编译性能优化**:
	- 升级至 **最新** 版本的 webpack，能有效提升编译性能；
	- 使用 **dev-server / 模块热替换 (HMR)** 提升开发体验；
		- 监听文件变动 **忽略 node_modules** 目录能有效提高监听时的编译效率；
	- **缩小编译范围**: 
		- modules: 指定模块路径，减少递归搜索；
		- mainFields: 指定入口文件描述字段，减少搜索；
		- noParse: 避免对非模块化文件的加载；
		- includes/exclude: 指定搜索范围/排除不必要的搜索范围；
		- alias: 缓存目录，避免重复寻址；
	- `babel-loader`:
		- 忽略`node_moudles`，避免编译第三方库中已经被编译过的代码；
		- 使用`cacheDirectory`，可以缓存编译结果，避免多次重复编译；
	- **多进程并发**:
		 - webpack-parallel-uglify-plugin: 可多进程并发压缩 js 文件，提高压缩速度；
		 - HappyPack: 多进程并发文件的 Loader 解析；
	- **第三方库模块缓存**:
		- DLLPlugin 和 DLLReferencePlugin 可以提前进行打包并缓存，避免每次都重新编译；
	- **使用分析**:
		- Webpack Analyse / webpack-bundle-analyzer 对打包后的文件进行分析，寻找可优化的地方；
		- 配置`profile：true`，对各个编译阶段耗时进行监控，寻找耗时最多的地方；
	- `source-map`:
		- 开发: `cheap-module-eval-source-map`；
		- 生产: `hidden-source-map`；

## 项目性能优化

### 1. 编码优化

编码优化，指的就是 在代码编写时的，通过一些 **最佳实践**，提升代码的执行性能。通常这并不会带来非常大的收益，但这属于 **程序猿的自我修养**，而且这也是面试中经常被问到的一个方面。

- **数据读取**:
	- 通过作用域链 / 原型链 读取变量或方法时，需要更多的耗时，且越长越慢；
	- 对象嵌套越深，读取值也越慢；
	- **最佳实践**: 
		- 尽量在局部作用域中进行 **变量缓存**；
		- 避免嵌套过深的数据结构，**数据扁平化** 有利于数据的读取和维护；

- **循环**: 循环通常是编码性能的关键点；
	- 代码的性能问题会再循环中被指数倍放大；
	- **最佳实践**:
		- 尽可能 **减少循环次数**；
			- 减少待遍历的数据量；
			- 完成目的后马上结束循环；
		- 避免在循环中执行大量的运算，避免重复计算，相同的执行结果应该使用缓存；
		- js 中使用 **倒序循环** 会略微提升性能；
		- 尽量避免使用 for-in 循环，因为它会枚举原型对象，耗时大于普通循环；
		
- **条件流程性能**: Map / Object > switch > if-else

```js
// 使用 if-else
if(type === 1) {

} else if (type === 2) {

} else if (type === 3) {

}

// 使用 switch
switch (type) {
	case 1:
		break;4
	case 2:
		break;
	case 3:
		break;
    default:
        break;
}

// 使用 Map
const map = new Map([
	[1, () => {}],
	[2, () => {}],
	[3, () => {}],
])
map.get(type)()

// 使用 Objext
const obj = {
	1: () => {},
	2: () => {},
	3: () => {},
}
obj[type]()
```

- **减少 cookie 体积**: 能有效减少每次请求的体积和响应时间；
	- 去除不必要的 cookie；
	- 压缩 cookie 大小；
	- 设置 domain 与 过期时间；

- **dom 优化**: 
	- **减少访问 dom 的次数**，如需多次，将 dom 缓存于变量中；
	- **减少重绘与回流**:
		- 多次操作合并为一次；
		- 减少对某些计算属性的访问；
		- 大量操作时，可将 dom 脱离文档流或者隐藏，待操作完成后再重新恢复；
		- DocumentFragment / cloneNode / replaceChild 进行操作；
	- 使用事件委托，避免大量的事件绑定；

- **css 优化**: 
	- **层级扁平**，避免过于多层级的选择器嵌套； 
	- **特定的选择器** 好过一层一层查找:  .xxx-child-text{} 优于 .xxx .child .text{}
	- **减少使用通配符与属性选择器**；
	- **减少不必要的多余属性**；
	- 使用 **动画属性** 实现动画，动画时脱离文档流，开启硬件加速，优先使用 css 动画；
	- 使用 `<link>` 替代原生 @import；

- **html 优化**:
	- **减少 dom 数量**，避免不必要的节点或嵌套；
	- **避免`<img src="" />`空标签**，能减少服务器压力，因为 src 为空时，浏览器仍然会发起请求
		- IE 向页面所在的目录发送请求；
		- Safari、Chrome、Firefox 向页面本身发送请求；
		- Opera 不执行任何操作。
	- 图片提前 **指定宽高** 或者 **脱离文档流**，能有效减少因图片加载导致的页面回流；
	- **语义化标签** 有利于 SEO 与浏览器的解析时间；  
	- 减少使用 table 进行布局，避免使用`<br />`与`<hr />`；

### 2. 页面基础优化

- **引入位置**: css 文件`<head>`中引入， js 文件`<body>`底部引入；
	- 影响首屏的，优先级很高的 js 也可以头部引入，甚至内联； 
- **减少请求** (http 1.0 - 1.1)，合并请求，正确设置 http 缓存；
- **减少文件体积**: 
	- **删除多余代码**:
		- tree-shaking
		- UglifyJs
		- code-spliting 
	- **混淆 / 压缩代码**，开启 gzip 压缩；
	- **多份编译文件按条件引入**:
		- 针对现代浏览器直接给 ES6 文件，只针对低端浏览器引用编译后的 ES5 文件；
		- 可以利用`<script type="module"> / <script type="module">`进行条件引入用
	- **动态 polyfill**，只针对不支持的浏览器引入 polyfill；
- **图片优化**:
	- 根据业务场景，与UI探讨选择 **合适质量，合适尺寸**；
	- 根据需求和平台，选择 **合适格式**，例如非透明时可用 jpg；非苹果端，使用 webp；
	- 小图片合成 **雪碧图**，低于 5K 的图片可以转换成 **base64** 内嵌；
	- 合适场景下，使用 **iconfont** 或者 **svg**；
- **使用缓存**:
	- **浏览器缓存**: 通过设置请求的过期时间，合理运用浏览器缓存；
	- **CDN缓存**: 静态文件合理使用 CDN 缓存技术；
		- HTML 放于自己的服务器上；
		- 打包后的图片 / js / css 等资源上传到 CDN 上，文件带上 hash 值；
		- 由于浏览器对单个域名请求的限制，可以将资源放在多个不同域的 CDN 上，可以绕开该限制；
	- **服务器缓存**: 将不变的数据，页面缓存到 内存 或 远程存储(redis等) 上；
	- **数据缓存**: 通过各种存储将不常变的数据进行缓存，介绍数据的获取；

### 3. 首屏渲染优化

- **css / js 分割**，使首屏依赖的文件体积最小，内联首屏关键 css / js；
- 非关键性的文件尽可能的 **异步加载和懒加载**，避免阻塞首页渲染；
- 使用`dns-prefetch / preconnect / prefetch / preload`等浏览器提供的资源提示，加快文件传输；
- 谨慎控制好 **Web字体**，一个大字体包足够让你功亏一篑；
	- 控制字体包的加载时机；
	- 如果使用的字体有限，那尽可能只将使用的文字单独打包，能有效减少体积；  
- 合理利用 Localstorage / server-worker 等存储方式进行 **数据与资源缓存**；
- **分清轻重缓急**:
	- 重要的元素优先渲染；
	- 视窗内的元素优先渲染；
- **服务端渲染(SSR)**:
	- 减少首屏需要的数据量，剔除冗余数据和请求；
	- 控制好缓存，对数据/页面进行合理的缓存；
	- 页面的请求使用流的形式进行传递；
- **优化用户感知**:
	- 利用一些动画 **过渡效果**，能有效减少用户对卡顿的感知；
	- 尽可能利用 **骨架屏(Placeholder) / Loading** 等减少用户对白屏的感知；
	- 动画帧数尽量保证在 **30帧** 以上，低帧数、卡顿的动画宁愿不要；
	- js 执行时间避免超过 **100ms**，超过的话就需要做:
		- 寻找可 缓存 的点； 
		- 任务的 分割异步 或 web worker 执行；

## 结语

不知不觉，下篇也写了整整一个多月了。🙂~ 其实下篇涉及的许多知识点都是有比较深的拓展空间，博主自己也无法面面俱到，也许甚至会有些争议或者错误的见解，还望小伙伴们共同指出和纠正。希望这些能帮助到大家，好好地将这些知识点进行消化和理解，闭关修炼虽然辛苦，但现在已经是时候出山征战江湖，收割 Offer 啦~

上下篇的知识点暂时还是比较局限于前端，后续如果大家想要继续提升，可以往自己感兴趣的方向进行深挖，例如:

- **全栈**: 那可能得更多的去了解 Node / Nginx / 反向代理 / 负载均衡 / PM2 / Docker 等服务端知识；
- **跨平台**: 可以学习 Hybrid / RN / Swift / Flutter 等；
- **视觉游戏**: WebGL / 动画 / Three.js / Canvas / 各大游戏引擎 / VR / AR 等；
- **底层框架**: 浏览器引擎 / 框架底层 / 机器学习 / 算法等；

总之，学无止境呐。。😂。感谢各位小伙伴的观看，共同进步，一起成长！[面试上篇](https://github.com/xd-tayde/blog/blob/master/interview-1.md)。


> Tips: 
> 
> 字节跳动招中高级前端或实习，有兴趣内推的同学可简历邮件至 guoxiaodong.tayde@bytedance.com (标题: 姓名-岗位-地点) 或关注下面公众号加我微信详聊哈。
> 
> 博主真的写得很辛苦，再不 star 下，真的要哭了。~ [github](https://github.com/xd-tayde/blog/blob/master/interview-2.md)。🤑

<img width="750" src="./images/qrcode.jpg">





