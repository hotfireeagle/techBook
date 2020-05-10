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

**createElement方法接下来做的功能就是处理children。**见代码如下所示：

```javascript
function createElement(type, config, children) {
  // ...... 省略来一些代码
  const childrenLength = arguments.length - 2 // 利用老语法arguments获取子节点的数量
  if (childrenLength === 1) {
    props.children = children
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength)
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2]
    }
    props.children = childArray
  }
}
```

**createElement方法在处理完了children之后，接下来重点要做的就是处理defauleProps**。这里先介绍一下，那就是defaultProps存储在哪？defaultProps是属于类的静态属性，也就是说是在type上面。关键代码如下所示：

```javascript
function createElement(type, config, children) {
  // ...... 省略了一些代码
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName]
      }
    }
  }
}
```

最终createElement使用ReactElement方法构造出了一个对象，并且对其进行返回，如下所示：

```javascript
function createElement(type, config, children) {
  // ...... 省略来一些代码
  return ReactElement(
  	type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  )
}
```

### 4.isValidElement

```javascript
function isValidElement(object) {
  return (
  	typeof object === 'object' &&
    	object !== null &&
    	object.$$typeof === REACT_ELEMENT_TYPE
  );
}
```

不愧是ducktype，如果是使用new的形式来创建ReactElement的话，还能通过instanceOf来稍微缓解一下这个情况。

