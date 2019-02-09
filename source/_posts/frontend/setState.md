---
title: 理解setState()
date: 2018-9-15
---

### state

一个组件的显示形态是可以由它数据状态和配置参数决定的。一个组件可以拥有自己的状态，就像一个点赞按钮，可以有“已点赞”和“未点赞”状态，并且可以在这两种状态之间进行切换。React.js 的  state 就是用来存储这种可变化的状态的。

那么 state 是如何改变的呢？显然直接赋值是不行的，this.state.name = 'tracy'？React 并不会根据此代码来重新渲染组件，正确的操作是使用 setState() 。

### setState()

```JavaScript
setState(updater, callback)
```

如上，setState()接受两个参数，一个是用于改变组件状态的 updater，另一个是当 setState()异步执行完毕后，调用的回调函数，用它可以进行任何的后期操作。

### 第一个参数

我们先来看 setState()的第一个参数，它既可以是一个对象，也可以是一个函数。当它是一个对象时，表示下一个 state 的值。

```JavaScript
//this.state={name:"Lili"}
handleClickOnChangeNameButton(){
   this.setState({name:"Tracy"})
}
```

这样操作起来非常直观，我们改变下一个 state 的值，将它的 name 由“Lili”改为“Tracy”，既然用对象就可以解决，那么为什么还需要函数呢？别急，让我们继续来探索。

你也许听过，setState()是一种异步的行为。这意味着当你调用  setState  的时候，React.js 并不会马上修改 state。而是把这个对象放到一个更新队列里面，稍后才会从队列当中把新的状态提取出来合并到  state  当中，然后再触发组件更新。让我们来做一个简单的实验

```JavaScript
//this.state = {name:"Lili"};
console.log(this.state.name);   // "Lili"
handleClickOnChangeNameButton(){
   this.setState({name:"Tracy"})
}
console.log(this.state.name);   // "Lili"
```

你会发现两次打印的结果都是“Lili”，这里 React.js 的  setState  把你的传进来的状态缓存起来，稍后才会帮你更新到  state  上，所以你获取到的还是原来的  name.
所以如果你想在  setState  之后使用新的  state  来做后续运算就做不到了，例如：

```JavaScript
handleClickButton (){
   this.setState({ count: 0 }) // => this.state.count 还是 undefined
   this.setState({ count: this.state.count + 1}) // => undefined + 1 = NaN
   this.setState({ count: this.state.count + 2}) // => NaN + 2 = NaN
}
```

上面的代码的运行结果并不能达到我们的预期，我们希望  count  运行结果是  3 ，可是最后得到的是  NaN。但是这种后续操作依赖前一个  setState  的结果的情况并不罕见。

这里就自然地引出了  setState  的第二种使用方式，可以接受一个函数作为参数。React.js 会把上一个  setState 的结果传入这个函数，你就可以使用该结果进行运算、操作，然后返回一个对象作为更新  state  的对象：

```JavaScript
...
  handleClickButton () {
    this.setState((prevState) => {
      return { count: 0 }
    })
    this.setState((prevState) => {
      return { count: prevState.count + 1 } // 上一个 setState 的返回是 count 为 0，当前返回 1
    })
    this.setState((prevState) => {
      return { count: prevState.count + 2 } // 上一个 setState 的返回是 count 为 1，当前返回 3
    })
    // 最后的结果是 this.state.count 为 3
  }
...
```

setState 合并
上面我们进行了三次  setState，但是实际上组件只会重新渲染一次，而不是三次；这是因为在 React.js 内部会把 JavaScript 事件循环中的消息队列的同一个消息中的  setState  都进行合并以后再重新渲染组件。

### 第二个参数

setState()的第二个参数是一个回调函数，它会在 setState()完成，并且组件被渲染完毕时被调用，类似于 componentDidUpdate。
由于 setState 是异步的，因此回调函数用于任何后期操作，比如，获取更新后的 state 值：

```JavaScript
this.state = {foo: 1};
this.setState({foo: 123});
console.log(this.state.foo);
// 1

this.state = {foo: 2};
this.setState({foo: 123}, ()=> {
    console.log(foo);
    // 123
});
```

关于 setState 中回调函数的更多用法，可以参考：
https://medium.com/@voonminghann/when-to-use-callback-function-of-setstate-in-react-37fff67e5a6c
https://segmentfault.com/a/1190000008051628
