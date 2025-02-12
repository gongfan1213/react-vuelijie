### React 的优先级机制：Lane 模型

在上一节我们提到 Scheduler 与 React 是两套优先级机制。在 React 中，存在多种使用不同优先级的情况，比如：

- 过期任务或者同步任务使用同步优先级
- 用户交互产生的更新（比如点击事件）使用高优先级
- 网络请求产生的更新使用一般优先级
- Suspense 使用低优先级

React 需要设计一套满足如下需求的优先级机制：

1. 可以表示优先级的不同
2. 可能同时存在几个同优先级的更新，所以还得能表示批的概念
3. 方便进行优先级相关计算

为了满足如上需求，React 设计了 lane 模型。接下来我们来看 lane 模型如何满足以上 3 个条件。

### 表示优先级的不同

想象你身处赛车场。

不同的赛车疾驰在不同的赛道。内圈的赛道总长度更短，外圈更长。某几个临近的赛道的长度可以看作差不多长。

lane 模型借鉴了同样的概念，使用 31 位的二进制表示 31 条赛道，位数越小的赛道优先级越高，某些相邻的赛道拥有相同优先级。

如下：

```javascript
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const SyncBatchedLane: Lane = /*                 */ 0b0000000000000000000000000000010;

export const InputDiscreteHydrationLane: Lane = /*      */ 0b0000000000000000000000000000100;
const InputDiscreteLanes: Lanes = /*                    */ 0b0000000000000000000000000011000;

const InputContinuousHydrationLane: Lane = /*           */ 0b0000000000000000000000000100000;
const InputContinuousLanes: Lanes = /*                  */ 0b0000000000000000000000011000000;

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000100000000;
export const DefaultLanes: Lanes = /*                   */ 0b0000000000000000000111000000000;

const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000001000000000000;
const TransitionLanes: Lanes = /*                       */ 0b0000000001111111110000000000000;

const RetryLanes: Lanes = /*                            */ 0b0000011110000000000000000000000;

export const SomeRetryLane: Lanes = /*                  */ 0b0000010000000000000000000000000;

export const SelectiveHydrationLane: Lane = /*          */ 0b0000100000000000000000000000000;

const NonIdleLanes = /*                                 */ 0b0000111111111111111111111111111;

export const IdleHydrationLane: Lane = /*               */ 0b0001000000000000000000000000000;
const IdleLanes: Lanes = /*                             */ 0b0110000000000000000000000000000;

export const OffscreenLane: Lane = /*                   */ 0b1000000000000000000000000000000;
```

其中，同步优先级占用的赛道为第一位：

```javascript
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
```

从 `SyncLane` 往下一直到 `SelectiveHydrationLane`，赛道的优先级逐步降低。

### 表示“批”的概念

可以看到其中有几个变量占用了几条赛道，比如：

```javascript
const InputDiscreteLanes: Lanes = /*                    */ 0b0000000000000000000000000011000;
export const DefaultLanes: Lanes = /*                   */ 0b0000000000000000000111000000000;
const TransitionLanes: Lanes = /*                       */ 0b0000000001111111110000000000000;
```

这就是批的概念，被称作 lanes（区别于优先级的 lane）。

其中 `InputDiscreteLanes` 是“用户交互”触发更新会拥有的优先级范围。

`DefaultLanes` 是“请求数据返回后触发更新”拥有的优先级范围。

`TransitionLanes` 是 Suspense、useTransition、useDeferredValue 拥有的优先级范围。

这其中有个细节，越低优先级的 lanes 占用的位越多。比如 `InputDiscreteLanes` 占了 2 个位，`TransitionLanes` 占了 9 个位。

原因在于：越低优先级的更新越容易被打断，导致积压下来，所以需要更多的位。相反，最高优的同步更新的 `SyncLane` 不需要多余的 lanes。

### 方便进行优先级相关计算

既然 lane 对应了二进制的位，那么优先级相关计算其实就是位运算。

比如：

#### 计算 a、b 两个 lane 是否存在交集

只需要判断 a 与 b 按位与的结果是否为 0：

```javascript
export function includesSomeLane(a: Lanes | Lane, b: Lanes | Lane) {
  return (a & b) !== NoLanes;
}
```

#### 计算 b 这个 lanes 是否是 a 对应的 lanes 的子集

只需要判断 a 与 b 按位与的结果是否为 b：

```javascript
export function isSubsetOfLanes(set: Lanes, subset: Lanes | Lane) {
  return (set & subset) === subset;
}
```

#### 将两个 lane 或 lanes 的位合并

只需要执行按位或操作：

```javascript
export function mergeLanes(a: Lanes | Lane, b: Lanes | Lane): Lanes {
  return a | b;
}
```

#### 从 set 对应 lanes 中移除 subset 对应 lane（或 lanes）

只需要对 subset 的 lane（或 lanes）执行按位非，结果再对 set 执行按位与：

```javascript
export function removeLanes(set: Lanes, subset: Lanes | Lane): Lanes {
  return set & ~subset;
}
```

### 总结

这就是 React 的优先级模型 lane 模型。

至此，我们已经了解 Fiber 架构、更新的优先级、Scheduler 的实现、lane 模型。从下一节开始，我们会逐步讲解 Concurrent Mode 的各种应用。
