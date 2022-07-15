# 前言

博主最近在开发 React 组件库——[Concis](http://react-view-ui.com:92/#/)，目前针对 PC 端已经开发了 30+组件，未来是期望可以转换出一套轻量的 Mobile 组件来支持 React 移动端项目，而博主原来的项目结构是这样的：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e89f272b747741aabbd056b02beda35f~tplv-k3u1fbpfcp-zoom-1.image)
其实这只是一个普通项目，在 src 中包含了一系列支持 PC 的 React 组件，而既然想要扩充，不妨使用 lerna 将多个子项目整合在一个大项目中。

# 重构过程

在本地全局安装 lerna

```javascript
npm i lerna -g  || yarn global add lerna
```

安装完成后执行 `lerna -v` 看下是否能够正确的输出 `lerna` 的版本号。

接下来进入到 Concis 中，在项目根目录初始化`lerna`

```javascript
lerna init
```

初始化之后会发现项目根目录多了`packages`目录和`lerna.json`

接下来在 packages 中创建两个目录，分别为 concis-react（目前的 PC 端组件库）和 concis-react-mobile（未来的移动端组件库），并将原来根目录下的 src 复制到 packages/concis-react 中，同时删除根目录下的 rollup.config.js，在 concis-react、concis-react-mobile 目录下分别创建出：

- rollup.config.js 用于分别打包子包的代码块，用于发 npm
- package.json 子包的包管理文件
- tsconfig.json TypeScript 配置文件

同时我把根目录的`README.md`复制了一份到子包目录中，为了发 npm 包时也可以出现介绍内容，因为发子包是基于 packages/xx 目录去发布的，而 github 不用这样处理，经过处理后我的项目结构是这样的：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad07eb4bc80841f5b57458b3b6fc6e95~tplv-k3u1fbpfcp-zoom-1.image)

接下来配置`package.json`文件。

concis-react/package.json:

```javascript
{
  "name": "concis",
  "version": "1.0.16",
  "description": "Concis Component library for PC",
  "authors": {
    "name": "fengxin",
    "email": "1244200081@qq.com"
  },
  "scripts": {
    "build": "rollup -c ./rollup.config.js"
  },
  "module": "es/index.js",
  "typings": "es/index.d.ts",
  "gitHooks": {
    "pre-commit": "lint-staged"
  },
  "files": [
    "web-react",
    "README.md",
    "README.zh-CN.md",
    "package.json"
  ],
  "publishConfig": {
    "registry": "https://registry.npmjs.org/"
  },
  "dependencies": {
    "@babel/plugin-transform-runtime": "^7.17.0",
    "@babel/preset-react": "^7.17.12",
    "core-js": "^3.22.2",
    "lodash": "^4.17.21"
  },
  "peerDependencies": {
    "react": "^16.8.0 || ^17.0.0 || ^18.0.0"
  },
  "devDependencies": {
    "@ant-design/icons": "^4.7.0"
  },
  "license": "MIT",
  "gitHead": "ce812c263bec669470e12af97e9c737cbc05d730"
}

```

concis-react-mobile/package.json:

```javascript
{
  "name": "concis-mobile",
  "version": "0.0.1",
  "description": "Concis Component library for Mobile",
  "authors": {
    "name": "fengxin",
    "email": "1244200081@qq.com"
  },
  "scripts": {
    "build": "rollup -c ./rollup.config.js"
  },
  "module": "es/index.js",
  "typings": "es/index.d.ts",
  "gitHooks": {
    "pre-commit": "lint-staged"
  },
  "files": [
    "web-react-mobile",
    "README.md",
    "README.zh-CN.md",
    "package.json"
  ],
  "publishConfig": {
    "registry": "https://registry.npmjs.org/"
  },
  "dependencies": {
    "@babel/plugin-transform-runtime": "^7.17.0",
    "@babel/preset-react": "^7.17.12",
    "core-js": "^3.22.2",
    "lodash": "^4.17.21"
  },
  "peerDependencies": {
    "react": "^16.8.0 || ^17.0.0 || ^18.0.0"
  },
  "devDependencies": {
    "@ant-design/icons": "^4.7.0"
  },
  "license": "MIT",
  "gitHead": "ce812c263bec669470e12af97e9c737cbc05d730"
}

```

两个子包的打包配置大同小异，配置项都一样，只需要对出口命名进行处理即可，这里贴一下`concis-react`的`rollup.config.js`

```javascript
import typescript from "rollup-plugin-typescript2"
import less from "rollup-plugin-less"
import clear from "rollup-plugin-clear"
import resolve from "rollup-plugin-node-resolve"
import commonjs from "rollup-plugin-commonjs"
import babel from "rollup-plugin-babel"
import { terser } from "rollup-plugin-terser"
import { uglify } from "rollup-plugin-uglify"
import copy from "rollup-plugin-copy"

export default {
  input: ["./src/index.ts"],
  output: [
    {
      file: "web-react-mobile/cjs.js",
      format: "cjs",
      name: "cjs.js",
    },
    {
      file: "web-react-mobile/umd.js",
      format: "umd",
      name: "umd.js",
    },
    {
      file: "web-react-mobile/index.js",
      format: "es",
      name: "index.js",
    },
  ],
  plugins: [
    typescript(),
    less({ output: "./web-react-mobile/style/index.css" }),
    clear({
      targets: ["web-react"],
    }),
    resolve(),
    commonjs(),
    babel({
      exclude: "node_modules/**",
      runtimeHelpers: true,
    }),
    terser(),
    uglify(),
    copy({
      targets: [
        {
          src: "../../scripts/globalStyle/compiled-colors.less",
          dest: "web-react/style",
        },
      ],
    }),
  ],
  external: ["react", "react-dom"],
}
```

`tsconfig.json`的处理则是继承根目录下的配置去做额外配置，有点类似`webpack-merge`的处理。

主配置:

```javascript
{
  "compilerOptions": {
    "target": "es6",
    "module": "es2015",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node", //node环境
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"],
      "@/": ["./packages"]
    },
    "experimentalDecorators": true
  },
  "include": [
    "typings.d.ts", //配置的.d.ts文件
    "docs/guide"
  ],
  "exclude": ["node_modules", "lib"]
}

```

子包:

```javascript
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "rootDir": "src",
    "include": [
      "src",
      "../../__tests__/concis-react",
      ]
  }
}

```

# 发包

`lerna`有多种发包方式，这里我选择了根据各个子包的`package.json`版本号去发，这里有两个坑需要注意一下：

- 发包前，需要`commit`项目中的代码
- 需要手动更新`package.json`中的版本号，否则不会被检测到更新信息

注意这两点以后，在项目根目录执行

```javascript
lerna publish from-package
```

出现提示后确认，即可发包，这里博主的发包结果：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/663d01225b87499db4353b2c18ebeac6~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06b0af8acbb84e8482c349fedc1231a5~tplv-k3u1fbpfcp-zoom-1.image)

就这样，组件库的项目结构重构完毕了，也更加清晰了，未来的阶段更多的就是在维护`packages/concis-react`的同时更新`packages/concis-react-mobile`中的组件。

重构后的项目结构如下：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8670f2c7a87d4f9e8f04d5b0970ca348~tplv-k3u1fbpfcp-zoom-1.image)

# Concis 的地址

Concis 已经开发了接近半年时间，也是越来越成熟了，这里留一下 Concis 的一些 Path：

Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

开源不易，如果文章内容对你有帮助，请支持一下，非常感谢。
