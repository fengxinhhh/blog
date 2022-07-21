![请添加图片描述](https://img-blog.csdnimg.cn/779600a4d17241a1b857e2438f6cda2c.jpeg)

# 写在前面

自上文发布以来，作者收到了很多的私信，也很高兴。

[组件库设计 | React 组件库 Concis 开源探索过程中的一些心路历程](https://blog.csdn.net/m0_46995864/article/details/125863812?spm=1001.2014.3001.5501)

大致内容有都是对作者的鼓励，心中十分感激，同时 Concis 收到了很多的 pr，贡献者也突破了 5 个...至少作者不会写着写着突然有一种迷迷糊糊很迷茫的感觉了，这点非常开心。

作者不断的在思考下一步 Concis 该做些什么，简单参考了一些成功的项目，最终花了三天时间做了暗黑模式。

# 组件库相关

Concis 已开工半年时间，开源免费，欢迎大家体验、一起折腾。你可以通过以下方式来支持作者。

1. 给 Concis 点 star，证明作者花的时间和汗水的值得的。
2. 给 Concis 发 pr 提 issue，修改或提出你认为不足不好的地方，帮助 Concis 成长。

其实作者刚开始一段时间是比较累的，因为并没有什么关注，同时是一个人，最近一段时间其实好很多了，因为社区的力量以及 Concis 不断完善，受到了一些支持，也给作者加了一把油。

[Github](https://github.com/fengxinhhh/Concis)

[线上文档](http://react-view-ui.com:92/#/)

# 暗黑模式

什么是暗黑模式？在我们翻阅一些网页的时候，白色的主色调会让人觉得刺眼，尤其是在夜晚的时候，这时候暗黑模式就有作用了。它可以大大的增加可读性，减轻眼睛的疲劳感。并且暗色的主题总感觉会有一种很高端的感觉，如果你经常喜欢逛 github，那当暗黑模式肯定是再熟悉不过了...

**先看一下 Concis 暗黑模式的效果吧。**

天黑，请闭眼。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7d721455ccac4b53af8c885347f08fc3.png)

天亮，请打工。

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-zIjO1tZO-1658420915832)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3238f935337a4cc9897ff65a55c1cf72~tplv-k3u1fbpfcp-watermark.image?)]](https://img-blog.csdnimg.cn/c6b60d575f3f4d239d0beeafe347459b.png)

如果你不喜欢 Concis 内置的主题色，你可以将暗黑模式和自定义主题结合，就像这样：

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-AqBP9crD-1658420915832)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ce0f26749d3443d99e6d04000ec0355~tplv-k3u1fbpfcp-watermark.image?)]](https://img-blog.csdnimg.cn/35405590c9ed4c7dbb33909b12598c0c.png)

对于优先级，自定义主题色的优先级会高于暗黑主题的颜色，当然你可以通过全局 className，组件局部 className，来深度自定化一些自己所需要的业务样式。

自定义主题实现文章可参考：

[五分钟告诉你 React 全局配置项目主题色是怎么实现的](https://juejin.cn/post/7119202163905003534)

# 配置方法

在下载配置完成 Concis 的情况下：

```js
import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"
import { GlobalConfig } from "concis/web-react"

const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <React.StrictMode>
    <GlobalConfig darkTheme globalColor="orange">
      <App />
    </GlobalConfig>
  </React.StrictMode>
)
```

其中`darkTheme`代表开启暗黑模式，`globalColor`代表自定义主题色。

# 推荐样式注入

Concis 内置了贴合暗黑模式的项目背景色和文本色：

```js
@import 'concis/web-react/style/compiled-colors.less';
@import 'concis/web-react/style/index.css';


.App {
  color: @dark-theme-text-color;
  background: @dark-theme-background;
}
```

# 实现思路

Concis 依靠主题前缀类名控制组件的模式，如`concis-dark-button`代表暗黑模式按钮，`concis-button`代表普通模式按钮，给每一款组件提供两份主题样式即可，主要的工程还是在自定义主题色那里去完成的，因为子组件需要借助`GlobalConfig`的传参，才可以去判断主题。

# 结束语

希望这次暗黑模式的特性可以让大家眼前一亮，最近作者也是一个人在开发暗黑模式同时去 review 多个小伙伴的 pr，顺便改一下小伙伴代码中不足的地方，后期考虑继续完善组件库，因为目前组件数量还不是很多（40+），也希望大家可以给 Concis 一些关注和支持。

- Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
- github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
- npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

开源不易，如果文章内容对你有帮助，请支持一下，非常感谢。
