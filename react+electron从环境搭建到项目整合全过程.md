## Electron 是什么

​Electron 是一个由 GitHub 开发的开源库，通过将 Chromium) 和 Node.js 组合并使用 HTML，CSS 和 JavaScript 进行构建 Mac，Windows，和 Linux 跨平台桌面应用程序。

**原理：**
上面已将说了，Electron 通过将 Chromium 和 Node.js 组合到单个 runtime 中来实现的.

**nodejs:**

如果你不知道 node.js，那还等什么快戳这里，看一看世界上最温柔可爱的语言。它借助于 Google 的 V8 引擎，Node.js 是一个能够在服务器端运行 JavaScript 的开放源代码、跨平台 JavaScript 运行环境，更多解释请戳维基百科。

**Chromium:**

Chromium 或许你没听说过，但是你一定听说过 chrome 吧！Chromium 是 Google 的开源浏览器，是 chrome 背后的那个不太稳定更新快的兄弟版，详情戳这里。

**组合:**

Electron 创建的应用使用网页作为其 GUI ,因此你可以将其当成由 JavaScript 控制的迷你精简版 Chromium 浏览器。也可以将 Electron 当成 node.js 变体，只不过它更专注于桌面应用而非 Web 服务器。在 Electron 中, 把 package.json 中设定的 main 脚本的所在进程称为 主进程。这个进程中运行的脚本也可通过创建网页这种方式来展现其 GUI。 因为 Electron 是通过 Chromium 来显示页面,所以 Chromium 自带的多进程架构也一同被利用。这样每个页面都运行着一个独立的进程,它们被统称为 渲染进程。通常来说,浏览器中的网页会被限制在沙盒环境中运行并且不允许访问系统原生资源。但是由于 Eelectron 用户可在页面中调用 Node.js API，所以可以和底层操作系统直接交互。

**优缺点？**
总之，优点肯定大于缺点。

**优点:**

方便快捷的开发桌面应用，跨平台，对前端开发者友好，活跃的社区，丰富的 api......

**缺点:**

性能肯定比不上原生的桌面应用，发布的包貌似有一点点大。

OK,接下来开始.........................................................

## 一、快速创建 react 项目

首先安装好 GIT 和 nodejs,安装好 nodejs 同时也安装好了 npm

这是使用 Facebook 开发的 reate-react-app 来快速创建一个 react 项目（命名为 react-electron）。

```javascript
# 安装 create-react-app 命令,如果已将安装请忽略
npm install -g create-react-app
# 创建 react项目
create-react-app react-electron
# 启动项目( create-react-app 真的超级方便啊)
cd react-electron && npm start

```

**相关配置**
react-electron 根目录(不是 src 目录)下面新建 main.js 文件,这个文件和 electron-quick-start 中的官方默认 main.js 几乎一模一样，只修改了加载应用这入口这一个地方：

```javascript
// 引入electron并创建一个Browserwindow
const { app, BrowserWindow } = require("electron")
const path = require("path")
const url = require("url")

// 保持window对象的全局引用,避免JavaScript对象被垃圾回收时,窗口被自动关闭.
let mainWindow

function createWindow() {
  //创建浏览器窗口,宽高自定义具体大小你开心就好
  mainWindow = new BrowserWindow({ width: 800, height: 600 })

  /* 
   * 加载应用-----  electron-quick-start中默认的加载入口
    mainWindow.loadURL(url.format({
      pathname: path.join(__dirname, './build/index.html'),
      protocol: 'file:',
      slashes: true
    }))
  */
  // 加载应用----适用于 react 项目
  mainWindow.loadURL("http://localhost:3000/")

  // 打开开发者工具，默认不打开
  // mainWindow.webContents.openDevTools()

  // 关闭window时触发下列事件.
  mainWindow.on("closed", function() {
    mainWindow = null
  })
}

// 当 Electron 完成初始化并准备创建浏览器窗口时调用此方法
app.on("ready", createWindow)

// 所有窗口关闭时退出应用.
app.on("window-all-closed", function() {
  // macOS中除非用户按下 `Cmd + Q` 显式退出,否则应用与菜单栏始终处于活动状态.
  if (process.platform !== "darwin") {
    app.quit()
  }
})

app.on("activate", function() {
  // macOS中点击Dock图标时没有已打开的其余应用窗口时,则通常在应用中重建一个窗口
  if (mainWindow === null) {
    createWindow()
  }
})

// 你可以在这个脚本中续写或者使用require引入独立的js文件.
```

**配置 package.json**

```javascript
{
  "name": "knownsec-fed",
  "version": "0.1.0",
  "private": true,
  "main": "main.js", // 这里 配置启动文件
  "homepage":".", // 这里
  "dependencies": {
    "electron": "^1.7.10",
    "react": "^16.2.0",
    "react-dom": "^16.2.0",
    "react-scripts": "1.1.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "electron-start": "electron ." // 这里 配置electron的start，区别于web端的start
  }
}

```

**启动 Electron**

```javascript
npm start	//启动react应用
electron .		//启动electron
```

其实很明显，项目的基本构成还是基于 react 来进行开发的，只是在原有的 chroms 基础下换做了 electron 进行开发，当然也可以在 main.js 中配置 electron-reloader 进行项目热更新的配置。

```javascript
npm i electron-reloader
//热加载
try {
    require('electron-reloader')(module)
} catch (_) {}
```

支持热调试，当你修改代码后，桌面应用也将会重新更新。

## react 路由配置与主进程配合通信

像默认 react 的项目是通过浏览器 url 进行页面的切换，如果在 electron 中想要达到点击某个按钮打开一个渲染进程并且加载指定的路由首先需要配置 react 路由，这里我以三个路由进行示例：

route.js 路由文件

```javascript
import Menu from "./components/Menu"
import Shopping from "./components/Shopping"
import Footer from "./components/Footer"

const routes = [
  {
    name: "menu",
    path: "/menu",
    component: Menu,
  },
  {
    name: "shopping",
    path: "/shopping",
    component: Shopping,
  },
  {
    name: "footer",
    path: "/footer",
    component: Footer,
  },
]
export default routes
```

在 App.js 中引入并配置

```javascript
return (
  <BrowserRouter>
    <h1 onClick={openRender}>首页</h1>
    <div className="App">
      <Link to="/menu">Menu</Link>
      <Link to="/shopping">Shopping</Link>
      <Routes>
        {routes.map((item, key) => {
          return (
            <Route
              key={key}
              path={item.path}
              element={<item.component />}
            ></Route>
          )
        })}
      </Routes>
    </div>
  </BrowserRouter>
)
```

现在有一个 button，点击后希望开启一个渲染进程并且加载 Footer 组件，因此需要在 App.js 中使用 ipcRenderer 进行发布订阅发射事件。

```javascript
import "./App.css"
import { Route, Routes, Link, BrowserRouter } from "react-router-dom"
import routes from "./Route"
const { ipcRenderer } = window.require("electron") //引入electron模块

function App() {
  //触发点击事件，发射给electron主进程
  const openRender = () => {
    ipcRenderer.send("openRender", {
      width: 400,
      height: 400,
    })
  }
  return (
    <BrowserRouter>
      <h1 onClick={openRender}>首页</h1>
      <div className="App">
        <Link to="/menu">Menu</Link>
        <Link to="/shopping">Shopping</Link>
        <Routes>
          {routes.map((item, key) => {
            return (
              <Route
                key={key}
                path={item.path}
                element={<item.component />}
              ></Route>
            )
          })}
        </Routes>
      </div>
    </BrowserRouter>
  )
}

export default App
```

在主进程 main.js 中接收并开启渲染进程

```javascript
ipcMain.on("openRender", (event, avg) => {
  let renderWindow = null
  const { width, height } = avg
  renderWindow = new BrowserWindow({
    width,
    height,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false,
    },
  })
  renderWindow.webContents.loadURL("http://localhost:3000/footer")
})
```

这样，就实现了 react 和 electron 的交互了，开启了一个渲染进程同时加载了 footer 路由

## 三、打包 react 项目

首先修改 main.js, 因为现在你要将 react 项目打包在 build 文件夹下面，所以加载应用处改成如下！当然也可在某个配置文件里面配置是否属于开发，此处用 if 判断一下从未进行选择执行哪段加载应用代码。但是这里为了简便，暂且使用直接修改的方式：

```javascript
// 加载应用----react 打包
mainWindow.loadURL(
  url.format({
    pathname: path.join(__dirname, "./build/index.html"),
    protocol: "file:",
    slashes: true,
  })
)
// 加载应用----适用于 react 开发时项目
// mainWindow.loadURL('http://localhost:3000/');
```

默认情况下，homepage 是 http://localhost:3000，build 后，所有资源文件路径都是 /static，而 Electron 调用的入口是 file :协议，/static 就会定位到根目录去，所以找不到静态文件。在 package.json 文件中添加 homepage 字段并设置为"."后，就可以开始打包了。

```javascript
npm run-script build
```

## 四、打包 electron

常用打包插件
安装 electron-packager

```javascript
# knownsec-fed目录下安装electron-packager包
npm install electron-packager --save-dev
# 安装electron-packager命令
npm install electron-packager -g
```

打包命令：

```javascript
electron-packager <location of project> <name of project> <platform> <architecture> <electron version> <optional options>
```

location of project: 项目的本地地址，此处我这边是 ~/knownsec-fed
location of project: 项目名称，此处是 knownsec-fed
platform: 打包成的平台
architecture: 使用 x86 还是 x64 还是两个架构都用
electron version: electron 的版本

于是，根据我这边的情况在 package.json 文件的在 scripts 中加上如下代码：

```javascript
"package": "electron-packager /home/react-electron react-electron --all --out ~/ --electron-version 2.0.6"
```

开始打包：

```javascript
npm run-script package
```

## 五、结束语

因为工作的原因需要用到 electron。通过一周时间的研究和学习，可以得出 electron 确实是一个跨平台开发应用的利器，通过 web 开发就能实现 wirte once, run every where 的理念。很棒。

在 electron 里面可以调用 nodejs 几乎所有的功能，当然前提是需要 require nodejs 的包；
在 react 的 js 页面或者公司项目用到的 Ant Design 的一些 js 页面需要用到 electron 时候，通过官方的

```javascript
const electron = require("electron")
```

语句并不能成功引入，此时需要通过

```javascript
const electron = window.require("electron")
```

引入；

还有，最最最重要的一点！！！！开发时候一般都是在 main 中通过 react 项目的 URL 去热调试应用，BUT！！此时请在 electron 生成的窗口中进行调试！！如果只在浏览器的页面查看效果，会提示 electron 的模块无法导入，无论你用啥方法！

最后，再提一点自己的感触。使用 nodejs 的 fs 包和 electron 的 dialog、app 类能够首先调用不同平台的文件选择器和一些特殊文件夹的的功能，比如说桌面、用户默认数据文件夹的修改。这里不多作描述了。～～

本文部分引入此篇博客的观点，谢谢作者的慷慨总结！！
