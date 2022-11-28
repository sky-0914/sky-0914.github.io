---
title: React基础入门教程
tags:
  - React
  - 前端
categories:
  - 前端
---

# React 基础入门教程

[React中文官网](https://react.docschina.org/)

**demo样例：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TEST</title>
</head>
<body>
<div id="test"></div>
</body>

<!-- react核心库 -->
<script src="https://unpkg.com/react@17/umd/react.production.min.js" crossorigin></script>
<!-- React虚拟DOM，用于支持react操作DOM -->
<script src="https://unpkg.com/react-dom@17/umd/react-dom.production.min.js" crossorigin></script>
<!-- 用于将jsx转为js -->
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

<script type="text/babel">
	//1.创建虚拟DOM
    const DOM = <h1>创建虚拟DOM方式一（JSX）</h1>
    //2.渲染虚拟DOM到页面
    ReactDOM.render(DOM,document.getElementById('test'))
</script>
</html>
```

## JSX规则
```
1. 定义虚拟DOM时，不要写引号。
2. 标签中混入JS表达式时要用{}。
3. 样式的类名指定不要用class，要用className。
4. 内联样式，要用style={{key:value}}的形式去写。
5. 只有一个根标签
6. 标签必须闭合
7. 标签首字母
   1. 若小写字母开头，则将该标签转为html中同名元素，若html中无该标签对应的同名元素，则报错
   2. 若大写字母开头，react就去渲染对应的组件。若组件没有定义，则报错
```

## 创建虚拟DOM方式

### 方式一（JSX）

```jsx
//1.创建虚拟DOM
const DOM = <h1>创建虚拟DOM方式一（JSX）</h1>
//2.渲染虚拟DOM到页面
ReactDOM.render(DOM,document.getElementById('test'))
```

### 方式二（JS）

```js
//1.创建虚拟DOM
const VDOM = React.createElement('h1',{id:'title'},React.createElement('span',{},'Hello,React'))
//2.渲染虚拟DOM到页面
ReactDOM.render(VDOM,document.getElementById('test'))
```

## 组件的定义方式

### 方式一（函数式组件）

```jsx
		//1.创建函数式组件
		function MyComponent(){
			console.log(this); //此处的this是undefined，因为babel编译后开启了严格模式
			return <h2>我是用函数定义的组件(适用于【简单组件】的定义)</h2>
		}
		//2.渲染组件到页面
		ReactDOM.render(<MyComponent/>,document.getElementById('test'))
		/* 
			执行了ReactDOM.render(<MyComponent/>.......之后，发生了什么？
					1.React解析组件标签，找到了MyComponent组件。
					2.发现组件是使用函数定义的，随后调用该函数，将返回的虚拟DOM转为真实DOM，随后呈现在页面中。
		*/
```

### 方式二（类式组件）

```jsx
		//1.创建类式组件
		class MyComponent extends React.Component {
			render(){
				//render是放在哪里的？—— MyComponent的原型对象上，供实例使用。
				//render中的this是谁？—— MyComponent的实例对象 <=> MyComponent组件实例对象。
				console.log('render中的this:',this);
				return <h2>我是用类定义的组件(适用于【复杂组件】的定义)</h2>
			}
		}
		//2.渲染组件到页面
		ReactDOM.render(<MyComponent/>,document.getElementById('test'))
		/* 
			执行了ReactDOM.render(<MyComponent/>.......之后，发生了什么？
					1.React解析组件标签，找到了MyComponent组件。
					2.发现组件是使用类定义的，随后new出来该类的实例，并通过该实例调用到原型上的render方法。
					3.将render返回的虚拟DOM转为真实DOM，随后呈现在页面中。
		*/
```

## 组件实例的三大属性

### State（状态）

> React 把组件看成是一个状态机（State Machines）。通过与用户的交互，实现不同状态，然后渲染 UI，让用户界面和数据保持一致。
>
> React 里，只需更新组件的 state，然后根据新的 state 重新渲染用户界面（不要操作 DOM）。

```jsx
class TestComponent extends React.Component {
    //在组件中定义state属性，并且给它赋值。state必须是个对象
    state = {
        flag: false
    }
    changeFlag(flag){
        console.log("changeFlag")
        //必须通过父类的setState()方法重新赋值，React才会重新去调用render()方法重新渲染页面
        //【严重注意】：状态必须通过setState进行更新,且更新是一种合并，不是替换。
        this.setState({flag: flag})
        //【严重注意】：状态state属性不可直接更改，下面这行就是直接更改！！！
		//this.state.flag = !flag //这是错误的写法
    }
	render() {
        console.log("render")
        //取出赋值属性
        const {flag} = this.state
        return (
            <div>
                <h1>今天天气好{flag ? "炎热" : "凉爽"}</h1>
                <button onClick={() => this.changeFlag(!this.state.flag)}>点击我</button>
            </div>
        )
    }
}
ReactDOM.render(<TestComponent/>, document.getElementById("test"))
```

### Props（传参）

> state 和 props 主要的区别在于 **props** 是不可变的，而 state 可以根据与用户交互来改变。
>
> 这就是为什么有些容器组件需要定义 state 来更新和修改数据。 而子组件只能通过 props 来传递数据。

```jsx
class TestComponent extends React.Component {
    render() {
        console.log("render")
        //取出传过来参数
        const {className, grade, list} = this.props
        return (
            <div>
                 <h2>{schoolName}-年级：{grade}-{classNum}班</h2>
                <div>
                    {
                        list.map((item, index) => {
                            return (
                                <ul key={item.id} onClick={() => this.alertInfo(item)}>
                                    <li>姓名：{item.name}</li>
                                    <li>年龄：{item.age}</li>
                                </ul>
                            )
                        })
                    }
                </div>
            </div>
        )
    }
}
const cla = {
    schoolName: "XX高级中学",
    classNum: 1,
    grade: "高中",
    list: [{id: 1, name: "张三", age: "18"}, {id: 2, name: "李四", age: 18}, {id: 3, name: "王五", age: 20}]
}
ReactDOM.render(<TestComponent {...cla}/>, document.getElementById("test"))
```

#### 参数限制

> 自 React v15.5 起，`React.PropTypes` 已移入另一个包中。请使用 [`prop-types` 库](https://www.npmjs.com/package/prop-types) 代替。

```js
class TestComponent extends React.Component{}...//组件类
//对标签属性进行类型、必要性的限制
TestComponent.propTypes = {
    schoolName: PropTypes.string.isRequired,    //限制schoolName必传，且为字符串
    classNum: PropTypes.number,                 //限制classNum为数值
    afterSchool: PropTypes.func.isRequired,     //限制afterSchool为函数
}

//指定默认标签属性值
TestComponent.defaultProps = {
    grade:'高中',   //grade默认值
    classNum:1,         //classNum默认值
}

const cla = {
    schoolName:"江宁高级中学",
    classNum: 1,
    // grade: "高三",
    list: [{id: 1, name: "张三", age: "18"}, {id: 2, name: "李四", age: 18}, {id: 3, name: "王五", age: 20}]
}
ReactDOM.render(<TestComponent {...cla} afterSchool={afterSchool}/>, document.getElementById("test"))

function afterSchool() {
    console.info("放学啦。。。");
}
```

**函数式组件使用**

```js
//创建组件
function Person (props){
    const {name,age,sex} = props
    return (
        <ul>
            <li>姓名：{name}</li>
            <li>性别：{sex}</li>
            <li>年龄：{age}</li>
        </ul>
	)
}
Person.propTypes = {
    name:PropTypes.string.isRequired, //限制name必传，且为字符串
    sex:PropTypes.string,//限制sex为字符串
    age:PropTypes.number,//限制age为数值
}

//指定默认标签属性值
Person.defaultProps = {
    sex:'男',//sex默认值为男
    age:18 //age默认值为18
}
//渲染组件到页面
ReactDOM.render(<Person name="jerry"/>,document.getElementById('test'))
```

### Ref（可以获取真实DOM节点）

> React提供的这个`ref`属性，**表示为对组件真正实例的引用，其实就是`ReactDOM.render()返回的组件实例`**；需要区分一下，`ReactDOM.render()`渲染组件时返回的是组件实例；而渲染dom元素时，返回是具体的dom节点。
>
> **通过event.target得到发生事件的DOM元素对象 ，不要过度使用ref**
>
> 官方说明：你可能首先会想到使用 refs 在你的 app 中“让事情发生”。如果是这种情况，请花一点时间，认真再考虑一下 state 属性应该被安排在哪个组件层中。通常你会想明白，让更高的组件层级拥有这个 state，是更恰当的。查看 [状态提升](https://react.docschina.org/docs/lifting-state-up.html) 以获取更多有关示例


#### 字符串形式

```js
class TestComponent extends React.Component {
    test1 = (e) => {
        //字符串形式已经被弃用
        let {test1} = this.refs;
        console.info(test1, test1.value)
    }
    //函数默认会带上当前事件绑定的DOM元素
    test2 = (e) => {
        console.info(e, e.target.value)
    }
    render() {
        return (
            <div>
                <input ref="test1" placeholder="test1"/>
                <br/>
                <button onClick={this.test1}>点击获取test1数据</button>
                <br/>
                <input placeholder="失去焦点获取当前输入数据" onBlur={this.test2}/>
            </div>
        )
    }
}
ReactDOM.render(<TestComponent/>, document.getElementById("test"))
```

#### 回调函数形式

```js
//回调函数形式：此种方式第一次都会默认调用一次，会有执行次数的问题
class TestComponent extends React.Component {
    test1 = (e) => {
        //字符串形式已经被弃用
        let {input1} = this;
        console.info(input1, input1.value)
    }
    //函数默认会带上当前事件绑定的DOM元素
    test2 = (e) => {
        console.info(e, e.target.value)
    }
    // 方式二
    saveInput = (a)=>{
        this.input1 = a;
        console.log('@',a);
    }
    render() {
        return (
            <div>
                {/*JSX里的注释得这么写*/}
                {/* 方式一 */}
                {/* a标识当前元素，{}方法体里表示：将当前元素a赋值给当前实例中的属性input1 */}
                {/* <input ref={a => {this.input1 = a;console.log('@',a)}} placeholder="test1"/>*/}
                {/* 方式二 */}
                <input ref={this.saveInput}  placeholder="test1"/>
                <br/>
                <button onClick={this.test1}>点击获取test1数据</button>
                <br/>
                <input placeholder="失去焦点获取当前输入数据" onBlur={this.test2}/>
            </div>
        )
    }
}
ReactDOM.render(<TestComponent/>, document.getElementById("test"))
```

#### `React.createRef()`形式

```js
//React 16.3 版本引入的 React.createRef() API
//Refs 是使用 React.createRef() 创建的，并通过 ref 属性附加到 React 元素。在构造组件时，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。
class TestComponent extends React.Component {
    input1 = React.createRef();
    input2 = React.createRef();
    test1 = (e) => {
        console.info(this.input1, this.input1.current.value)
    }
    //函数默认会带上当前事件绑定的DOM元素
    test2 = (e) => {
        console.info(e, this.input2.current.value)
    }
    render() {
        return (
            <div>
                <input ref={this.input1} placeholder="test1"/>
                <br/>
                <button onClick={this.test1}>点击获取test1数据</button>
                <br/>
                <input ref={this.input2} placeholder="失去焦点获取当前输入数据" onBlur={this.test2}/>
            </div>
        )
    }
}
ReactDOM.render(<TestComponent/>, document.getElementById("test"))
```

## React的受控组件和非受控组件

> 受控组件就是可以被 react 状态控制的组件
> 在 react 中，Input textarea 等组件默认是非受控组件（输入框内部的值是用户控制，和React无关）。但是也可以转化成受控组件，就是通过 onChange 事件获取当前输入内容，将当前输入内容作为 value 传入，此时就成为受控组件。
> 好处：可以通过 onChange 事件控制用户输入，使用正则表达式过滤不合理输入。
>
> **React没有实现数据的双向绑定，一句话概括：双向数据绑定就是受控组件**

## 高阶函数定义

> 如果一个函数符合下面2个规范中的任何一个，那该函数就是高阶函数。
>
> 1. 若某xx函数，接收的参数是一个函数，那么xx函数就可以称之为高阶函数。
> 2. 若某xx函数，调用的返回值依然是一个函数，那么xx函数就可以称之为高阶函数。
>
> 常见的高阶函数有：Promise、setTimeout、arr.map()等等。。。

## React生命周期

### （旧）生命周期

![](https://i.loli.net/2021/03/24/qY2bIOLzhsgK1fV.png)

#### （旧）生命周期详解：

> 18版本可能会移除的钩子函数：
>
> ~~`componentWillMount()`【组件将要挂载的钩子】已弃用~~
>
> ~~`componentWillUpdate()`【组件将要更新的钩子】已弃用~~
>
> ~~`componentWillReceiveProps(props)`【子组件将要接收新的props的钩子】已弃用~~
>
> 17版本还可以用，但是必须得加上`UNSAFE_`前缀
>
> 举例：`UNSAFE_componentWillMount()`

1. 初始化阶段: 由`ReactDOM.render()`触发---初次渲染
   1. `constructor(props)`【构造器】
   2. ~~`componentWillMount()`【组件将要挂载的钩子】已弃用~~
   3. **`render()`【将虚拟DOM渲染到页面】（必须用）**
   4. **`componentDidMount()` 【组件挂载完毕的钩子】（常用：一般在这个钩子中做一些初始化的事，例如：开启定时器、发送网络请求、订阅消息）**
   
2. 更新阶段: 由组件内部`this.setSate()`或父组件`render()`触发
   1. `shouldComponentUpdate()`【控制组件更新的“阀门”，有返回值：true/false】
   2. ~~`componentWillUpdate()`【组件将要更新的钩子】已弃用~~
   3. **`render()`【将虚拟DOM渲染到页面】（必须用）**
   4. `componentDidUpdate()`【组件更新完毕的钩子】
   
   > ~~`componentWillReceiveProps(props)`【子组件将要接收新的props的钩子】已弃用~~
   
3. 卸载组件: 由`ReactDOM.unmountComponentAtNode({真实DOM元素})`触发
   
   > 示例：ReactDOM.unmountComponentAtNode(document.getElementById('test'))
   
   1. **`componentWillUnmount()` 【组件将要卸载的钩子】 （常用：一般在这个钩子中做一些收尾的事，例如：关闭定时器、取消订阅消息）**

**代码示例：**

```js
class TestComponent extends React.Component {
    /**
     * 构造器
     * @param props
     */
    constructor(props) {
        console.log('constructor-构造器', props);
        super(props)
        this.state = {count: 0}
    }

    /**
     * 组件将要挂载的钩子
     */
    componentWillMount() {
        console.info("componentWillMount-组件将要挂载的钩子")
    }

    /**
     * 组件挂载完毕的钩子
     */
    componentDidMount() {
        console.info("componentDidMount-组件挂载完毕的钩子-（常用：一般在这个钩子中做一些初始化的事，例如：开启定时器、发送网络请求、订阅消息）")
    }

    /**
     * 控制组件更新的“阀门”
     * @returns {boolean} true：继续、false：不走了
     */
    shouldComponentUpdate() {
        console.info("shouldComponentUpdate-控制组件更新的“阀门”，有返回值：true/false")
        //返回值控制接下来的钩子函数走向，true：继续、false：不走了
        return this.state.count % 2 === 0;
    }

    /**
     * 组件将要更新的钩子
     */
    componentWillUpdate() {
        console.info("componentWillUpdate-组件将要更新的钩子")
    }

    /**
     * 组件更新完毕的钩子
     */
    componentDidUpdate() {
        console.log('componentDidUpdate-组件更新完毕的钩子');
    }

    /**
     * 将虚拟DOM渲染到页面
     */
    render() {
        console.info("render-将虚拟DOM渲染到页面")
        let {count} = this.state;
        return (
            <div>
                <h1>我是父组件，当前数字：{count}</h1>
                <button onClick={() => this.addCount()}>点我+1</button>
                <button onClick={() => this.forceUpdate()}>强制更新</button>
                <button onClick={() => ReactDOM.unmountComponentAtNode(document.getElementById('test'))}
                        style={{backgroundColor: "red"}}>卸载组件
                </button>
                <A count={this.state.count}></A>
            </div>
        )
    }

    /**
     * 组件将要卸载的钩子
     * @param props
     */
    componentWillUnmount() {
        console.info("componentWillUnmount-组件将要卸载的钩子-（常用：一般在这个钩子中做一些收尾的事，例如：关闭定时器、取消订阅消息）")
    }

    //=============================================================
    //自定义方法
    addCount() {
        this.setState({count: this.state.count + 1});
    }
}

//子组件
class A extends React.Component {
    /**
     * 组件将要接收新的props的钩子
     * @param props
     */
    componentWillReceiveProps(props) {
        console.log('A---componentWillReceiveProps', props);
    }

    //控制组件更新的“阀门”
    shouldComponentUpdate() {
        console.log('A---shouldComponentUpdate');
        return true
    }

    //组件将要更新的钩子
    componentWillUpdate() {
        console.log('A---componentWillUpdate');
    }

    //组件更新完毕的钩子
    componentDidUpdate() {
        console.log('A---componentDidUpdate');
    }

    render() {
        return (
            <div>
                <h2>我是子组件“A”，父组件传过来的值是：{this.props.count}</h2>
            </div>
        )
    }
}

ReactDOM.render(<TestComponent a="aaa" b="bbb"/>, document.getElementById("test"));
```

### （新）生命周期

***展示不常用生命周期：**

![](https://i.loli.net/2021/03/24/jbJCSoNipsql2X7.png)

**展示常用生命周期：**

![](https://i.loli.net/2021/03/26/l3HKgexf7y2wnmc.png)

#### （新）生命周期详解：

[官方文档：生命周期](https://react.docschina.org/docs/react-component.html#shouldcomponentupdate)

1. 初始化阶段: 由`ReactDOM.render()`触发---初次渲染
	1. `constructor(props)`【构造器】
	
	2. `getDerivedStateFromProps(props,state)`
	
	   > 代码示例：
	   >
	   > ```js
	   > static getDerivedStateFromProps(props,state){
	   >     console.log('getDerivedStateFromProps',props,state);
	   >     //返回快照值
	   >     return null
	   > }
	   > ```
	   >
	   > 新增（极少用到）
	   >
	   > 若state的值在任何时候都取决于props，那么可以使用getDerivedStateFromProps
	
	3. **`render()`【将虚拟DOM渲染到页面】（必须用）**
	
	4. **`componentDidMount()` 【组件挂载完毕的钩子】（常用：一般在这个钩子中做一些初始化的事，例如：开启定时器、发送网络请求、订阅消息）**
	
2. 更新阶段: 由组件内部`this.setSate()`或父组件重新`render()`触发
     1. `getDerivedStateFromProps(props,state)`

        > 代码示例：
        >
        > ```js
        > static getDerivedStateFromProps(props,state){
        >     console.log('getDerivedStateFromProps',props,state);
        >     //返回快照值
        >     return null
        > }
        > ```
        >
        > 新增（极少用到）
        >
        > 若state的值在任何时候都取决于props，那么可以使用getDerivedStateFromProps

     2. `shouldComponentUpdate()`【控制组件更新的“阀门”，有返回值：true/false】

     3. **`render()`【将虚拟DOM渲染到页面】（必须用）**

     4. `getSnapshotBeforeUpdate(prevProps, prevState)`【在更新之前获取快照】

        > 新增（极少用到）
        >
        > 使用场景：类似微信朋友圈，朋友圈消息在不断更新动态新增时，刷到中间时停住，同时，消息也在不断新增。

     5. `componentDidUpdate(prevProps, prevState, snapshot)`【组件更新完毕的钩子】

3. 卸载组件: 由`ReactDOM.unmountComponentAtNode({真实DOM元素})`触发

   > 示例：ReactDOM.unmountComponentAtNode(document.getElementById('test'))

   1. **`componentWillUnmount()` 【组件将要卸载的钩子】 （常用：一般在这个钩子中做一些收尾的事，例如：关闭定时器、取消订阅消息）**

**代码示例：**

```js
class TestComponent extends React.Component {
    /**
     * 构造器
     * @param props
     */
    constructor(props) {
        console.log('constructor-构造器', props);
        super(props)
        this.state = {count: 0}
    }

    /**
     * 若state的值在任何时候都取决于props
     * @param props
     * @param state
     */
    static getDerivedStateFromProps(props, state) {
        console.log('getDerivedStateFromProps-若state的值在任何时候都取决于props', props, state);
        // return {count: state.count + 1};
        return null;
    }

    /**
     * 控制组件更新的“阀门”
     * @param props
     * @param state
     * @param value
     * @returns {boolean} true：继续、false：不走了
     */
    shouldComponentUpdate(props, state, value) {
        console.info("shouldComponentUpdate-控制组件更新的“阀门”，有返回值：true/false", props, state, value)
        //返回值控制接下来的钩子函数走向，true：继续、false：不走了
        return this.state.count % 2 === 0;
    }

    /**
     * 组件挂载完毕的钩子
     */
    componentDidMount() {
        console.info("componentDidMount-组件挂载完毕的钩子-（常用：一般在这个钩子中做一些初始化的事，例如：开启定时器、发送网络请求、订阅消息）")
    }

    /**
     * 组件更新完毕的钩子
     * @param prevProps
     * @param prevState
     * @param snapshot
     */
    componentDidUpdate(prevProps, prevState, snapshot) {
        console.log('componentDidUpdate-组件更新完毕的钩子', prevProps, prevState, snapshot);
    }

    /**
     * 将虚拟DOM渲染到页面
     */
    render() {
        console.info("render-将虚拟DOM渲染到页面")
        let {count} = this.state;
        return (
            <div>
                <h1>我是父组件，当前数字：{count}</h1>
                <button onClick={() => this.addCount()}>点我+1</button>
                <button onClick={() => this.forceUpdate()}>强制更新</button>
                <button onClick={() => ReactDOM.unmountComponentAtNode(document.getElementById('test'))}
                        style={{backgroundColor: "red"}}>卸载组件
                </button>
            </div>
        )
    }

    /**
     * 在更新之前获取快照
     */
    getSnapshotBeforeUpdate() {
        console.log('getSnapshotBeforeUpdate-在更新之前获取快照');
    }

    /**
     * 组件将要卸载的钩子
     */
    componentWillUnmount() {
        console.info("componentWillUnmount-组件将要卸载的钩子-（常用：一般在这个钩子中做一些收尾的事，例如：关闭定时器、取消订阅消息）")
    }

    //=============================================================
    //自定义方法
    addCount() {
        this.setState({count: this.state.count + 1});
    }
}

ReactDOM.render(<TestComponent a="aaa" b="bbb"/>, document.getElementById("test"));
```

## Diffing算法

**diff算法其实就是对DOM进行different比较不同的一种算法(虚拟的比较更节约性能) 补丁:用来更新DOM的任务**

![](https://i.loli.net/2021/03/26/5humsyKqxLcRCP8.png)

**遍历规则：先序深度优先遍历(从根节点向下级子节点遍历)**

> 0是根节点、1 2 5叫广度优先、1 2 3 4 5深度优先

![](https://i.loli.net/2021/03/26/91itJMZoAcTDa78.png)

## For循环为什么key不能用index索引值作为key

> 可能会出现严重的效率问题

```js

	/*
   经典面试题:
      1). react/vue中的key有什么作用？（key的内部原理是什么？）
      2). 为什么遍历列表时，key最好不要用index?

			1. 虚拟DOM中key的作用：
					1). 简单的说: key是虚拟DOM对象的标识, 在更新显示时key起着极其重要的作用。

					2). 详细的说: 当状态中的数据发生变化时，react会根据【新数据】生成【新的虚拟DOM】,
												随后React进行【新虚拟DOM】与【旧虚拟DOM】的diff比较，比较规则如下：

									a. 旧虚拟DOM中找到了与新虚拟DOM相同的key：
												(1).若虚拟DOM中内容没变, 直接使用之前的真实DOM
												(2).若虚拟DOM中内容变了, 则生成新的真实DOM，随后替换掉页面中之前的真实DOM

									b. 旧虚拟DOM中未找到与新虚拟DOM相同的key
												根据数据创建新的真实DOM，随后渲染到到页面

			2. 用index作为key可能会引发的问题：
								1. 若对数据进行：逆序添加、逆序删除等破坏顺序操作:
												会产生没有必要的真实DOM更新 ==> 界面效果没问题, 但效率低。

								2. 如果结构中还包含输入类的DOM：
												会产生错误DOM更新 ==> 界面有问题。

								3. 注意！如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，
									仅用于渲染列表用于展示，使用index作为key是没有问题的。

			3. 开发中如何选择key?:
								1.最好使用每条数据的唯一标识作为key, 比如id、手机号、身份证号、学号等唯一值。
								2.如果确定只是简单的展示数据，用index也是可以的。
   */

	/*
		慢动作回放----使用index索引值作为key

			初始数据：
					{id:1,name:'小张',age:18},
					{id:2,name:'小李',age:19},
			初始的虚拟DOM：
					<li key=0>小张---18<input type="text"/></li>
					<li key=1>小李---19<input type="text"/></li>

			更新后的数据：
					{id:3,name:'小王',age:20},
					{id:1,name:'小张',age:18},
					{id:2,name:'小李',age:19},
			更新数据后的虚拟DOM：
					<li key=0>小王---20<input type="text"/></li>
					<li key=1>小张---18<input type="text"/></li>
					<li key=2>小李---19<input type="text"/></li>

	-----------------------------------------------------------------

	慢动作回放----使用id唯一标识作为key

			初始数据：
					{id:1,name:'小张',age:18},
					{id:2,name:'小李',age:19},
			初始的虚拟DOM：
					<li key=1>小张---18<input type="text"/></li>
					<li key=2>小李---19<input type="text"/></li>

			更新后的数据：
					{id:3,name:'小王',age:20},
					{id:1,name:'小张',age:18},
					{id:2,name:'小李',age:19},
			更新数据后的虚拟DOM：
					<li key=3>小王---20<input type="text"/></li>
					<li key=1>小张---18<input type="text"/></li>
					<li key=2>小李---19<input type="text"/></li>


	 */
	class Person extends React.Component{

		state = {
			persons:[
				{id:1,name:'小张',age:18},
				{id:2,name:'小李',age:19},
			]
		}

		add = ()=>{
			const {persons} = this.state
			const p = {id:persons.length+1,name:'小王',age:20}
			this.setState({persons:[p,...persons]})
		}

		render(){
			return (
				<div>
					<h2>展示人员信息</h2>
					<button onClick={this.add}>添加一个小王</button>
					<h3>使用index（索引值）作为key</h3>
					<ul>
						{
							this.state.persons.map((personObj,index)=>{
								return <li key={index}>{personObj.name}---{personObj.age}<input type="text"/></li>
							})
						}
					</ul>
					<hr/>
					<hr/>
					<h3>使用id（数据的唯一标识）作为key</h3>
					<ul>
						{
							this.state.persons.map((personObj)=>{
								return <li key={personObj.id}>{personObj.name}---{personObj.age}<input type="text"/></li>
							})
						}
					</ul>
				</div>
			)
		}
	}

	ReactDOM.render(<Person/>,document.getElementById('test'))
```

