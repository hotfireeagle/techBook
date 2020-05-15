### 1.FiberNode方法

FiberNode对象的构造函数，关键源码如下所示：

```javascript
function FiberNode(tag: WorkTag, pendingProps, key, mode: TypeOfMode) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;
  
  // Fiber
  this.return = null;
  this.children = null;
  this.sibling = null;
  this.index = 0;
  
  this.ref = null;
  
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies_new = null;
  
  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  this.expirationTime_opaque = NoWork;
  this.childExpirationTime_opaque = NoWork;

  this.alternate = null;
}
```

这里重点介绍一下下面几个参数（或者说值）

+ tag参数，可以查阅同目录下的ReactWorkTags这一篇，标明了当前FiberNode的标签类型。
+ mode参数，标明当前FiberNode处于何种模式下。常见的模式有：无模式、StrictMode。可参阅ReactTypeOfMode文章。
+ NoEffect：标明默认的FiberNode是不存在Effect的。Effect也分类型的，可参阅ReactSideEffectTags文章。
+ NoWork：值为0，值为数字0，表示计算时间？在React源码中，使用number的别名ExpirationTimeOpaque来定义计算时间，NoWork便属于该类型，其值为0，是个常量。

### 2.createFiber方法

```javascript
function createFiber(tag: WorkTag, pendingProps: mixed, key: null | string, mode: TypeOfMode): Fiber {
  return new FiberNode(tag, pendingProps, key, mode);
}
```

可以看出，它就只是调用FiberNode构造函数来构造来一个Fiber对象。为什么要引入中间这一层而不是直接依靠FiberNode方法，暂时不是很清楚。

### 3.shouldConstructor

被用来判断一个component（虽然在react系统中存在很多component，但是这里所讨论的是开发者所接触到的class component以及function component）是否是class component。判断方法如下所示（注意，这里已经确定来是component，只是区别function c和class c）：

```javascript
function shouldConstructor(Component: Function) {
  const prototype = Component.prototype;
  return !!(prototype && prototype.isReactComponent);
}
```

可以发现，相比较function component，class component实例对象上面会有一个isReactComponent属性。

### 4.resolveLazyComponentTag

在react系统中，reactElement只有通过两种写法创建：1.jsx语法糖；2.React.createElement。而在react中，component有很多，只不过对于使用者来说就是ClassComponent和FunctionComponent，但是其实还有那些没有暴露出来的component的，比如说通过memo方法所暴露出来的MemoComponent。

```javascript
function resolveLazuComponentTag(Component: Function) {
  if (typeof Component === 'function') {
    return shouldConstructor(Component) ? ClassComponent : FunctionComponent;
  } else if (Component !== undefined && Component !== null) {
    const $$typeof = Component.$$typeof;
    if ($$typeof === React_MEMO_TYPE) {
      return MemoComponent;
    }
    if ($$typeof === REACT_FORWARD_REF_TYPE) {
      return ForwardRef;
    }
  }
}
```

### 5.createWorkInProgress

由于看的源码还不够多，所以对一些概念还是模糊不清，初步感觉对于一个Fiber来说，alternate很重要呀！？

该函数关键代码如下所示：

```javascript
function createWorkInProgress(current: Fiber, pendingProps: any): Fiber {
  let workInProgress = current.alternate;
  
  // 省略若干代码
  workInProgress.pendingProps = pendingProps;
  
  workInProgress.effectTag = NoEffect;
  workInProgress.nextEffect = null;
  workInProgress.firstEffect = null;
  workInProgress.lastEffect = null;
  
  workInProgress.childExpirationTime_opaque = current.childExpirationTime_opaque;
  workInProgress.expirationTime_opaque = current.expirationTime_opaque;
  
  workInProgress.child = current.child;
  workInProgress.memoizedProps = current.memoizedProps;
  workInProgress.memoizedState = current.memoizedState;
  workInProgress.updateQueue = current.updateQueue;
  
  workInProgress.sibling = current.sibling;
  workInProgress.index = current.index;
  workInProgress.ref = current.ref;
  
  const currentDependencies = current.dependencies_new;
  workInProgress.dependencies_new =
    currentDependencies === null
      ? null
      : {
        expirationTime: currentDependencies.expirationTime,
        firstContext: currentDependencies.firstContext,
        responders: currentDependencies.responders,
      };
}
```

从代码我们可以看出，createWorkInProgress方法所创建出来的Fiber几乎就是当前Fiber的复刻了。wtf，这是什么鬼，只能接着往下看才能搞清楚了。

这里解释一下为什么单独一个dependencies_new需要单独拎出来赋值：因为在render parse中，dependencies_new是mutable的，所以不能指向同一块内存，避免引起问题。

### 6.resetWorkInProgress

对一个fiber进行reset，reset规则是什么？如果该fiber不具有alternate的话，那么把需要reset的属性都reset为null；如果该fiber节点具有alternate的话，那么就把需要reset的属性都reset为alternate的对于属性，关键代码如下所示：

```javascript 
function resetWorkInProgress(workInProgress: Fiber, renderExpirationTime: ExpirationTimeOpaque) {
  workInProgress.nextEffect = null;
  workInProgress.firstEffect = null;
  workInProgress.lastEffect = null;
  
  const current = workInProgress.alternate;
  
  if (current === null) {
    // 省略部分代码
  } else {
    // 省略部分代码
  }
}
```

除了上面的nextEffect、firstEffect、lastEffect都被初始化为null外，这些属性也被初始化了：childExpirationTime_opaque、expirationTime_opaque、child、memoizedProps、memoizedState、updateQueue、dependencies_new、stateNode。区别是前者被初始化为null（形如createFiber），后者是初始化为alternate（形如createWorkInProgress）。

### 7.createFiberFromTypeAndProps

这个方法的源代码就不贴了，大致作用就是根据传入的参数type来创建指定的类型的Fiber。

