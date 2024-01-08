---
title: Vue3 源码阅读.2 - npm run dev 做了什么？
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
可以看到 `dev` 命令实际上是执行了 `vite` 命令，而 `vite` 命令是在 `vite` 包的 `bin` 目录下的 `vite.js` 文件：
```bash
.
├── vite
│   ├── packages
│   │   └── vite
│   │       ├── bin
│   │       │   └── vite.js
```
因本文的目的是学习 `Vue.js` 源码，所以不会去很深入的解析 `vite` 的源码。只跟着 `npm run dev` 的流程看下去。

`npm run dev` 对应的不带参数的 `vite` 命令会直接进入start方法

vite/packages/vite/bin/vite.js: 50
```js
// 前面的代码判断了启动参数，日志输出级别，是否开启node inspector等逻辑
function start() {
  return import('../dist/node/cli.js')
}
```
`start` 方法加载了 `../dist/node/cli.js`，对应源码为 `vite/packages/vite/src/node/cli.ts`。在 `cli.ts` 中我们可以看到 `vite` 使用 [cac](https://github.com/cacjs/cac) 库构建命令行参数，`vite` 命令对应设置如下：
```js
// dev
cli
  // 对应vite命令的解释
  .command('[root]', 'start dev server') 
  // 对 `vite` 命令取别名, 既 `vite serve` 命令等价于 `vite`
  .alias('serve') 
  // 另一个别名`dev`
  .alias('dev')
  // 可以通过 `--host` 或 `-h` 参数指定host
  .option('--host [host]', `[string] specify hostname`, { type: [convertHost] })
  // 可以通过 `--port` 或 `-p` 参数指定port
  .option('--port <port>', `[number] specify port`)
  // 设置 `--open [path]` 参数设置是否打开浏览器和打开浏览器的路径
  .option('--open [path]', `[boolean | string] open browser on startup`)
  // 设置 `--cors` 参数设置是否开启跨域
  .option('--cors', `[boolean] enable CORS`)
  // 设置 `--strictPort` 参数设置是否在端口被占用时退出
  .option('--strictPort', `[boolean] exit if specified port is already in use`)
  // 设置 `--force` 参数设置是否强制忽略缓存重新打包
  .option(
    '--force',
    `[boolean] force the optimizer to ignore the cache and re-bundle`,
  )
  .action(async (root: string, options: ServerOptions & GlobalCLIOptions) => {
    // 这个方法用于处理命令行重复设置的属性，如果一个属性重复则取最后一个设置的值
    filterDuplicateOptions(options)
    
    const { createServer } = await import('./server')
    try {
      const server = await createServer({
        root,
        base: options.base,
        mode: options.mode,
        configFile: options.config,
        logLevel: options.logLevel,
        clearScreen: options.clearScreen,
        optimizeDeps: { force: options.force },
        server: cleanOptions(options),
      })

      if (!server.httpServer) {
        throw new Error('HTTP server not available')
      }

      await server.listen()

      const info = server.config.logger.info

      const viteStartTime = global.__vite_start_time ?? false
      const startupDurationString = viteStartTime
        ? colors.dim(
            `ready in ${colors.reset(
              colors.bold(Math.ceil(performance.now() - viteStartTime)),
            )} ms`,
          )
        : ''
      const hasExistingLogs =
        process.stdout.bytesWritten > 0 || process.stderr.bytesWritten > 0

      info(
        `\n  ${colors.green(
          `${colors.bold('VITE')} v${VERSION}`,
        )}  ${startupDurationString}\n`,
        {
          clear: !hasExistingLogs,
        },
      )

      server.printUrls()
      const customShortcuts: CLIShortcut<typeof server>[] = []
      if (profileSession) {
        customShortcuts.push({
          key: 'p',
          description: 'start/stop the profiler',
          async action(server) {
            if (profileSession) {
              await stopProfiler(server.config.logger.info)
            } else {
              const inspector = await import('node:inspector').then(
                (r) => r.default,
              )
              await new Promise<void>((res) => {
                profileSession = new inspector.Session()
                profileSession.connect()
                profileSession.post('Profiler.enable', () => {
                  profileSession!.post('Profiler.start', () => {
                    server.config.logger.info('Profiler started')
                    res()
                  })
                })
              })
            }
          },
        })
      }
      server.bindCLIShortcuts({ print: true, customShortcuts })
    } catch (e) {
      const logger = createLogger(options.logLevel)
      logger.error(colors.red(`error when starting dev server:\n${e.stack}`), {
        error: e,
      })
      stopProfiler(logger.info)
      process.exit(1)
    }
  })
```
