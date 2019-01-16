---
title: react基本概念解析
date: 2018/06/24
tags: [js,react]
toc: false
---

[读react官网](http://www.css88.com/react/docs/hello-world.html)

## react阻止默认事件
在react阻止默认行为 必须使用
  e.preventDefault();
   
```
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }
  这里， e 是一个合成的事件。 React 根据 W3C 规范 定义了这个合成事件，所以你不需要担心跨浏览器的兼容性问题。查看 SyntheticEvent 参考指南了解更多。

```

在JSX回调中你必须注意 this 的指向。 在 JavaScript 中，类方法默认没有 绑定 的。如果你忘记绑定 this.handleClick 并将其传递给onClick，那么在直接调用该函数时，this 会是 undefined 。

这不是 React 特有的行为；这是 JavaScript 中的函数如何工作的一部分。 一般情况下，如果你引用一个后面没跟 () 的方法，例如 onClick={this.handleClick} ，那你就应该 绑定(bind) 该方法。

```
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // 这个绑定是必要的，使`this`在回调中起作用
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```
解决方式


```
class LoggingButton extends React.Component {
  // 这个语法确保 `this` 绑定在 handleClick 中。
  // 警告：这是 *实验性的* 语法。
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}

```

或者


```
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 这个语法确保 `this` 被绑定在 handleClick 中
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
这样不好  问题是
这个语法的问题是，每次 LoggingButton 渲染时都创建一个不同的回调。在多数情况下，没什么问题。然而，如果这个回调被作为 prop(属性) 传递给下级组件，这些组件可能需要额外的重复渲染。我们通常建议在构造函数中进行绑定，以避免这类性能问题。
```

如果需要参数传递给事件处理程序

```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```


使用逻辑 && 操作符的内联 if 用法 来进行条件渲染

```
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 8 ?
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
       : ''
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);


==


function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 8 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
       
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);

条件渲染的一种简便写法（类似于三目运算符）

它可以正常运行，因为在 JavaScript 中， true && expression 总是会评估为 expression ，而 false && expression 总是执行为 false 。

因此，如果条件为 true ，则 && 后面的元素将显示在输出中。 如果是 false，React 将会忽略并跳过它。

最low的一种方式也是借助于元素变量


    let button = null;
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }
    这可以帮助您有条件地渲染组件的一部分，而输出的其余部分不会更改。
```

使用条件操作符的内联if-else
另一个用于条件渲染元素的内联方法是使用 JavaScript 的条件操作符 condition ? true : false 。

```
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```
## 防止组件渲染 返回null

```
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

 render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}
```

## react和key

键（key）
键(Keys) 帮助 React 标识哪个项被修改、添加或者移除了。数组中的每一个元素都应该有一个唯一不变的键(Keys)来标识：

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
```

### 如何指定key？

如果你提取 一个 ListItem 组件，应该把 key 放置在数组处理的 <ListItem /> 元素中，不能放在 ListItem 组件自身中的 <li> 根元素上。

```
function ListItem(props) {
  // 正确！这里不需要指定 key ：
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 正确！key 应该在这里被指定
    <ListItem key={number.toString()}
              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);

一个好的经验准则是元素中调用 map() 需要 keys 。


```
## 组合 继承

react拥有一个强大的组合模型 建议使用组合而不是继承以实现代码的重用

包含  如果你不确定要使用什么组件 在 弹层等通用的容器中比较常见
建议这种组件使用特别的 children props 来直接传递


```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}

function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```


```
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />

        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

## 自定义组件写法
用户自定义的组件必须以大写字母开头

但一个元素类型以小写字母开头 他表示引用一个类似于<div>或者<span>的内置组件
以大写字母开头的类型，类似于 <Foo />，会被编译成 React.createElement(Foo) ，对应于自定义组件 或者在 JavaScript 文件中导入的组件。


```
import React from 'react';

// 错误！这是一个组件，首字母应该大写：
function hello(props) {
  // 正确！这种使用 <div> 是合法的，因为 div 是一个有效的HTML标记：
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // 错误！React 认为 <hello /> 是一个 HTML 标签，因为它首字母应不是大写的：
  return <hello toWhat="World" />;
}
```
字符串字面量

```
<MyComponent message="hello world" />
====
<MyComponent message={'hello world'} />
```

props(属性)默认为true

```
<MyTextBox autocomplete />
===
<MyTextBox autocomplete={true} />
```
jsx中的children

字符串变量

```
<MyComponent>Hello world!</MyComponent>
这是有效的 JSX ，MyComponent 组件中的 props.children 值为字符串 "Hello world!" 。
```
JSX会删除每行开头和结尾的空格，并且也会删除空行。邻接标签的空行也会被移除，字符串之间的空格会被压缩成一个空格，因此下面的渲染效果都是相同的

```
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```
Booleans, Null, 和 Undefined 被忽略
false，null，undefined，和 true 都是有效的的 children(子元素) 。但是并不会被渲染，下面的JSX表达式渲染效果是相同的：

```
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```
在有条件渲染的时候非常有用 如果 showheader为true的时候 <header >会被渲染（必须保证&&表达式之前总是布尔值）

```
<div>
  {showHeader && <Header />}
  <Content />
</div>

```

如果在输出中想要渲染false true null或者undefined 必须先将其转化为字符串

```
<div>
  My JavaScript variable is {String(myVariable)}.
</div>

```

使用propType进行类型检查
可以通过赋值特定的default属性为pros定义默认值


```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// 指定 props 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};

// 渲染为 "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);

或者

 class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    )
  }
}

propTypes 的类型检测是在defaultProps 解析之后发生的，因此也会对默认属性 defaultProps 进行类型检测。

```

## ES6 声明class组件


```
class Greeting extends React.Component {
  // ...
}

Greeting.defaultProps = {
  name: 'Mary'
};

```
### 设置初始化组件

```
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: props.initialCount};
  }
  // ...
}
```


## 一致性比较（diff算法的实现）

```
使用react的时候 任何一个单点时刻可以认为 render（）函数的作用是创建react元素树
在下一个state或者props更新的时候 render（）函数将会返回一个不同的react元素树
接下来 react将会找出如何高效的更新ui来匹配最近时刻的react元素树
```
react基于以下两个假设实现了时间复杂度为o（n）的算法

* 不同类型的两个元素将会产生不同的树。
* 开发人员可以使用一个key prop来指示在不同的渲染中那个那些元素可以保持稳定

### diff算法
当比较不同的树 react会首先比较两个根元素 根据根的类型不同 它有不同的行为
#### 元素类型不相同时
无论什么时候 当根元素类型不同时，react将会销毁原先的树并重写构建新的树
当销毁原先的树时 之前的dom节点将销毁，实例组件执行 componentWillUnmount() 。当构建一个新的树 新的dom节点将会插入dom中 组件将会执行componentWillMount() 以及 componentDidMount() 。与之前旧的树相关的 state 都会丢失。
#### dom元素类型相同时
当元素类型相同 react会比较检查它们的属性（attributes）保留相同的底层dom节点 只更新发生改变的属性

```
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```
通过比较两个元素，React 会仅修改底层 DOM 节点的 className 属性。
在处理完当前 DOM 节点后，React 会递归处理子节点。

#### 相同类型的组件
当一个组件更新的时候 组件实例保持不变 以便在渲染中保持state react会更新组件实例的属性来匹配新的元素 并在组件实例中调用componentWillReceiveProps() 和 componentWillUpdate()。接下来， render() 方法会被调用并且diff算法对上一次的结果和新的结果进行递归。
#### 子元素递归

```
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
React 会比较两个 <li>first</li> 树与两个 <li>second</li> 树，然后插入 <li>third</li> 树。
```

```
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
React 将会改变每一个子节点而没有意识到需要保留 <li>Duke</li> 和 <li>Villanova</li> 两个子树。这种低效是一个问题。
```
怎么解决类似的问题
为了解决这个问题 react支持一个key属性 当子节点有了key react使用这个key去比较原来的树的子节点和之后的子节点 

```
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
现在 react知道2014的key元素是新的 key为2015和2016的两个元素仅仅只是被移动而已
```

## 片段

解决痛点

```
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}

为了渲染有效的 HTML ， <Columns /> 需要返回多个 <td> 元素。如果 <Columns /> 的 render() 函数里面使用一个父级 div ，那么最终生成的 HTML 将是无效的。


class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}

最后会渲染成
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>

```

```
使用方式
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
新的声明方式（看起来像空标签）
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}

带key的片段

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 没有`key`，将会触发一个key警告
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
key 是唯一可以传递给 Fragment 的属性。在将来，我们可能增加额外的属性支持，比如事件处理。


```
### 插槽
Portals 提供了一种很好的方法，将子节点渲染到父组件 DOM 层次结构之外的 DOM 节点。

`ReactDOM.createPortal(child, container)`

```
第一个参数（child）是任何可渲染的 React 子元素，例如一个元素，字符串或 片段(fragment)。
第二个参数（container）则是一个 DOM 元素。
```

```
render() {
  // React 装载一个新的 div，并将 children 渲染到这个 div 中
  return (
    <div>
      {this.props.children}
    </div>
  );
}
然而，有时候将子元素插入到 DOM 节点的其他位置会有用的：

render() {
  // React *不* 会创建一个新的 div。 它把 children 渲染到 `domNode` 中。
  // `domNode` 可以是任何有效的 DOM 节点，不管它在 DOM 中的位置。
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}
```
### 错误边界
部分 UI 中的 JavaScript 错误不应该破坏整个应用程序，错误边界是 React 组件，它可以在子组件树的任何位置捕获 JavaScript 错误，记录这些错误，并显示一个备用 UI ** ，而不是使整个组件树崩溃。错误边界(Error Boundaries) 在渲染，生命周期方法以及整个组件树下的构造函数中捕获错误。

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
 有这个生命周期方法 证明是错误边界方式
  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
调用
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```
注意错误边界(Error Boundaries) 仅可以捕获其子组件的错误。错误边界无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会向上冒泡至最接近的错误边界。这也类似于 JavaScript 中 catch {} 的工作机制

https://codepen.io/anon/pen/VdyBrE?editors=0010
### 未捕获错误的新行为
这一改变有非常重要的意义。自 React 16 开始，任何未被错误边界捕获的错误将会卸载整个 React 组件树。


时间处理器如何处理
错误边界无法捕获事件处理器内部的错误。


因为时间处理器不会在渲染周期内触发
因此若他们抛出异常 react仍然能够知道需要在屏幕中显示什么
如果你需要在事件处理器内部捕获错误，使用普通的 JavaScript try / catch 语句：

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
  }
  
  handleClick = () => {
    try {
      // Do something that could throw
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    if (this.state.error) {
      return <h1>Caught an error.</h1>
    }
    return <div onClick={this.handleClick}>Click Me</div>
  }
```

代码拆分

```
import { add } from './math';

console.log(add(16, 26));
```
以后

```
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```
当 Webpack 遇到这个语法时，它会自动启动 代码拆分 来拆分你的应用程序。


react component
###  Mounting(装载)
 当组件实例被创建并且将其插入dom时 z和谐方法将会被调用

```
constructor()
componentWillMount()
render()
componentDidMount()
```
### Updating(更新)
改变props或者state可以触发更新事件，在重新渲染组件时将会调用

```
componentWillReceiveProps()
shouldComponentUpdate()
componentWillUpdate()
render()
componentDidUpdate()

```
### Unmounting(卸载)
当一个组件从Dom中删除时 将会调用

```
componentWillUnmount()
```
### render() 方法

render() 函数应该是纯函数，这意味着它不会修改组件状态，每次调用它时返回相同的结果 如果 shouldComponentUpdate() 方法返回 false ，render() 不会被调用。


当被调用时，它会检查this.props 和 this.state并返回其中一个类型

```
react元素
字符串和数字
Portals  ReactDOM.createPortal 创建。
null:不渲染任何东西
布尔值:不渲染任何东西 （通常存在于 return test && <Child />写法，其中 test 是布尔值。）
```
你可以返回 null 或 false 来表示你不想要渲染任何东西 当返回 null 或 false 时，ReactDOM.findDOMNode(this) 将返回 null
当你想返回多个元素时 用数组操作

```
render() {
  return [
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```
### constructor()方法
组件被装载前调用，当实现`react.component`时子类的constructor(构造函数)时 应该最先调用`super(props)` 否则将找不到this 报错

可以在构造函数中初始化state 采用
this.state={} 的方式而不是使用setState()  构造函数也经常用于将事件处理程序绑定到类实例(目前可以用箭头函数解决此类问题)
如果state需要根据props的值来初始化，就是你需要考虑变量提升的时候

如果你没有初始化 状态(state) ，并且没有绑定方法，不用实现一个构造函数

### componentWillMount()
在组件装载之前立即被调用的方法 在render之前渲染 所以 不会触发render
也是服务端渲染调用的唯一的生命周期的
钩子
### componentDidMount()
组件装载之后立即被调用
初始化需要的dom节点或从远程加载数据的地方
在这里调用`setState()`会触render()
但会在浏览器更新屏幕之前发生。在这种情况下，即使 render() 会被调用两次， 也可以保证用户不会看到中间状态
### componentWillReceiveProps()
 在已装载组件接收新 props 之前被调用,
 可以在此方法中比较this.props 和 nextProps 并使用 this.setState() 执行状态转换。
### shouldComponentUpdate(nextProps, nextState) 
让 React 知道组件的输出是否不受 state 或 props 当前变化的影响，默认行为是在每次 state 更改时重新渲染，并且在绝大多数情况下，你应该依赖于默认行为。
当接收到新的 props 或 state 时，shouldComponentUpdate() 在渲染之前被调用。 默认返回 true ，对于初始(第一次)渲染，不调用此方法，返回 false 不会阻止子组件在 state 更改时重新渲染 但是
`componentWillUpdate()` ，`render()`和`componentDidUpdate() `将不会被调用
### componentWillUpdate()
当接收到state或者props，它会在渲染之前被调用
### componentDidUpdate()
在更新发生之立即被调用，当组件已经更新时，也可以在这里操作dom 也可以做网络请求
### componentWillUnmount()
当组件被卸载和销毁之前 被调用 可以在此执行必要的清理 计时器 网络请求 或者在didmount创建的元素

```
  componentDidMount() {
    this.serverRequest = axios.get('/api')
      .then(posts => {
        this.setState({
          posts
      })
    })
  }
  
  componentWillUnmount() {
    this.serverRequest.abort()
  }
```
### componentDidCatch()
错误边界 可以在其_子组件树 _中的任何位置捕获js错误 显示备用ui
错误边界在渲染过程中，在生命周期方法中，以及整个树下的构造函数中捕获错误。
### setState()
setState() 总是会导致重新渲染，除非 shouldComponentUpdate() 返回 false

setState是作为一个请求而不是立即命令来更新组件 为了性能考虑 react可能会批量 或 延迟到后面更新它 然后合并多个setState()更新多个组件
React不保证 state 更新就立即应用(重新渲染)。所以更新之后立即读取可能会有问题
你可以使用 componentDidUpdate或者setState回调 `（setState(updater, callback)）`

```
例如，假设我们想通过 props.step 在 state 中增加一个值：


this.setState((prevState, props) => {
  return {counter: prevState.counter + props.step};
});

setState(stateChange, [callback])
这将执行 stateChange 的浅合并到新的 state 

这种也是异步的会进行批处理 如果同一时间段需要多次操作 可能会被覆盖
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)

可以这么使用
this.setState((prevState) => {
  return {counter: prevState.quantity + 1};
});
```

传递一个更新函数允许你在更新中访问当前的状态值。由于 setState 调用是批处理的,这允许你链式更新并确保它们建立在彼此之上，而不是产生冲突：
```
incrementCount() {
  this.setState((prevState) => {
    return {count: prevState.count + 1}
  });
}

handleSomething() {
  // this.state.count 是 1，然后我们这样做：
  this.incrementCount();
  this.incrementCount(); // count 现在是 3
}
```
## ReactDOMServer

ReactDOMServer 对象允许您在服务器上渲染组件。

renderToString()
renderToStaticMarkup()

渲染流
renderToNodeStream()
renderToStaticNodeStream()

渲染流可以减小第一个字节(TTFB)渲染时间，在文档的下一个部分生成之前，将文档的开头向下发送到浏览器。所有主流浏览器都会在服务器以这种方式流出内容时开始解析和呈现文档。


### 在组件中使用跟组件处理函数
#### 节流
是阻止函数在给定时间内被多次调用。下面这个例子会阻止“click”事件每秒钟的多次调用。

```
import throttle from 'lodash.throttle';

class LoadMoreButton extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.handleClickThrottled = throttle(this.handleClick, 1000);
  }

  componentWillUnmount() {
    this.handleClickThrottled.cancel();
  }

  render() {
    return <button onClick={this.handleClickThrottled}>Load More</button>;
  }

  handleClick() {
    this.props.loadMore();
  }
}

```
#### Debounce（防抖）

防抖确保函数上次执行后的一段时间内，不会再次执行。
下面这个例子以250ms的延迟来改变文本输入。

```
import debounce from 'lodash.debounce';

class Searchbox extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.emitChangeDebounced = debounce(this.emitChange, 250);
  }

  componentWillUnmount() {
    this.emitChangeDebounced.cancel();
  }

  render() {
    return (
      <input
        type="text"
        onChange={this.handleChange}
        placeholder="Search..."
        defaultValue={this.props.value}
      />
    );
  }

  handleChange(e) {
    // React pools events, so we read the value before debounce.
    // Alternately we could call `event.persist()` and pass the entire event.
    // For more info see reactjs.org/docs/events.html#event-pooling
    this.emitChangeDebounced(e.target.value);
  }

  emitChange(value) {
    this.props.onChange(value);
  }
}
```

#### 操作css

```
render() {
  let className = 'menu';
  if (this.props.isActive) {
    className += ' menu-active';
  }
  return <span className={className}>Menu</span>
}
```
### Virtual DOM and Internals
#### 什么是虚拟dom
虚拟dom是一种编程概念，是指虚拟的视图被保留在内存中 通过如reactDom这样的库与“真实”的dom 保持同步
虚拟DOM是由JavaScript库在浏览器API之上实现的一种概念。

#### 什么是“React Fiber”？
fiber是React 16中新的和解引擎。它的主要目的是使虚拟DOM能够进行增量渲染。了解更多。







 






