# 前言

博主最近在自研 Concis 组件库，全局配置项目主题色的大体意思其实和 antd 一样，可以通过一款色系来全局配置 Concis 中所有组件的主题色，比如 Concis 默认的色系是蓝色(#1890FF)，如果公司业务的主题色是绿色、黄色，一个个去给单独组件加样式是很麻烦的，因此就有了需求的来源，效果图是这样的：

![请添加图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ba9262c445a4b91a59cc13000e8bd63~tplv-k3u1fbpfcp-zoom-1.image)

# 思路

抛开 Concis 不谈，单从 create-react-app 来讲，其实思路很简单，在入口`<App />`组件为它再包裹上一层 createContext hook，把我们所需要定制的主题色向下传给所有子组件，在子组件中做一些兼容性处理，即可实现。

预期`App`组件的样子是这样的：

```typescript
import React from "react"
import ReactDOM from "react-dom/client"
import "./index.css"
import App from "./App"
import reportWebVitals from "./reportWebVitals"
import { GlobalConfig } from "concis/web-react"

const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <React.StrictMode>
    <GlobalConfig globalColor="orange">
      <App />
    </GlobalConfig>
  </React.StrictMode>
)
```

接下来我们先开始 GlobalConfig 组件的编写。

# GlobalConfig 全局配置组件

首先`GlobalConfig`会创建顶层通信装置，即 React.createContext，将调用`GlobalConfig`所传入的色号接受，并向下传递给所有子组件，因此可以得出代码：

```typescript
import React, { createContext } from "react"
import { GlobalConfigProps } from "./interface"

export const globalCtx = createContext<GlobalConfigProps>(
  {} as GlobalConfigProps
) //顶层通信装置

const GlobalConfig = (props: GlobalConfigProps) => {
  const { children } = props

  return <globalCtx.Provider value={props}>{children}</globalCtx.Provider>
}

export default GlobalConfig
```

这里我们直接把 props 作为 context 值内容放进去就好，这里除了主题色也可以定制一些其他的颜色，比如在 Menu 组件中，选中背景样式其实是主题色更浅一些的颜色，就像这样：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a886dc1e1a24432ebd39f2248aa1b832~tplv-k3u1fbpfcp-zoom-1.image)

因此可以自定义一些其他参数，如 menuActiveTextBg，单独配置 Menu 的背景色，当然这只是例外，大部分场景在上面动图上其实单靠主题色一个参数就可以满足。

# 子组件中的处理

这里拿`CheckBox组件`举例，先忽略源代码，我们把`GlobalConfig`中导出的 Context 引入进来：

```typescript
//CheckBox一些其他依赖包导入
import { GlobalConfigProps } from "../GlobalConfig/interface"
import { globalCtx } from "../GlobalConfig"

//CheckBox业务代码
```

并且在组件中使用这个 Context：

```typescript
const CheckBox: FC<checkBoxProps> = (props) => {

  const { globalColor } = useContext(globalCtx) as GlobalConfigProps;

  //CheckBox业务代码
});
```

那么问题来了，声明在 CSS 文件中的颜色，如何在 tsx 组件中替换呢?

# React tsx 与 CSS 的通信

我们可以通过 css 自定义变量，在组件根标签中引入，就像这样：

```typescript
return (
    <Fragment>
      {group && group.length ? (
        <div className="checkGroup" style={{ '--global-color': globalColor || '#1890ff' } as any}>

         //CheckBox模板代码

        </div>
      )}
    </Fragment>
  );
};

export default memo(CheckBox);
```

这里我们在根标签引入了一个`global-color`属性，并且判断，如果自定义主题色了值就为主题色，否则就是默认颜色。

在 CheckBox.module.less 中就可以这样处理：

```typescript
.checkBox-activedLess {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 12px;
    height: 12px;
    margin-left: 5px;
    color: #fff;
    border: 1px solid #d9d9d9;
    .less-box {
      width: 8px;
      height: 8px;
      background-color: var(--global-color);
    }
  }
```

直接把原来的`background-color: #1890ff;` 替换为`background-color: var(--global-color);`

这样，打开页面，就可以看到 CheckBox 原本的蓝色选中样式变成了自定义颜色，当然这里的处理方式只是博主想到的，如果有更好的方案也欢迎评论区留言，因为博主的 css 其实学的并不好~~~

# 在 Concis 组件库中代码

这里贴一下博主在组件库中写的一些源码，文章底部也会有整个组件库源码和 npm 链接。

index.tsx:

```typescript
import React, { createContext } from "react"
import { GlobalConfigProps } from "./interface"

export const globalCtx = createContext<GlobalConfigProps>(
  {} as GlobalConfigProps
) //顶层通信装置

const GlobalConfig = (props: GlobalConfigProps) => {
  const { children } = props

  return <globalCtx.Provider value={props}>{children}</globalCtx.Provider>
}

export default GlobalConfig
```

interface.ts:

```typescript
import { ReactNode } from 'react';

interface GlobalConfigProps {
  children?: ReactNode;
  /**
   * @description 主题颜色
   * @default #1890FF
   */
  globalColor?: string;
  /**
   * @description Input输入框组件聚焦、点击时的外发光颜色
   * @default #C6F7FF
   */
  input?: string;
  /**
   * @description Tree选择器组件选中时的字体颜色
   * @default #1890FF
   */
  treeSelectTextColor?: string;
  /**
   * @description Menu导航菜单组件选中时的背景颜色
   * @default #C6F7FF
   */
  menuSelectBgColor?: string;
}

export type { GlobalConfigProps };

```

代码其实很简单，主要 Concis 在之前都是写死了主题色为蓝色(#1890ff)，在所有子组件中的处理花费了比较大的时间。

# GlobalConfig 在 Concis 中的文档

![请添加图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb5f9819c3014f10bdddee75a30ca617~tplv-k3u1fbpfcp-zoom-1.image)

# Concis 地址信息

Concis 已经开发了接近半年时间，也是越来越成熟了，这里留一下 Concis 的一些 Path：

- Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
- github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
- npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

开源不易，如果文章内容对你有帮助，请支持一下，非常感谢。
