---
title: React学习笔记
date: 2019-01-19 18:15:15
tags: javascript
categories:
- js
---
### 一、React初探

#####es6写法 [code](https://codepen.io/xiaobinwu/pen/mKagVy)
```javascript
  import React from 'react';
  import ReactDOM from 'react-dom';
  import PropTypes from 'prop-types';
  
  class App extends React.Component { 
      state = {
          title: '环球大前端'
      }
      render() {
          const { title } = this.state;
          const { name } = this.props
          return (
            <div>
                <h2>{title}</h2>
                <p> Hello {name}! </p>
            </div>
          )
      }
  }
  App.propTypes = {
      name: PropTypes.string
  }
  App.defaultProps = {
      name: '帅气小伙子'
  }
  ReactDOM.render(<App name="24小清新" />, document.getElementById('app'));
```

#####es5写法（遗憾的是现在最新版本的react，已经不再能使用createClass去创建react组件了，由于react将createClass剥离出去，减少react的体积，我们可以使用create-react-class 模块来创建  [code](https://codepen.io/xiaobinwu/pen/gKZygr)）
```javascript
var App = React.createClass({
    getDefaultProps: function() {
      return {
        name: '帅气小伙子'  
      }  
    },
    getInitialState: function() {
        return {
            title: '环球大前端'
        }
    },
    render: function() {
      return (
        <div>
            <h2>{this.state.title}</h2>
            <p> Hello {this.props.name}! </p>
        </div>
      )
    }
})
React.render(<App name="24小清新" />, document.getElementById('app'));
```
核心思想：封装组件，各个组件维护自己的状态(state, prop)和UI,当状态变更，自动重新渲染组件，数据流向是单向的。

需要明白的几个基础概念：

1、什么是JSX?

2、如何修改组件state，从而修改组件UI?

3、事件处理

对于上述那些既不是字符串也不是 HTML的的标签语法，被称为JSX，是一种 JavaScript 的语法扩展，用来描述用户界面。

常用的是在JSX中使用表达式，例如 2 + 2， user.firstName， 以及 formatName(user)，条件判断（三目运算符、&&）, 数组Map函数遍历获取React元素 都是可以使用的。如：
```javascript
    formatName(user) {
        return `${user.firstName}-${user.name}`;
    }
    const user = {
        firstName: 'wu',
        name: 'shaobin'
    }
    const show = true; //我可以是this.state属性哦！！！
    const arr = ['xiaobin', 'kaizi', 'liujun'];
    const element = (
      <div>
        <h1>Hello, {formatName(user)}!</h1>
        <h1>Hello, {user.name}!</h1>
        <h1>Hello, { 1 + 1 }!</h1>
        <h1>Hello, { show ? 'I am show' : null }</h1>
        <h1>Hello, { arr.length > 0 && <span>数组长度大于0</span> }</h1>
        {
            arr.map((item, index) => {
                return <span key={item}>item</span>    
            })
        }
        //记住数组Map函数遍历获取React元素的时候，必须要记得必须➕keys属性
        // 为啥呀？
        //Keys可以在DOM中的某些元素被增加或删除的时候帮助React识别哪些元素发生了变化。因此你应当给数组中的每一个元素赋予一个确定的标识。没有唯一值的时候可以使用index，但是官方不建议，会导致渲染变慢。
      </div>
    );
    ReactDOM.render(
      element,
      document.getElementById('root')
    );
```
切记每个组件的render函数返回的JSX结构都需要根元素去包裹着，当然也有例外，如[React.Fragment](https://codepen.io/xiaobinwu/pen/mKvRVG)

对于JSX,react最终经babel的转换会调用React.createElement相应api转换成react能识别的对象，如上述例子转换后得到：

```javascript
React.createElement(
    'div',
    null,
    React.createElement(
        'h2', //可以是一个html标签名称字符串,也可以是也可以是一个 React component 类型
        null,
        title
    ),
    React.createElement(
        'p',
        null,
        ' Hello ',
        name,
        '! '
    )
);
```
[babel查看es6->es5的结果](https://babeljs.io/repl)

既然我们可以为组件初始化状态，也必须要能够去改变它，以达到改变视图。
当然`this.state.xxx = xxx`不会触发渲染组件的动作，而是使用`this.setState({ xxx: xxx })`方法来修改状态，同时多个setState() 调用合并成一个调用能提高性能。

对于事件处理，需要注意的一点就是this的绑定，其他跟普通Dom绑定监听事件一样，this的绑定有以下几种方式：

1. 在构造函数中使用bind绑定this
2. 在调用的时候使用bind绑定this
3. 在调用的时候使用箭头函数绑定this
4. 使用属性初始化器语法绑定this

[setState与事件处理的例子](https://codepen.io/xiaobinwu/pen/bKzYZd)

同时需要注意的是，React合成事件（onClick={}）和原生事件（document.addEventListener）的阻止冒泡存在差异，需要明白的是，所有合成事件都是绑定在document上的（代理），所以执行合成事件中的event.stopPropagation()，实际原生事件还是会冒泡到document上，同时需要注意的是，setState 只在合成事件和生命周期函数中是 "异步" 的，在原生事件和 setTimeout 中都是同步的。
[合成事件与原生事件列子](https://codepen.io/xiaobinwu/pen/gBRZRQ)
参考文章：
[从 Dropdown 的 React 实现中学习到的](https://juejin.im/post/5bb1812a6fb9a05d082a3361)
[React合成事件和DOM原生事件混用须知](https://juejin.im/post/59db6e7af265da431f4a02ef)



### 二、React进阶

1、有哪些生命周期，生命周期的执行顺序？

2、Ref的引用

3、高阶组件的使用

[生命周期图示(React16)：](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

![](https://user-gold-cdn.xitu.io/2018/7/31/164ee7c89679c44b?w=1138&h=680&f=png&s=69928)

#### Mounting/挂载
constructor()    //React组件的构造函数，用于super(props)，初始化state，bind绑定事件等

static getDerivedStateFromProps()

UNSAFE_componentWillMount()    // 组件挂载前(组件渲染到页面前)

render()    // 渲染函数，不做实际的渲染动作，它只是返回一个JSX描述的结构，生成虚拟Dom树，执行patch，最终由React来操作渲染过程

componentDidMount()  //组件挂载后(组件渲染到页面上)，可以在这个钩子添加异步请求以及定时器,或是socket连接

#### Updating/更新
UNSAFE_componentWillReceiveProps() // 组件接收到属性时触发

static getDerivedStateFromProps()

shouldComponentUpdate(prevProps, prevState)  // 当组件接收到新属性，或者组件的状态发生改变时触发。组件首次渲染时并不会触发，此钩子可做性能优化

UNSAFE_componentWillUpdate()  //组件即将被更新时触发

render() // 渲染函数，不做实际的渲染动作，它只是返回一个JSX描述的结构，生成虚拟新Dom树，与旧树进行diff， 执行patch

getSnapshotBeforeUpdate()

componentDidUpdate(prevProps, prevState, snapshot) // 组件被更新完成后触发，生命周期中由于state的变化触发请求，在componentDidUpdate中进行

#### Unmounting/卸载

componentWillUnmount()  // 卸载组件，注销监听事件或是定时器，socket关闭

详细看[生命周期例子](https://codepen.io/xiaobinwu/pen/Qxowzq)

#### [补充Tip](https://juejin.im/post/5aca20c96fb9a028d700e1ce)：
[getSnapshotBeforeUpdate()](https://doc.react-china.org/docs/react-component.html#getsnapshotbeforeupdate)与[static getDerivedStateFromProps()](https://doc.react-china.org/docs/react-component.html#static-getderivedstatefromprops)两个新增生命周期钩子是被用来代替 UNSAFE_componentWillMount() ，UNSAFE_componentWillUpdate()， UNSAFE_componentWillReceiveProps()三个生命周期的，但是这个三个生命周期仍是可以使用。为什么勒？React为了1.7版本实现Async Rendering。


Refs 提供了一种方式，用于访问在 render 方法中创建的 DOM 节点或 React 元素，官方建议少用。获取Ref有三种场景：

![](https://user-gold-cdn.xitu.io/2018/7/31/164eee2ba2bc9294?w=725&h=394&f=png&s=18656)
获取Ref的常用方式（通过this.myRef.current来获取Dom节点或实例）：
```javascript
class MyComponent extends React.Component {
   constructor(props) {
     super(props);
     this.myRef = React.createRef(); // 调用React.createRef API
   }
   render() {
     return <input ref={this.myRef} />;
   }
}
```
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <input ref={(ref) => { this.myRef = ref; }} />;
  }
}
```

[Ref获取Dom元素例子](https://codepen.io/xiaobinwu/pen/mKoLzy)， [Ref获取React元素（子组件实例）例子](https://codepen.io/xiaobinwu/pen/eKXjxB)。

补充Tip： 

对于第三种情况，获取子组件的Dom节点，官方有提供Forwarding Refs（转发Ref）的方法，来获取子组件的Dom节点的Ref，此方法返回的是一个React元素，对应方法为
`React.forwardRef((props, ref) => { ... })`

[Ref获取子组件Dom元素或是React元素的ref例子](https://codepen.io/xiaobinwu/pen/pKYOeR)

[高阶组件](https://doc.react-china.org/docs/higher-order-components.html)（HOC）是react中对组件逻辑进行重用的高级技术。但高阶组件本身并不是React API。它只是一种模式，其实就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件。高阶组件在React第三方库中很常见，比如Redux的connect方法和react-router的withRouter()方法。

注意事项：

1、不要再render函数中使用高阶组件，不然会导致每次重新渲染，都会重新创建高阶组件实例，销毁掉旧的高阶组件，导致所有状态和子组件都被卸载。

2、必须将静态方法做拷贝，当使用高阶组件包装组件，原始组件被容器组件包裹，得到新组件会丢失原始组件的所有静态方法，假如原始组件有静态方法，可以使用hoist-non-react-statics进行静态方法拷贝。[例子](https://codepen.io/xiaobinwu/pen/jKJJmo)

3、Refs属性不能传递，高阶组件可以传递所有的props属性给包裹的组件，但是不能传递refs引用，但是有时候我们确实需要把ref的引用传给包裹组件，可以传一个非ref命名的props属性给到高阶组件上，由高阶组件绑定到包裹组件的ref上，也可以使用转发Ref。[例子](https://codepen.io/xiaobinwu/pen/YvgMab)

### 三、React第三方库

#### 1、[Redux](https://unpkg.com/redux@4.0.0/lib/redux.js)与[react-redux](https://cdnjs.cloudflare.com/ajax/libs/react-redux/5.0.7/react-redux.js)

Redux主要分成三部分，分别为Store，Action，Reducer，下面是对三部分的通俗的讲解：

Store：Redux应用只有一个单一的Store，就是单一数据源，将整个应用共享的状态state储存在一棵对象树上面，注意的是，对象树上面的state是只读的，只能通过纯函数来执行修改，创建store，是通过Redux的 createStore（reducer）方法来创建的，store里面会有getState()、dispatch()、subscribe(listener)的方法。

Action：一个普通的Javascript对象，描述了应用state发生了什么变化，通过dispatch方法来通知store调用reducer方法。

Reducer：描述应用如何更新state，本身是一个函数，接受Action参数，返回新的state。

[不结合react-redux的Redux使用例子](https://codepen.io/xiaobinwu/pen/YvMLrd)

需要明白的一点Redux跟React一点关系都没有，但是React搭配Redux来实现状态管理时最好的实现方案。那么如何搭配呢？本来我们可以subscribe(listener)在react的组件注册redux的监听器，但是这种方式繁琐，而且会导致多次渲染。所以搭配着react-redux来使用。基本使用如下：
``` javacript
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import App from './components/App'

const todoApp = (state = {}, action) {
   if (actions.type === 'SHOW') {
       return Object.assign({}, state, {
        show: action.show
       });
   }
   return state;
}

let store = createStore(todoApp)
render(
  // 使用指定的 React Redux 组件 <Provider> 来让所有容器组件都可以访问 store，而不必显示地传递它。只需要在渲染根组件时使用即可。
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

```javascript
import React from 'react'
import { connect } from 'react-redux';
import Child from './components/Child'
class App extends Component {
    render() {
        const { show } = this.props;
        return (
            <div>
                <Child show={show }/>
            </div>
        );
    }
}
const stateToProps = (state) => ({
    show: state.show
});

export default connect(stateToProps)(App);
```

[结合react-redux的例子](https://codepen.io/xiaobinwu/pen/PagXWg)

react-redux内容实现原理，使用的[Context API](https://doc.react-china.org/docs/legacy-context.html#如何使用context)，简单来说，父组件声明需要跨层级组件传递的属性（childContextType）以及监听属性变化的getChildContext()函数，子组件声明可以接收上层组件传递的属性（contextType）。

如果存在多个reducer的话，可以使用redux中的combineReducers进行合并，代码如下：
```javascript
import { createStore, combineReducers } from 'redux';
const todoApp = combineReducers({
  reducer1,
  reducer2
})
const store = createStore(todoApp);
```
代码等价于：
```javascript
import { createStore} from 'redux';
const todoApp = (state = {}, action) => {
  return {
    reducer1: state.reducer1,
    reducer2: state.reducer2
  }
}
const store = createStore(todoApp);
```
combineReducers最终只是生成一个函数，这个函数来调用你的一系列 reducer，每个 reducer 根据它们的 key 来筛选出 state 中的一部分数据并处理，然后这个生成的函数再将所有 reducer 的结果合并成一个大的对象。详细逻辑可以看[redux源码](https://cdnjs.cloudflare.com/ajax/libs/redux/4.0.0/redux.js)。

#### 2、react-router

对于路由规则，我们在项目里面搭配的是react-router v4这个库来完成的，由于我们这之前也没接触过react-router，所以版本v3与v4之间模式和策略的差异不同也没有带来思维模式转换的困难，下面先帖码简单看看v3与v4版本之间的差异性（摘自[掘金](https://juejin.im/post/5995a2506fb9a0249975a1a4)）：
```javascript
import { Router, Route, IndexRoute } from 'react-router'
const PrimaryLayout = props => (
    <div className="primary-layout">
        <header>
          Our React Router 3 App
        </header>
        <main>
          {props.children}
        </main>
  </div>
)
const HomePage =() => <div>Home Page</div>
const UsersPage = () => <div>Users Page</div>
const App = () => (
  <Router history={browserHistory}>
    <Route path="/" component={PrimaryLayout}>
      <IndexRoute component={HomePage} />
      <Route path="/users" component={UsersPage} />
    </Route>
  </Router>
)
render(<App />, document.getElementById('root'))
```

```javascript
import { BrowserRouter, Route } from 'react-router-dom'

const PrimaryLayout = () => (
    <div className="primary-layout">
    <header>
      Our React Router 4 App
    </header>
    <main>
      <Route path="/" exact component={HomePage} />
      <Route path="/users" component={UsersPage} />
    </main>
  </div>
)

const HomePage =() => 
<div>Home Page</div>

const UsersPage = () => 
<div>Users Page</div>


const App = () => (
  <BrowserRouter>
    <PrimaryLayout />
  </BrowserRouter>
)

render(<App />, document.getElementById('root'))
```

差异性：

1、v3是集中性路由，所有路由都是集中在一个地方， 而v4则相反。

2、v3布局和页面嵌套是通过 组件的嵌套而来的，而v4不会互相嵌套

3、v3布局和页面组件是完全纯粹的，它们是路由的一部分，而v4路由规则位于布局和 UI 本身之间

4、使用v4需要在我们的组件根部用BrowserRouter组件(用于浏览器)去包装，实现原理与react-redux的Provider组件一样（Context API），以便组件可以去拿到路由信息。

这里主要介绍包容性路由、排他性路由、嵌套路由，以及withRouter的一些基本用法。
