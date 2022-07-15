# 前言

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7665c9c0744644fbbbac8e44cec4853e~tplv-k3u1fbpfcp-zoom-1.image)

学习过 Webpack 都知道，Webpack 在打包项目模块的过程中，有两个配置项是编译中的主角，分别是：

- Loader 模块预处理器
- Plugin 插件

Loader 会在解析每一个模块前判断文件名是否在 loader 配置项中存在，如果存在，则会在打包前进行文件预处理，因为 Webpack 本身只支持 js 模块的打包。

而 Plugin 则会在整个打包过程中，以"恰当的时机"，来进行插件的运行，常用于优化、运行服务、dist 文件处理等等。那么问题来了，恰当的时机是什么时机？在接下来的内容中，你将了解，plugin 是什么、plugin 在打包构建过程中的原理、如何仿写一个 plugin。

# Tapable

Webpack 是建立于插件系统之上的事件流工作系统，而插件系统的根基则是 tapable 这个库，tapable 通过观察者模式实现了事件的监听和触发，提供给了 Webpack 很多 Hook 类，这些 Hook 类可以用来生成实例对象。

```javascript
const {
  SyncHook,
  SyncBailHook,
  SyncWaterfallHook,
  SyncLoopHook,
  AsyncParalleHook,
  AsyncParalleBailHook,
  AsyncSeriesHook,
  AsyncSeriesBailHook,
  AsyncSeriesLoopHook,
  AsyncSeriesWaterfallHook,
} = require("tapable")
```

tapable 提供了上述十大 Hook 类以及三种实例方法：tap、tapAsync 和 tapPromise，为 Webpack 整个工作创建了观察 -> 触发插件的工作流。

# Webpack Plugin 生命周期

webpack 源码中对于 plugin 执行前有如下代码:

```javascript
compiler = new Compiler(options.context)
compiler.options = options
if (options.plugins && Array.isArray(options.plugins)) {
  for (const plugin of options.plugins) {
    if (typeof plugin === "function") {
      plugin.call(compiler, compiler)
    } else {
      plugin.apply(compiler)
    }
  }
}
```

可以看到，如果在 webpack.config.js 中配置了 plugins 并且他是个数组，webpack 会依次去执行每一个 plugin，并且把 compiler 传给每一个 plugin。

那么 compiler 到底是什么？顺着代码到 compiler.js 可以看到：

```javascript
class Compiler extends Tapable {}
class Compilation extends Tapable {}
```

其实就是继承于 Tapable 的子类，那么 Tapable 的观察者模式就可以对上了，Compiler 基于观察者模式，设计出了一系列的生命周期钩子函数。基于 tapable 的支持，Webpack 插件系统中提供了大量的生命周期钩子函数，让每一个插件都可以在需要的时候去执行。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/277cd24222e340f397079a15c489fe69~tplv-k3u1fbpfcp-zoom-1.image)

上图是 Webpack 在 compiler 在整个打包中所涉及到的生命周期，可以试想一下。
如 clean-webpack-plugin 插件在打包前，就应该清除目录，因此它应该会在 done（解析完毕）执行，去清除我们的 output 目录；html-webpack-plugin 插件会根据打包产出资源构建出一个 web 服务器，因此它应该会在所有钩子结束后，无任何 error 的情况下去执行该 plugin，开启一个服务器。

# 手写一个 plugin

我们在项目中创建一个 HelloPlugin.js 文件，在其中写入：

```javascript
class HelloPlugin {
	constructor(options) {
		console.log(options);
	}
	apply(compiler) {
		compiler.hooks.done.tap('HelloPlugin', () => {
			console.log('HelloPlugin');
		}
	}
}
```

并在 webpack.config.js 中定义：

```javascript
const HelloPlugin = require('./HelloPlugin.js');

module.exports = {
	...
	plugins: [
		new HelloPlugin(),
	],
	...
}
```

**执行 npx webpack，可以看到打印结果。**

在 Webpack 官方推荐：使用 es6 class 去定义每一个 plugin，使用 new 实例化出一个构造函数后，可以再 constructor 获取到对应的传参。并且 apply 方法会在插件初始化时被调用一次，内部方法就是插件的功能，而 compiler.hooks 后可以选择插件在某一个生命周期时去执行，并且可以在 class 中声明多个这样的钩子函数。

钩子函数的第一个参数为插件名，第二个参数是回调函数，在回调函数中可以获取 Compilation 对象。
compiler 和 compilation 都存放着编译相关的信息，compiler 存放的是 webpack 全局环境信息，而 compilation 存放的是开发模式运行时一次性、不间断编译的信息。

# 模拟 copy-webpack-plugin

copy-webpack-plugin 可以将项目目录中某一个文件夹或文件直接复制到打包目标文件夹中，这里我们实现一下。

创建一个 CopyPlugin.js 文件，代码如下：

```javascript
const fs = require("fs")

class CopyPlugin {
  constructor(options) {
    this.from = options.from
    this.to = options.to
  }
  apply(compiler) {
    const { from, to } = this
    const isDir = fs.statSync(from).isFile() ? false : true
    compiler.hooks.done.tap("CopyPlugin", () => {
      fs.cp(from, to, { recursive: isDir }, (err) => {
        if (err) {
          throw err
        }
      })
    })
  }
}

module.exports = CopyPlugin
```

在 webpack.config.js 中配置：

```javascript
const path = require('path');
const CopyPlugin = require('./copyPlugin');

module.exports = {
	...
	plugins: [
		new CopyPlugin({
 			from: path.resolve(__dirname, 'static'),
    		to: path.resolve(__dirname, 'dist', 'static')
		})
	],
	...
}
```

这里我们选择将 static 文件夹复制到 dist 目录中，使用方式和原来的一样，执行 npx webpack 可以看到：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb56e090911e4e49be63fbd8ce84b996~tplv-k3u1fbpfcp-zoom-1.image)
具体逻辑就是复制一个文件夹或文件，执行时间这里是选择在打包结束后进行复制的，对应生命周期 compiler.hooks.done

# 模拟 clean-webpack-plugin

clean-webpack-plugin 会在打包出新的文件前清空整个打包目录，所以应该在生命周期偏起始的时候去执行，这里我们选择 compiler.hooks.make。

创建 CleanPlugin.js，写入代码：

```javascript
const fs = require("fs")
const path = require("path")

class CleanPlugin {
  apply(compiler) {
    compiler.hooks.make.tap("CleanPlugin", () => {
      function clearDir(dirPath) {
        let files = []
        if (fs.existsSync(dirPath)) {
          files = fs.readdirSync(dirPath)
          files.forEach((f) => {
            const absPath = dirPath + "/" + f
            if (fs.statSync(absPath).isDirectory()) {
              clearDir(absPath)
            } else {
              fs.unlinkSync(absPath)
            }
          })
        }
      }
      clearDir(path.resolve(__dirname, "dist"))
    })
  }
}

module.exports = CleanPlugin
```

在 webpack.config.js 中配置：

```javascript
const path = require('path');
const CleanPlugin = require('./CleanPlugin');

module.exports = {
	...
	plugins: [
		new CleanPlugin(),
	],
	...
}
```

运行 npx webpack，会发现，在打包中，目录被清除了，最后又出现文件了。

# 总结

最后结合了两个案例的仿写，相信你已经对 Webpack plugin 的组成和运行原理有了一定的理解，它其实就如同 Vue 组件的生命周期一样，Webpack 在打包过程中也是会有这样的生命周期去执行一系列的插件，如果觉得文章对你有帮助，请点个赞吧~
