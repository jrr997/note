# React Render阶段

```javascript

  function workLoopSync() {
    // Already timed out, so perform work without checking if we need to yield.
    while (workInProgress !== null) {
      performUnitOfWork(workInProgress);
    }
  }


  function performUnitOfWork(unitOfWork) {
    var next;
	next = beginWork$1(current, unitOfWork, subtreeRenderLanes); //  beginWork$1 return fiber.child
    if (next === null) {
      // If this doesn't spawn new work, complete the current work.
      completeUnitOfWork(unitOfWork);
    } else {
      workInProgress = next;
    }
  }


   beginWork$1 = function (current, unitOfWork, lanes) {
	  ...
      return beginWork(current, unitOfWork, lanes); // beginWork的工作是传入当前Fiber节点，创建子Fiber节点
      ...
    };
       
       
	// 如果要通知Renderer将Fiber节点对应的DOM节点插入页面中，需要满足两个条件：
	// 1.fiber.stateNode存在，即Fiber节点中保存了对应的DOM节点
	// 2.(fiber.effectTag & Placement) !== 0，即Fiber节点存在Placement effectTag
	// 我们知道，mount时，fiber.stateNode === null，且在reconcileChildren中调用的mountChildFibers不会为Fiber节点赋值effectTag。那么首屏渲染如何完成呢？
	// 针对第一个问题，fiber.stateNode会在completeWork中创建，我们会在下一节介绍。
	// 第二个问题的答案十分巧妙：假设mountChildFibers也会赋值effectTag，那么可以预见mount时整棵Fiber树所有节点都会有Placement effectTag。那么commit阶段在执行DOM操作时每个节点都会执行一次插入操作，这样大量的DOM操作是极低效的。
	// 为了解决这个问题，在mount时只有rootFiber会赋值Placement effectTag，在commit阶段只会执行一次插入操作。
   function reconcileChildren(current, workInProgress, nextChildren, renderLanes) {
    if (current === null) { // 挂载时直接添加节点，而不是通过effect tag的方式
      // If this is a fresh new component that hasn't been rendered yet, we
      // won't update its child set by applying minimal side-effects. Instead,
      // we will add them all to the child before it gets rendered. That means
      // we can optimize this reconciliation pass by not tracking side-effects.
      workInProgress.child = mountChildFibers(workInProgress, null, nextChildren, renderLanes);
    } else { // 更新时通过追踪effect tag来找到需要更新的节点并进行更新
      // If the current child is the same as the work in progress, it means that
      // we haven't yet started any work on these children. Therefore, we use
      // the clone algorithm to create a copy of all the current children.
      // If we had any progressed work already, that is invalid at this point so
      // let's throw it out.
      workInProgress.child = reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes);
    }
  }
```

