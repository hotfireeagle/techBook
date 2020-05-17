### 1.legacyCreateRootFromDOMContainer

```typescript
function legacyCreateRootFromDOMContainer(container: Container, forceHydrate: boolean) {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container); // 表示是否服务端渲染
  
  if (!shouldHydrate) {
    let rootSibling;
    while((rootSibling = container.lastChild)) {
      container.removeChild(rootSibling);
    }
  }
  
  return createLegacyRoot(
  	container,
    shouldHydrate ? { hydrate: true } : undefined,
  )
}
```

从源码我们可以看出，哪怕开发者在模版html文件中的react应用挂载点#root下面写了一些DOM元素，经过react的一通操作后，它们也会被去掉。

createLegacyRoot：创建了一个工作在legacy模式下的根fiber节点。具体可参照同目录下ReactDOMRoot文件。

调用这个方法之后，最终创建出了一个legacy模式下的根fiber节点并且进行返回。

### 2.legacyRenderSubtreeIntoContainer

开启渲染过程。关键代码如下所示：

```javascript
function legacyRenderSubtreeIntoContainer(
	parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: Container,
  forceHydrate: boolean,
  callback: ?Function,
) {
    let root = container._reactRootContainer;
    let fiberRoot;
    
    if (!root) { // 表示第一次创建的过程
      root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      	container,
        forceHydrate,
      ); // 第一次挂载，还不存在根fiber节点，那就创建出来
      fiberRoot = root._internalRoot; // 所创建出来的根fiber节点
      
      // ......省略若干代码
      
      // 第一次整个应用的挂载不应该batched
      unbatchedUpdates(() => {
        updateContainer(children, fiberRoot, parentComponent, callback);
      })
    } else { // 表示已经创建来根fiber了，那么此时应该执行update过程（意味着是要被调度的）
      fiberRoot = root._internalRoot;
      
      // ......省略若干代码
      
      updateContainer(children, fiberRoot, parentComponent, callback);
    }
    
    return getPublicRootInstance(fiberRoot);
  }
```

函数作用：开启整个react应用的渲染过程。如果是第一次挂载的话，那么直接渲染，不会被批处理进入调度；如果是更新的话，那么进入调度。

参数解释：children，一般表示ReactElement或者js语言里面的简单值；container：即整个应用的挂载dom节点。

### 3.render

render方法我们使用的再多不过了，一般的使用方式都是传入传入两个参数，分别是第一个用来表示component，第二个用来表示dom。

```typescript
function render(component, ele, cb) {
  // 省略若干代码
  return legacyRenderSubtreeIntoContainer(
  	null,
    component,
    ele,
    false, // 客户端渲染
    cb,
  );
}
```

