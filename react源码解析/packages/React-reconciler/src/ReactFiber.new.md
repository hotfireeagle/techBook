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