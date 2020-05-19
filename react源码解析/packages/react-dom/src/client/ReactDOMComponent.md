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

### 5.setInitalProperties

作用：给dom元素设置一些默认值，这里只研究input元素。针对input的精简代码如下所示：

```javascript
ReactDOMInputInitWrapperState(domElement, rawProps);
props = ReactDOMInputGetHostProps(domElement, rawProps);
legacyTrapBubbledEvent(TOP_INVALID, domElement);
```

看下ReactDOMInputInitWrapperState对dom元素做了些什么操作：

```typescript
function ReactDOMInputInitWrapperState(ele, props) {
  const defaultValue = props.defaultValue == null ? '' : props.defaultValue;
  node._wrapperState = {
    initialChecked:
      props.checked != null ? props.checked : props.defaultChecked,
    initialValue: getToStringValue(
      props.value != null ? props.value : defaultValue,
    ),
    controlled: isControlled(props),
  };
}
```

可以看到，react给input元素添加了一些属性，并且挂载在其dom节点上面。不妨随便打开一个react应用，获取到一个input元素，可以发现在这个dom对象上面拥有一个_wraperState属性。其中有一个属性挺重要的，那就是controlled，判断一个节点是否是受控的，那么对于react来说，它是怎么判断一个dom节点是受控节点呢？如下所示：

```typescript
function isControlled(props) {
  const usesChecked = props.type === 'checkbox' || props.type === 'radio';
  return usesChecked ? props.checked != null : props.value != null;
}
```

可以发现，如果一个元素是checkbox或者radio的话，那么只要props里面有checked，就说明它是一个受控组件；其余节点，只要props上面存在value，也认为它是一个受控节点。

在回归正题，看看ReactDOMInputGetHostProps做了些什么：

```typescript
function getHostProps(element: Element, props: Object) {
  const node = ((element: any): InputWithWrapperState);
  const checked = props.checked;

  const hostProps = Object.assign({}, props, {
    defaultChecked: undefined,
    defaultValue: undefined,
    value: undefined,
    checked: checked != null ? checked : node._wrapperState.initialChecked,
  });

  return hostProps;
}
```

不懂，这里为啥不顾为啥要使用undefined来覆盖props.defaultChecked等几个属性。

接下来看看legacyTrapBubbledEvent做了些啥（省略后的代码）：

```javascript
function legacyTrapBubbledEvent(topLevelType, element, listenerMap) {
  const listener = addTrappedEventListener(element, topLevelType, PLUGIN_EVENT_SYSTEM, false);
}
```

看看addTrappedEventListener做了些什么：

```typescript
function addTrappedEventListener(targetContainer, topLevelType, eventSystemFlags, capture) {
  const rawEventName = getRawEventName(topLevelType); // 事件名称
  const listener = createEventListenerWrapperWithPriority(
  	targetContainer,
    topLevelType,
    eventSystemFlags,
  );
  const unscribe = capture ? 
        addEventCaptureListener(targetContainer, rawEventName, listener)
  			:
  			addEventBubbleListener(targetContainer, rawEventName, listener);
  return unscribe;
}
```

关于这个事件是怎么合成的，后续出一篇关于react事件处理的文章，这里就不继续深入下去了。

### 6.diffProperties

作用：该方法用来计算两个代表property的props具有哪些不同。