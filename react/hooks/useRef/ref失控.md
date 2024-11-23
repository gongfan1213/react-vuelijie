- 保存对dom对象的引用
- 任何需要被引用的数据都可以保存在ref当中
- 在mount和update的时候对应两个不同的dispatcher

```js
function mountRef(initialvalue){
const hook = mountworkInProgressHook();
const ref ={curreny:initialvalue);
const memoizedState = ref;
return ref;
}
function updateRef（initialValue）{
const hook = updatWorkInProgressHook（）；
//返回保存的ref
return hook。memoizedState;
}
```
- react。createRef方法也会创建同样的数据结构的ref

```js
function createRef(){
const refObject = {
current:null;}
return refObject;}
```

- 有多种组建类型可以赋值ref props,比如HostComponent,ClasscOmponent,ForwardRef,
- ref的工作流程分为两个阶段
- render阶段标记ref flag
- commit阶段，根据ref flag执行ref相关的操作
- markrRef的方法用于标记ref flag

```js
function markRef(current,workInProgress){
const ref = workInProgress.ref;
if((current === null && ref!==null)|| (current !==null )&&current.ref !==ref)){
//标记ref tag
workInProgress。flags ｜= ref;
}
}
```


- 以下这两种情况会标记ref flag
- mount的时候，current ===null ref props存在的时候
- update的时候并且ref,props发生了变化
- 对于标记了ref flags的fibernode在commit阶段的mutation子阶段，首先会移除旧的ref


```js
function commitMutationEffectedOnFiber（finishedWork，root）{
if(flags &Ref){
const current = finishedWork.alternate;
if(current!==null){
//移除旧的ref
commitDetachRef（current）；
}}}
```


