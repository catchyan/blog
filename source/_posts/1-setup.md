---
title: Vue3 源码阅读.1-环境搭建
---
> 本系列文章旨在学习 Vue.js 源码，从源码的角度去理解 Vue.js 的设计思想和实现原理。本系列文章的源码版本为发文时最新版本 [3.4.3](https://github.com/vuejs/core/tree/v3.4.3)。本系列文章顺序会按照Vue.js官网的[深度指南](https://cn.vuejs.org/guide/introduction.html)顺序进行分析。笔者能力有限，如有错误，欢迎 [issue](https://github.com/catchyan/catchyan.github.io/issues) 指正。

## 环境搭建
### 准备工具
- [Git](https://git-scm.com/)
- [Node.js](https://nodejs.org/en/)
- [pnpm](https://pnpm.io/zh/)
- [VSCode](https://code.visualstudio.com/)
- 最后在 VSCode 中搜索 [Vitest](https://marketplace.visualstudio.com/items?itemName=ZixuanChen.vitest-explorer) 插件并安装，这个插件可以让我们在 VSCode 中直接运行 vitest 测试用例。
### 下载源码
```bash
git clone https://github.com/vuejs/core.git
```
```bash
git clone https://github.com/vitejs/vite.git
```
```bash
git clone https://github.com/vitejs/vite-plugin-vue.git
```
## 本地源码链接
想要直接在源码中打断点进行调试，可以使用 pnpm 的 [Workspace 协议](https://pnpm.io/zh/workspaces#workspace-%E5%8D%8F%E8%AE%AE-workspace)从工作区中链接本地源码。之后通过开启 rollup 的 sourceMap 选项，就可以在源码中打断点进行调试了。
### 创建demo项目
跟随[快速上手](https://cn.vuejs.org/guide/quick-start.html)创建一个demo项目
```bash
npm create vue@latest
```
```bash
✔ Project name: … <your-project-name>
✔ Add TypeScript? … No / Yes
✔ Add JSX Support? … No / Yes
✔ Add Vue Router for Single Page Application development? … No / Yes
✔ Add Pinia for state management? … No / Yes
✔ Add Vitest for Unit testing? … No / Yes
✔ Add an End-to-End Testing Solution? … No / Cypress / Playwright
✔ Add ESLint for code quality? … No / Yes
✔ Add Prettier for code formatting? … No / Yes

Scaffolding project in ./<your-project-name>...
Done.
```
当前项目目录结构如下：
```bash
.
├── core    // Vue3 源码
├── demo    // demo 项目
├── vite    // vite 源码
└── vite-plugin-vue // vite-plugin-vue 源码
```

### 修改 demo 项目的包依赖
在脚手架创建好的工程的中修改 package.json 文件：
```diff
...
  "dependencies": {
-   "vue": "^3.3.11"
+   "vue": "workspace:../core/packages/vue"
  },
  "devDependencies": {
-   "@vitejs/plugin-vue": "^4.5.2",
+   "@vitejs/plugin-vue": "workspace:../vite-plugin-vue/packages/plugin-vue",
-   "vite": "^5.0.10"
+   "vite": "workspace:../vite/packages/vite"
  }
...
```

