---
title: 深拷贝
author: Dale
date: 2021-10-05 09:52:00 +0800
categories: [Blogging, JavaScript]
tags: [JavaScript, 深拷贝]
math: true
mermaid: true
---

## 1. 需要考虑的问题

- 基本实现
  - 递归能力
- 循环引用
  - 考虑问题的全面性
  - 理解 weakmap 的真正意义
- 多种类型
  - 考虑问题的严谨性
  - 创建各种引用类型的方法，JS API 的熟练程度
  - 准确的判断数据类型，对数据类型的理解程度
- 通用遍历：
  - 写代码可以考虑性能优化
  - 了解集中遍历的效率
  - 代码抽象能力
- 拷贝函数：
  - 箭头函数和普通函数的区别
  - 正则表达式熟练程度

## 2. 实现

```js
/*
 * @Author: Dalegac
 * @Date: 2021-09-30 14:28:05
 * @LastEditTime: 2021-10-04 10:11:43
 * @LastEditors: Dalegac
 * @Description: Just say something
 */
// 可继续遍历的数据类型
const mapTag = "[object Map]";
const setTag = "[object Set]";
const arrayTag = "[object Array]";
const objectTag = "[object Object]";
const argsTag = "[object Arguments]";

// 不可继续遍历的数据类型
const boolTag = "[object Boolean]";
const dateTag = "[object Date]";
const numberTag = "[object Number]";
const stringTag = "[object String]";
const symbolTag = "[object Symbol]";
const bigintTag = "[object BigInt]";
const errorTag = "[object Error]";
const regexpTag = "[object RegExp]";
const funcTag = "[object Function]";

const deepTag = [mapTag, setTag, arrayTag, objectTag, argsTag];

/**
 * @description: 工具函数 - 通用while循环
 * @param {Array}array 待遍历数组
 * @param {Function}iteratee 执行函数
 * @return {Array}
 */
function forEach(array, iteratee) {
  let index = -1;
  const length = array.length;
  while (++index < length) {
    iteratee(array[index], index);
  }
  return array;
}
/**
 * @description: 工具函数 - 判断是否为引用类型
 * @param {Any} target
 * @return {Boolean} 是否是引用类型
 */
function isObject(target) {
  const type = typeof target;
  return target !== null && (type === "object" || type === "function");
}
/**
 * @description: 工具函数 - 获取实际类型
 * @param {Any} target
 * @return {String} 格式化类型
 */
function getType(target) {
  return Object.prototype.toString.call(target);
}

/**
 * @description:工具函数 - 初始化被克隆函数
 * @param {Any} target
 * @return {*}
 */
function getInit(target) {
  const Ctor = target.constructor;
  return new Ctor();
}
/**
 * @description: 工具函数 - 克隆Symbol
 * @param {Symbol}
 * @return {*}
 */
function cloneSymbol(target) {
  return Object(Symbol.prototype.valueOf.call(target));
}

/**
 * @description: 工具函数 - 克隆正则
 * @param {Regexp} target
 * @return {*}
 */
function cloneReg(target) {
  const { source, flags, lastIndex } = target;
  const result = new RegExp(source, flags);
  result.lastIndex = lastIndex;
  return result;
}

/**
 * @description: 工具函数 - 克隆函数
 * @param {Function} target
 * @return {*}
 */
function cloneFunction(target) {
  const bodyReg = /(?<={).+(?=})/ms;
  const paramReg = /(?<=\().+(?=\)\s+{)/;
  const funcString = target.toString();
  if (target.prototype) {
    const param = paramReg.exec(funcString);
    const body = bodyReg.exec(funcString);
    if (body) {
      if (param) {
        const paramArr = param[0].split(",");
        return new Function(...paramArr, body[0]);
      } else {
        return new Function(body[0]);
      }
    } else {
      return null;
    }
  } else {
    return eval(funcString);
  }
}

/**
 * @description: 工具函数 - 克隆不可遍历类型
 * @param {Any} target 被克隆对象
 * @param {String} type 对象类型
 * @return {*}
 */
function cloneOtherType(target, type) {
  const Ctor = target.constructor;
  switch (type) {
    case boolTag:
    case numberTag:
    case stringTag:
    case bigintTag:
    case errorTag:
    case dateTag:
      return new Ctor(target);
    case regexpTag:
      return cloneReg(target);
    case symbolTag:
      return cloneSymbol(target);
    case funcTag:
      return cloneFunction(target);
    default:
      return null;
  }
}

/**
 * @description: 克隆函数
 * @param {Any} target 被克隆对象
 * @param {WeakMap} map 引用对象缓存池
 * @return {*}
 */
function clone(target, map = new WeakMap()) {
  // 克隆原始类型
  if (!isObject(target)) {
    return target;
  }
  // 初始化
  const type = getType(target);
  let cloneTarget;
  if (deepTag.includes(type)) {
    cloneTarget = getInit(target);
  } else {
    return cloneOtherType(target, type);
  }

  // 防止循环引用
  if (map.get(target)) {
    return map.get(target);
  }
  map.set(target, cloneTarget);

  // 克隆set
  if (type === setTag) {
    target.forEach((value) => {
      cloneTarget.add(clone(value, map));
    });
    return cloneTarget;
  }

  // 克隆map
  if (type === mapTag) {
    target.forEach((value, key) => {
      cloneTarget.set(key, clone(value, map));
    });
    return cloneTarget;
  }

  // 克隆对象和数组
  const keys = type === arrayTag ? undefined : Object.keys(target);
  forEach(keys || target, (value, key) => {
    if (keys) {
      key = value;
    }
    cloneTarget[key] = clone(target[key], map);
  });
  return cloneTarget;
}
module.exports = { clone };
```

## 3. 参考

- [如何写出一个惊艳面试官的深拷贝](http://www.conardli.top/blog/article/JS%E8%BF%9B%E9%98%B6/%E5%A6%82%E4%BD%95%E5%86%99%E5%87%BA%E4%B8%80%E4%B8%AA%E6%83%8A%E8%89%B3%E9%9D%A2%E8%AF%95%E5%AE%98%E7%9A%84%E6%B7%B1%E6%8B%B7%E8%B4%9D.html#%E5%AF%BC%E8%AF%BB)
- [MDN BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
- [阮一峰博客 ES6 RegExp](https://es6.ruanyifeng.com/#docs/regex#RegExp-%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
