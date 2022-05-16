React

```javascript
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

过程: JSX -> React.createElement() -> element tree



fiber.tag

```javascript
  var FunctionComponent = 0;
  var ClassComponent = 1;
  var IndeterminateComponent = 2; // Before we know whether it is function or class

  var HostRoot = 3; // Root of a host tree. Could be nested inside another node.

  var HostPortal = 4; // A subtree. Could be an entry point to a different renderer.

  var HostComponent = 5;
  var HostText = 6;
  var Fragment = 7;
  var Mode = 8;
  var ContextConsumer = 9;
  var ContextProvider = 10;
  var ForwardRef = 11;
  var Profiler = 12;
  var SuspenseComponent = 13;
  var MemoComponent = 14;
  var SimpleMemoComponent = 15;
  var LazyComponent = 16;
  var IncompleteClassComponent = 17;
  var DehydratedFragment = 18;
  var SuspenseListComponent = 19;
  var FundamentalComponent = 20;
  var ScopeComponent = 21;
  var Block = 22;
  var OffscreenComponent = 23;
  var LegacyHiddenComponent = 24;
```



`performUnitOfWork`方法会创建下一个`Fiber节点`并赋值给`workInProgress`，并将`workInProgress`与已创建的`Fiber节点`连接起来构成`Fiber树`。



`performUnitOfWork`分两部分: beginWork和completeWork



```javascript
function beginWork(
  current: Fiber | null, // 上一次更新时的Fiber节点，mount时为null
  workInProgress: Fiber, // 当前组件对应的Fiber节点
  renderLanes: Lanes, // 优先级
): Fiber | null {
  // ...省略函数体
}

// beginWork的作用是为workInProgress创建下一个节点，注意workInProgress是Fiber节点，此时的Fiber节点还没有最新的DOM，会在completeWork时添加上

// beginWork生成了新的子Fiber节点并赋值给workInProgress.child，作为本次beginWork返回值，并作为下次performUnitOfWork执行时workInProgress的传参


```



beginWork在update时会为Fiber创建effectTag，而mount不会。

```javascript
// DOM需要插入到页面中
export const Placement = /*                */ 0b00000000000010;
// DOM需要更新
export const Update = /*                   */ 0b00000000000100;
// DOM需要插入到页面中并更新
export const PlacementAndUpdate = /*       */ 0b00000000000110;
// DOM需要删除
export const Deletion = /*                 */ 0b00000000001000;
```



completeWork

`mount`时的主要逻辑包括三个：

- 为`Fiber节点`生成对应的`DOM节点`
- 将子孙`DOM节点`插入刚生成的`DOM节点`中
- 与`update`逻辑中的`updateHostComponent`类似的处理`props`的过程

**最终根节点上生成了一棵要被渲染到页面上的DOM树，这个DOM树可以直接添加到页面中**



当`update`时，`Fiber节点`已经存在对应`DOM节点`，所以不需要生成`DOM节点`。需要做的主要是处理`props`，比如：

- `onClick`、`onChange`等回调函数的注册
- 处理`style prop`
- 处理`DANGEROUSLY_SET_INNER_HTML prop`
- 处理`children prop`

在update过后，如果Fiber存在effectTag，说明其需要操作(更新、删除、添加等)对应的DOM。我们从根节点开始遍历整颗Fiber树来更新DOM的话，效率太低。我们可以在为Fiber节点创建effectTag时，把这个节点放进链表、当需要渲染DOM时，可以直接从链表中找到所有需要更新的DOM(Fiber.stateNode)。在React中，节点放进链表的操作是在completeWork完成时。

