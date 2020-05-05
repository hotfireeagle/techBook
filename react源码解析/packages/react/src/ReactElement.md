### 1.功能概述

定义了ReactElement数据类型，使用React.createElement方法将会返回一个ReactElement类型的数据。在react中，有两个数据类型还是挺重要的，他们分别是ReactElement类型和Fiber类型。

### 2.内容基础

使用React.createElement来创建一个ReactElement，这个方法的第二个参数config中不仅包括普通的props数据，还包括一些react的保留props，react中有哪些保留props呢，如下所示：

```javascript
const RESERVED_PROPS = {
  key: true,
  ref: true,
  __self: true,
  __source: true,
} // 这些都是保留关键props
```

### 3.createElement

先看看方法签名：

```javascript
(type, config, children) => ReactElement
```

下面来看createElement的关键代码。首先看看关于config数据的处理，由于config中不仅可以带有普通的props，还可以携带保留props，而这部分保留props是需要剥离出来的，下面是关于剥离的逻辑片段：

```javascript
function createElement(type, config, children) {
  // ...... 省略来一些代码
  const props = {} // 普通的props数据
  let key = null // 保留props之key
  let ref = null // 保留props之ref
  let self = null // 同上
  let source = null // 同上
  
  if (config != null) { // 过滤掉了undefined以及null
    if (hasValidRef(config)) ref = config.ref
    if (hasValidKey(config)) key = '' + config.key
    self = config.__self === undefined ? null : config.__self
    source = config.__source === undefined ? null : config.__source
    
    // 处理普通的props数据
    for (let propName in config) {
      if (
      	Object.prototype.hasOwnProperty.call(config, propsName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName]
      }
    }
  }
}
```



