- reconciler工作的阶段在react内部被称为render阶段，
- classComponent的render函数,FunctionComponent后文被简称为FC函数本身都在该阶段被调用
- 根据scheduler调度的结果不同，render阶段可能开始于performSyncWorkOnRoot同步更新流程，performConcurrentWorkOnRoot并发更新的流程
- workInProgress代表生成Fiber Tree工作已经进行到了wip fiberNode,performUnitOfWork方法会创建下一个fiberNode并且会赋值给wip,wip和已经创建的fiberNode连接起来构成Fiber Tree
-  wip==null代表fiber Tree的构建工作已经结束的
-  另种方法唯一区别是是否调用shouldYield是否可以中断

  ```js
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
}
}
//perfomrConcurrentWorkOnRoot会执行改方法
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
}
}
```
## 流程概览
- Fiber Reconciler是从stack reconciler重构而来的，通过遍历的方式实现可以中断的递归
- performUnitOfWork工作可以分为两部分:递和归
- 递阶段会从HostRootFiber开始以DFS的方式遍历，为遍历到的每一个fiberNode执行beginWork方法，该方法会根据传入的fiberNode创建下一级的fiberNode
- 1.下一级只有一个元素，beginWork方法会创建子fiberNode并且和wip连接
- <ul><li></li></ul>
- LiFierber.return =UlFiber;
- 2.下一级有多个元素，这个时候的beginWork方法会依次创建所有的子fiberNode,并且连接在一起的，为首的子fiberNode和wip连接
```js
<ul>
<li></li>
<li></li>
<li></li>
</ul>
Li0Fiber.sibling =Li1FIber;
Li1Fiber.sibling = Li2Fiber;
Li0Fiber.return =ULFiber;
```
- 当遍历到叶子元素不包含子fiberNode的时候，performUnitOfWork就会进入归的阶段
- 归的阶段会调用complete方法处理fiberNode当某个fiberNode执行完了completeWork方法以后，如果存在兄弟fiberNode(fiberNode.sibling!=null)会进入其兄弟的fiberNode的递阶段，如果不存在兄弟兄弟ifberNode则进入父fiberNode的归
- 递和归会交替执行，直到HostRootFiber的归的阶段render工作结束
- App DIV hhello hellobeginWork hello completeWork, span beginWork span completeWork div completework app completework HostRootFiber completeWork
```js
function performUnitOfWork(fiberNode) {
  // 省略执行beginWork工作
  if(fiberNode.child) {
    performUnitOfWork(fiberNode.child)
  }
  //省略执行completeWork工作
  if((fiberNode.sibling) {
    performUnitOfWork(fiberNode.sibling)
  })
}
```
