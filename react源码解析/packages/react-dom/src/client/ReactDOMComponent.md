### 1.功能概述

该文件定义了一个js所描述的element对象如何转换回实际DOM的逻辑功能，都是属于真实的DOM操作。

### 2.setInitialDomProperties

将props中的一些属性转移到实际DOM中。关键代码如下所示：

```typescript
function setInitialDomProperties(
	tag: string,
   domElement: Element,
   rootContainerElement: Element | Document,
   nextProps: Object,
   isCustomComponentTag: boolean,
) {
    for (const propKey in nextProps) { // 不管是在原型上还是示列上，可遍历还是不可遍历
      if (!nextProps.hasOwnProperty(propKey)) { // 非示列上
        continue;
      }
      const nextProp = nextProps[propKey];
      
      if (propKey === STYLE) { // props中的style 样式数据
        setValueForStyles(domElement, nextProp);
      } else if (propKey === DANGEROUSLY_SET_INNER_HTML) { // 设置html
        setInnerHTML(domElement, nextProp[__html]);
      } else if (propKey === CHILDREN) {
        if (typeof nextProp === 'string') {
          setTextContent(domElement, nextProp);
        } else if (typeof nextProp === 'number') {
          setTextContent(domElement, '' + nextProp);
        }
      } // ......省略若干代码
      else if (nextProp != null) {
        setValueForProperty(domElement, propKey, nextProp, isCustomComponentTag);
      }
    }
  }
```

如何将props中的数据同步到dom中，这个是分情况来看的，下面我们来考虑setValueForStyles的实现：

```typescript
function setValueForStyles(node, styles) {
  const style = node.style; // 获取到dom元素的style数据
  
  for (let styleName in styles) {
    if (!styles.hasOwnProperty(styleName)) {
      return;
    }
    
    const styleValue = dangerousSetValue(styleName, styles[styleName], isCustom);
    
    if (styleName === 'float') {
      styleName = 'cssFloat'; // float属性在cssStyleDeclaration中的正确表示
    }
    
    style.setProperty(styleName, styleValue); //形如下面
    style[styleName] = styleValue;
  }
}
```

如上面所示：获取DOM元素的style对象，然后通过setProperty方法设置样式；或者直接改变对象上的数据也行。

而dangerousSetValue方法就是对value做了一些处理，比如说允许开发者在写px单位的width时不用携带单位px等等便利操作。

### 3.updateDOMProperties

调用一次，可更新多个dom上面的属性。方法代码如下所示：

```typescript
function updateDOMProperties(domElement, updatePayload, wasCustomComponentTag) {
  for (let i = 0; i < updatePayload.length; i += 2) {
    const propKey = updatePayload[i];
    const propValue = updatePayload[i + 1];
    
    if (propKey === STYLE) {
      setValueForStyles(domElement, propValue);
    } else if (propKey === DANGEROUSLY_SET_INNER_HTML) {
      setInnerHTML(domElement, propValue);
    } else if (propKey === CHILDREN) {
      setTextContent(domElement, propValue);
    } else {
      setValueForProperty(domElement, propKey, propValue, isCustomComponentTag);
    }
  }
}
```

### 4.createElement

我们写的jsx元素最终还是需要反馈到DOM元素上去的，因此创建DOM的过程也是被react所一手包办的，关于代码实现细节，这里就不多做解释了，总之就是返回了一个DOM节点。

