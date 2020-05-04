### 1.功能概述

在这个文件中定义了整个框架中比较重要的一些系统公用常量，比如说react中的元素类型定义：ELEMENT_TYPE、FRAGMENT_TYPE等。



### 2.源码中为什么常量的值采用16进制

采用16进制来表示常量性能会好一些。为什么，搜遍整个中文互联网也没找到靠谱的解释答案（这里站在计算机角度进行考虑，而非编写者），基本上都是围绕下面这点：

+ 进行位运算性能更加优异。但是关于这个逻辑为什么成立，暂时还是搞不清楚的，应该是和V8的实现有关。
+ ~~16进制能够节约系统资源。~~个人来看的话，这个依据是站不住脚的，因为在JavaScript中，所有的数字类型都是双精度浮点形的64位数据，[下面是摘抄自ecmascript语法中所描述的一句话](http://www.ecma-international.org/ecma-262/6.0/#sec-terms-and-definitions-number-value)：

> The Number type has exactly 18437736874454810627 (that is, 264−253+3) values, representing the double-precision 64-bit format IEEE 754-2008 values as specified in the IEEE Standard for Binary Floating-Point Arithmetic, except that the 9007199254740990 (that is, 253−2) distinct “Not-a-Number” values of the IEEE Standard are represented in ECMAScript as a single special **NaN** value. (Note that the **NaN** value is produced by the program expression `NaN`.)



### 3.源码中为什么要使用Symbol来定义常量

首先我们先来理清一下这里对系统公有常量的定义：

+ 首先得是一个常量：如果一块内存空间是个常量的话，那么存在它里面的东西就不应该发生变化来。
+ 系统公有的：这块内存空间应该是不可被替代的。对于计算机来说，value更有意义。但是对于编程人员来说，value所相关的变量标识符更加被高频使用。那么问题来了，如果使用另一个内存单元也存储同一个常量value的话，那岂不是这两个变量标识符都在这个系统里面起到了同样的作用？但是这样做很显然是不对的，它引起了混乱。

因此，基于上面亮点原因，使用const和symbol来组合搭配，能够更好的定义出一个健壮的系统公有常量。它实现了：

+ 这块内存空间的值不可变。
+ 值和内存空间的对应关系是一对一的，不容易被其他同值的内存空间冒牌使用。



### 4.Symbol.for是什么

先搞清楚Symbol是什么：Symbol类型是js语言的基本数据类型，可以通过Symbol函数来返回一个Symbol类型的值。每个Symbol函数所返回的值都是唯一的。Symbol函数接受一个可选参数，这个可选参数仅仅只是为了描述symbol，没有其它任何用处了。

考一下symbol的正确使用方法：

```javascript
const s1 = Symbol() // right
const s2 = new Symbol() // wrong
```

**对于Symbol来说，不能通过使用构造函数的形式进行使用，就当普通方法来使用就行了。**

在MDN官网上面，对于Symbol.for的描述如下所示：这个方法接受一个key，如果运行时中的symbol注册表中有对应symbol的话，那么返回它；如果没有的话，则新建一个和这个key参数所关联的symbol，然后将其放到注册表里面。

```javascript
Symbol.for('foo') // 创建和foo关联的一个Symbol
Symbol.for('foo') // 直接取出symbol注册表中foo这个key所关联的symbol
```



### 5.如何判断一个对象是否支持iterator接口

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


