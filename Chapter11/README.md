# 第十一章 前端构建

<!-- toc -->

- [1、什么是webpack，他与grunt、gulp的不同](#1、什么是webpack，他与grunt、gulp的不同)
- [2、什么是bundle，什么是chunk，什么是module](#2、什么是bundle，什么是chunk，什么是module)
- [3、有哪些常见的Loader？他们是解决什么问题的？](#3、有哪些常见的Loader？他们是解决什么问题的？)
- [4、有哪些常见的Plugin？他们是解决什么问题的？](#4、有哪些常见的Plugin？他们是解决什么问题的？)
- [5、webpack的构建流程](#5、webpack的构建流程)
- [6、性能优化](#6、性能优化)
  <!-- tocstop -->


## 1、什么是webpack，他与grunt、gulp的不同

#### 1.1、 webpack

本质上，webpack 是一个JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，**它会从入口文件的依赖开始解析，递归地构建一个依赖关系图**(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

**在 Webpack 里一 切文件皆模块，通过 Loader转换文件，通过 Plugin注入钩子，最后输出由多个模块组合成的文件。 Webpack 专注于构建模块化项目。**

一切文件如 JavaScript、 css、 scss、图片、模板，对于 Webpack来说都是一个个模块， 这样的好处是能清晰地描述各个模块之间的依赖关系，以方便 Webpack 对模块进行组合和打 包。经过 Webpack 的处理，最终会输出浏览器能使用的静态资源。

#### 1.2、 webpack与grunt、gulp的不同？

他们都是用于前端构建的工具。

grunt和gulp是基于任务和流（Task、Stream）的。对这些task进行一系列链式操作，更新流上的数据， 整条链式操作就构成了整个web的构建流程。

webpack是基于模块的。webpack会自动地递归解析入口需要加载的所有资源模块，然后用不同的Loader来处理不同的文件，用Plugin来扩展功能。



## 2、什么是bundle，什么是chunk，什么是module

bundle是由webpack打包出来的文件。

chunk是指webpack在进行模块依赖分析的时候，分割出来的代码块。

module是指模块，在Webpack中一切皆模块，一个模块即为一个文件。Webpack会从Entry开始递归找出所有的依赖模块



## 3、有哪些常见的Loader？他们是解决什么问题的？

**Loader用于对模块文件进行编译转换和加载处理**，在module.rules中进行配置，它用于告诉Webpack在遇到哪些文件时使用哪些Loader去加载和转换。

- babel-loader：把 ES6 转换成 ES5
- awesome-typescript-loader：解析ts语法
- url-loader：打包静态文件（图片）（大于limit时用file-loader）
- css-loader：加载 CSS模块，支持模块化、压缩、文件导入等特性
- style-loader：把 CSS 代码注入到页面中
- source-map-loader：加载额外的 Source Map 文件，以方便断点调试
- image-loader：加载并且压缩图片文件
- eslint-loader：通过 ESLint 检查 JavaScript 代码



## 4、有哪些常见的Plugin？他们是解决什么问题的？

**Plugin可以扩展webpack的功能，这些扩展功能可以参与到整个webpack打包的各个流程(生命周期)**。

**实现原理是在构建流程里注入钩子函数。**

每一个内部插件，都是通过监听任务点的方式，来实现自定义的逻辑

- DefinePlugin：定义环境变量
- commons-chunk-plugin：提取公共代码
- CleanWebpackPlugin：清除dist目录
- MiniCssExtractPlugin：将CSS提取为独立的文件



**【tips】什么是钩子函数？**

钩子函数：钩子函数是在一个事件触发的时候，在系统级捕获到并做一些操作，一段用以处理系统消息的程序。“钩子”就是在某个阶段给你一个做某些处理的机会。

钩子（hook）函数就是处理拦截在软件组件之间传递的函数调用或事件或消息的代码

在计算机编程中，**钩子函数主要用于通过拦截在软件组件之间传递的函数调用（或消息/事件）来改变或增强操作系统，应用程序或其他软件组件的行为。**处理这种截获的函数调用，事件或消息的代码称为钩子，它的本质就是用以处理系统消息的程序，通过系统调用，把它挂入系统。钩子函数可用于许多目的，包括调试和扩展功能。常见的钩子函数：react的生命周期函数、vue的生命周期函数等。



## 5、webpack的构建流程

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
- 确定入口：根据配置中的 entry 找出所有的入口文件；
- 编译模块：从入口文件出发，**调用所有配置的 Loader 对模块进行翻译**，**再找出该模块依赖的模块，再递归的完成所有文件的处理；**

​       使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；

- 输出资源：**根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk**，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
- 输出完成：在确定好输出内容后，根据配置中的输出路径和文件名，把文件内容写入到文件系统。

在以上过程中，**Webpack 会在特定的时间点广播出特定的事件，插件在监听到对应的事件后会执行相应的处理逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。**



## 6、性能优化

用webpack优化前端性能是指优化webpack的输出结果，让打包的最终结果在浏览器运行快速高效。

- 提取公共代码，减少打包体积

CommonsChunkPlugin->splitChunks

- 压缩代码

删除多余的代码、注释、简化代码的写法等等方式。可以利用webpack的UglifyJsPlugin和ParallelUglifyPlugin来压缩JS文件， 利用cssnano（css-loader?optimization.minimizer.OptimizeCSSAssetsPlugin）来压缩css

使用UglifyJS插件压缩JS代码时，需要先将代码解析成Object表示的AST（抽象语法树），再去应用各种规则去分析和处理AST，所以这个过程计算量大耗时较多。ParallelUglifyPlugin可以开启多个子进程，每个子进程使用UglifyJS压缩代码，可以并行执行，能显著缩短压缩时间。

- 减少构建搜索或编译路径

--优化Loader的文件搜索范围：增加exlude，include缩小搜索范围

--设置resolve.modules:[path.resolve(__dirname, 'node_modules')]避免层层查找

--对庞大的第三方模块设置**resolve.alias**

```javascript
resolve.alias:{
    'react': path.resolve(__dirname, './node_modules/react/dist/react.min.js')
}
```

或

```javascript
resolve: {
    alias: {
        'react-dom': '@hot-loader/react-dom'
    }
}
```

--合理配置resolve.extensions，减少文件查找

- 多进程处理

[happypack](http://link.zhihu.com/?target=https%3A//github.com/amireh/happypack) 的原理是让loader可以多进程去处理文件

- 利用[CDN](https://cloud.tencent.com/product/cdn?from=10680)加速。在构建过程中，将引用的静态资源路径修改为CDN上对应的路径。可以利用webpack对于output参数和各loader的publicPath参数来修改资源路径
- 删除死代码（Tree Shaking）

将代码中永远不会走到的片段删除掉。可以通过在启动webpack时追加参数--optimize-minimize来实现

- 缓存与增量构建

把Babel编译过的文件缓存起来，下次只需要编译更改过的代码文件即可

`loader: 'babel-loader?cacheDirectory=ture'`

- DllPlugin 

DllPlugin可以将特定的类库提前打包然后引入。这种方式可以极大的减少打包类库的次数，只有当类库更新版本才有需要重新打包，并且也实现了将公共代码抽离成单独文件的优化方案.



