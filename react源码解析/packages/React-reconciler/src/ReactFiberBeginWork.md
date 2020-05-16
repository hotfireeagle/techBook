### 1.reconcileChildren方法

```javascript
function reconcileChildren(
	current: Fiber | null,
   workInProgress: Fiber,
   nextChildren: any,
   renderExpirationTime: ExpirationTimeOpaque,
) {
    if (current === null) {
      workInProgress.child = mountChildFibers(
        workInProgress,
        null,
        nextChildren,
        renderExpirationTime,
      );
    } else {
      workInProgress.child = reconcileChildFibers(
        workInProgress,
        current.child,
        nextChildren,
        renderExpirationTime,
      );
    }
  }
```

看到这里的疑惑是：这个参数current到底是代表了什么呢？

