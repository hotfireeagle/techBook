**依据：如果对象具有Symbol.iterator属性或者@@iterator属性，并且是函数的话，那么就认为这个对象是实现了iterator接口的。**

实现iterator接口可以怎么样？答：比如说可以使用for...of...循环来进行遍历。说说哪些类型的数据实现了iterator接口？答：比如说数组、generator function。

判断代码如下所示：

```javascript
const key1 = typeof Symbol === 'function' && Symbol.iterator
const key2 = '@@iterator'

function isIteratorable(obj) {
  if (obj === null || typeof obj !== 'object') return false
  const value = key1 && obj[key1] || obj[key2]
  if (typeof value === 'function') return true
  return false
}
```

下面在举个例子看看generator function搭配for...of...使用的效果

```javascript
function* fun() {
  yield 1
  yield 2
  yield 3
}
const ob = fun()
for (let n of ob) {
  console.log(n)
}
```

