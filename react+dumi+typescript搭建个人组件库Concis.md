## dumi为我们创建个人组件库提供了很好的平台，开箱即用，可以把专注度放在组件业务部分的编写上。
### 搭建步骤：
## 1.创建文件夹并初始化脚手架

```javascript
mkdir my-app
cd my-app
$ npx @umijs/create-dumi-lib        # 初始化一个文档模式的组件库开发脚手架
# or
$ yarn create @umijs/dumi-lib

$ npx @umijs/create-dumi-lib --site # 初始化一个站点模式的组件库开发脚手架
# or
$ yarn create @umijs/dumi-lib --site
```
## 2.安装项目依赖并运行

```javascript
yarn add -d dumi
npm start
```
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c2e6304b81e4e47a425bf9afc181604~tplv-k3u1fbpfcp-zoom-1.image)

 如图为博主个人修改好的组件库主页，进入组件库文档页面也可以根据脚手架内部的配置文件route.ts、tsconfig.json、umirc.ts来配置文档的目录、菜单，具体可以参照官网配置。
 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d55efe350c9143ba8577961ddd3c2368~tplv-k3u1fbpfcp-zoom-1.image)
如图是博主所写好的第一个Button组件

## 3.线上部署
采用官方推荐gh-pages进行线上部署。

```javascript
yarn add -d gh-pages
```

安装依赖后，在package.json中添加部署代码：

```javascript
"deploy": "gh-pages -d docs-dist"
```
接下来打包+部署

```javascript
npm run docs:build
npm run deploy
```
出现"pubilshed"说明打包后的分支代码提交到git仓库中了，去github的仓库中将gh-pages分支的代码部署到线上即可，但是会出现一个问题，任何页面刷新后都会变成404，这是因为默认只输出一个 index.html 作为入口 HTML 文件，服务器在 serve / 时可以找到文件但 /some-route 却没有对应的 /some-route/index.html，所以会出现 404。设置 config.exportStatic 为 {} 根据路由按文件夹结构输出所有 HTML 文件即可。
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd6ec8fc604f4b248c87f1bc4b7bd97f~tplv-k3u1fbpfcp-zoom-1.image)


具体配置如下：
.umirc.ts加入代码片段

```javascript
exportStatic: {
    htmlSuffix: true
  },
  history: {
    type: "hash"
  }
```
再次最早的操作，组件库刷新不会出现问题了。

以上就是ums.js+react+typescript搭建个人组件库的完整流程，也留下我的线上地址和github地址：

- Concis组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
- github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
- npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

好了好了~都看到这里了，不管有没有成功帮忙点个赞，github点个星星吧哈哈哈，后续也会不断发出每一个组件的代码和文档，敬请关注！