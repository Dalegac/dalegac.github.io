---
title: 发布-订阅模式
author: Dale
date: 2021-09-15 22:45:00 +0800
categories: [Blogging, 设计模式]
tags: [设计模式]
math: true
mermaid: true
---

发布—订阅模式可以广泛应用于异步编程中，这是一种替代传递回调函数的方案。
比如，我们可以订阅 ajax 请求的 error、success 等事件。或者如果想在动画的每一帧完成之后做一些事情，那我们可以订阅一个事件，然后在动画的每一帧完成之后发布这个事件。在异步编程中使用发布—订阅模式，我们就无需过多关注对象在异步运行期间的内部状态，而只需要订阅感兴趣的事件发生点。

发布—订阅模式可以取代对象之间硬编码的通知机制，一个对象不用再显式地调用另外一个对象的某个接口。发布—订阅模式让两个对象松耦合地联系在一起，虽然不太清楚彼此的细节，但这不影响它们之间相互通信。当有新的订阅者出现时，发布者的代码不需要任何修改；同样发布者需要改变时，也不会影响到之前的订阅者。只要之前约定的事件名没有变化，就可以自由地改变它们。

## 1. 发布-订阅模式的例子

### 1.1 EventTarget 接口

DOM 节点的事件操作（监听和触发），都定义在`EventTarget`接口。在 DOM 上绑定事件函数，使用的就是发布订阅模式。

- `addEventListener()`：绑定事件的监听函数
- `removeEventListener()`：移除事件的监听函数
- `dispatchEvent()`：触发事件

### 1.2 Node 的 EventEmitter

#### 1.2.1 同步/异步

EventEmitter 按照注册的顺序同步地调用所有监听器。 这确保了事件的正确排序，并有助于避免竞争条件和逻辑错误。 在适当的时候，监听器函数可以使用 setImmediate() 或 process.nextTick() 方法切换到异步的操作模式：

## 2. 实现

链式调用，最大监听数，未注册提示

```js
class EventEmitter {
  constructor(options = {}) {
    this.events = {};
    this.maxListeners = options.maxListeners || Infinity;
  }
  /**
   * 发布事件
   * @param {String} event 事件类型
   * @param  {...any} args 参数列表，把emit传递的参数赋给回调函数
   */
  emit = (event, ...args) => {
    const cbs = this.events[event];

    if (!cbs) {
      console.warn(`${event} is not registered`);
    }

    cbs.forEach((cb) => cb.apply(this, args));
    return this;
  };
  /**
   * 注册事件监听者
   * @param {String} event 事件类型
   * @param {Function} cb 回调函数
   */
  on = (event, cb) => {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    if (
      this.maxListeners !== Infinity &&
      this.events[event].length >= this.maxListeners
    ) {
      console.log(`${event} has reached max listeners`);
      return this;
    }
    this.events[event].push(cb);

    return this;
  };

  /**
   * 注册事件监听者，仅执行依次回调
   * @param {String} event 事件类型
   * @param {Function} cb 回调函数
   */
  once = (event, cb) => {
    // 用完进行销毁
    const func = (...args) => {
      this.off(event, func);
      cb.apply(this, args);
    };

    this.on(event, func);
    return this;
  };
  /**
   * 移除某个事件的一个监听者
   * @param {String} event 事件类型
   * @param {Function} cb 回调函数
   */
  off = (event, cb) => {
    if (!cb) {
      this.events[event] = null;
    } else {
      this.events[event] = this.events[event].filter((item) => cb !== item);
    }

    return this;
  };
}
```

## 3. 进阶问题

### 3.1 先发布后订阅

### 3.2 Event 对象创建命名空间

### 3.3 触发优先级

## 4. 参考

- 《JavaScript 设计模式与开发实践》---曾探
- [《深入设计模式》](https://refactoringguru.cn/design-patterns/observer)
- [《JavaScript 教程》EventTarget 接口](https://wangdoc.com/javascript/events/eventtarget.html)
- [Node 的 EventEmitter 实现](https://github.com/nodejs/node/blob/v16.9.1/lib/events.js)
