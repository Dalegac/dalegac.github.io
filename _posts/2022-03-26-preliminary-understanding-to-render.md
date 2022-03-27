---
title: 初识渲染器-创建篇
author: Dale
date: 2022-03-26 09:52:00 +0800
categories: [Blogging, JavaScript]
tags: [JavaScript, Vue, Vuejs设计与实现]
math: true
mermaid: true
---

## 1. 前置知识

### 1.1 UI 内容

- DOM 元素：例如是 div 标签还是 a 标签。
- 属性：如 a 标签的 href 属性，再如 id、class 等通用属性。
- 事件：如 click、keydown 等。
- 元素的层级结构：DOM 树的层级结构，既有子节点，又有父节点。

### 1.2 UI 的描述方式

- 模板描述

```html
<h1 @click="handler"><span></span></h1>
```

- JavaScript 对象描述

```js
const title = {
  // 标签名称
  tag: "h1",
  // 标签属性
  prop: {
    onClick: handler,
  },
  // 子节点
  children: [{ tag: "span" }],
};
```

相较于模板的描述方式，使用 JavaScript 对象描述 UI 更加灵活。

### 1.3 虚拟 DOM

**描述 UI 的 JavaScript 对象，就是的虚拟 DOM。**

## 2. 渲染器

渲染器的作用就是把虚拟 DOM 渲染为真实 DOM。

它的工作原理是递归地遍历虚拟 DOM 对象，并调用原生 DOM API 来完成真实 DOM 的创建。渲染器的精髓在于后续的更新，他会通过 Diff 算法找出变更点，并且只会更新需要更新的内容。

```js
// 虚拟DOM
const vnode = {
  tag: "div",
  props: {
    onClick: () => alert("hello"),
  },
  children: "click me",
};
```

```js
/**
 * @description: 渲染器
 * @param {Object}node 虚拟DOM对象
 * @param {Object}container 一个真实DOM元素，作为挂载点
 */
function renderer(vnode, container) {
  // 使用 vnode.tag 作为标签名称创建 DOM 元素
  const el = document.createElement(vnode.tag);
  // 遍历 vnode.pros，将属性、事件添加到 DOM 元素
  for(const key in vnode.props) {
    if(/^on/.test(key)){
      // 如果key以on开头，说明它是事件
      el.addEventListener(
        key.substr(2).toLowerCase(),// 事件名称onClick--->click
        vnode.props[key] // 事件处理函数
      )
    }
  }
  // 处理children
  if(typeof vnode.children === 'string') {
    // 如果children是字符串，说明它是元素的文本子节点
    el.appendChild(document.createTextNode(vnode.children))
  } else if(Array.isArray(vnode.children)){
    // 递归地调用render函数渲染子节点，使用当前元素el作为挂载点
    vnode.children.forEach(child=>renderer(child,el))
  }

  // 将元素添加到挂载点下
  container.appendChild(el)
}
```


实现思路：
- 创建元素：把vnode.tag作为标签名称来创建DOM元素。
- 为元素添加属性和事件：遍历vnode.props对象，如果key以on字符开头，说明它是一个事件，把字符on截取掉后再调用toLowerCase函数将事件名称小写化，最终得到合法的事件名称，例如onClick会变成click，最后调用addEventListener绑定事件处理函数。
- 处理children：如果children是一个数组，就递归地调用render继续渲染，注意，此时我们要把刚刚创建地元素作为挂载点（父节点）；如果children是字符串，则使用createTextNode函数创建一个文本节点，并将其添加到新创建的元素内。


## 3. 参考资料
- 《Vue.js 设计与实现》 --霍春阳
