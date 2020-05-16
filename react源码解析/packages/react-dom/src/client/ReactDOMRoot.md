### 1.ReactDOMRoot

```javascript
function ReactDOMRoot(container, options) {
  this._internalRoot = createRootImpl(container, ConcurrentRoot, options);
}

ReactDOMRoot.prototype.render = function(children) {
  const root = this._internalRoot;
  updateContainer(children, root, null, null);
}

ReactRoot.prototype.unmount = function() {
  const root = this._internalRoot;
  const container = root.containerInfo;
  updateContainer(null, root, null, () => {
    unmarkContainerAsRoot(container);
  });
}
```

这是一个构造函数，用来创建RootType类型的数据。什么是RootType类型的数据？答：react应用所挂载的DOM节点对象上的_reactRootContainer就是属于RootType类型。想了解更多关于该类型的内容，可以查看同目录下《一些重要类型》文章。

ConcurrentRoot是什么？答：是react的一种新的仍旧处于开发阶段的工作模式。当前的工作模式（使用ReactDOM.render）在架构上有一些web开发的痛点未解决：一旦触发了一次渲染，那么它是不可被打断的；异步渲染UI；因此react引入了concurrent模式，这个模式可以通过ReactDOM.createRoot(ele).render(Component)来开启，仍旧处于开发阶段。

其中updateContainer方法来自于react-reconciler这个模块，后续在进行介绍。

### 2.ReactDOMBlockingRoot

通过前面的了解，我们也明白了react存在三种工作模式：

+ legacy模式，目前版本的默认模式。通过ReactDOM.render
+ current模式，开发中的模式。提出了：一次UI渲染可被打断重启；异步渲染机制。通过ReactDOM.createRoot(ele).render(component)
+ blocking模式，过渡到current的中间版本。通过ReactDOM.createBlockingRoot(ele).render(component)

而这里的ReactDOMBlockingRoot便是用来创建legacy和blocking模式的构造函数（上面所提到的的ReactDOMRoot是用来创建current模式的）。

```javascript
function ReactDOMBlockingRoot(
  container: Container,
  tag: RootTag,
  options: void | RootOptions,
) {
  this._internalRoot = createRootImpl(container, tag, options);
}
```

在原型上，ReactDOMBlockingRoot和ReactDOMRoot是一致的，这里就不进行过多讨论了。

### 3.createRootImpl方法

把关于服务端渲染的相关代码去掉后，该方法的核心代码如下所示：

```typescript
function createRootImpl(container, tag, options) {
  const root = createContainer(container, tag, hydrate, hydrationCallbacks);
  markContainerAsRoot(root.current, container);
  
  return root;
}
```

其中createContainer来自于react-reconciler模块，后续在进行介绍。毋庸置疑，createContainer方法都是创建了一个fiber的。

### 4.createRoot方法

```typescript
function createRoot(container: Container, options?: RootOptions) {
  return new ReactDOMRoot(container, options);
}
```

用来创建current模式下的根fiber节点。

### 5.createBlockingRoot

```typescript
function createBlockingRoot(container: Container, options?: RootOptions) {
	return new ReactDOMBlockingRoot(container, BlockingRoot, options);
}
```

用来创建blocking模式下的根fiber节点，其中BlockingRoot是一个常量值，为1。

### 6.createLegacyRoot

```typescript
function createBlockingRoot(container: Container, options?: RootOptions) {
	return new ReactDOMBlockingRoot(container, LegacyRoot, options);
}
```

用来创建legacy模式下的根fiber节点，LegacyRoot是一个常量值，值为0。