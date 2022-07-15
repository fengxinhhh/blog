## 前段时间一直在更新 react 组件库的每一个组件，今天来测试一下在实际业务中使用组件库~

## 我使用了 rollup 来打包组件库

## 1.搭建库打包脚手架：

首先我们安装一下 rollup：

```javascript
npm i rollup -g
```

我的组件库项目目录是这样的，所有组件都在 index.ts 中暴露出去
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1a4541a1e634dfc9e656dc5449f84a9~tplv-k3u1fbpfcp-zoom-1.image)
代码写好以后，最基础的 rollup.config.js 版本可以这样配置：

```javascript
// rollup.config.js
export default {
  input: "src/index.ts",
  output: {
    file: "cjs.js",
    format: "cjs",
  },
}
```

在终端执行：

```javascript
rollup - c
```

即可完成打包

## 2.rollup 插件使用

为了更灵活的打包库文件，我们可以配置 rollup 插件，比较实用的插件有： rollup-plugin-node-resolve ---帮助 Rollup 查找外部模块，然后导入 rollup-plugin-commonjs ---将 CommonJS 模块转换为 ES2015 供 Rollup 处理 rollup-plugin-babel --- 让我们可以使用 es6 新特性来编写代码 rollup-plugin-terser --- 压缩 js 代码，包括 es6 代码压缩 \* rollup-plugin-eslint --- js 代码检测

打包一个库用以上插件完全够用了，不过如果想实现对 react 等组件的代码，可以有更多的插件可以使用，这里就不一一介绍了。

我们可以这样使用，类似于 webpack 的 plugin 配置：

```javascript
import resolve from "rollup-plugin-node-resolve"
import commonjs from "rollup-plugin-commonjs"
import babel from "rollup-plugin-babel"
import { terser } from "rollup-plugin-terser"
import { eslint } from "rollup-plugin-eslint"

export default [
  {
    input: "src/main.js",
    output: {
      name: "timeout",
      file: "/lib/tool.js",
      format: "umd",
    },
    plugins: [
      resolve(), // 这样 Rollup 能找到 `ms`
      commonjs(), // 这样 Rollup 能转换 `ms` 为一个ES模块
      eslint(),
      babel(),
      terser(),
    ],
  },
]
```

其实看起来和 webpack 差不多，只是在 rollup 中 plugin 变成了一个个函数的调用，看起来更加清晰。

## 3.利用 babel 来编译 es6 代码

首先我们先安装 babel 相关模块：

```javascript
npm i core-js @babel/core @babel/preset-env @babel/plugin-transform-runtime
```

然后设置.babelrc 文件

```javascript
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules": false,
        "useBuiltIns": "usage",
        "corejs": "2.6.10",
        "targets": {
          "ie": 10
        }
      }
    ]
  ],
  "plugins": [
      // 解决多个地方使用相同代码导致打包重复的问题
      ["@babel/plugin-transform-runtime"]
  ],
  "ignore": [
      "node_modules/**"
    ]
}
```

@babel/preset-env 可以根据配置的目标浏览器或者运行环境来自动将 ES2015+的代码转换为 es5。需要注意的是，我们设置"modules": false，否则 Babel 会在 Rollup 有机会做处理之前，将我们的模块转成 CommonJS，导致 Rollup 的一些处理失败。

为了解决多个地方使用相同代码导致打包重复的问题，我们需要在.babelrc 的 plugins 里配置@babel/plugin-transform-runtime，同时我们需要修改 rollup 的配置文件：

```javascript
babel({
  exclude: 'node_modules/**', // 防止打包node_modules下的文件
  runtimeHelpers: true,       // 使plugin-transform-runtime生效
}),
```

## 4. external 属性

使用 rollup 打包，我们在自己的库中需要使用第三方库，例如 lodash 等，又不想在最终生成的打包文件中出现 jquery。这个时候我们就需要使用 external 属性。比如我们使用了 lodash，

```javascript
import _ from 'lodash'

// rollup.config.js
{
    input: 'src/main.js',
    external: ['lodash'],
    globals: {
        lodash: '_'
    },
    output: [
    { file: pkg.main, format: 'cjs' },
    { file: pkg.module, format: 'es' }
    ]
}
```

## 6.导出模式

我们可以将自己的代码导出成 commonjs 模块，es 模块，以及浏览器能识别的模块，通过如下方式设置：

```javascript
{
  input: 'src/main.js',
  external: ['ms'],
  output: [
    { file: pkg.main, format: 'cjs' },
    { file: pkg.module, format: 'es' },
    { file: pkg.module, format: 'umd' }
  ]
}
```

## 7.发布到 npm

你可以通过如下方式配置 package.json：

```javascript
{
  "name": "react-view-ui",
  "version": "1.0.1",
  "description": "ui package",
  "main": "dist/tool.cjs.js",
  "module": "dist/tool.esm.js",
  "browser": "dist/tool.umd.js",
  "author": "xin_feng",
  "dependencies": {
    // ...
  },
  "devDependencies": {
    // ...
  },
  "scripts": {
    "build": "NODE_ENV=production rollup -c",
    "dev": "rollup -c -w",
    "test": "node test/test.js",
    "pretest": "npm run build"
  },
  "files": [
    "dist"
  ]
}
```

dist 为上传到 npm 库的指定目录，我们将打包后的 dist 目录传上去即可。

## 8.业务中使用

我们可以在实际项目中通过：

```javascript
yarn add react-view-ui | npm i react-view-ui
```

来安装 UI 库，安装完成后，在项目中通过：

```javascript
//jsx中:
import {
  Button,
  Radio,
  RadioGroup,
  Divider,
  Layout,
  Slider,
  Menu,
  Header,
  Content,
  Footer,
  Pagination,
} from "react-view-ui/dist/my-lib-esm"
//App.jsx根目录中引入css样式
import "react-view-ui/dist/index.css"
```

我是把之前写的 8 个组件都试了一下，项目运行界面是这样的：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69f40f1b3ec7491f95dc0d7d35ea3286~tplv-k3u1fbpfcp-zoom-1.image)

实际使用方式在文档中有写，可参考：[react-view-ui 官方文档](http://124.222.161.174:8080/#/)

```javascript
import "./App.css"
import {
  Button,
  Radio,
  RadioGroup,
  Divider,
  Layout,
  Slider,
  Menu,
  Header,
  Content,
  Footer,
  Pagination,
} from "react-view-ui/dist/my-lib-esm"
import { useEffect } from "react"
import {
  AppstoreOutlined,
  MailOutlined,
  SettingOutlined,
} from "@ant-design/icons"
import Hello from "./component/hello"

function App() {
  const options = [10, 20, 30, 50]
  const handleClick = () => {
    console.log(1)
  }
  const handleRouteChange = (a) => {
    console.log(a)
  }
  const getItem = (label, key, level, icon, children) => {
    return {
      label,
      key,
      level,
      icon,
      children,
    }
  }
  const items = [
    getItem("Navigation One", "sub1", 1, null, [
      getItem("Option 1", "1", 2),
      getItem("Option 2", "2", 2),
      getItem("Option 3", "3", 2),
      getItem("Option 4", "4", 2),
    ]),
    getItem("Navigation Two", "sub2", 1, null, [
      getItem("Option 5", "5", 2),
      getItem("Option 6", "6", 2, null, [
        getItem("Option 13", "13", 3),
        getItem("Option 14", "14", 3),
      ]),
      getItem("Submenu", "sub3", 2, null, [
        getItem("Option 15", "15", 3),
        getItem("Option 16", "16", 3),
      ]),
    ]),
    getItem("Navigation Three", "sub4", 1, null, [
      getItem("Option 9", "9", 2),
      getItem("Option 10", "10", 2),
      getItem("Option 11", "11", 2),
      getItem("Option 12", "12", 2),
    ]),
  ]

  return (
    <div className="App" style={{ marginLeft: "20px" }}>
      <Button handleClick={handleClick}>123</Button>
      <RadioGroup value={0} canAddOption boxStyle>
        <Radio>Apple</Radio>
        <Radio>Orange</Radio>
        <Radio>Watch</Radio>
      </RadioGroup>
      <Layout>
        <Header>header</Header>
        <Layout>
          <Slider
            row={3}
            extraStyle={{ height: "100%", padding: "0 0 50px 0" }}
          >
            <Menu
              items={items}
              handleRouteChange={handleRouteChange}
              defaultOpen
            />
          </Slider>
          <Content row={7}>content</Content>
        </Layout>
        <Footer>footer</Footer>
      </Layout>
      <Pagination
        total={200}
        showSizeChanger
        pageSizeOptions={options}
        showJumpInput
      />
      <div style={{ marginTop: "30px" }}>
        <Divider>文档分割</Divider>
      </div>
    </div>
  )
}
export default App
```

这样以后就可以在项目中使用自己喜欢风格的 react 组件啦~

- Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
- github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
- npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)
