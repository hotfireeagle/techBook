## 1.isValidElementType

这里直接贴代码了：

```javascript
function isValidElementType(type) {
  return (
  	typeof type === 'string' ||
    typeof type === 'function' ||
    type === REACT_FRAGMENT_TYPE ||
    type === REACT_PROFILER_TYPE ||
    type === REACT_DEBUG_TRACING_MODE_TYPE ||
    type === REACT_STRICT_MODE_TYPE ||
    type === REACT_SUSPENSE_TYPE ||
    type === REACT_SUSPENSE_LIST_TYPE ||
    (typeof type === 'object' &&
      type !== null &&
      (type.$$typeof === REACT_LAZY_TYPE ||
        type.$$typeof === REACT_MEMO_TYPE ||
        type.$$typeof === REACT_PROVIDER_TYPE ||
        type.$$typeof === REACT_CONTEXT_TYPE ||
        type.$$typeof === REACT_FORWARD_REF_TYPE ||
        type.$$typeof === REACT_FUNDAMENTAL_TYPE ||
        type.$$typeof === REACT_RESPONDER_TYPE ||
        type.$$typeof === REACT_SCOPE_TYPE ||
        type.$$typeof === REACT_BLOCK_TYPE ||
        type[(0: any)] === REACT_SERVER_BLOCK_TYPE))
  );
}
```

从上面这个判断方法我们可以看出，对于ReactElement（jsx或者createElement所返回的对象）来说，它是返回false的。

## 2.memo

省略部分开发模式的代码后，长下面这样：

```javascript
type TC = (oldProps: Props, newProps: Props) => boolean
function memo(type, compare: TC) {
  if (__DEV__) {
    if (!isValidElementType(type)) console.error('...')
  }
  const elementType = {
    $$typeof: REACT_MEMO_TYPE,
    type,
    compare: compare === undefined ? null : compare
  }
  return elementType
}
```

从源码分析，我们便可以发现React官网上面描述的那句memo只适用于函数式组件是什么意思了。