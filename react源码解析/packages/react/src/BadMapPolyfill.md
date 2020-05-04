### 1.功能概述

测试运行环境是否支持Map、Set。如果支持返回false，不支持返回true。

### 2.代码分析

```javascript
new Map([[{}, null]])
```

上面的这段代码的作用类似于下面这样：

```javascript
const m = new Map()
m.set({}, null)
```

同理，针对set的使用方法

```javascript
new Set([1])
```

类似于下面这样

```javascript
const s = new Set()
s.add(1)
```