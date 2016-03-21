---
layout: post
title: "webpack简明使用指南"
tags: [webpack, javascript, tools]
---

  本文是收集相关资料后整理的关于Webpack学习和使用的简要指南，目的是帮助自己归整思路并成为日后使用的参考手册，同时也为有相同需要的人提供一份参考。

## 背景

  今天的Web网站正在演化成WebApp：
  
  * 页面中的js越来越多
  * 现代浏览器功能越来越强大，能做的事越来越多
  * 很少有页面被全部重新加载，更多的是部分页面被加载
  
这些导致的结果是在客户端有大量的代码。为了便于维护，我们需要对代码进行有效的组织，模块系统是一种组织代码的方式。

当下流行的模块系统主要有：[CommonJS](http://requirejs.org/docs/commonjs.html)，[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)，[ES6 modules](http://www.ecma-international.org/ecma-262/6.0/#sec-modules)等

除了JS的代码组织问题外，还会碰到以下问题：
  
  * 代码转移(transferring)
  
    网站需要把服务器端的module转移到浏览器，有两种极端的方式：1）1次请求一个模块；2）1次请求所有模块。还有一种妥协方案是chunked transferring。
  
  * 其它资源模块化
    
    除了JS外，其它资源的组织，例如css等也需要模块系统

  * 模块依赖
  
    模块化之后，模块之间会产生依赖，这些依赖需要管理


## Webpack概述

  webpack是一个模块打包器(bundler)。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

  webpack的目标:

  * 将依赖树拆分成按需加载的块
  * 初始化加载的耗时尽量少
  * 各种静态资源都可以视作模块
  * 将第三方库整合成模块的能力
  * 可以自定义打包逻辑的能力
  * 适合大项目，无论是单页还是多页的 Web 应用

webpack的不同之处:

  * 代码拆分
  
    Webpack 有两种组织模块依赖的方式，同步和异步。异步依赖作为分割点，形成一个新的块。在优化了依赖树后，每一个异步区块都作为一个文件被打包。
  
  * Loader
    
    Webpack 本身只能处理原生的 JavaScript 模块，但是 loader 转换器可以将各种类型的资源转换成 JavaScript 模块。这样，任何资源都可以成为 Webpack 可以处理的模块。
  
  * 智能解析
  
    Webpack 有一个智能解析器，几乎可以处理任何第三方库，无论它们的模块形式是 CommonJS、 AMD 还是普通的 JS 文件。甚至在加载依赖的时候，允许使用动态表达式 require("./templates/" + name + ".jade")。
  
  * 插件系统
  
    Webpack 还有一个功能丰富的插件系统。大多数内容功能都是基于这个插件系统运行的，还可以开发和使用开源的 Webpack 插件，来满足各式各样的需求。

## 安装和使用

### 安装

  首先要安装 [Node.js](https://nodejs.org/en/download/)，Webpack 需要 Node.js v0.6 以上支持，建议使用最新版 Node.js。

  用npm安装Webpack：
```shell
  $ npm install webpack -g
```

  在项目中使用webpack：
```shell
  $ npm init
  $ npm install webpack --save-dev
```

### 使用

  使用可参考[webpack-your-bags](http://blog.madewithlove.be/post/webpack-your-bags/)，示例代码可访问[这里](https://github.com/totothink/webpack-article)

## 开发环境

  当项目逐渐变大，webpack 的编译时间会变长，可以通过参数让编译的输出内容带有进度和颜色。
```shell
  $ webpack --progress --colors
```

  如果不想每次修改模块后都重新编译，那么可以启动监听模式。开启监听模式后，没有变化的模块会在编译后缓存到内存中，而不会每次都被重新编译，所以监听模式的整体速度是很快的。
```shell
  $ webpack --progress --colors --watch
```

  使用 webpack-dev-server 开发服务是一个更好的选择。它将在 localhost:8080 启动一个 express 静态资源 web 服务器，并且会以监听模式自动运行 webpack，在浏览器打开 http://localhost:8080/ 或 http://localhost:8080/webpack-dev-server/ 可以浏览项目中的页面和编译后的资源输出，并且通过一个 socket.io 服务实时监听它们的变化并自动刷新页面。
```shell
  # 安装
  $ npm install webpack-dev-server -g

  # 运行
  $ webpack-dev-server --progress --colors
```

## 相关资源

  * [webpack源码](https://github.com/webpack/webpack)
  * [webpack示例](https://github.com/webpack/webpack/tree/master/examples)
  * [loader列表](https://webpack.github.io/docs/list-of-loaders.html)
  * [plugin列表](http://webpack.github.io/docs/list-of-plugins.html)
  * [webpack可视化工具](http://chrisbateman.github.io/webpack-visualizer/)
  * [Webpack分析](http://webpack.github.io/analyse/)


## 参考文献
  * [webpack官方文档](http://webpack.github.io/docs/)
  * [webpack中文指南](http://zhaoda.net/webpack-handbook/index.html)