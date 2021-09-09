<!--
 * @Author: Dalegac
 * @Date: 2021-09-07 13:58:34
 * @LastEditTime: 2021-09-08 12:22:53
 * @LastEditors: Dalegac
 * @Description: Fiber架构学习理解
-->

## 1. 设计理念

### React 哲学

> 我们认为，React 是用 JavaScript 构建**快速响应**的大型 Web 应用程序的首选方式。他在 Facebook 和 Instagram 上表现优秀。

#### 制约快速响应的因素

- 计算能力——CPU 瓶颈(creating nodes,re-rendering)
- 网络延迟——IO 瓶颈(data fetching,code splitting)

#### 计算能力限制的解决方案————Time Slicing

主流浏览器刷新频率为 60Hz，即每（1000ms / 60Hz）16.6ms 浏览器刷新一次。我们知道，JS 可以操作 DOM，GUI 渲染线程与 JS 线程是互斥的。所以 JS 脚本执行和浏览器布局、绘制不能同时执行。在每 16.6ms 时间内，需要完成如下工作：

```
JS脚本执行 ————>  样式布局 ————> 样式绘制
```

当 JS 执行时间过长，超出了 16.6ms，这次刷新就没有时间执行样式布局和样式绘制了。[demo](https://codesandbox.io/s/concurrent-3h48s?file=/src/index.js)

React 提出了解决方案：在浏览器每一帧的时间中，预留一些时间给 JS 线程，React 利用这部分时间更新组件(预留的初始时间是 [5ms](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/src/forks/SchedulerHostConfig.default.js#L119))。

当预留的时间不够用时，React 将线程控制权交还给浏览器使其有时间渲染 UI，React 则等待下一帧时间到来继续被中断的工作。

> 这种将长任务分拆到每一帧中，像蚂蚁搬家一样一次执行一小段任务的操作，被称为时间切片

Time Slicing 带来的**好处**：

- React 在渲染（render）的时候，不会阻塞现在的线程
- 如果你的设备足够快，你会感觉渲染是同步的
- 如果你设备非常慢，你会感觉还算是灵敏的
- 虽然是异步渲染，但是你将会看到完整的渲染，而不是一个组件一行行的渲染出来
- 同步书写组件的方式

解决 CPU 瓶颈的关键是实现时间切片，而时间切片的关键是：将同步的更新变为可中断的异步更新。

React 的 Concurrent Mode 启用了时间切片：此时我们的长任务被拆分到每一帧不同的 task 中，JS 脚本执行时间大体在 5ms 左右，这样浏览器就有剩余时间执行样式布局和样式绘制，减少掉帧的可能性。

## 2. Fiber

### 什么是 Fiber？

> This Fiber is just a plain JavaScript object and it has one to one relationship with an instance. It manages the work for an instance so it keeps track of which instance is for using the property state node. It also keeps track of its relationships to other fibers in the tree.
> [Lin Clark - A Cartoon Intro to Fiber - React Conf 2017](https://www.youtube.com/watch?v=ZCuYPiUIONs)

**Fiber 的含义**：作为静态的数据结构来说，每个 Fiber 节点对应一个 React element，保存了该组件的类型（函数组件/类组件/原生组件...）、对应的 DOM 节点等信息。
作为动态的工作单元来说，每个 Fiber 节点保存了本次更新中该组件改变的状态、要执行的工作（需要被删除/被插入页面中/被更新...）

### Fiber 的数据结构

源文件[ReactFiber.new.js](https://github1s.com/facebook/react/blob/HEAD/packages/react-reconciler/src/ReactFiber.new.js)

```js

function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  // 作为静态数据结构的属性
  this.tag = tag; // 对应的组件类型FunctionComponent,ClassComponent,HooksComponent(DOM结点对应的Fiber结点)
  this.key = key; // 我们常用的元素key
  this.elementType = null; // 大部分情况下elementType与type相同，在某些情况下，比如FunctionComponent使用React.Memo包裹时，两者不同
  this.type = null; // 对于FunctionComponent是函数本身，对于ClassComponent是class，对于HooksComponent是DOM结点的TagName
  this.stateNode = null; // 对于HooksComponent是是它真实对应的DOM结点

  // Fiber
  // 用于连接其他Fiber节点形成Fiber树
  this.return = null;
  this.child = null;
  this.sibling = null;
  // 对于多个同级Fiber节点，代表他们插入DOM的位置索引
  this.index = 0;
  // 就是我们常用的ref属性
  this.ref = null;

  // 从这里往下，都是作为动态的工作单元的属性
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  // Effects
  // 名称中带有Effect代表副作用相关，对于HostComponent副作用包括DOM节点的增删查改，
  // 对于FunctionComponent副作用代表了我们使用useEffect或者useLayoutEffect这两个hook
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber
  this.alternate = null;
  ......
}
```

### 调度器的优先级

源文件[SchedulerPriorities.js](https://github1s.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/src/SchedulerPriorities.js)

```js
/**
 * Copyright (c) Facebook, Inc. and its affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

export type PriorityLevel = 0 | 1 | 2 | 3 | 4 | 5;

// TODO: Use symbols?
// 初始化时的无优先级
export const NoPriority = 0;

// 立刻执行的优先级，也就是同步的优先级，react中最高优的优先级
export const ImmediatePriority = 1;

// 由用户触发的更新，比如在点击事件回调onClick触发了this.setState,
//那么这个this.setState创建的Update就是此优先级
export const UserBlockingPriority = 2;

// 比如在应用中请求服务的数据，数据返回时更新状态，这里的更新就是此优先级
export const NormalPriority = 3;

// Suspense就是
export const LowPriority = 4;

// 空闲的优先级
export const IdlePriority = 5;
```

## 3. Fiber 架构的工作原理

### 工作阶段

Lin Clark 举了一个例子
React 的工作流程分为两个阶段：

- render 阶段(可中断的)
- commit 阶段(不可中断)
  ![Fiber工作流程](https://gitee.com/Dalegac/static-pic/raw/master/images/Fiber%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.jpg)

render 阶段，从 workInProgress Tree 的根出发，深度优先遍历树，组件会经过 beginWork 和 completeWork 阶段，最后 react 将 workInProgress Tree 作为 pendingCommit，第一阶段到此结束。至此我们更新了 workInProgress Tree，并且得出了树中需要更新的结点列表 effectList。
![effectList](https://gitee.com/Dalegac/static-pic/raw/master/images/effectList.jpg)
![workInProgress Tree](https://gitee.com/Dalegac/static-pic/raw/master/images/pendingcommit.jpg)

commit 阶段，react 会依次遍历 effectList，将这次的更新 commit 给 DOM。

![遍历effectList](https://gitee.com/Dalegac/static-pic/raw/master/images/%E9%81%8D%E5%8E%86effectlist.jpg)

### Double Buffering

在构建 workInProgress Fiber 树时会尝试复用 current Fiber 树中已有的 Fiber 节点内的属性。类似于 canavas 的双缓存，这样能节省内存分配和垃圾回收的时间。
![双缓存机制](https://gitee.com/Dalegac/static-pic/raw/master/images/doublebuffering.png)

## 4. 参考

- [Beyond React 16 by Dan Abramov - JSConf Iceland](https://www.youtube.com/watch?v=v6iR3Zk4oDY&list=PL3j9y0zHLR-nWZ66DsnexvS0TNhtiqElY&index=1&t=58s)

- [Lin Clark - A Cartoon Intro to Fiber - React Conf 2017](https://www.youtube.com/watch?v=ZCuYPiUIONs&list=PL3j9y0zHLR-nWZ66DsnexvS0TNhtiqElY&index=2&t=576s)

<!--[React Fiber Architecture](https://github.com/acdlite/react-fiber-architecture)-->
