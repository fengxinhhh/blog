# 写在前面

React 组件库 Concis 在大量募集小伙伴一起参与开源项目开发，目前仍有很多计划需要去实施，包含 PC 端组件扩充、移动端组件开发、Concis 生态插件开发等等，具体可以看此文：

[React 组件库 Concis,寻求社区有兴趣的小伙伴加入...](https://blog.csdn.net/m0_46995864/article/details/125747677?spm=1001.2014.3001.5501)

**Concis 组件库相关链接，希望大家多多鼓励支持、多多 Star，给作者更多的动力**

[Github](https://github.com/fengxinhhh/Concis)
[线上文档](http://react-view-ui.com:92/#/)

# 需求来源

博主在开发组件库以及实际项目测试过程中，突然发现`Concis`的组件都需要去手写，而回想以前使用`antd/el`组件库时都是可以自动生成代码块的，并且移到标签中可以看到对应的一些提示信息，于是博主开始探索 Vscode 插件的开发了。

# 生成项目

首先全局安装 vscode 脚手架

```javascript
npm install -g yo generator-code
```

安装完成后在命令行输入

```javascript
yo code
```

在提示中直接选择**New Extension**，生成一个**vscode**插件工程，下面几项是**vscode**所提供单独功能开发的脚手架，可以理解成小插件，由于这里是具体化到整个`Concis`工程，因此选择第一个。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d35c5cb16a84690a08b877bba3f1676.png)

**接下来输入一些插件的命名即可生成项目，经过处理后的项目目录是这样的：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/1464d62923c641ddac1f274789e82f18.png)
这里后续会讲到自定义打包的一些配置。

# 代码补全(snippets)

代码补全功能相对简单，只需在项目中创建`snippets.json`文件，在里面配置 snippets 就可以。
有一个网站可以帮助我们快速的创建 code snippet [https://snippet-generator.app/](https://snippet-generator.app/)

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20d642ceb2344a1e94436f222f5e23da~tplv-k3u1fbpfcp-zoom-1.image)

把对应的 snippet 生成，复制到`snippets.json`中即可，就像这样：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0151666ff7b14ba6ae83441930525476~tplv-k3u1fbpfcp-zoom-1.image)

同时在**package.json**中配置`contributes`，指定插件所生效的文件，博主是 React 组件库，因此定了`js/ts/jsx/tsx`四种文件类型

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bb9b82d5cdc4a0d9532b9983ea0b291~tplv-k3u1fbpfcp-zoom-1.image)

很简单，不需要你动脑筋写一行代码，自动补全的功能实现了。

# 智能提示

提示的话就需要我们入门一下，写一写 vscode 的 api 啦，打开项目生成的`extension.ts`文件，将代码写成这样：

```javascript
import * as vscode from "vscode"

const compileFiles = [
  "react",
  "typescript",
  "javascript",
  "javascriptreact",
  "typescriptreact",
]

function providerHover(
  document: vscode.TextDocument,
  position: vscode.Position
) {}

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.languages.registerHoverProvider(compileFiles, {
      provideHover,
    })
  )
}

export function deactivate() {}
```

上面代码我们通过`vscode.languages.registerHoverProvider`注册了鼠标移入事件，并通过`compileFiles`指定了文件类型，而插件所要实现的业务功能就在`provideHover`中实现。

```javascript
function provideHover(
  document: vscode.TextDocument,
  position: vscode.Position
) {
  //移入Concis组件Dom，出现介绍
  const line = document.lineAt(position)
  let isConcisComponentDom = false
  let matchComponent = ""
  for (let i = 0; i < componentList.length; i++) {
    const component = componentList[i]
    if (line.text.includes(`<${component}`)) {
      isConcisComponentDom = true
      matchComponent = component
    }
  }
  if (isConcisComponentDom) {
    const isCN = vscode.env.language === "zh-cn"
    let componentDocPath = ""
    for (let i = 0; i < matchComponent.length; i++) {
      const str = matchComponent[i]
      if (i !== 0 && str.charCodeAt(0) >= 65 && str.charCodeAt(0) <= 90) {
        componentDocPath += "-"
      }
      componentDocPath += str
    }
    let text = isCN
      ? `查看${matchComponent}组件官方文档\n
Concis -> http://react-view-ui.com:92/#/common/${componentDocPath.toLowerCase()}`
      : `View the official documentation of the Button component\n
Concis -> http://react-view-ui.com:92/#/common/${componentDocPath.toLowerCase()}`

    return new vscode.Hover(text)
  }
}
```

接下来我们解读一下这里面的功能：

1.  首先，通过`document.lineAt`获取代码行，判断 Concis 组件关键词是否出现在代码内容`line.text`中；
2.  对满足条件的情况，获取组件线上文档地址，编辑提示的内容信息，对应代码段中的`text`；
3.  将`text`通过`new vscode.Hover`返回；

至此，代码自动补全+智能提示功能完成了，来看一下效果吧：

![请添加图片描述](https://img-blog.csdnimg.cn/15c2e88b0b084b11a5b7b7eccf918528.jpeg)

![请添加图片描述](https://img-blog.csdnimg.cn/b04fb521f9bf48139b2cac57906295d4.jpeg)

# 打包发布

## 1.第一步，全局安装 vscode 打包工具

```javascript
npm i -g vsce
```

## 2.第二步，创建 vscode 账号

首先访问 [https://login.live.com/](https://login.live.com/) 登录你的 Microsoft 账号，没有的先注册一个，然后访问： [https://aka.ms/SignupAzureDevOps](https://aka.ms/SignupAzureDevOps)，如果你从来没有使用过 Azure，那么就要先创建一个 Azure DevOps 组织，默认会创建一个以邮箱前缀为名的组织。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3a6894a281f45e9a08021883c4cf48a~tplv-k3u1fbpfcp-zoom-1.image)

## 3.第三步，创建组织令牌

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aeb5903b4dd47f68077299a05024346~tplv-k3u1fbpfcp-zoom-1.image)
点击右上角的用户设置，点击创建新的个人访问令牌。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6973b658bdd4462bb005622fbaacd1c5~tplv-k3u1fbpfcp-zoom-1.image)

这里请选择`All accessible organizations` 否则会无法部署插件。

创建完毕会出现一个 token，请手动记录下来这个 token，因为官方不会为你保存，并且这个 token 在发布时会用到。

## 4.第四步，创建 publisher

前往插件市场[https://marketplace.visualstudio.com/](https://marketplace.visualstudio.com/)创建一个`publisher`，并在`package.json`中添加
`publisher`字段，名字保持一致。

## 配置

好了，到这里你已经完成了准备工作。

博主通过`rollup`将代码打包后再使用 vscode 内置的打包发布，因此打包流程如下：

1.  rollup 打包
2.  vsce 打包
3.  vsce 发布

`rollup.config.js`配置如下：

```javascript
import typescript from "rollup-plugin-typescript2"
import clear from "rollup-plugin-clear"
import resolve from "rollup-plugin-node-resolve"
import commonjs from "rollup-plugin-commonjs"
import { terser } from "rollup-plugin-terser"
import { uglify } from "rollup-plugin-uglify"

export default {
  input: ["./src/extension.ts"],
  output: [
    {
      file: "dist/extension.js",
      format: "cjs",
      name: "cjs.js",
    },
  ],
  plugins: [
    typescript(), // 会自动读取 文件tsconfig.json配置
    clear({
      targets: ["dist"],
    }),
    resolve(),
    commonjs(),
    terser(),
    uglify(),
  ],
  external: ["react", "react-dom"],
}
```

在 package.json 中配置一下命令行，让我们的打包发布过程像如下所说的顺序：

```javascript
	"scripts": {
		"upgrade": "node ../../scripts/replace-version-in-vscode.ts",					//升级package.json中版本
		"compile": "rollup -c ./rollup.config.js",										//rollup打包
		"build": "vsce package",														//vsce打包
		"generate:readme": "node ../../scripts/vscode/generate-vscode-snippet.ts",		//生成README.md内容
		"publish": "npm run compile && npm run build && vsce publish"					//部署
	},
```

这里`upgrade`和`generate:readme`是博主针对于 Concis 组件库自己加的两个脚本，我们看一下`publish`，可以看到按上述所讲的顺序进行打包构建，这样代码层面的配置已经做完了。

执行`npm run publish`，输入之前创建保存的 token，这样就发布成功了。

![请添加图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad86f94b0702445787b73208f23c081c~tplv-k3u1fbpfcp-zoom-1.image)

看到这里，你已经掌握了开发一个`vscode`工程并打包发布的内容了，博主针对组件库的需求进行的扩展，也就是上面所提到的两个命令：

```javascript
npm run upgrade				//升级package.json中版本
npm run generate:readme		//根据组件项目目录，自动生成README.md
```

从`Concis-snippets`插件的介绍可以看到，下面有一段表格，表格介绍了每个`snippet`对应的介绍，看到这你应该想象到了吧？此时博主又开发了两个组件，但是不想手动去更新 README.md，这时就需要自定义脚本了。

## 自动更新 README.md

```javascript
//根据项目目录写入vscode-snippets README.md
const fs = require("fs-extra")
const path = require("path")

const baseTemplate = fs.readFileSync(
  path.join(__dirname, "base-template.md"),
  "utf-8"
)
const componentList = fs.readdirSync(
  path.join(__dirname, "../../packages/concis-react/src")
)

let tableTemplate = ""

componentList.forEach((f) => {
  if (f[0].charCodeAt(0) >= 65 && f[0].charCodeAt(0) <= 90) {
    //组件
    tableTemplate += `|c${f.toLowerCase()}|snippet a Concis ${f} Component|\n`
  }
})

const fileContent = `${baseTemplate}${tableTemplate}`
console.log(fileContent)

fs.writeFile(
  path.resolve(__dirname, "../../packages/concis-vscode-snippets/README.md"),
  fileContent,
  "utf-8"
).then(() => {
  console.log("生成concis-vscode-snippets 成功") // eslint-disable-line
})
```

这里我们有一个基础模板，放着`README.md`上部分一直不会变的内容，对应代码中`base-template.md`，然后读取了组件目录，将基础模板和表格内容拼接起来，就可以了。

## 自动升级 package.json 版本

```javascript
//更新concis-vscode-snippets/package.json中的版本
const fs = require("fs-extra")
const path = require("path")
const { version } = require("../packages/concis-vscode-snippets/package.json")

const fileText = fs.readFileSync(
  path.join(__dirname, "../packages/concis-vscode-snippets/package.json"),
  "utf-8"
)

const replacedVersion = fileText.split("\n").map((t) => {
  if (/"version":/.test(t)) {
    let newVersion = version.split(".")
    let [first, secord, third] = newVersion
    if (third >= 10 || secord >= 10) {
      if (third >= 10) {
        third = 0
        secord = Number(secord) + 1
      }
      if (secord >= 10) {
        third = 0
        secord = 0
        first = Number(first) + 1
      }
    } else {
      third = Number(third) + 1
    }

    newVersion[newVersion.length - 1] =
      Number(newVersion[newVersion.length - 1]) + 1
    console.log(`"version": "${first}.${secord}.${third}"`)
    return `  "version": "${first}.${secord}.${third}",`
  }
  return t
})

fs.outputFile(
  path.resolve(__dirname, "../packages/concis-vscode-snippets/package.json"),
  replacedVersion.join("\n")
).then(() => {
  console.log("替换 package.json 中的 version 成功！") // eslint-disable-line
})
```

代码实现了一个替换`package.json`文件中版本的功能，有了这两个自动脚本的支持，我们就可以把`publish`改成这样：

```javascript
"scripts": {
	"upgrade": "node ../../scripts/replace-version-in-vscode.ts",
	"compile": "rollup -c ./rollup.config.js",
	"build": "vsce package",
	"generate:readme": "node ../../scripts/vscode/generate-vscode-snippet.ts",
	"publish": "npm run upgrade && npm run generate:readme && npm run compile && npm run build && vsce publish"
}
```

# 总结

本文通过博主对于 Concis 实际案例，讲解了开发一个 vscode 插件从准备工作到功能实现最后打包发布的全过程，如果文章有用，希望多多支持。

**Concis 组件库地址**

![请添加图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8aa42937125d4a6e8c1985e59a6245b6~tplv-k3u1fbpfcp-zoom-1.image)

Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

开源不易，欢迎学习和体验，喜欢请多多支持，有问题请留言，如果此文对你有帮助，博主需要你的支持，感谢。
