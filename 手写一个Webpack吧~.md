# 前言

Webpack 是前端热门打包工具，是前端工程化的根基，在 Webpack 眼中，万物皆模块，因此 Webpack 打包的核心目的其实就是把所有模块都打包到一个文件中（bundle.js）

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/126ba763572f482b9bbb9cab5d8cd164~tplv-k3u1fbpfcp-zoom-1.image)

# 原理分析

如果现在有这些模块：

index.js

```jypescript
import add from './add.js';
console.log(add(1, 2));
```

add.js

```jypescript
export default function add(a, b) {
  return a + b;
}
```

我们配置一个最基本的 webpack.config.js 文件：

```javascript
const path = require("path")

module.exports = {
  entry: "./index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "bundle.js",
  },
  mode: "none",
}
```

执行 npx webpack，打包后结果：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16174bce7e45449d8b4ebaa109f7c4e0~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，webpack 将多个 js 文件打包到了一个文件中，这就是 webpack 的原理，要实现这个原理，需要这些步骤：

- 分析 entry 文件；
- 基于 entry 文件的导入依赖，递归查找出整个项目的依赖；
- 将所有依赖转换成依赖图；
- 写入 bundle.js；

# 分析 entry 文件

由于需要 es6 转 es5，以及 ast 语法树的生成，便于我们查找每个文件中的依赖关系，下载这些依赖包：

```javascript
npm i --save-dev @babel/parser @babel/traverse @babel/core
```

并且在项目中新建 webpack.js 文件，导入包：

```javascript
const fs = require("fs")
const path = require("path")
const parser = require("@babel/parser")
const traverse = require("@babel/traverse").default
const babel = require("@babel/core")
```

接下来开始写第一个方法 getModuleInfo，获取单个文件的信息，代码如下：

```javascript
//生成单文件依赖树
function getModuleInfo(file) {
  //读取入口文件内容
  const body = fs.readFileSync(file, "utf-8")
  //转为ast语法树
  const ast = parser.parse(body, {
    sourceType: "module",
  })
  //收集所有模块依赖
  const deps = {}
  traverse(ast, {
    ImportDeclaration({ node }) {
      const dirname = path.dirname(file)
      //转换当前文件的绝对路径
      const absPath = path.join(dirname, node.source.value)
      deps[node.source.value] = absPath
    },
  })
  //es6转es5代码
  const { code } = babel.transformFromAst(ast, null, {
    presets: ["@babel/preset-env"],
  })
  const moduleInfo = { file, deps, code }
  return moduleInfo
}
```

从代码可以看到，读取到文件信息后，将文件转为 ast 语法树，并开始收集依赖，这里遍历器用的是 babel-traverse 自带的遍历器，让我们快速收集到所有 import 导入依赖的语句，最后将文件代码转为 es5 代码。

执行`getModuleInfo('./index.js');`出现结果：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c03562d401aa4e618b33c0bdd7491426~tplv-k3u1fbpfcp-zoom-1.image)

这就是依赖树种一个文件的内容，接下来的思路很简单，基于 entry 文件的 deps，递归查找所有 deps 文件中所用到的依赖，直到无依赖，并将收集到的信息全部保存在一个对象中。

# 收集依赖

这里我们创建两个函数，parseModules 和 getDeps，代码如下：

```javascript
//收集entry文件依赖，并整合所有依赖
function parseModules(file) {
  const entry = getModuleInfo(file)
  let temps = [entry]
  const depsResult = {} //整个项目最终所有文件 -> 代码块的映射表
  getDeps(entry, temps)
  temps.forEach((item) => {
    depsResult[item.file] = {
      deps: item.deps,
      code: item.code,
    }
  })
  return depsResult
}

//收集entry文件所依赖的子文件其他依赖
function getDeps({ deps }, temps) {
  //遍历入口文件的所有依赖，寻找更多依赖
  Object.keys(deps).forEach((d) => {
    const child = getModuleInfo(deps[d])
    temps.push(child)
    getDeps(child, temps)
  })
}
```

在 parseModules 函数中，我们首先获取到了入口文件的依赖树，基于他的 deps，进行更多 deps 的获取，并记录他们的依赖树，最后保存在 depsResult 中，结果如下：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e78860a43e5492793a2249d9bb7c4fe~tplv-k3u1fbpfcp-zoom-1.image)
接下来，我们将所有文件的代码合并起来即可。

# 写入 bundle

这里，我们写一个自执行函数，里面包括了自定义 require 函数和 exports 对象，让浏览器识别到我们自定义的导入导出：

```javascript
function bundle(file) {
  const depsGraph = JSON.stringify(parseModules(file))
  return `(function (graph) {
        function require(file) {
            function absRequire(relPath) {
                return require(graph[file].deps[relPath])
            }
            var exports = {};
            (function (require,exports,code) {
                eval(code)
            })(absRequire,exports,graph[file].code)
            return exports
        }
        require('${file}')
    })(${depsGraph})`
}

const content = bundle("./index.js")
!fs.existsSync("./dist") && fs.mkdirSync("./dist")
fs.writeFileSync("./dist/bundle.js", content)
```

最后把这块代码写入 dist/bundle.js 即可。

# 测试

我们在 index.html 中引入 dist/bundle.js：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f51fb3c600e64c21aa692c8b74c10885~tplv-k3u1fbpfcp-zoom-1.image)
打开浏览器：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/971a7306ee1f49e7bdab9b06fc352bfb~tplv-k3u1fbpfcp-zoom-1.image)

# 源码

```javascript
const fs = require("fs")
const path = require("path")
const parser = require("@babel/parser")
const traverse = require("@babel/traverse").default
const babel = require("@babel/core")

//生成单文件依赖树
function getModuleInfo(file) {
  //读取入口文件内容
  const body = fs.readFileSync(file, "utf-8")
  //转为ast语法树
  const ast = parser.parse(body, {
    sourceType: "module",
  })
  //收集所有模块依赖
  const deps = {}
  traverse(ast, {
    ImportDeclaration({ node }) {
      const dirname = path.dirname(file)
      //转换当前文件的绝对路径
      const absPath = path.join(dirname, node.source.value)
      deps[node.source.value] = absPath
    },
  })
  //es6转es5代码
  const { code } = babel.transformFromAst(ast, null, {
    presets: ["@babel/preset-env"],
  })
  const moduleInfo = { file, deps, code }
  return moduleInfo
}

//收集entry文件依赖，并整合所有依赖
function parseModules(file) {
  const entry = getModuleInfo(file)
  let temps = [entry]
  const depsResult = {} //整个项目最终所有文件 -> 代码块的映射表
  getDeps(entry, temps)
  temps.forEach((item) => {
    depsResult[item.file] = {
      deps: item.deps,
      code: item.code,
    }
  })
  return depsResult
}

//收集entry文件所依赖的子文件其他依赖
function getDeps({ deps }, temps) {
  //遍历入口文件的所有依赖，寻找更多依赖
  Object.keys(deps).forEach((d) => {
    const child = getModuleInfo(deps[d])
    temps.push(child)
    getDeps(child, temps)
  })
}

function bundle(file) {
  const depsGraph = JSON.stringify(parseModules(file))
  return `(function (graph) {
        function require(file) {
            function absRequire(relPath) {
                return require(graph[file].deps[relPath])
            }
            var exports = {};
            (function (require,exports,code) {
                eval(code)
            })(absRequire,exports,graph[file].code)
            return exports
        }
        require('${file}')
    })(${depsGraph})`
}

const content = bundle("./index.js")
!fs.existsSync("./dist") && fs.mkdirSync("./dist")
fs.writeFileSync("./dist/bundle.js", content)
```

ok，学费了~
