## 1.概述

在这个文件里面定义了react系统中的各种hook方法，但是具体实现逻辑并不在这里，这里只是调用了一下而已。

## 2.基础

dispatcher是个什么？先看该目录下的ReactCurrentDispatcher.js文件，代码如下所示：

```javascript
import { Dispatcher } from 'react-reconciler/ReactInternalTypes';

const ReactCurrentDispatcher = {
  current: (null: null | Dispatcher),
};

export default ReactCurrentDispatcher;
```

Wtf，这什么鬼类型声明？这个current到底是表示null或者Dispatcher还是？fuck.

看看这个Dispatcher是个什么：

```javascript
export type Dispatcher = {
  useContext<T>(
    context: ReactContext<T>,
    observedBits: void | number | boolean,
  ): T,
  readContext<T>(
  context: ReactContext<T>,
  observedBits: void | number | boolean,
): T,
};
```

很显然，这个Dispatcher就是一个类型定义，与逻辑实现无关，所以就先不继续看下去了。

## 3.useContext

**useContext是存在第二个参数的！！可以用来表示在context发生变化的时候，是否需要触发组件进行更新。**

先看下useContext函数的定义：

```javascript
export function useContext(context, observedBits) {
  return dispatcher.useContext(context, observedBits);
}
```

如上所示：useContext除了常见的第一个参数之外，还有第二个observedBits参数，可以用来控制context变化的时候是否触发更新。如果你显式的使这个参数的值为0或者false并且搭配memo使用的话，那么将会达到即使context发生变化，也不会触发组件更新的效果。代码如下所示：

```javascript
const Fun = () => {
  const v = useContext(ac, 0);
  return <h1>{v}</h1>;
};
export default React.memo(Fun);
```

由于这个文件并没有关于hooks的内部实现逻辑，所以就不再介绍其它hooks函数了。