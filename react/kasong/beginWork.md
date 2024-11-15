- 首先会判断当前流程是mount还是update阶段，判断依据是current FiberNode是否存在的
- 如果当前流程是update流程，则wip fiberNode存在对应的current FiberNode如果本次更新不影响fiberNode.child则可以复用对应的current fiberNode，
  # 【插入图片1】
- 如果无法复用current fiberNode则mount和update的流程是一致的
- 根据wip.tag进入不同的类型的元素的处理分支
- 使用reconcile算法生成下一级的fiberNode
- 两个流程的区别是最终是否会为生成子的fiberNode标记的副作用的flags
```js
// beginWork.js

// 引入必要的模块和常量
import { HostRoot, HostComponent, HostText, IndeterminateComponent } from './ReactWorkTags';
import { NoFlags } from './ReactFiberFlags';
import { createFiber } from './ReactFiber';
import { updateHostComponent, updateHostText } from './ReactFiberReconciler';

// beginWork 函数的定义
function beginWork(current, workInProgress, renderExpirationTime) {
    // 如果当前 Fiber 节点不存在，说明是初始渲染
    if (current === null) {
        // 根据节点类型创建新的 Fiber 节点
        switch (workInProgress.tag) {
            case HostRoot:
                // 处理根节点
                return updateHostRoot(current, workInProgress, renderExpirationTime);
            case HostComponent:
                // 处理宿主组件
                return updateHostComponent(current, workInProgress, renderExpirationTime);
            case HostText:
                // 处理文本节点
                return updateHostText(current, workInProgress, renderExpirationTime);
            case IndeterminateComponent:
                // 处理不确定的组件
                return updateIndeterminateComponent(current, workInProgress, renderExpirationTime);
            default:
                // 处理其他类型的组件
                return updateFunctionComponent(current, workInProgress, renderExpirationTime);
        }
    }

    // 如果当前 Fiber 节点存在，说明是更新过程
    switch (workInProgress.tag) {
        case HostComponent:
            // 更新宿主组件
            return updateHostComponent(current, workInProgress, renderExpirationTime);
        case HostText:
            // 更新文本节点
            return updateHostText(current, workInProgress, renderExpirationTime);
        case IndeterminateComponent:
            // 更新不确定的组件
            return updateIndeterminateComponent(current, workInProgress, renderExpirationTime);
        default:
            // 更新其他类型的组件
            return updateFunctionComponent(current, workInProgress, renderExpirationTime);
    }
}

// 更新根节点的逻辑
function updateHostRoot(current, workInProgress, renderExpirationTime) {
    // 处理根节点的更新逻辑
    // 这里可以包括状态更新、子树的处理等
    return null; // 返回下一个要处理的 Fiber 节点
}

// 更新宿主组件的逻辑
function updateHostComponent(current, workInProgress, renderExpirationTime) {
    // 处理宿主组件的更新逻辑
    // 这里可以包括 props 的比较、子节点的处理等
    return null; // 返回下一个要处理的 Fiber 节点
}

// 更新文本节点的逻辑
function updateHostText(current, workInProgress, renderExpirationTime) {
    // 处理文本节点的更新逻辑
    return null; // 返回下一个要处理的 Fiber 节点
}

// 更新不确定组件的逻辑
function updateIndeterminateComponent(current, workInProgress, renderExpirationTime) {
    // 处理不确定组件的更新逻辑
    return null; // 返回下一个要处理的 Fiber 节点
}

// 更新函数组件的逻辑
function updateFunctionComponent(current, workInProgress, renderExpirationTime) {
    // 处理函数组件的更新逻辑
    return null; // 返回下一个要处理的 Fiber 节点
}

export { beginWork };

```

- HostComponment代表的是原生的elemnt类型div,span
- IndeterminateComponent是FC mount进入的分支，update的时候键入Function Component
- HostText代表的是文本元素类型
- 没有命中优化策略，最终会进入reconcileChildren方法，
```js
  function reconcileChildren(current, workInProgress, nextChildren, renderLanes) {
    // 1. 如果当前没有子节点，直接挂载新的子节点
    if (current === null) {
        // 使用 mountChildFibers 方法挂载新的子节点
        return mountChildFibers(workInProgress, null, nextChildren, renderLanes);
    }

    // 2. 如果当前有子节点，进行协调
    return reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes);
}
function childReconciler(shouldTrackSideEffects) {
    return function reconcileChildren(current, workInProgress, newChildren) {
        // 1. 获取当前的子 Fiber
        let currentChild = current ? current.child : null;
        let newChild = newChildren;

        // 2. 处理新子节点的协调
        if (currentChild === null) {
            // 如果当前没有子节点，直接挂载新子节点
            return mountChildFiber(workInProgress, newChild, shouldTrackSideEffects);
        } else {
            // 如果当前有子节点，进行协调
            return reconcileChildFibers(workInProgress, currentChild, newChild, shouldTrackSideEffects);
        }
    };
}
function mountChildFiber(workInProgress, newChild, shouldTrackSideEffects) {
    // 创建新的 Fiber 节点
    const newFiber = createFiberFromElement(newChild);
    
    // 将新 Fiber 连接到工作中的 Fiber 树
    newFiber.return = workInProgress;
    workInProgress.child = newFiber;

    // 处理副作用
    if (shouldTrackSideEffects) {
        // 记录挂载的副作用
        workInProgress.effectTag |= Placement;
    }

    return newFiber;
}
function reconcileChildFibers(workInProgress, currentChild, newChild, shouldTrackSideEffects) {
    // 这里会有复杂的逻辑来处理不同类型的子节点
    // 例如：比较类型、key、props 等

    // 假设我们有一个简单的逻辑来处理
    if (currentChild.key === newChild.key) {
        // 如果 key 相同，更新现有的 Fiber
        const updatedFiber = updateFiber(currentChild, newChild);
        updatedFiber.return = workInProgress;
        return updatedFiber;
    } else {
        // 否则，可能需要删除旧的 Fiber 并挂载新的 Fiber
        // 这里省略了具体的删除逻辑
        return mountChildFiber(workInProgress, newChild, shouldTrackSideEffects);
    }
}

```
- mountChildFiber 方法用于挂载新的子 Fiber。它会创建一个新的 Fiber 节点并将其添加到工作中的 Fiber 树中
- reconcileChildFibers 方法用于协调现有的子 Fiber 和新的子节点。它会根据新的子节点的类型和状态来决定是更新、删除还是挂载新的 Fiber。
- shouldTrackSideEffects 参数用于决定是否需要记录副作用
```js
function placeChild(newFiber, lastPlacedIndex, newIndex, shouldTrackSideEffects) {
newFiber.index= newIndex;
 if (!shouldTrackSideEffects) {
//不标记Placement
newFiber.flags |=Forked;
return  lastPlacedIndex;
}
    if (shouldTrackSideEffects) {
        // 需要跟踪副作用
        if (newFiber.index < lastPlacedIndex) {
            // 需要移动
            newFiber.effectTag = Placement; // 标记为需要放置
        }
    }

    // 更新 lastPlacedIndex
    return newIndex;
}

```
- ChildDeletion删除操作
- placement插入或者移动操作
- 
