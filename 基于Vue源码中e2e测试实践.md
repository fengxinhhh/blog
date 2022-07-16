@[TOC](基于Vue源码中e2e测试实践)
![在这里插入图片描述](https://img-blog.csdnimg.cn/284651244ee74abb9e939b6190f4bb61.jpeg#pic_center)

# 前言

最近半年博主一直在抽空自研一款 React 组件库 Concis，对于测试原本只支持于`jest+enzyme单元测试`，而单元测试的缺陷是无法模拟用户的一些操作，如表单组件，只能测试到一些组件渲染 mount 准确性的测试，对于一系列用户操作的模拟测试是无法做到的，因此博主考虑加入`e2e测试`从而让组件上线后更加健全。

关于单元测试，博主之前整理了一份基于 Concis 项目自用的总结：

[全网最细：Jest+Enzyme 测试 React 组件（包含交互、DOM、样式测试）](https://blog.csdn.net/m0_46995864/article/details/125365202?spm=1001.2014.3001.5501)

# 技术选型&对 Vue 的参考

其实博主本身对`e2e测试`的技术栈比较陌生，于是决定学习一下目前主流框架 Vue 的`e2e测试`是如何去做的，学习一下尤大大的技术选型，哈哈哈~

这是 github 中 Vue 的 test 文件夹：

![在这里插入图片描述](https://img-blog.csdnimg.cn/3c23b28bb55e4e548fe95ebdf84eb86e.png)

我们进入到 e2e 中可以看到，圈起来的都是 e2e 测试用例，而 utils 应该就是 e2e 的初始化用于在每一个测试用例文件执行前初始化的，看一下`e2eUtils.ts`这个文件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/30410953eb6d42c38adfff301579a3b2.png)

好了，技术选型确认完毕，使用的是`puppeteer`这个包，大概研究一下使用，这里尤大大其实是基于`pupeteer`一系列 page 执行性 api 进行了二次封装并且最终导出，在每一个测试用例中去使用：

![在这里插入图片描述](https://img-blog.csdnimg.cn/76658098df90487ca55faec9f3d951ba.png)
而这些方法博主最后整理了一下，有很详细的备注，具体代码在最后~

# Puppeteer 测试流程

这里简单介绍一下使用`Puppeteer`做`e2e测试`的整体流程：

1.  初始化 browser 实例，构造测试模拟浏览器;
2.  基于 browser 实例，新建一个页面实例;
3.  打开测试 url;
4.  一系列的操作，DOM 操作，mock 用户;
5.  关闭页面;
6.  关闭浏览器;

**这里整理了一份入门版简易代码，对照上述流程是这样的：**

```typescript
const testPupeteer = async () => {
  let brwoser: puppeteer.Browser
  let page: puppeteer.Page

  brwoser = await puppeteer.launch({
    //打开浏览器
    headless: false,
  })
  await page.goto("http://localhost:8000") //page页面跳转测试url
  await page.type("body .testDiv .username", "username") //输入用户名
  await page.type("body .testDiv .password", "123456") //输入密码
  await page.click("body .testdiv .login-btn") //登录
}
```

可以看到，代码很清晰，由于是真实操作，可能有的操作会涉及网络请求，因此都是异步访问按顺序阻塞执行操作。

代码中模拟了一次用户打开浏览器、打开页面、输入用户名密码、点击登录按钮的一次登录操作，但是`pupeteer`只提供了模拟操作，并没有对操作进行反馈做判断，这时就需要配合`jest`的断言来进行 e2e 整体测试了。

# 在 Concis 中的实践

这里博主借鉴了 Vue 的 e2e 测试方法，即同样的写了一个 setup 方法用于初始化`pupeteer`的起步动作，并且二次封装了一系列方法。

`e2eUtils.ts`代码如下：

```typescript
import puppeteer from "puppeteer"

const getExampleUrl = (componentName: string) => {
  //获取测试组件的页面url
  return `http://localhost:8000/#/common/${componentName}`
}
const e2eTestTimeout = 30 * 1000

const setupPuppeteer = () => {
  let browser: puppeteer.Browser
  let page: puppeteer.Page

  beforeAll(async () => {
    browser = await puppeteer.launch({
      headless: false,
    })
  })
  beforeEach(async () => {
    page = await browser.newPage()

    await page.evaluateOnNewDocument(() => {
      localStorage.clear()
    })
    page.on("console", (e) => {
      if (e.type() === "error") {
        const err = e.args()[0]
        console.error(`Error from Puppeteer-loaded page:\n`, err)
      }
    })
  })
  afterEach(async () => {
    await page.close()
  })
  afterAll(async () => {
    await browser.close()
  })

  const click = async (dom: string, options?: puppeteer.ClickOptions) => {
    //点击元素
    await page.click(dom, options)
  }
  const getCount = async (dom: string) => {
    //获取元素数量
    return (await page.$$(dom)).length
  }
  const getText = async (dom: string) => {
    //获取元素文本内容
    return await page.$eval(dom, (node) => node.textContent)
  }
  const getValue = async (dom: string) => {
    //获取文本框的内容
    return await page.$eval(dom, (node) => (node as HTMLInputElement).value)
  }
  const getHtml = async (dom: string) => {
    //获取元素的innerHTML
    return await page.$eval(dom, (node) => node.innerHTML)
  }
  const getClassList = async (dom: string) => {
    //获取元素所有类名
    return await page.$eval(dom, (node) => [...node.classList])
  }
  const getChildrenCount = async (dom: string) => {
    //获取子元素数量
    return await page.$eval(dom, (node) => node.children.length)
  }
  const domIsShow = async (dom: string) => {
    //判断元素是否在document中
    const display = await page.$eval(dom, (node) => {
      return window.getComputedStyle(node).display
    })
    return display !== "none"
  }
  const isChecked = async (dom: string) => {
    //判断多选框是否被选中
    return await page.$eval(dom, (node) => (node as HTMLInputElement).checked)
  }
  const isFocused = async (dom: string) => {
    //判断元素是否被聚焦
    return await page.$eval(dom, (node) => node === document.activeElement)
  }
  const setValue = async (dom: string, value: string) => {
    //设置输入框内容
    const el = (await page.$(dom))!
    await el.evaluate((node) => ((node as HTMLInputElement).value = ""))
    await el.type(value)
  }
  const setText = async (dom: string, value: string) => {
    //设置元素内容
    const el = (await page.$(dom))!
    await el.evaluate((node) => ((node as HTMLElement).innerText = ""))
    await el.evaluate((node) => ((node as HTMLElement).innerText = value))
  }
  const enterValue = async (dom: string, value: string) => {
    //设置输入框内容后回车
    const el = (await page.$(dom))!
    await el.evaluate((node) => ((node as HTMLInputElement).value = ""))
    await el.type(value)
    await el.press("Enter")
  }
  const clearValue = async (dom: string) => {
    //清空输入框内容
    await page.$eval(dom, (node) => ((node as HTMLInputElement).value = ""))
  }
  const timeout = async (time: number) => {
    //延时
    return page.evaluate((time) => {
      return new Promise((r) => {
        setTimeout(r, time)
      })
    }, time)
  }
  return {
    page: () => page,
    click,
    getCount,
    getText,
    getValue,
    getHtml,
    getChildrenCount,
    getClassList,
    domIsShow,
    isChecked,
    isFocused,
    setValue,
    setText,
    enterValue,
    clearValue,
    timeout,
  }
}

export { getExampleUrl, setupPuppeteer, e2eTestTimeout }
```

尤大大的源码路径是这个：[https://github.com/vuejs/vue/blob/main/test/e2e/e2eUtils.ts](https://github.com/vuejs/vue/blob/main/test/e2e/e2eUtils.ts)
以上代码主要做了这些事情：

1.  getExampleUrl 函数用于获取测试的 url 地址，博主是测试组件库，因此就是每次测试单个组件文档的页面地址;
2.  setupPupeteer 函数用于初始化测试环境（浏览器、打开页面）并且封装了一系列方法导出;

**接下来看一下博主对于 Form 组件的测试：**

```typescript
import { getExampleUrl, setupPuppeteer, e2eTestTimeout } from "../e2eUtils"

describe("form e2e test", () => {
  const {
    click,
    page,
    getCount,
    getText,
    getChildrenCount,
    setValue,
    getValue,
  } = setupPuppeteer()

  const formTest = async () => {
    await page().goto(getExampleUrl("form"), {
      waitUntil: "domcontentloaded",
    })
    //基本demo展示Dom测试
    expect(await getCount("#form-index1 .concis-form .concis-form-item")).toBe(
      4
    )
    expect(
      await getText(
        "#form-index1 .concis-form .concis-form-item:nth-child(1) .concis-form-item-label"
      )
    ).toBe("Username")
    expect(
      await getText(
        "#form-index1 .concis-form .concis-form-item:nth-child(2) .concis-form-item-label"
      )
    ).toBe("Post")
    expect(
      await getText(
        "#form-index1 .concis-form .concis-form-item:nth-child(3) .concis-form-item-label"
      )
    ).toBe("")
    expect(
      await getText(
        "#form-index1 .concis-form .concis-form-item:nth-child(4) .concis-form-item-label"
      )
    ).toBe("")
    //切换布局radioGroup测试
    expect(await getCount("#form-index2 .concis-radio-group")).toBe(1)
    //全局禁用测试
    expect(await getCount("#form-index5 .disabled")).toBe(1)
    //单行禁用测试
    expect(
      await getCount(
        "#form-index7 .concis-form .concis-form-item:nth-child(2) .concis-form-item-disabled"
      )
    ).toBe(1)
    //测试添加校验label icon显示
    expect(
      await getChildrenCount(
        "#form-index8 .concis-form .concis-form-item:nth-child(1) .concis-form-item-label"
      )
    ).toBe(1)
    //校验失败的测试
    await click(
      "#form-index8 .concis-form .concis-form-item:nth-child(3) .concis-form-item-content"
    )
    expect(await getText("#form-index8 .concis-form .show-rule-label")).toBe(
      "必须包含a"
    )
    //提交后弹窗测试
    await click(
      "#form-index9 .concis-form .concis-form-item:nth-child(3) .concis-form-item-content"
    )
    expect(await getChildrenCount(".all-container")).toBe(1)
    //测试重置
    await setValue(
      "#form-index6 .concis-form .concis-form-item:nth-child(1) input",
      "123"
    )
    expect(
      await getValue(
        "#form-index6 .concis-form .concis-form-item:nth-child(1) input"
      )
    ).toBe("123")
    await click(
      "#form-index6 .concis-form .concis-form-item:nth-child(8) .concis-form-item-content .concis-button:nth-child(2)"
    )
    expect(
      await getValue(
        "#form-index6 .concis-form .concis-form-item:nth-child(1) input"
      )
    ).toBe("")
  }
  test("test e2e test", async () => await formTest(), e2eTestTimeout)
})
```

其实有了`e2eUtils.ts`后，编写测试用例方便了很多，只需要在调用封住好的 api 基础上加入 jest 的一些断言，就可以很好的测试组件的交互性。
这里 Form 的测试都有注释，博主不再依次阐述。

# 项目目录整理

由于之前只有单元测试，因此需要整理项目目录，整理后的目录如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/57f8ecb25e724249bd307bf2bebe6bbe.png)

对于 package.json 也是可以重新配置：

```javascript
 "scripts": {
    "build": "rollup -c ./rollup.config.js",
    "test:unit": "jest ./__tests__/unit",		//单元测试
    "test:e2e": "jest ./__tests__/e2e"			//e2e测试
  },
```

# Concis 组件库

博主自研 Concis 组件库已经有半年时间，组件库也是慢慢成型，目前已整合前端组件 30+、组件 unit/e2e 测试、组件库文档、全局配置等等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/39fd09fc79a245378aa4f11635031962.png)

**一路人也是就自己一个人，也是真的挺花费功夫的...博主也是需要更多的有兴趣小伙伴可以关注一下参与进来，一起贡献开源项目，建设一个基于 Concis 的技术社区，相互讨论、学习、帮助，这是博主从最初的兴趣想去实现一个小项目到目前为止一个更大的愿景。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/c4a81f7b28044ec1a37772cdc7f76e36.png)

更具体的可以参考 [React 组件库 Concis，寻求社区中有兴趣的小伙伴加入...](https://blog.csdn.net/m0_46995864/article/details/125747677?spm=1001.2014.3001.5501)

Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

开源不易，欢迎学习和体验，喜欢请多多支持，有问题请留言，如果此文对你有帮助，博主需要你的支持，感谢。
