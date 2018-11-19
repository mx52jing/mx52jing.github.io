---
title: React的PureComponent
date: 2018-11-19 18:28:40
categories:
- React
tags:
- React
---

PureComponent提供了一个具有浅比较的`shouldComponentUpdate`方法。

<!-- more -->

当`props`和`state`改变时，PureComponent将会对`props`和`state`进行浅比较，只会浅检查组件的props和state，所以嵌套对象和数组是不会被比较的。

```
if (this._compositeType === CompositeTypes.PureClass) {
  shouldUpdate = !shallowEqual(prevProps, nextProps)
  || !shallowEqual(inst.state, nextState);
}
```
> 浅比较将检查原始值是否有相同的值（例如：1 === 1或者ture===true）,数组和对象引用是否相同

#### 基础类型值的state
如果state是基础类型的值，那么每次改变值都会触发render

```
import React, { PureComponent } from 'react';

class App extends PureComponent {
  state = {
    loading: 111
  }
  handleClick = _ => {
    const {loading} = this.state
    this.setState({loading: loading == 333 ? 111 : 333}) //每次都会会触发render
  }
  render() {
    console.log('render')
    const {loading} = this.state
    return (
      <div className="App">
        <span>{loading}</span>
        <button onClick={this.handleClick}>change loading</button>
      </div>
    )
  }
}


export default App;
```

#### 复杂类型的state

##### 易变数据不能使用一个引用

```
import React, { PureComponent } from 'react';

class App extends PureComponent {
  state = {
    arr: [1,2,3]
  }
  handleClick = _ => {
    const {arr} = this.state
    arr.push(6)
    console.log(arr)
    this.setState({arr}) //不会触发render
  }
  render() {
    console.log('render')
    const {arr} = this.state
    return (
      <div className="App">
        {
            arr.map((item, index) => <li key={index}>{item}</li>)
        }
        <button onClick={this.handleClick}>change arr</button>
      </div>
    )
  }
}


export default App;
```

上面的代码是没有效果的，通过控制台输出的数据发现，arr已经每次都增加了6，数组在变化，但是***数组的引用**并没有改变，因此不会触发render.`如果改成使用Component每次都会重新render.`

正确使用

```
  handleClick = _ => {
    const {arr} = this.state
    this.setState({arr: [...arr, 6]}) //会触发render
  }
```
或者

```
  handleClick = _ => {
    const {arr} = this.state
    arr.push(6)
    this.setState({arr: [].concat(arr)}) //会触发render
  }
```

这样每次改变都会产生一个新的数组，也就可以`render`了


#### 其他注意点

* **条件渲染中的&&(见官网深入JSX一节)**

`JavaScript 中的一些 “false” 值(比如数字0)，依然会被渲染在页面中`

```
state = {
  arr = []
}

render

{ arr.length && <span>xxxx</span>}

当arr数组长度为零时，期望页面不展示，但是此时页面展示的是0

要解决这个问题，就要确保 && 前面的表达式始终为布尔值：

可以直接强制类型转换:

{ !!arr.length && <span>xxxx</span>}
```
