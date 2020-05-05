### 1.概念

阻止给对象添加新属性、阻止删除对象上的属性、给对象上所有属性的configurable属性描述符设置为false（这意味着属性描述符是不可被修改的了）。

常见误区：被Object.seal处理后的对象就不能在对属性的值进行修改了？*错误*

+ 如果该属性之前的writable属性符就是false的话，那么自然还是不可修改的。
+ 如果该属性之前的writable访问属性符是true的话，那么自然是可以修改的。

下面看个例子：

```javascript
const obj = Object.seal({a: 1})
obj.b = 2;
console.log(obj) // {a: 1}
obj.a = 2
console.log(obj) // {a: 2}
```

### 2.和Object.freeze有什么区别？

挺多不同的，很关键的一点就是Object.freeze不允许对象的属性被重新赋值。如下所示：

```javascript
const obj = Object.freeze({a: {b: 1}})
obj.a = {b: 2}
console.log(obj.a) // {b: 1}
```

Object.freeze真能做到一块内存里面值不能被修改吗？答：做不到，如下所示：

```javascript
const obj = Object.freeze({a: {b: 1}})
obj.a.b = 2
console.log(obj.a) // {b: 2}
```

Object.freeze的使命就只能使第一层属性常量化，其它的也做不了了。