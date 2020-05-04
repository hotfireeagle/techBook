### 1.功能概述

在react中，如果要定义一个类组件，那么它要么是继承了Component类型，要么就是继承了PureComponent类型。在这个文件中，描述了这两个构造函数的基本形状。为什么说是只描述了基本形状，因为在这里只涉及到了：props、context、refs、updater、setState(挂在原型上)、forceUpdate(挂在原型上)。对于Component来说，原型上面还有个isReactComponent的标志。对于PureComponent来说，其原型上则有一个isPureReactComponent的标志。

下面用一张图来说明一下：

![Component类型](https://s1.ax1x.com/2020/05/04/Y9jAfJ.md.png)

对于PureComponent来说也是同理，同时它继承了Component了，通过如下所示代码进行继承：

```javascript
Object.assign(PureComponent.prototype, Component.prototype)
```

### 2.看看源码

这里先重点关注一下setState以及forceUpdate的代码，关键代码如下所示：

```javascript
Component.prototype.setState = function(partialState, cb) {
  this.updater.enqueueSetState(this, partialState, cb, 'setState')
}
Component.prototype.forceUpdate = function(cb) {
  this.updater.enqueueForUpdate(this, callback, 'forceUpdate')
}
```

从上面我们可以看出，对于一个Component来说，当调用setState来刷新页面UI的时候，setState会把具体的逻辑交给交给Component实例上面的updater对象来进行调度，也就是进入调度是同步的，更新state或者UI是保证不了同步的。

接下来看看PureComponent继承Component的代码，感觉有点冗余了：

```javascript
function ComponentDummy() {}
ComponentDummy.prototype = Component.Prototype

function PureComponent(props, context, updater) {
  this.props = props
  this.context = context
  this.refs = emptyObject
  this.updater = updater || ReactNoopUpdateQueue
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy())
pureComponentPrototype.constructor = PureComponent // 2
Object.assign(pureComponentPrototype, Component.prototype) // 3
pureComponentPrototype.isPureReactComponent = true
```

关于继承：搞不懂直接第2步和第3步应该就可以满足的情况为何需要引入ComponentDummy组件。