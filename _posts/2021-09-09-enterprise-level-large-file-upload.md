---
title: 企业级大文件上传
author: Dale
date: 2021-10-09 7:21:00 +0800
categories: [Blogging, 文件服务]
tags: [Vue, 文件上传]
math: true
mermaid: true
---

# 大文件上传

一个企业级上传组件,支持的功能：

- [x] axios.post
- [x] 体验优化，粘贴，拖拽，进度条
- [x] 断点续传+秒传，类型判断
- [x] web-worker，时间切片，抽样 hash（MD5）
- [x] 异步任务并发数，切片报错重试
- [ ] 慢启动，碎片清理，其他优化（完成部分）

[GitHub 地址](https://github.com/Dalegac/file-upload.git)

## 1. 预备知识

### 1.1 专用工作者线程（web-worker）

> 专用工作者线程是最简单的 Web 工作者线程，网页中的脚本可以创建专用工作者线程来执行在页面线程之外的其他任务。这样的线程可以与父页面交换信息、发送网络请求、执行文件输入/输出、进行密集计算、处理大量数据，以及实现其他不适合在页面执行线程里做的任务（否则会导致页面响应迟钝）。

### 1.2 慢启动（slow-start）

> 在慢启动状态，cwnd 的值以一个 MSS 开始并且每当传输的报文段首次被确认就增加 1 个 MSS，这一过程没过一个 RTT，发送速率就翻番。因此，TCP 发送速率起始慢，但是在慢启动阶段以指数增长。

![TCP慢启动](https://github.com/Dalegac/static-pic/raw/master/images/TCP%E6%85%A2%E5%90%AF%E5%8A%A8.png)

## 2. 工程目录

```
// 客户端
client （基于vue）
├── public
│   ├── favicon.ico
│   ├── hash.js                             // web-worker
│   ├── index.html
│   └── spark-md5.min.js                    // MD5算法脚本
├── src
│   ├── App.vue                             // 上传组件
│   ├── assets
│   ├── main.js                             // vue工程入口
│   └── plugins
│       └── element.js                      // 引入elementUI
├── babel.config.js
├── package.json
├── vue.config.js
└── README.md


// 服务端（基于egg）
server
├── app
│   ├── controller
│   │   └── home.js                         // 接收文件方法
│   ├── public                              // 上传文件、切片存储文件夹
│   ├── router.js                           // 路由接口映射
│   └── service
│       └── upload.js                       // 文件处理方法
├── config
│   └── config.default.js                   // 配置上传协议和目录
└── package.json
```

## 3. 功能

### 3.1 文件校验

![文件校验](https://github.com/Dalegac/static-pic/blob/master/images/文件校验.png?raw=true)

### 3.2 上传文件

![上传文件](https://github.com/Dalegac/static-pic/blob/master/images/上传文件.png?raw=true)

### 3.3 接收文件

#### 3.3.1 /check

![check](https://github.com/Dalegac/static-pic/blob/master/images/接收文件check.png?raw=true)

#### 3.3.2 /upload

![upload](https://github.com/Dalegac/static-pic/blob/master/images/接收文件upload.png?raw=true)

#### 3.3.1 /merge

![merge](https://github.com/Dalegac/static-pic/blob/master/images/接收文件merge.png?raw=true)

## 4. 优化

### 4.1 文件 hash 值的计算

![优化hash值计算](https://github.com/Dalegac/static-pic/blob/master/images/优化文件hash值计算.png?raw=true)

存在问题：文件较大时，直接计算 hash 值，时间过长，导致页面阻塞卡顿

有一些解决方案。

#### 4.1.1 web-worker

在页面线程之外，开辟专用工作者线程，用于 hash 值的计算。计算就不会阻塞当前页面

```
// web-worker
self.importScripts("spark-md5.min.js");

self.onmessage = e => {
  // 接受主线程的通知
  const { chunks } = e.data;
  const spark = new self.SparkMD5.ArrayBuffer();
  let progress = 0;
  let count = 0;

  const loadNext = index => {
    const reader = new FileReader();
    reader.readAsArrayBuffer(chunks[index].file);
    reader.onload = e => {
      // 累加器 不能依赖index，
      count++;
      // 增量计算md5
      spark.append(e.target.result);
      if (count === chunks.length) {
        // 通知主线程，计算结束
        self.postMessage({
          progress: 100,
          hash: spark.end()
        });
      } else {
        // 每个区块计算结束，通知进度即可
        progress += 100 / chunks.length;
        self.postMessage({
          progress
        });
        // 计算下一个
        loadNext(count);
      }
    };
  };
  // 启动
  loadNext(0);
};
```

#### 4.1.2 requestIdleCallback

[window.requestIdleCallback()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)方法将在浏览器的空闲时段内调用的函数排队。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。相当于在浏览器每帧的闲暇时间进行 hash 值的计算，此方法存在兼容性问题，requestIdleCallback 方法本身也存在问题，参考 react 团队因此自己实现了一个。

```
async calculateHashIdle(chunks) {
      return new Promise(resolve => {
        const spark = new sparkMd5.ArrayBuffer();
        let count = 0;
        const appendToSpark = async file => {
          return new Promise(resolve => {
            const reader = new FileReader();
            reader.readAsArrayBuffer(file);
            reader.onload = e => {
              spark.append(e.target.result);
              resolve();
            };
          });
        };
        const workLoop = async deadline => {
          // 有任务，并且当前帧还没结束
          while (count < chunks.length && deadline.timeRemaining() > 1) {
            await appendToSpark(chunks[count].file);
            count++;
            // 没有了 计算完毕
            if (count < chunks.length) {
              // 计算中
              this.hashProgress = Number(
                ((100 * count) / chunks.length).toFixed(2)
              );
              // console.log(this.hashProgress)
            } else {
              // 计算完毕
              this.hashProgress = 100;
              resolve(spark.end());
            }
          }
          console.log(`浏览器有任务拉，开始计算${count}个，等待下次浏览器空闲`);

          window.requestIdleCallback(workLoop);
        };
        window.requestIdleCallback(workLoop);
      });
    },
```

#### 4.1.3 抽样 hash

牺牲一定的准确率换来效率，hash 一样的不一定是同一个文件， 但是不一样的一定不是。所以可以考虑用来预判

```
async calculateHashSample() {
      return new Promise(resolve => {
        const spark = new sparkMd5.ArrayBuffer();
        const reader = new FileReader();
        const file = this.file;
        // 文件大小
        const size = this.file.size;
        let offset = 2 * 1024 * 1024;

        let chunks = [file.slice(0, offset)];

        // 前面100K

        let cur = offset;
        while (cur < size) {
          // 最后一块全部加进来
          if (cur + offset >= size) {
            chunks.push(file.slice(cur, cur + offset));
          } else {
            // 中间的 前中后去两个字节
            const mid = cur + offset / 2;
            const end = cur + offset;
            chunks.push(file.slice(cur, cur + 2));
            chunks.push(file.slice(mid, mid + 2));
            chunks.push(file.slice(end - 2, end));
          }
          // 前取两个子杰
          cur += offset;
        }
        // 拼接
        reader.readAsArrayBuffer(new Blob(chunks));

        // 最后100K
        reader.onload = e => {
          spark.append(e.target.result);
          this.hashProgress = 100;
          resolve(spark.end());
        };
      });
    },
```

### 4.2 并行上传任务数限制

限制开启的文件上传任务数，减少服务器压力

```
const limit = 4 // 设置并行请求的个数
while (limit > 0) {
  setTimeout(() => {
    // 模拟延迟,后续可以删除
    start();   // 上传请求方法
  }, Math.random() * 2000);

  limit -= 1;
}
```

### 4.3 上传错误重试

当前切片在上传中出错了，尝试重试机制，重新 push 到待上传队列中。设置重试次数，错误次数超过限制则报错。

### 4.4 进度条优化

进度条的展示能够使得用户直观感知上传的速度和进度。要是要计算单次上传进度的话，可以采用[XMLHttpRequest.upload](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/upload)监听上传进度。

## 5. 参考文献：

- Professional JavaScript for Web Developers,4th Edition 第 27 章工作者线程
- [《Web Worker 使用教程》](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)，阮一峰
- [MDN Web Docs](https://developer.mozilla.org/zh-CN/)
- 掘金：[字节跳动，我也实现了大文件上传和断点续传](https://juejin.cn/post/6844904055819468808)
