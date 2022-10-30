# 什么是 Turbopack

Turbopack 是针对 JavaScript 和 TypeScript 优化的增量打包器，由 Vercel 的 Webpack 和 Next.js 的创建者用 Rust 编写。换句话说，就是基于 Rust 重新实现了 webpack.

# 为什么要写 Turbopack

-   vite 采用的是 Bundless 的方式，由于只需要编译一个文件，相应更新比较快，但是处理大型应用的时候，大量的级联请求会导致很慢的启动速度
-   采用了增量编译，采用了缓存机制，缓存的粒度是函数级别的，意味着做过的任务不会再做
-   为啥不用 esbuild
    -   没有 HMR
    -   没有缓存 做过的事情要重复做
    -   不支持懒打包
-   在 dev 模式下面支持懒打包，根据请求建立依赖图，然后返回给浏览器（例如你访问 localhost:3000，它只会返回 pages/index.jsx 和它导入的模块）。采取这种方式的话，会比 Native ESM 更快。
    

# 核心的概念

## **The Turbo engine**

> Turbopack 之所以如此之快，是因为它建立在一个可重用的 Rust 库之上，该库支持称为 Turbo 引擎的增量计算

### **Function-level caching**

![](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=YzdjOWI0ZGI4MTk2MjkyZDA1YjI0NmYxYjFlZWI3Y2RfSlM0OUpDbkpuUGR2YVgxYUxCa1BoOHd1bHdyT2kxYnFfVG9rZW46Ym94Y242a2ZBaExVT2Vpd1dtYThPZlhvbURjXzE2NjcxMzkyNTY6MTY2NzE0Mjg1Nl9WNA)

**update sdk.ts**

![](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWY4NmUzNGJmZTUyOWM1N2JlMGZhOTMzNTM0MjVlOThfSHZuc3NGTEd2TDBwZlp2c1lyQjRFRHBKdk5aMnZzYzNfVG9rZW46Ym94Y250cDF1ZEN3YWs2cmRxcVM2U1BmQUdnXzE2NjcxMzkyNTY6MTY2NzE0Mjg1Nl9WNA)

现在的缓存是存在内存中，取消服务的时候，内存就会被释放掉。未来的规划是持久化存储，存储在文件系统中，或者采用和像 Turborepo’s 的远程缓存，这就意味着 Turbopack 可以跨机器的记忆。

### **Compiling by Request**

> 这个是为了解决项目的启动时间，目的就是为了在启动的时候编译需要用到的代码

**Page-level compilation**
	一开始 next 是编译整个 app。在 11 版本之后，只需编译请求 page 代码。这样是一个好的解决办法，但是不是完美的。采用这种方式的话，会同时编译 client 和 server 的模块，动态导入的，css 和图片，也会在视图之后被编译。
**Request-level compilation**
	只会编译请求的资源，例如浏览器请求了 HTML，他就只编译 HTML，内部引用的东西不会被编译。
	不再像基于纯 esm 的工具一个入口请求过来，后续所有依赖都是一个单独的请求，所以一旦拆解模块很小很细，在 dev 环境很容易造成网络请求上的排队延迟，turpo 目前这种就是按照你当前入口的请求把后续所有依赖请求都打包所以理论上还是只有一个请求，并且只针对当前的请求打包，也不是启动的时候都打包，所以在冷启动和运行时请求都照顾到了，并且再加上缓存的特性，让重复工作得以直接输出结果所以更快。

# 规划

-   现在内置到 Next 13 中，支持 dev 模式，未来会支持到 build 模式
-   计划支持 svelte
-   计划持久化缓存，本地和远程的
-   提供 webpack 的迁移
-   将 turborepo 用 rust 重写与 turbopack合并
# feature

-   JS 和 TS 基于 SWC 打包，可以在 package.json里面配置 Browserslist，不支持 babel 未来可能提供 babel 的插件支持
-   支持在 tsconfig.json 配置 `paths` 和 `baseUrl` 不支持 type checker。需要你自己运行 `tsc —-watch` 或者依赖 IDE 的能力
-   框架层面 支持 jsx 和 React Server Components。不支持 ****`next/dynamic`。**不支持 `vue` 和 `svelte`
-   CSS 打包依赖于 swc_css。支持 css 和 css modules. 使用****`postcss-nested`**** 语法，支持嵌套。支持 @import 语法。不支持 PostCSS 插件，LESS、 SCSS、 Tailwind CSS
-   Dev Server 支持 HMR 和 Fast Refresh
-   支持导入静态资源，支持 public 目录和 import JSON
-   import 支持 CJS ESM 和部分的 AMD，支持 `require()` `import type` `Dynamic Imports`
-   支持 `.env` 文件并且会自动 reload，支持注入 `process.env`

# Turbopack vs vite

[https://github.com/yyx990803/vite-vs-next-turbo-hmr](https://github.com/yyx990803/vite-vs-next-turbo-hmr)

![](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=NDdiMjkxMzA3ZDUxYWJkMGI0ZTAzNmVmMDVjN2Q0ZDRfS2Q2UDZvT21wcVoyQ1FDYkdSa0RSQjFpQ2RZeHlHNjlfVG9rZW46Ym94Y25JOG5oVDhIMzdCZ2ZEcnVvVERJMXBnXzE2NjcxMzkyNTY6MTY2NzE0Mjg1Nl9WNA)

![](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=OWE5NjkxMDAxZmNiZGRkNmQ0ZDg4ZjAzZDI1Y2Q3MzZfSGYzT01LZUR0bE9xTW1hZkdNR3F3d2Y4NmVZVkE0ejBfVG9rZW46Ym94Y25uaE9mNU1OZGdWTDdFT1VWNTJPOVh4XzE2NjcxMzkyNTY6MTY2NzE0Mjg1Nl9WNA)

# 拓展阅读

[https://leerob.io/blog/rust](https://leerob.io/blog/rust) Rust Is The Future of JavaScript Infrastructure
https://www.youtube.com/watch?v=NiknNI_0J48&t=2129s next conf
[https://github.com/rolldown-rs/rolldown](https://github.com/rolldown-rs/rolldown) rust 重写的 rollup
[https://github.com/telecss/telecss](https://github.com/telecss/telecss) rust 写的 css 的 parser
[https://github.com/parcel-bundler/lightningcss](https://github.com/parcel-bundler/lightningcss) rust 写的 css 的 parser
[https://github.com/marcj/TypeRunner](https://github.com/marcj/TypeRunner) c++ 实现的 type checker
[https://github.com/dudykr/stc](https://github.com/dudykr/stc) rust 实现的 type checker
[https://github.com/HerringtonDarkholme/vue-compiler](https://github.com/HerringtonDarkholme/vue-compiler) rust 实现的 vue template compiler