---
layout: post
title: Deep into React 2 - setState 更新
date: 2021-03-16
categories: 前端
tags: frontend react
---

上次我们分析了下面这个例子首次 mount 的过程

```
class App extends React.Component {
  componentDidMount() {
    console.log('App componentDidMount');
  }
  render() {
    return (
      <div className="wrap">
        <Box />
        <span>list组件</span>
      </div>
    );
  }
}
class Box extends React.Component {
  state = {
    count: 0,
  };
  handleClick = () => {
    this.setState(state => {
      return {
        count: ++state.count,
      };
    });
  };
  componentDidMount() {
    console.log('Box componentDidMount');
  }
  render() {
    return (
      <button onClick={this.handleClick}>点击次数({this.state.count})</button>
    );
  }
}
export default App;
```

本次我们来分析 button 按钮被点击后，页面的渲染过程。

首次 mount 后的 fiber 树状态如下：

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/efbf460e-59c4-6fc2-b0ed-52ce3b6064c9.png)

## setState
我们跳过事件分发机制，从 setState 执行说起

```
Component.prototype.setState = function (partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState')
}

const classComponentUpdater = {
  isMounted,
  enqueueSetState(inst, payload, callback) {
    // 1. 获取class实例对应的Fiber节点
    const fiber = getInstance(inst)
    // 2. 创建update对象
    …
    const update = createUpdate(currentTime, expirationTime, suspenseConfig)
    /**
    payload 就是setState的这个回调函数
    state => {
      return {
        count:++state.count,
      };
    }
    */
    update.payload = payload
    if (callback !== undefined && callback !== null) {
      update.callback = callback
    }
    // 3. 将update对象添加到当前Fiber节点的updateQueue队列当中
    enqueueUpdate(fiber, update)
    // 4. 在当前Fiber节点上进行调度更新
    scheduleUpdateOnFiber(fiber, expirationTime)
  }
}
```

`setState` 在 Box 组件对应 fiber 的 updateQueue 上, 然后触发更新。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/5e858ed6-0f42-694d-eae1-4e049944fa2b.png)

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/ffaff1a3-266d-bdf1-4852-f02bb045461b.png)

```
type ExecutionContext = number;

const NoContext = /*                    */ 0b000000;
// 批量更新 batchedUpdates, 对应非批量更新，即直接更新
const BatchedContext = /*               */ 0b000001;
// 批量事件更新 batchedEventUpdates
const EventContext = /*                 */ 0b000010;
// 事件 dispatch 的时候
const DiscreteEventContext = /*         */ 0b000100;
const LegacyUnbatchedContext = /*       */ 0b001000;
// 比如 renderRootSync，也就是建立 fiber 树的过程
const RenderContext = /*                */ 0b010000;
// 渲染后 commit 的时候
const CommitContext = /*                */ 0b100000;
```

```
export function scheduleUpdateOnFiber(
  fiber: Fiber,
  expirationTime: ExpirationTime,
) {
  const root = markUpdateTimeFromFiberToRoot(fiber, expirationTime);

  // leagcy下, expirationTime = Sync
  if (expirationTime === Sync) {
    if (
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      // ... 第一次render进入
    } else {
      // 事件分发的时候在 discreteUpdates 函数里, 会给 executionContext 赋值
      // executionContext |= DiscreteEventContext;
      // 更新时进入
      ensureRootIsScheduled(root);
      ...
    }
  }
}
```

## markUpdateTimeFromFiberToRoot
从当前 fiber 开始一直访问到根节点，修改所有节点的 childExpirationTime.

```
export const MAX_SIGNED_31_BIT_INT = 1073741823;
export const Sync = MAX_SIGNED_31_BIT_INT; // 最大的符号整形数

function markUpdateTimeFromFiberToRoot(fiber, expirationTime) {
  // Update the source fiber's expiration time
  if (fiber.expirationTime < expirationTime) {
    fiber.expirationTime = expirationTime;
  }
  let alternate = fiber.alternate;
  if (alternate !== null && alternate.expirationTime < expirationTime) {
    alternate.expirationTime = expirationTime;
  }

  // Walk the parent path to the root and update the child expiration time.
  let node = fiber.return;
  let root = null;
  if (node === null && fiber.tag === HostRoot) {
    root = fiber.stateNode;
  } else {
    while (node !== null) {
      alternate = node.alternate;
      if (node.childExpirationTime < expirationTime) {
        node.childExpirationTime = expirationTime;
        if (
          alternate !== null &&
          alternate.childExpirationTime < expirationTime
        ) {
          alternate.childExpirationTime = expirationTime;
        }
      } else if (
        alternate !== null &&
        alternate.childExpirationTime < expirationTime
      ) {
        alternate.childExpirationTime = expirationTime;
      }
      if (node.return === null && node.tag === HostRoot) {
        root = node.stateNode;
        break;
      }
      node = node.return;
    }
  }
}
```
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/9cb97956-fa8f-de3d-b7df-9cc48468aeab.png)

> 经过这个过程，可以保证后续的 render 阶段，如果发现孩子的过期时间小于当前时间，意思是这个孩子没有更新，则停止调度

## ensureRootIsScheduled
ensureRootIsScheduled 涉及调度机制，我们简单的把它理解为最终触发了 performSyncWorkOnRoot，进入 render 阶段。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/663bde0a-5eb4-5308-1b69-8a737674073a.png)

## prepareFreshStack
render 阶段一开始，先准备环境。新建了一些指针，最重要的是把 workInProgress 指到了 fiberRoot.current.alternate
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/d707a41d-509f-57b5-5122-aefcb869292d.png)

## 进入 workLoop
进入 render 阶段，回溯 fiber 树，交替执行 beginWork 和 completeWork。

```
let workInProgress = null

function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}

// ... 函数中省略了与首次render无关代码, 先关心主流程
function performUnitOfWork(unitOfWork: Fiber): void {
  // 第一次render时, unitOfWork=HostRootFiber, alternate已经初始化
  const current = unitOfWork.alternate;
  let next;
  // 1. 创建Fiber节点
  next = beginWork(current, unitOfWork, renderExpirationTime);
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  if (next === null) {
    //2. 处理beginWork中的Fiber节点
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }
  // ...
}
```

## beginWork

```
function beginWork(current, workInProgress, renderExpirationTime) {
  const updateExpirationTime = workInProgress.expirationTime;

  if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;

    if (oldProps !== newProps || hasLegacyContextChanged()) {
      // props 或 context 改变了
      didReceiveUpdate = true;
    } else if (updateExpirationTime < renderExpirationTime) {
      // 此 fiber 没有变化，NoWork, 这个分支的主要逻辑是克隆孩子并返回
      didReceiveUpdate = false;
      // ...
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderExpirationTime,
      );
    } else {
      // setState
      didReceiveUpdate = false;
    }
  }

  // Before entering the begin phase, clear pending update priority.
  workInProgress.expirationTime = NoWork;

  switch (workInProgress.tag) {
    case XXXComponent: {
      // ...
      return updateXXXComponent(...)
      // ...
    }
  }
}
```

函数开始有 3 个分支：
1. props 或者 context 改变，didReceiveUpdate 设置成 true，调 updateXXX 函数
2. 无 props 变化也无 state 变化，则执行bailoutOnAlreadyFinishedWork
3. 如果是 state 变化，则 didReceiveUpdate 为 false，调 updateXXX 函数

# 分析例子
下面我们来分析文章开头说的例子，首先是 `RootFiber` 的 `beginWork`。

## beginWork for RootFiber

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/29037114-30b2-ead5-68b8-00674a16c815.png)

进入第 2 个分支，调用 bailoutOnAlreadyFinishedWork 对孩子进行克隆。
```
function bailoutOnAlreadyFinishedWork(current, workInProgress, renderExpirationTime,) {
  ...
  const childExpirationTime = workInProgress.childExpirationTime;
  if (childExpirationTime < renderExpirationTime) {
    // 此 fiber 的孩子也没有更新
    return null;
  } else {
    // 孩子有更新，对孩子进行一层克隆
    cloneChildFibers(current, workInProgress);
    return workInProgress.child;
  }
}
```
注意 `childExpirationTime < renderExpirationTime` 这个分支，会跳过对此分支的更新，这就是 `markUpdateTimeFromFiberToRoot` 的作用，优化了不需要更新的分支。

此次 beginWork 后，fiber 树的示意：
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/26084b59-00df-ef53-30ae-e5373aa59399.png)

## beginWork for App and div
过程和 RootFiber 类似，此 2 次 beginWork 后，fiber 树的示意：
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/c583e639-d057-aba6-e00f-1ac90e8f2be3.png)

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/52978704-1ba4-a7c4-f182-dd9132d673b1.png)

## beginWork for Box
显然这次会进入 beginWork 的第 3 个分支。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/df3b68a7-b71b-59e5-1697-776451807af8.png)

```
function beginWork() {
  // ...
  case ClassComponent: {
     // ...
    return updateClassComponent(
      current,
      workInProgress,
      Component,
      resolvedProps,
      renderExpirationTime,
    );
  }
}
```

`updateClassComponent` 主要就是 2 个步骤：`updateClassInstance` 和 `finishClassComponent`。

```
function updateClassComponent(...) {
  const instance = workInProgress.stateNode;
  let shouldUpdate;

  shouldUpdate = updateClassInstance(
    current,
    workInProgress,
    Component,
    nextProps,
    renderExpirationTime,
  );

  const nextUnitOfWork = finishClassComponent(
    current,
    workInProgress,
    Component,
    shouldUpdate,
    hasContext,
    renderExpirationTime,
  );
  return nextUnitOfWork;
}
```

### updateClassInstance
`updateClassInstance` 主要做以下工作：
- 处理 `updateQueue`
-  调用 `getDerivedStateFromProps`
- 调用 `souldComponentUpdate`

```
// Invokes the update life-cycles and returns false if it shouldn't rerender.
function updateClassInstance(current, workInProgress, ctor, newProps, renderExpirationTime,) {
  const instance = workInProgress.stateNode;
  cloneUpdateQueue(current, workInProgress);
  ...
  const getDerivedStateFromProps = ctor.getDerivedStateFromProps;

  const oldState = workInProgress.memoizedState;
  let newState = (instance.state = oldState);
  // 处理 updateQueue
  processUpdateQueue(workInProgress, newProps, instance, renderExpirationTime);
  newState = workInProgress.memoizedState;

  // …

  // 调用生命周期 getDerivedStateFromProps
  if (typeof getDerivedStateFromProps === 'function') {
    applyDerivedStateFromProps(workInProgress, ctor, getDerivedStateFromProps, newProps);
    newState = workInProgress.memoizedState;
  }

  // 调用 shouldComponentUpdate
  const shouldUpdate = checkHasForceUpdateAfterProcessing()
      || checkShouldComponentUpdate(…);

  if (shouldUpdate) {
    if (typeof instance.componentDidUpdate === 'function') {
      workInProgress.effectTag |= Update;
    }
    if (typeof instance.getSnapshotBeforeUpdate === 'function') {
      workInProgress.effectTag |= Snapshot;
    }
  } else {
    ...
  }

  // Update the existing instance's state, props, and context pointers even
  // if shouldComponentUpdate returns false.
  instance.props = newProps;
  instance.state = newState;
  instance.context = nextContext;

  return shouldUpdate;
}
```

`processUpdateQueue` 主要就是遍历 `updateQueue` 依次对 state 进行更新，从而得到最新的 state。

```
export function processUpdateQueue<State>(…): void {
  // This is always non-null on a ClassComponent or HostRoot
  const queue: UpdateQueue<State> = (workInProgress.updateQueue: any);

  let firstBaseUpdate = queue.firstBaseUpdate;

  // These values may change as we process the queue.
  if (firstBaseUpdate !== null) {
    // Iterate through the list of updates to compute the result.
    let newState = queue.baseState;

    // ...

    let update = firstBaseUpdate;
    // 遍历 queue 对没一个 update 做处理
    do {
      // Process this update.
      newState = getStateFromUpdate(
        workInProgress,
        queue,
        update,
        newState,
        props,
        instance,
      );

      const callback = update.callback;
      if (callback !== null) {
        workInProgress.effectTag |= Callback;
        const effects = queue.effects;
        if (effects === null) {
          queue.effects = [update];
        } else {
          effects.push(update);
        }
      }

      update = update.next;

      // ...

    } while (true);

    if (newLastBaseUpdate === null) {
      newBaseState = newState;
    }

    queue.baseState = ((newBaseState: any): State);

    // ...

    workInProgress.memoizedState = newState;
  }
}
```

经历以上的 `updateClassInstance` 过程后，我们就得到了该组件的最新 state，现在需要重新执行组件的 render 函数来获得新的虚拟 dom、执行 diff 算法、然后给 fiber 打上更新标记。这正是 `finishClassComponent` 要做的事。

### finishClassComponent

```
function finishClassComponent(current, workInProgress, …) {
  // ...
  const instance = workInProgress.stateNode;

  // Rerender
  ReactCurrentOwner.current = workInProgress;
  let nextChildren;
  // ...
  // 调用 render
  /**
    render() {
     return (
      <button onClick={this.handleClick}>点击次数({this.state.count})</button>
     );
   }
   */
  nextChildren = instance.render();
  // ...
  // React DevTools reads this flag.
  workInProgress.effectTag |= PerformedWork;

  // 调用 render, 协调孩子, 即所谓的 diff 算法
  reconcileChildren(
    current,
    workInProgress,
    nextChildren,
    renderExpirationTime,
  );
  // ...
  return workInProgress.child;
}
```

nextChildren 显然是 render 渲染出来的虚拟 dom，他是一个对象：

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/34506eb4-44a4-11bc-a6c7-be7f6957dd18.png)

下面即将进入 diff 算法，即用代表页面当前的内存结构和 代表页面将变成的内存结构 对对比，并找出差异。
- 现在：current fiber
- 未来：render 的这个返回，即虚拟 dom 对象
我们现在看当前这个简单 singleElement 场景。

> 可以看出 react diff 算法是 fiber 链表和 vdom (对象、数组) 的对比，这和 vue 就很不相同，vue 的 diff 算法就是 vdom 和 vdom 之间的对比。那么，想一想哪一个更高效呢？

### reconcileChildren - singleElement

```
export function reconcileChildren(current, workInProgress,
  nextChildren, renderExpirationTime) {
  // 此时 current 肯定不是空, 为空是首次渲染该组件的场景
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
      current.child, // 你能说出这个变量是上面内存示意图里的哪一个么？
      nextChildren,
      renderExpirationTime,
    );
  }
}
```

`nextChildren` 是一个对象，而非数组，这意味着我们的目标是渲染一个单元素，所以这次会走进 `reconcileSingleElement` 的单元素协调逻辑。

```
function reconcileChildFibers(returnFiber, currentFirstChild, newChild, expirationTime,) {
  // ...
  // 现在 newChild 是对象类型
  const isObject = typeof newChild === 'object' && newChild !== null;

  if (isObject) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE:
        return placeSingleChild(
          reconcileSingleElement(
            returnFiber,
            currentFirstChild,
            newChild,
            expirationTime,
          ),
        );
    }
  }
  // ...
}
```

想一想，给你一个单链表，而你的目标是渲染一个元素，用最小的代价创建出新的 fiber。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/7eea1903-a0c2-bc43-cb69-9838dd26e590.png)

```
function reconcileSingleElement(returnFiber, currentFirstChild, element, expirationTime) {
  const key = element.key;
  let child = currentFirstChild;

  while (child !== null) {
    // 如果 key 和 child.key 都是 null，那么只有第一个元素会被遍历到
    if (child.key === key) { // key 是目标的, 进入这个分支说明找到了 key 相同的。
      switch (child.tag) {
        // ...
        default: {
          if (child.elementType === element.type) { // 如果 type 也相同
            deleteRemainingChildren(returnFiber, child.sibling);
            const existing = useFiber(child, element.props);
            existing.ref = coerceRef(returnFiber, child, element);
            existing.return = returnFiber;
            return existing;
          }
          break;
        }
      }
      // 找到 key 相同的，但是 type 不一样，那么全删了
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // 如果不是，删了
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }

  // ...
  // 如果没找到，创建一个，并返回
  const created = createFiberFromElement(
    element,
    returnFiber.mode,
    expirationTime,
  );
  created.ref = coerceRef(returnFiber, currentFirstChild, element);
  created.return = returnFiber;
  return created;
}
```

遍历 fiber 链，尝试找到 key 相同的。如果找到了，还要看 type 是否相同
- 如果 type 也相同，那么即复用此 fiber, 删除剩余的
- 如果 type 不相同，那么则删除所有的 fiber

我们来看些例子：
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/e1ea6fdb-b337-04cb-1da9-eb4e05a5ba00.png)

回到我们的例子里，fiber 链只有 1 个元素，目标也是一个元素，key 都是 undefined,  type 都是 button，那自然就复用了。 所以这一步后，fiber 树是这样的：

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/c9094ff0-283f-5526-b876-e97cf53e6960.png)

下面继续对 button fiber 做 beginWork

## beginWork for button fiber
button 是 HostFiber 的类型，所以 beginWork 会走进 updateHostComponent 的逻辑。接入还是走进了 reconcileChildren 的 diff 逻辑。
> react 里的 diff 算法，叫 reconcile，所以也叫协调算法。

```
function updateHostComponent(current, workInProgress, renderExpirationTime) {
  // ...
  const nextProps = workInProgress.pendingProps;

  let nextChildren = nextProps.children;

  reconcileChildren(
    current,
    workInProgress,
    nextChildren,
    renderExpirationTime,
  );
  return workInProgress.child;
}
```

```
function reconcileChildFibers(returnFiber, currentFirstChild, newChild, expirationTime,) {
  // ...
  // 现在 newChild 这次是数组了
  if (isArray(newChild)) {
    return reconcileChildrenArray(
      returnFiber,
      currentFirstChild,
      newChild,
      expirationTime,
    );
  }
  // ...
}
```

`reconcileChildrenArray` 就是 react diff 的核心部分了，即设计链表和数组之间的 diff。

### reconcileChildren - Array - reconcileChildren

我们面临的问题：

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/700fdc21-c4d1-0260-ed1f-4c78df3e93b4.png)

> 此例的场景十分简单，实际情况会有插入，移动，甚至反转等复杂场景。


此算法主要由 3 个循环组成：
* 第 1 个循环，遍历 newChildren 和 fiber 链，对 key 相同的 fiber 做 updateSlot 操作。

```
for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
  nextOldFiber = oldFiber.sibling;

  // 如果 Key match 则对 old fiber 做更新，否则返回 null
  const newFiber = updateSlot(
    returnFiber,
    oldFiber,
    newChildren[newIdx],
    expirationTime,
  );

  // 一旦出现 key 不 match 的 bad case，则跳出循环。
  if (newFiber === null) {
    break;
  }

  if (oldFiber && newFiber.alternate === null) {
    // We matched the slot, but we didn't reuse the existing fiber, so we
    // need to delete the existing child.
    // key 相同但是 type 不同的场景，即 updateSlot 里的 createFiber 分支
    deleteChild(returnFiber, oldFiber);
  }

  lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
  // 如果找到了，则将拼在 resultingFirstChild 这个链上。
  if (previousNewFiber === null) {
    resultingFirstChild = newFiber;
  } else {
    previousNewFiber.sibling = newFiber;
  }
  previousNewFiber = newFiber;
  oldFiber = nextOldFiber;
}
```

- updateSlot 的逻辑：
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/a7184286-20c5-cbfc-47f8-cf80d28ae76c.png)


- placeChild 的逻辑

```
 // 只有在原位置找到元素的时候才会更新 index
 function placeChild(
    newFiber,
    lastPlacedIndex,
    newIndex,
  ) {
    newFiber.index = newIndex;

    const current = newFiber.alternate;
    if (current !== null) {
      const oldIndex = current.index;
      if (oldIndex < lastPlacedIndex) {
        // This is a move.
        newFiber.effectTag = Placement;
        return lastPlacedIndex;
      } else {
        // This item can stay in place.
        return oldIndex;
      }
    } else {
      // This is an insertion.
      newFiber.effectTag = Placement;
      return lastPlacedIndex;
    }
  }
```
我们来看个例子：
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/5eee5c9a-763f-5000-69e9-50d7099251e4.png)


第 1 个循环后，如果新的孩子列表已经遍历完了，那么清空老的 fiber 链表。
```
if (newIdx === newChildren.length) {
  // We've reached the end of the new children. We can delete the rest.
  deleteRemainingChildren(returnFiber, oldFiber);
  return resultingFirstChild;
}
```

* 如果新孩子列表 newChildren 没有遍历完，老孩子链表 oldFiber 走完了，那么进入第 2 个循环，即遍历剩余的新孩子，依次创建即可。
```
if (oldFiber === null) {
  // If we don't have any more existing children we can choose a fast path
  // since the rest will all be insertions.
  for (; newIdx < newChildren.length; newIdx++) {
    const newFiber = createChild(
      returnFiber,
      newChildren[newIdx],
      expirationTime,
    );
    if (newFiber === null) {
      continue;
    }
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
  }
  return resultingFirstChild;
}
```
![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/2d016931-0347-4c6e-8a66-ad462662b9c3.png)


* 如果新孩子列表 newChildren 没有遍历完，老孩子 oldFiber 也没完，那么进入第三个循环。
  - 把老孩子放在一个 map 里用 fiber 的 key 或者 index 作为 map 的 key, fiber 对象作为 value
  - 继续遍历 newChildren ，在 map 里找 fiber，找到了就复用，找不到就新建。
  - 最后如果还有剩余的老孩子，则全部标记为删除。

```
// 构建老 fiber 的 map
const existingChildren = mapRemainingChildren(returnFiber, oldFiber);

// 扫描剩余的新孩子，在 map 里查找
for (; newIdx < newChildren.length; newIdx++) {
  const newFiber = updateFromMap(
    existingChildren,
    returnFiber,
    newIdx,
    newChildren[newIdx],
    expirationTime,
  );

  if (newFiber !== null) {
    if (shouldTrackSideEffects) {
      if (newFiber.alternate !== null) {
        // 我们复用了这个 fiber，不会销毁它，所以从 map 中删除。
        existingChildren.delete(
          newFiber.key === null ? newIdx : newFiber.key,
        );
      }
    }
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
  }
}

if (shouldTrackSideEffects) {
  // 还有剩余的老 fiber，全标记为删除，后续会销毁。
  existingChildren.forEach(child => deleteChild(returnFiber, child));
}
```

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/2aff7a76-83a2-3fb4-ea66-db0146b079c5.png)

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/3aa77192-ab91-a159-aa32-e792dda7f670.png)



可见对于交换的场景，react 的效率并不是很高，尤其当列表比较长的时候，移动的元素会原来越来越多。vue 的双指针优化，在这种场景下更加优秀。

最后我们看一下这个算法作者的注释：
> This algorithm can't optimize by searching from both ends since we don't have backpointers on fibers. I'm trying to see how far we can get with that model. If it ends up not being worth the tradeoffs, we can add it later.
Even with a two ended optimization, we'd want to optimize for the case where there are few changes and brute force the comparison instead of going for the Map. It'd like to explore hitting that path first in forward-only mode and only go for the Map once we notice that we need lots of look ahead. This doesn't handle reversal as well as two ended search but that's unusual. Besides, for the two ended optimization to
work on Iterables, we'd need to copy the whole set. In this first iteration, we'll just live with hitting the bad case (adding everything to a Map) in for every insert/move.

我大概翻译了一下：

> 因为 fiber 结构不是双向链表，所以我们无法对此算法进行双向搜索优化（Vue 的方案）。我正在调查双向搜索优化能达到的高度，如果有值得的话，我们可以以后再优化。
~~双端优化可以让我们优化以下这样的场景：只有一点点变化的时候，不进行 Map 查找了，直接进行暴力对比。~~本算法采用了仅向后查找的模式，一旦发现需要回头找的时候，则进入 Map 查找的方案。本算法和双指针优化一样没有处理列表反转的场景，但还好它并不常见。~~除了这些以外，如果采用双向搜索优化，在处理 Iterables 的时候，我们还需要将整个 Iterable 复制一份。~~这是算法的第一版，我们遇到 bad case（插入/移动）就会转向 Map 查找的方案。

> 划掉的地方是我硬翻的，我太理解他的意思。希望有高手给解释解释。

anyway，回到例子，经过 reconcileChildren, button 子元素的 fiber 会被建立好。

![](https://raw.githubusercontent.com/njleonzhang/image-bed/master/assets/75bbfd53-c8eb-83b2-34ce-77a8c0b4cc62.png)

再往后，就和新建的场景类似啦。completeWork 建立 effectList，在进入commit 阶段，根据 effectList 对 dom 进行更新。

# 参考文档
https://github.com/7kms/react-illustration-series
