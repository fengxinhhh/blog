# 写在前面

本文可能无法从细节层面教会你如何做好一个开源组件库，作者也在不断探索和学习，但是也许会对你有所启发。这篇文章既是分享，也是记录，在写这篇文章的此刻，已经是作者一拍脑袋要做一个开源项目将近半年时间了。半年前作者对于如何开发一个组件库一无所知，对于开源项目也是了解甚少，抱着什么不会学什么的态度，独自一人踏上了开源之旅。

## Concis 组件库相关链接，希望多多鼓励和支持

[Github](https://github.com/fengxinhhh/Concis)

[线上文档](http://react-view-ui.com:92/#/)

## 为什么要做一套组件库？

`Concis`在今年三月底完成了第一行代码，在这第一行代码写下之前，作者参考了市面上成熟的组件库，尝试理解其中的架构思维和设计模式，如`antd-design` `element-ui` `element-plus` `arco-design` 而作者想要做一套组件库的原因其实也很单纯简单，工作了一段时间，手上的公司业务暂时空闲，想证明一下自己的能力，从此一发不可收拾，每天投入`Concis`的时间甚至比上班还多...

## 文档项目技术选型

最早其实博主并不知道任何组件库文档构建的`cli`工具，在全网到处搜索关键词——React 组件库项目搭建（哈哈哈，太真实了），之后了解到了`docz`、`storybook`、`dumi`，由于作者之前开发过`umi`，并对其作者陈成（支付宝前端架构师、各种框架作者）崇拜，就是这么佛系的情况下，作者使用 dumi 创造`Concis`的项目文档。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfab3e39c1064827af3a0d5f8e3977d6~tplv-k3u1fbpfcp-zoom-1.image)

对于 dumi 搭建组件库的教程，作者在三月就记录下来了，现在这篇文章正好用上~~~

[react+dumi+typescript 搭建个人组件库 Concis](https://juejin.cn/post/7115414761646342152)

## 开发第一个组件——Button

有了 dumi 的支持，作者也是很轻松的开发出了第一个 Button 组件。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/963c94b076df4c50bf4af4db0353fe79~tplv-k3u1fbpfcp-zoom-1.image)

在这之中，对于作者组件封装代码层面提升最大的就是`arco-design`字节跳动去年诞生的组件库。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/749aee128708438ab2a473fbf2526650~tplv-k3u1fbpfcp-zoom-1.image)

作者很多时候的电脑界面是这样的。。。在开发每个组件之前，学习一下别人是如何设计这个组件的，读懂这个组件，从而提升自己，在这半年对于作者的编码能力以及 React 代码质量提升是巨大的。

# 20 个组件时

## 加入 Jest 单元测试

作者维持了上述的情况持续了大约 2 个月，加入了`jest单元测试`，从而保证组件的健壮性以及作者每次上线前的自信心（虽然也没人看）

具体的 React+Jest 单元测试组件的文章，作者也早已记录好~

[全网最细：Jest+Enzyme 测试 React 组件（包含交互、DOM、样式测试）](https://juejin.cn/post/7115420446375444516)

关于每一份`component.test.tsx`作者也是去借鉴`arco-design`的测试用例编写方式，因此这篇文章的质量有很大的保障。

## 加入前端埋点

嗯，就是上面所说的，作者还真的很好奇到底有没有人看`Concis`的文档，每天有多少人看，于是作者加入了组件库线上文档的埋点，对于埋点的具体设计和实现文章在这里~

[前端埋点实现](https://juejin.cn/post/7115419772405153829)

实现完埋点以后第二天醒来看到数据库的记录还是很惊讶的。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec1d0ab7e8740f09db25ef8511b2b98~tplv-k3u1fbpfcp-zoom-1.image)

还是有一定的浏览量的，也是在这个时候，给作者提升了很大的自信心，也就是开源的那种成就感（自己的东西被别人使用或浏览）

# 加入更多的灵活配置

借鉴了一些成熟组件库文档不难发现他们都有共同点：

- 自定义主题
- 国际化

作者想了一下，确实，如果用户永远只能使用组件库内置的配色，那这个组件库的应用范围是很小的，因此作者实现了全局主题色切换功能，效果是这样的：

![请添加图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5431f856f624f2bbf6d3fa49f2e8b17~tplv-k3u1fbpfcp-zoom-1.image)
对应的实现文章也准备好了，在这里~

[五分钟告诉你 React 全局配置项目主题色是怎么实现的](https://juejin.cn/post/7119202163905003534)

# 让 Concis 不再只是一个组件库

在之前，作者的组件库文档其实只有一个菜单——组件，其余一无所有，此时的心态其实已经从最早的玩一玩、体验一下、证明一下自己的能力转向了想把`Concis`做成一个充满希望、有未来的产品，这时作者加入了指南、贡献相对应的介绍性文档，而不只是单纯的一个个组件的说明书。

**介绍页面：**
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff67af630c514fd9828262281c2b4d88~tplv-k3u1fbpfcp-zoom-1.image)

**快速开始页面：**
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d8d2f931d0941d1b4c264c723f05507~tplv-k3u1fbpfcp-zoom-1.image)

# 重构项目

此时作者其实发现自己的组件库项目目录和成熟产品并不一样，作者的是这样的：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6e1bbf89a9c4c60954b737caf6a6ede~tplv-k3u1fbpfcp-zoom-1.image)

而`arco-design`的是这样的：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4deeba2beb74b66ac11ba4f3c770f98~tplv-k3u1fbpfcp-zoom-1.image)

这其实涉及到了一个架构——**分包架构**，作者之后了解到了，选择了使用`lerna`对项目进行重构，因为本身`dumi`对`lerna`提供了支持，其实之所以作者只有单纯的一个 src 目录，原因也很简单，因为此时`Concis`只是一个组件库，去查看成熟组件库的 packages 目录，可以看到其中除了组件库，还有一系列围绕着组件库的生态产品，如 cli 工具、各种插件等等。

经过重构以后，`Concis`的项目目录变成了这样：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/388c74c73630465db967de063771f24c~tplv-k3u1fbpfcp-zoom-1.image)

具体重构过程，作者也还是写了文章保存，哈哈哈。

[基于 lerna 重构 Concis 组件库](https://juejin.cn/post/7119553385971318821)

# 新鲜的血液——Varlet

偶然间刷 github，看到了一款 Vue 移动端组件库：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd384ec4400f471f9595c0c7b6551cab~tplv-k3u1fbpfcp-zoom-1.image)

后续了解到了作者其实最早和博主很像，都是小团队开发（Varlet 是三人开始），比作者好一点点哈哈哈，这里也有一篇 Varlet 的文章：

[从 0 到 1400star，从阮一峰周刊到尤雨溪推荐，小透明开源项目的 2021 年总结](https://juejin.cn/post/7038379264852361246)

过了 2 年时间，Varlet 从 0 到目前（已经 3kstar 了）对于作者的激励程度不是一点点，一晚上刷完了 Varlet 作者所有的掘金文章，同样在社区保持了一点的联系，也让作者从 Varlet 中获得了一些新的灵感。

Varlet 有许多做的很不错的地方：

- 在线编辑器
- 开发工具插件支持
- varlet-cli 工具
- 很不错的 UI 设计（对于 Varlet，我也是用户同样也是评审团）

这些也都记录到了作者未来的规划中，同样看到 Varlet2 年所产生的社区，作者不禁羡慕。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85a1704b4fcb49a6aa4824cae26d3113~tplv-k3u1fbpfcp-zoom-1.image)

# 加入 e2e 测试

其实 e2e 的概念作者也是偶然间在朋友嘴里知道的，后来想着不妨也加入到 Concis 中吧。在技术选型中作者并没有经验，于是看了一眼 Vue 的源码，看到了 e2e 目录，就这样，跟着尤大的脚步一步步把测试放在了 Concis 中了。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4265cc7ade954e0ab8f9655bd7aa962a~tplv-k3u1fbpfcp-zoom-1.image)

**具体的配置实现过程在这里：**

[基于 Vue 源码中 e2e 测试实践](https://juejin.cn/post/7121053442239365133)

# 加入 vscode 代码补全、智能提示插件

看过了`Varlet`，作者对于这项技术非常有兴趣，于是就尝试了起来，同样也是学习了`Varlet`的源码，一步步学习探索，最后在也是成功实现。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/033de2de1566404bacdf5a3c69ad6107~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9155993fb9c44d25bb229aa1e4845971.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c54b7cb84f54a8583eb319da781fea7.png)

# 未来的展望

半年不到的时间，作者将自己从零到现在开发 Concis 的心路历程吐露了出来，也许这篇文章并没有一些技术层面的内容，但是每一个点都可以通过作者总结的文章去学习，对于 Concis 的未来，作者其实有很多的想法，当然很多都是基于成熟框架而来的。

对于看到这里的你，很感谢可以感受作者的心路历程到文末，如果你也想创造一个组件库，作者可能路不是完全对的，但是希望`Concis`的发展道路可以给你提供一些想法和收货。

同样，作者也非常想通过 Concis 可以创造出一个社区，认识更多的朋友，喜欢折腾，喜欢研究的朋友，一起创造，让作者开源的路上不再孤单。希望接下来的时间能够进一步的成长，沉下心来好好做技术，不忘初心，不要浮躁。支持我们可以给我们的仓库点一个`star`

同样，实现的文章在这里：

[组件库设计 | 让你的 React 组件获得代码补全和属性提示功能](https://juejin.cn/post/7121817655765368845)

**相关的链接在下方。**

# 链接相关

Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)

github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)

npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)
