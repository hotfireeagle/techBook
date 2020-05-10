## 1.关键代码

删掉开发模式的check代码之后，关键代码如下所示：

```javascript
function forwardRef(render) {
  const elementType = {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };
  return elementType;
}
```