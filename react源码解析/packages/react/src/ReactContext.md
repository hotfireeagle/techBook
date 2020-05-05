### 1.功能概述

定义了React.createContext方法。通过查看源码学习到的内容有：

+ React.createContext方法接受两个参数，第一个参数是都知道的defaultValue，而第二个参数是calculateChangedBits。

+ context内部维护了两个currentValue，为什么需要维护两个currentValue，源代码中的注释如下：

  > As a workaround to support multiple concurrent renderers, we categorize some renderers as primary and others as secondary

### 2.calculateChangedBits是什么？

关于createContext的第二个参数calculateChangedBits，在官方文档中并没有提到它是干嘛用的，甚至没有提到createContext存在第二个参数。但是无论如何，createContext都是存在第二个参数calculateChangedBits的，而且在某些场景之中还能发挥不错的作用，下面进行举例：

```javascript
// context.js
const calculateChangedBits = (newV, oldV) => 0
const AppContext = React.createContext(1, calculateChangedBits)
export default AppContext

// app.js
const App = () => {
  const [value, setValue] = useState(1)
  
  const clickHandler = () => { setValue(n => n + 1) }
  
  return (
  	<AppContext.Provider value={value}>
    	<button onClick={clickHandler}>change</button>
			<b>{value}</b>
			<Count />
    </AppContext.Provider>
  )
}

// count.js
const Count = () => {
  const val = useContext(AppContext)
  return <h1>{val}</h1>
}
export default React.memo(Count)
```

如上所示，我们在创建context的时候，传入了一个calculateChangedBits作为第二个参数，并且一直返回0，这表示oldContextValue和newContextValue是一样的，没有发生变化。因此，当点击button的时候，b标签的innerText在不断累加，但是Count元素却是一直显示1。

如果想要正常的话，那么把calculateChangeBits去掉，或者显式的返回1。

同时，在上面还有一点需要注意一下，那就是：Count元素必须用React.memo包装一下，因为父元素App状态发生了变化，默认是会使得子元素进行更新的，那一旦触发更新，这个context自然也是取的新的值了，此时显式返回0的calculateChangedBits函数也发挥不了作用。

### 3.当Provider的value发生变化时，消费context的子元素是怎么做到更新的？

先描述一下所讨论问题的边界：很显然，**Provider作为父组件，当传给自己的props-value发生变化的时候，会触发自己的更新过程，默认情况下，子组件也会被波及到引起更新。而这里所讨论的是特殊情况下，比如说如果函数子组件使用React.memo把子组件给裹住了，或者子class组件定义了shouldComponentUpdate并且显式返回false（父组件Provider也这样做的话，那么context的value发生变化，是肯定引起不了更新的）。**在这种特殊情况下，此时子组件还是能够正常更新到最新的context值，这又是为什么呢？

答：首先Provider组件被触发了更新过程，此时这个更新过程taskA会进入react调度，当有机会轮到更新过程taskA进行执行的时候，React中的beginWork会接管这个调度task，它的方法签名如下所示：

```javascript
function beginWork(current, workInProgress, renderExpirationTime) {}
```

在这个beginWork中，会判断一下workInProgress的tag，此时我们看一下对于ContextProvider这个tag会发生什么：

```javascript
case ContextProvider:
      return updateContextProvider(current, workInProgress, renderExpirationTime)
```

调用了updateContextProvider方法，再看一下这个方法做了哪些逻辑（这里只粘贴关键代码）：

```javascript
var changedBits = calculateChangedBits(context, newValue, oldValue)
if (changedBits === 0) {
  if (oldProps.children === newProps.children && !hasContextChanged()) {
  	return bailoutOnAlreadyFinishedWork(current, workInProgress,renderExpirationTime);
  }
} else {
  propagateContextChange(workInProgress, context, changedBits, renderExpirationTime);
}
var newChildren = newProps.children;
reconcileChildren(current, workInProgress, newChildren, renderExpirationTime);
```

通过上面的代码我们可以发现，如果判断认定前后context value发生了变化的话，那么就会调用propagateContextChange方法（而不仅仅只是调用reconcileChildren方法来更新子组件），个人认为正是这个propagateContextChange做到了子组件的context value同步（哪怕不想让子组件进行更新）。

通过查阅propagateContextChange方法的源代码，我们也可以发现它大概就是想下遍历，如果属于自己context的consume的话，那么将对对其进行更新。

**END：这个源码所涉及到的fiber知识点比较多，但是那部分代码还没有开始着手，所以这里的源码注释阅读难免很多不足，后续进行进一步补全，同时还有Provider部分、Consumer部分，useContext部分。**