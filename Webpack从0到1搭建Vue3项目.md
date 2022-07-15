# 前言

Vue 是当下非常火热的前端框架，选择 Vue-Cli 是大部分人主流的选择，因为其快捷、便利可以让我们一键生成整个项目模板，如果需要自定义配置下从 0 开始搭建一个 Vue 项目，其实也不难，接下来就来记录一下。

# 项目初始化

通过命令：

```typescript
npm init -y
```

生成一个初始的带包管理器的项目模板，参考 Vue3 脚手架生成的目录，我们也同样同步目录文件：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a25dd8351064476b92168389c121b794~tplv-k3u1fbpfcp-zoom-1.image)
文件内容笔者直接参照 vue-cli 生成的文件内容复制粘贴~~就不贴代码了。

## 构建打包

这里我们选择 Webpack 进行打包，首先下载依赖包。

```typescript
npm i --save-dev webpack webpack-cli webpack-dev-server webpack-merge html-webpack-plugin
```

这里首先下载这几个最基本的依赖包，简单说明下：

- webpack 我们需要打包的包
- webpack-cli 打包依赖包+1
- webpack-dev-server 开发环境建立本地网络服务器所需的包
- webpack-merge 开发环境、生产环境分离打包
- html-wepback-plugin 打包 html 文件的依赖包

OK，下载完成，接下来我们新建 3 个文件
**webpack.config.js、webpack.config.dev.js、webpack.config.pro.js**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d6184ae64fb46568556583c348d3a7e~tplv-k3u1fbpfcp-zoom-1.image)

这里简单说明下，如果不理解可以看笔者最近的一篇文章[Webpack 分环境打包（生产/开发两套打包）](https://blog.csdn.net/m0_46995864/article/details/125532571?spm=1001.2014.3001.5502)

- webpack.config.js 公共配置项
- webpack.config.dev.js 开发环境的配置项
- webpack.config.pro.js 生产环境的配置项

## 配置 npm 命令

这里配置一下生产环境和开发环境的打包命令。

```typescript
scripts: {
 	"dev": "webpack-dev-server --mode=development --config webpack.config.dev.js",
    "build": "npx webpack --mode=production --config webpack.config.pro.js"
}
```

## 配置 webpack.config.js

配置项：

```typescript
const path = require("path")
const { VueLoaderPlugin } = require("vue-loader")
const HtmlWebpackPlugin = require("html-webpack-plugin")
const { CleanWebpackPlugin } = require("clean-webpack-plugin")

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "bundle.js",
  },
  resolve: {
    extensions: [".js", ".vue"],
    alias: {
      "@": path.resolve(__dirname, "src"),
    },
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        use: ["vue-loader"],
      },
      {
        test: /\.(png|jpe?g|gif|svg|webp|ico)$/,
        type: "asset/resource",
      },
      {
        test: /\/js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
            cacheDirectory: true,
          },
        },
      },
    ],
  },
  plugins: [
    new VueLoaderPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html", // 这是html模板存放的地址
      filename: "index.html",
    }),
    new CleanWebpackPlugin(),
  ],
}
```

简单说明下，首先安装公共配置的依赖包：

```jypescript
npm i --save-dev vue-loader clean-webpack-plugin babel-loader @babel/preset-env
```

- entry、output 分别配置了项目的打包入口和打包出口，这个是公共配置项，毋庸置疑；
- resolve 配置了项目中的便捷路径转义写法，这里就先写一个常用的@；
- loader 中配置了 vue-loader、静态资源、babel 三个预处理器进行文件处理；
- plugins 中配置了 vue 的插件、html 模板编译、打包清空目录三个插件；

这些都是生产、开发环境不会改变的固定配置。

## 配置 webpack.config.dev.js

接下来进行开发环境的打包配置：

```typescript
const path = require("path")
const { merge } = require("webpack-merge")
const baseConfig = require("./webpack.config.js")

module.exports = merge(baseConfig, {
  module: {
    rules: [
      {
        test: /\.css|scss|sass$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
    ],
  },
  devServer: {
    open: true,
    host: "127.0.0.1",
    port: 8080,
    client: {
      logging: "none",
    },
    hot: true,
    historyApiFallback: true,
  },
  mode: "development",
  devtool: "inline-source-map",
})
```

这里开发环境比较简单，主要就是 devServer 本地服务器的配置，可以让我们方便的开发，以及样式的预处理器，由于开发中不需要进行压缩，因此选择 style-loader 即可。

最后通过 webpack-merge 插件将当前配置和 webpack.config.js 配置合并。

**开发配置中所需的依赖包：**

```typescript
npm i --save-dev style-loader css-loader sass-loader sass
```

## 配置 webpack.config.pro.js

接下来进行生产环境的配置：

```typescript
const path = require("path")
const { merge } = require("webpack-merge")
const baseConfig = require("./webpack.config.js")
const TerserPlugin = require("terser-webpack-plugin")
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
const CopyPlugin = require("copy-webpack-plugin")

module.exports = merge(baseConfig, {
  module: {
    rules: [
      {
        test: /\.css|scss|sass$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"],
      },
    ],
  },
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, "public", "favicon.ico"),
          to: path.resolve(__dirname, "dist/image/"),
        },
      ],
    }),
  ],
  optimization: {
    usedExports: true,
    minimize: true,
    minimizer: [
      new TerserPlugin(),
      new MiniCssExtractPlugin({
        filename: "index-[contenthash:8].css",
        chunkFilename: "[id].css",
      }),
    ],
  },
  cache: {
    type: "filesystem",
  },
  mode: "production",
  devtool: "cheap-module-source-map",
})
```

生产环境的打包因注重优化，如打包文件体积、打包速度，因此都是一些优化配置项。
先说一下生产环境的依赖包把。

```typescript
npm i --save-dev mini-css-extract-plugin copy-webpack-plugin
```

配置项介绍：

- 样式预处理器采用 **mini-css-extract-plugin** 将所有样式抽离成一行；
- **copy-webpack-plugin** 将一些静态资源直接转移至 dist 目录，这里用 Vue ico 举例；
- **optimization** 进行了代码压缩，包括 tree-shaking、js 压缩、css 压缩；
- **cache** 开启文件缓存，可以让我们的打包速度飞跃性的提升；

## 产物

至此，我们的所有 webpack 配置就结束了，展示一下产物吧。

**生产环境打包：**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f24012ef3ad3460a896736efbd2c96e7~tplv-k3u1fbpfcp-zoom-1.image)
**开发环境页面：**
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b509cb4b27ef47b1ac1e35ed5ccd53f9~tplv-k3u1fbpfcp-zoom-1.image)

# Vue 全家桶加入

## Vue Router

首先下载依赖包：

```typescript
npm i vue-router --save-dev
```

接下来在 src 目录下新建 router 文件夹，并创建 index.js、routes.js。

**index.js:**

```typescript
import { createRouter, createWebHistory } from "vue-router"
import routes from "./routes" //导入router目录下的router.js

const router = createRouter({
  history: createWebHistory(), //路由模式
  routes,
})

export default router //导出
```

**routes.js:**

```typescript
const routes = [
  {
    name: "a",
    path: "/a",
    component: () => import("@/view/A"),
  },
  {
    name: "b",
    path: "/b",
    component: () => import("@/view/B"),
  },
]
export default routes
```

在 src 目录下创建 view 目录，并创建 A.vue、B.vue 用于测试路由跳转。

**A.vue:**

```typescript
<template>
  <div>
    A
  </div>
</template>

<script setup>
import { onMounted } from "vue"

onMounted(() => {
  console.log("A页面渲染了")
})
</script>

<style lang="scss" scoped></style>

```

测试：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e094d44a1e14ec4b200b22110734e6a~tplv-k3u1fbpfcp-zoom-1.image)

## Vuex

下载依赖包：

```typescript
npm i --save-dev vuex
```

在 src 目录下创建 store.js 并写入：

```typescript
// VueX 创建了一个全局唯一的仓库，用来存放全局的数据
import { createStore } from "vuex"

export default createStore({
  state: {
    name: "John",
  },
  getters: {},
  mutations: {
    changeName(state, value) {
      state.name = value
    },
  },
  actions: {},
  modules: {},
})
```

接下来还是使用 A.vue 进行测试，A.vue 文件修改为如下：

```typescript
<template>
  <div>
    A
    <span @click="changeName">{{ store.state.name }}</span>
  </div>
</template>

<script setup>
import { onMounted } from "vue"
import { useStore } from "vuex"

let store = useStore()
onMounted(() => {
  console.log("A页面渲染了")
})
const changeName = () => {
  store.commit("changeName", "Mike")
}
</script>

<style lang="scss" scoped></style>

```

测试：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db9a8f9f26f74e40a9094b2ad31bc936~tplv-k3u1fbpfcp-zoom-1.image)

## Axios

下载依赖包：

```typescript
npm i axios --save-dev
```

src 目录下新建\_utils/request.js：

```typescript
import axios from "axios"

// 创建一个 axios 实例
const service = axios.create({
  baseURL: "/api", // 所有的请求地址前缀部分
  timeout: 60000, // 请求超时时间毫秒
  withCredentials: true, // 异步请求携带cookie
  headers: {
    // 设置后端需要的传参类型
    "Content-Type": "application/json",
    token: "your token",
    "X-Requested-With": "XMLHttpRequest",
  },
})

// 添加请求拦截器
service.interceptors.request.use(
  function(config) {
    // 在发送请求之前做些什么
    return config
  },
  function(error) {
    // 对请求错误做些什么
    console.log(error)
    return Promise.reject(error)
  }
)

// 添加响应拦截器
service.interceptors.response.use(
  function(response) {
    console.log(response)
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    // dataAxios 是 axios 返回数据中的 data
    const dataAxios = response.data
    // 这个状态码是和后端约定的
    const code = dataAxios.reset
    return dataAxios
  },
  function(error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    console.log(error)
    return Promise.reject(error)
  }
)

export default service
```

至此，一个基础的 Vue 项目搭建完毕了，项目结构如图：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb8cdfca7bfe4d88b6845acc63e19351~tplv-k3u1fbpfcp-zoom-1.image)

# 总结

使用 Vue-Cli 搭建项目很方便，但是自己从 0 搭建一个完整的 Vue 项目运行环境也是巩固 Webpack 学习的一个很好的过程，项目已经上传 github 给大家学习参考。

github：[https://github.com/fengxinhhh/vue-cli](https://github.com/fengxinhhh/vue-cli)
