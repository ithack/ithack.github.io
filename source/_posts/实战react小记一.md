---
title: 实战react小记一
date: 2017-06-19 17:13:04
tags: [react,随笔]
---

​	最近公司调整，我有机会从0开始做一个项目感觉很爽；下面简单说一下我们的技术选型：

​	1.前端展示页选用公司自己的nodejs框架搭建前端页面

​	2.后台系统用react作为框架搭建后台页面

​	3.后台系统是用nodejs作为中间层接收后端接口数据后渲染模版传给客户端（浏览器）

​	之前一直在看react的demo和API从来没有在项目中用过，之前看demo和api时觉得react的写法上实在让我这个纯前端觉得高大上同时望而却步的感觉，这次有机会用react在实际项目中使用内心却有那么一点的小激动；好了废话不多说，下面就说说react吧（由于第一次用，记录不对的地方还请看官大大批评指正）

​	由于是实战小记，这里就不说react是什么，react解决了什么等问题了，这些问题可以在搜索引擎搜到一堆大大们的文章学习，今天以实战经历为主；我们用了（[antDesign](https://ant.design/)）这个前端框架的，先说说react创建组件的ES5和ES6两种方法：

ES5：

```
import React from 'react';
import reactDOM from 'react-dom';
/*子组件*/
var Hello = React.createClass({
  getIntialState: function(){}, //初始化组件状态
  getDefaultProps： function(){}, //初始化组件的默认属性
  
  propTypes: {}, //规定属性的存在性和类型
  
  //一些生命周期相关函数
  componentWillMount: function(){}, //组件被嵌入之前触发
  componentDidMount: function(){}, //组件被嵌入之后触发
  componentWillReceiveProps: function(){}, //当props有改变时触发
  componentWillUnmount: function(){}, //组件被注销之前触发
  //渲染函数，不能省略    
  render: function() {
    return <h1>Hello {this.props.name}</h1> ;     
  }
});
export default Hello
/*父组件*/
ReactDOM.render(
  <Hello name="John" />,
  document.getElementById('App')
);
```

<!--more-->ES6：

```
import React ,{ Component }from 'react';
import reactDOM from 'react-dom';
/*子组件*/
class Hello extends React.Component {
  constructor() {
    super();
    this.state = {
    //组件状态
    };
  }
  static defaultProps = {
    //组件属性    
  };
  handleClick(){
    alert("你点击我了")
  }
  render() {
    return (
      <div onClick={this.handleClick}>
       {this.props.name}
      </div>
    );
  }
}
export default Hello
/*父组件*/

ReactDOM.render(
  <Hello name="John" />,
  document.getElementById('App')
);
```

这里简单说一下props和state：

**props**当我们在使用定义好的组件时，可以向其添加一些属性，在组件定义的内部，可以通过this.props来访问这些属性，如下在render方法里渲染动态数据：

```
var MyProps = React.createClass({
    getDefaultProps: function(){
      return {
        name: 'world'
      }
    },
    propTypes: {
    name: React.PropTypes.string //设置属性的类型
    },
    render: function(){
        return (
            <h1>Hello, {this.props.name}!</h1> //访问name属性
        );
    }
});
```

其中getDefaultProps用来设置默认的属性值，[propTypes](https://facebook.github.io/react/docs/reusable-components.html#prop-validation)来设置属性的类型，在使用时如果属性类型不匹配会提示。

this.props是只读的，它通常用来传递来自父组件的数据，也是**React**构建单向数据流的方式。

**state**相同的组件之所以能够表现出不一样的UI是因为它们内部拥有不同的状态（states），每个组件都可以拥有自己的state，并且可以在需要的时候通过props传递给子组件。组件可以通过setState函数来修改内部的状态，同时触发界面的渲染。

```
var MyState = React.createClass({
  getInitialState: function(){
    return {
      name: 'tim',
      friend: 'han mei mei' 
    }
  },
  handleClick: function(e){
    this.setState({name: 'kevin'}); //更改状态
  },
  render: function(){
    return (
      <div>
        <h3> Name: {this.state.name} </h3>
        //点击触发事件，调用handleClick改变组件状态
        <button onClick={this.hanleClick}>change name</button>
        <Component name={this.state.friend} /> //传递状态到子组件
      </div>
    )
  }
});
```

getInitialState是用来初始化组件内部状态的默认值，不要在该方法里面使用this.props，详细请参看[这里](https://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)。

##### props和state的差异

props是只读的，不能够在组件内部修改，它是父组件向子组件传递数据的途径；state是组件自身的状态，它不能是父组件传递过来的数据，并且state是可以改变的。

合理的状态操作是创建多个只负责渲染数据的无状态（stateless）组件，在它们的上层创建一个有状态（stateful）组件并把它的状态通过 props 传给子组件。这个有状态的组件封装了所有用户的交互逻辑，而这些无状态组件则负责声明式地渲染数据

**由于我自己项目用的是ES6的写法，所以我在用绑定实例方法的时候遇到了一点点小问题代码如下：**

```
class schoolTable extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            visible: false,
            count:0,
            tableData: props.data,
        };
        this.handleOk = this.handleOk.bind(this);
        this.handleCancel = this.handleCancel.bind(this);
        this.showModal=this.showModal.bind(this)
        this.handleAdd=this.handleAdd.bind(this)
    }
    showModal(obj) {
        this.setState({
            visible: true,
        });
    }
    handleOk(){}
    handleCancel(){}
    .....
}
```

每一个实例对象都要手动绑定这个非常痛苦，按照es6的箭头函数和属性初始化程序应该：

```
class schoolTable extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            visible: false,
            count:0,
            tableData: props.data,
        };
    }
    showModal=(obj)=>{
        this.setState({
            visible: true,
        });
    }
    handleOk=()=>{}
    handleCancel=()=>{}
    .....
}
```

如上写法可以不用手动去绑定，但是用箭头的写法我的项目会报错，目前还没有查到问题，只能手动去绑定，如果那位大大知道原因还请告知，后面我找到原因会更新问题解决方案；

20170619使用react心得总结：

React 它主要做几件事情：

- 组件化
- 利用 props 形成单向的数据流
- 根据 state 的变化来更新 view
- 利用虚拟 DOM 来提升渲染性能

