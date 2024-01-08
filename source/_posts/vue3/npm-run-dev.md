---
title: npm run dev 做了什么？
date: 2024-01-08 13:52:08
categories:
- Vue3 源码阅读
---
> 本系列文章旨在学习 `Vue.js` 源码，从源码的角度去理解 `Vue.js` 的设计思想和实现原理。本系列文章的源码版本为发文时最新版本 [3.4.3](https://github.com/vuejs/core/tree/v3.4.3)。本系列文章顺序会按照 `Vue.js` 官网的[深度指南](https://cn.vuejs.org/guide/introduction.html)顺序进行分析。笔者能力有限，如有错误，欢迎 [issue](https://github.com/catchyan/blog/issues) 指正。

<!-- more -->
```bash
npm run dev
```
在 `demo` 项目中执行 `npm run dev` 命令，会执行 `package.json` 中的 `scripts` 中的 `dev` 命令，`dev` 命令的内容如下：
```json
...
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
...
```
可以看到 `dev` 命令实际上是执行了 `vite` 命令，而 `vite` 命令是在 `vite` 包的 `bin` 目录下的 `vite.js` 文件，文件在源码中的路径为：
```bash
.
├── vite
│   ├── packages
│   │   └── vite
│   │       ├── bin
│   │       │   └── vite.js
```