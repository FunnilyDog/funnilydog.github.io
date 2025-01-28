---
title: "React18 阅读笔记 -- debug 环境搭建"
date: 2025-01-19T18:45:18+08:00
draft: false
summary: "简述如何利用打包工具 webpack / vite 配置 react debug 环境"
series: ["React18 阅读笔记"]
series_order: 1
---

{{< alert "github" >}}
配置好的仓库自取(带阅读笔记版): [react-debug github link](https://github.com/FunnilyDog/react-debug)
{{< /alert >}}

## vite

### 创建项目

```zsh
pnpm create vite
#  (根据提示配置项目)
...
```

### 下载 react 源码

github 地址：https://github.com/facebook/react
切换 tag ，找到对应版本 本文是 18.2.0 版本

```zsh
# 切换目录到src下，直接clone整个react资源包
cd src
git clone git@github.com:facebook/react.git
```

### 配置 vite.config.js

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  resolve: {
    // 配置别名
    alias: {
      react: path.posix.resolve("src/react/packages/react"),
      "react-dom": path.posix.resolve("src/react/packages/react-dom"),
      "react-dom-bindings": path.posix.resolve(
        "src/react/packages/react-dom-bindings"
      ),
      "react-reconciler": path.posix.resolve(
        "src/react/packages/react-reconciler"
      ),
      "react-client": path.posix.resolve("src/react/packages/react-client"),
      scheduler: path.posix.resolve("src/react/packages/scheduler"),
      shared: path.posix.resolve("src/react/packages/shared")
    }
  },
  // 配置环境变量
  define: {
    __DEV__: false,
    __EXPERIMENTAL__: true,
    __PROFILE__: true
  }
});
```

由于 DEV 配置的 false，需要手动改赋值一下 jsxDEV

```js
// path: src/react/packages/react/src/jsx/ReactJSX.js

// 注释掉原来的 赋值 直接赋值为 _jsxDEV(之前赋值为 jsxProd 会报错 没解决掉)
// const jsxDEV      = __DEV__ ? _jsxDEV : undefined;
const jsxDEV = _jsxDEV;
```

### 配置 tsconfig.json / jsconfig.json

```js
// ...
 "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "react/*": [
        "src/react/packages/react/*"
      ],
      "react-dom/*": [
        "src/react/packages/react-dom/*"
      ],
      "react-dom-bindings/*": [
        "src/react/packages/react-dom-bindings/*"
      ],
      "react-reconciler/*": [
        "src/react/packages/react-reconciler/*"
      ],
      "scheduler/*": [
        "src/react/packages/scheduler/*"
      ],
      "shared/*": [
        "src/react/packages/shared/*"
      ]
    }
  },
//   ...
```

### vscode 调试

![初始化launch.js](/imgs/launch.jpg "初始化launch.js")

```zsh
# 回退到项目根目录
cd ..
# 安装依赖
pnpm i
# 起项目
pnpm dev
```

F5 启动调试

### 解决报错

参考:

> [React 18 debugger 源码分析配置](https://www.skillgroup.cn/framework/react/scanalysis/react-debugger.html#_3-the-requested-module-src-react-packages-react-reconciler-src-reactfiberconfig-js-does-not-provide-an-export-named-getchildhostcontext)

## webpack

### 创建项目

```zsh
npx create-react-app debug-react

# 暴露webpack 配置
npm run eject
```

### 下载 react 源码

github 地址：https://github.com/facebook/react
切换 tag ，找到对应版本 本文是 18.2.0 版本

```zsh
# 切换目录到src下，直接clone整个react资源包
cd src
git clone git@github.com:facebook/react.git
```

### 修改配置

```js
// config/webpack.config.js
'react': path.resolve(__dirname, '../src/react/packages/react'),
'react-dom': path.resolve(__dirname, '../src/reactpackages/react-dom'),
'react-dom-bindings': path.resolve(__dirname, '../srcreact/packages/react-dom-bindings'),
'react-reconciler': path.resolve(__dirname, '../srcreact/packages/react-reconciler'),
'shared': path.resolve(__dirname, '../src/react/packagesshared'),
'scheduler': path.resolve(__dirname, '../src/reactpackages/scheduler'),
```

![alias配置](/imgs/alias_webpack.jpg "webpack 配置 alias")

```js
// /config/env.js
__DEV__: true,
__PROFILE__: true,
__UMD__: true,
__EXPERIMENTAL__: true,
__VARIANT__: true,
```

![全局变量](/imgs/env_webpack.jpg "全局变量")

### 解决报错

参考

> [手把手教你配置 React18 调试环境](https://juejin.cn/post/7248531609389416506)
