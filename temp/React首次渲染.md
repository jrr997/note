# React首次渲染

流程：

1. `ReactDOM.render`创建`fiberRootNode`（代码中叫`fiberRoot`）和`rootFiber`,其中`fiberRootNode`是整个应用的根节点，`rootFiber`是`<App/>`所在组件树的根节点。
2. 接下来进入`render阶段`，根据组件返回的`JSX`在内存中依次创建`Fiber节点`并连接在一起构建`Fiber树`，被称为`workInProgress Fiber树`。
3. 在`commit`阶段，渲染构建完的`workInProgress Fiber树`。并且`fiberRootNode`的`current`指针指向`workInProgress Fiber树`使其变为`current Fiber 树`。



Fiber树的创建过程：

fiber树的创建是自上到下吗？