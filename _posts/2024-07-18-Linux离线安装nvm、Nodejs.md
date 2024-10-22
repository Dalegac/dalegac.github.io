---
title: Linux离线安装nvm Nodejs
author: Dale
date: 2024-07-18 7:21:00 +0800
categories: [Blogging, 环境安装]
tags: [Linux, nvm, Nodejs]
math: true
mermaid: true
---

# Linux 前端CICD环境搭建

## 1. 安装nvm

> 准备事项
下载安装包[nvm-0.39.7.tar.gz](https://github.com/nvm-sh/nvm/tags)

```bash
# step1 /usr/local目录下，新建.nvm文件夹，用于存放nvm
cd /usr/local
mkdir -p .nvm

# step2 解压nvm安装包进.nvm目录
tar -zxvf nvm-0.39.7.tar.gz -C .nvm

# step3 在/etc/profile文件末尾，配置nvm的环境变量
# 如果需要系统所有用户都使用你，建议放在/usr/local目录下
vim /etc/profile

export NVM_DIR="/usr/local/.nvm/nvm-0.39.7"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

# step4 使配置生效，并验证nvm安装成功
source /etc/profile
nvm -v   # 返回0.39.7
```

## 2. 安装Nodejs

> 准备事项[Nodejs下载地址](https://nodejs.org/download/release/)
下载安装包node-v16.20.2-linux-x64.tar.gz

```bash
# 在 nvm 安装目录下新建一个文件夹，用于存放 node
cd /usr/local/.nvm/nvm-0.39.7/versions
mkdir -p node

# 解压 node 安装包
tar -zxvf node-v16.20.2-linux-x64.tar.gz -C node

# 重命名
cd node
mv node-v16.20.2-linux-x64 v16.20.2

```

## 3. 安装 node-sass

> 准备事项[node-sass下载地址](https://github.com/sass/node-sass/releases)
下载`linux-x64-72_binding.node`
window为win32-x64-72_binding.node,具体需根据使用的node-sass版本及操作系统确定

```bash
# 添加环境变量
vim /etc/profile

# 末尾追加
export SASS_BINARY_PATH=/usr/local/.nvm/nvm-0.39.7/lib/linux-x64-72_binding.node

```

## 4. 其他问题

### 4.1 普通用户全局安装pnpm的时候无权限，errno -13

```bash
sudo chown -R $(whoami) "/usr/local/.nvm"
```

### 4.2 node运行内存溢出

```bash
# 添加环境变量,指定old-space为4G
vim /etc/profile

# 末尾追加
export NODE_OPTIONS=--max_old_space_size=4096

```

### 4.3 nx/lerna提示 cannot find module '@nrwl/nx-linux-64-gnu'

```bash
# 脚本中手动添加
npm install @nrwl/nx-linux-64-gnu@15.
```

## 参考文档

https://github.com/nvm-sh/nvm