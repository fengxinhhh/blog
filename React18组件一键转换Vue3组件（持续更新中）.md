## 需求来源

博主最近一段时间其实是在自研 React 组件库的业务的，目前也有了大约二十几个组件，这里顺便先留一下组件库的地址哈~

React-View-UI 组件库线上链接：[http://react-view-ui.com:92/#/](http://react-view-ui.com:92/#/)
github：[https://github.com/fengxinhhh/React-View-UI-fs](https://github.com/fengxinhhh/React-View-UI-fs)
npm：[https://www.npmjs.com/package/react-view-ui](https://www.npmjs.com/package/react-view-ui)

**组件库文档：**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcac2a9fdfe4407e9973c8d5b72a8832~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/171fffce369342c0a18a14616d10f015~tplv-k3u1fbpfcp-zoom-1.image)

嗯，先卖个关子，然后回归主题，其实现在 Vue 也是很火的框架随着 Vue3 的诞生，博主其实最终目标是想整合一套 React+一套 Vue 组件库在一起的，但是重写一遍 React 的组件很费工作量也不现实，因为我是单人开发，于是就萌生了写一个 React 组件转换 Vue 组件的工具，功能性将逐步开发更新到博客，喜欢的可以关注一下。

## 市场环境

目前市场上也有这样的转换工具，但是局限性比较高，如 React Class 组件转换 Vue 组件，类组件其实在现在用的是很少的，随着 React16.8 的出现，hooks 的到来，函数组件火热，让 React 的组件写的更加灵活舒适，因此只能选择自研一个转换工具。

## 效果演示

目前工具已经开发一天，只能实现一些基本的语法转换，后期也会更新工具研发流程，最后会有 github，可以关注一下。

目前的转换效果是这样的：
**js 部分：**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78755152cae34ed18f6a6f4de5a7e844~tplv-k3u1fbpfcp-zoom-1.image)
**h5 部分：**
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb8633b420ba4a7bba36894bd5150c9e~tplv-k3u1fbpfcp-zoom-1.image)

目前写了除基本模板以外，react 的 useEffect、useState 转换 -> vue 的 ref、recative、setup、mounted、unmounted、watch。

因此本文目的主要为了记录以及分享开发的过程和思路。

## 转换

转换的入口就在 index.js -> transformOptions 变量中，可以配置 css 为 less/scss，配置入口源文件路径和出口路径，在终端执行 node src/index 即可转换。

```typescript
const transformOptions = {
  sourcePath: path.resolve(__dirname, "sourceFile", "index.jsx"),
  outputPath: path.resolve(__dirname, "outputFile", "index.vue"),
  styleType: "less",
  componentName: "Index",
}
```

## 设计

入口函数就在 index.js 的第 26 行：

```typescript
readFileFromLine(transformOptions.sourcePath, (res) => {

}
```

将转换的配置传入函数，进行处理。readFileFromLine 函数可以将我们输入的文件按行输出，变成一个数组，让我们一行一行进行判断处理。

```typescript
const fs = require("fs")
const readline = require("readline")

const readFileFromLine = (path, callback) => {
  var fRead = fs.createReadStream(path)
  var objReadline = readline.createInterface({
    input: fRead,
  })
  var arr = new Array()
  objReadline.on("line", function(line) {
    arr.push(line)
  })
  objReadline.on("close", function() {
    callback(arr)
  })
}

module.exports = {
  readFileFromLine,
}
```

这里传入了源文件的路径，并且返回了文件切割后的以行为元素的数组，并且准备了一些全局变量，用于模板的保存、导入依赖包的记录、栈的实例，这里栈主要用作记录任务优先级，举个例子吧，在 useEffect 中遇到了 return()=>，原本在做对应 vue mounted 的模板处理，应在遇到 return()=>之后去做 vue unmounted 的处理，这就是栈的需求，可以调度我们的任务分配，每行代码会去执行当前栈顶的任务。

**设计变量如下：**

```typescript
const vueTemplateList = [] //vue模板内容
const vueScriptList = [] //vue script内容
let fsContent = [] //react源文件
let compileStack = new Stack() //以栈顶值作为优先级最高的编译任务调度，1 -> 常规编译 ，2 -> 副作用函数编译中， 3 -> 模板编译中， 4 -> 引用数据类型状态编译中， 5 -> 组件销毁生命周期
compileStack.unshift(1)
let reactFileHasStateType = [] //1为基本数据类型ref，2为引用数据类型reactive，3为mounted，4为watch，5为unmounted
let allStateList = new Map() //所有状态的列表
let resultFileTotalLine = 0 //结果文件总行数
let mountedContainer = "" //mounted临时容器
let unMountedContainer = "" //unMounted临时容器
let stateContainer = "" //复杂state临时容器
let jsxContainer = "" //jsx模板临时容器
let jsxCompileParams = {} //jsx模板解析时的记录信息
let functionContainer = "" //函数内容临时容器
let functionModuleNum = 0 //函数中花括号呈对数，0表示{}成对
```

1.  vueTemplateList 用于保存模板代码段；
2.  vueScriptList 用于保存逻辑代码段；
3.  fsContent 用于接收入口函数处理后的代码片段（代码行为分割的数组）；
4.  compileStack 用于分配任务调度优先级，默认为 1。1 -> 常规编译 ，2 -> 副作用函数编译中， 3 -> 模板编译中， 4 -> 引用数据类型状态编译中， 5 -> 组件销毁生命周期；
5.  allStateList 记录所有的状态，以状态名：状态值为键值对形式保存在 map 中；
6.  resultFileTotalLine 用于在最后记录输出文件的行数；
7.  mountedContainer 用于存放在 mounted 处理时的代码保存容器（任务标识->2）；
8.  unMountedContainer 用于存放在 unmounted 处理时的代码保存容器（任务标识->5）；
9.  stateContainer 用于存放复杂状态的代码段，因为它由多行代码保存（任务标识->4）；
10. jsxContainer 用于存放遍历渲染的代码段。（解析到 jsx .map 转换到 v-for 时，任务标识->6）；
11. jsxCompileParams 用于存放外层渲染的一些数据，如遍历的数组状态名、key 值、遍历的元素名称；
12. functionContainer 用于存放函数体的代码段；
13. functionModuleNum 用于判断为函数结束还是函数内块级作用域结束；

有了栈的任务分配，因此在每行代码做处理前，我们需要确定它的任务标识号是多少，从而进行处理，而在某个 react 关键词结束时，如 useEffect ---> ,[])，将任务栈栈顶弹出，转做新的任务，这就是栈的好处。

**接下来就是主体处理流程了，先看一下六个条件分支吧：**

## 处理（7 个任务）

**comipleStack.peek() === 1 时**，是常规编译，这时会对当前行的代码进行判断，是否有 react 的一些关键词，如'return('开头应是 template 处理；'useEffect'应对应 vue 的 mounted/watch 处理。
如果捕捉到了关键字，则 comipleStack.peek()会改变，同时在下一行代码的处理时直接进行 2/3/4/5/6 的分支进行对应的编译；如果没有捕捉到关键字，则跳过。

```typescript
if (compileStack.peek() === 1) {
  //常规编译，捕捉当前行是否为特殊编译
  if (lineItem === "return(") {
    //开始输出模板
    compileStack.unshift(3)
  } else if (lineItem.startsWith("const[") && lineItem.includes("useState")) {
    //如果是状态声明
    const {
      returnCodeLine,
      returnAllStateList,
      returnCompileStack,
      returnReactFileHasStateType,
    } = saveState(lineItem, allStateList, compileStack, reactFileHasStateType)
    vueScriptList.push(returnCodeLine)
    allStateList = returnAllStateList
    compileStack = returnCompileStack
    reactFileHasStateType = returnReactFileHasStateType
  } else if (lineItem.startsWith("useEffect")) {
    //副作用函数
    compileStack.unshift(2)
  } else {
    //被舍弃的，跳过循环，不做编译，一些特殊react专属语法，如export default function Index()/import './index.module.less';
  }
}
```

**comipleStack.peek() === 2 时**，是副作用函数编译，也就是 useEffect 转 mounted/watch，在这分支中每一行只要捕捉到了'return()=>'则截取上段代码，作为 mounted 代码片段，并在 return()=>之后进行 unmounted 代码片段的截取，在最后截取的时候，如果是"},[])"，代表这是一个 mounted 函数；如果是
"},["，代表这是一个状态监听函数（vue watch），则切割出副作用状态，进行处理，编译模板。

```typescript
else if (compileStack.peek() === 2) {          //副作用函数编译中
      const saveCodeResult = saveCodeInUseEffect(allStateList, lineItem, compileStack)
      if (saveCodeResult.action === 'unmounted') {        //调度到unmounted
        compileStack = saveCodeResult.compileStack;
      } else {                        //仍然在执行mounted任务
        if (lineItem.startsWith('},[])')) {       //mounted结束，批量插入
          mountedContainer = '\nonMounted(() => {\n' + mountedContainer + '})\n';
          vueScriptList.push(mountedContainer);
          vueScriptList.push(unMountedContainer);
          mountedContainer = "";
          unMountedContainer = "";
          compileStack.shift();
          if (!reactFileHasStateType.includes(3)) {
            reactFileHasStateType.push(3);
          }
        } else if (lineItem.startsWith('},[')) {      //编译成watch函数，批量插入
          const params = lineItem.split('[')[1].split(']')[0].split(',');
          vueScriptList.push(formatWatchToVue(params, mountedContainer));
          if (!reactFileHasStateType.includes(4)) {
            reactFileHasStateType.push(4);
          }
          compileStack.shift();
        } else {
          mountedContainer += saveCodeResult;
        }
      }
    }
```

**compileStack.peek() === 3**代表模板编译，以"return("作为开始（在 compileStack.peek() === 1）中会捕获到，并改变，以结尾")"作为模板编译结束，中间对 className 处理为 class。
这里博主功能还没有完善好，后期会做遍历渲染、条件渲染、vue 语法糖的转换，这里仅做参考。

```typescript
else if (compileStack.peek() === 3) {             //模板编译中
      lineItem = lineItem.replace('className', 'class')
      if (lineItem.includes(')') && lineItem.replace(/\s*/g, "") === ')') {                 //模板输出结束
        compileStack.shift();
      }
      else if (lineItem.includes('{') && lineItem.includes('}')) {        //带状态的模板
        vueTemplateList.push(formatStateInTemplate(lineItem));
      } else {
        vueTemplateList.push(lineItem);
      }
    }
    else if (compileStack.peek() === 4) {         //复杂状态编译中
      if (lineItem.includes(')')) {         // 编译结束
        stateContainer += `\n${lineItem}`;
        vueScriptList.push(stateContainer);
        stateContainer = '';
        compileStack.shift();
      } else {
        stateContainer += `${lineItem}`;
      }
    }
```

**compileStack.peek() === 4**代表复杂状态编译，什么是复杂状态编译，比如有这样的状态声明：

```typescript
const [newList, setNewList] = useState({
  age: 20,
  hobby: "学习",
})
```

这样的状态并不在一行，因此要捕捉到从首行起"({"对应之后的"})"，对复杂的状态进行存储，改变。

```typescript
else if (compileStack.peek() === 4) {         //复杂状态编译中
      if (lineItem.includes(')')) {         // 编译结束
        stateContainer += `\n${lineItem}`;
        vueScriptList.push(stateContainer);
        stateContainer = '';
        compileStack.shift();
      } else {
        stateContainer += `${lineItem}`;
      }
    }
```

对应的状态处理在这里：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cda62abc1300426a93cd17ec80bc88fb~tplv-k3u1fbpfcp-zoom-1.image)

对应的 saveState 方法在这里：

```typescript
const saveState = (
  lineItem,
  allStateList,
  compileStack,
  reactFileHasStateType
) => {
  //保存状态
  //处理useState hook
  const stateKey = lineItem.split("[")[1].split(",")[0]
  const stateVal = lineItem.split("useState(")[1].split(")")[0] //状态值
  let returnCodeLine = ""
  if (!lineItem.includes(")")) {
    compileStack.unshift(4)
  }
  //判断state 类型，保存
  if (stateVal.startsWith("[") || stateVal.startsWith("{")) {
    returnCodeLine = `const ${stateKey}=reactive(${stateVal}${
      compileStack.peek() === 4 ? "" : ")"
    }`
    allStateList.set(
      { state: stateKey, stateAction: `set${formatUseStateAction(stateKey)}` },
      "reactive"
    )
    if (!reactFileHasStateType.includes(2)) {
      reactFileHasStateType.push(2)
    }
  } else {
    returnCodeLine = `const ${stateKey}=ref(${stateVal})`
    allStateList.set(
      { state: stateKey, stateAction: `set${formatUseStateAction(stateKey)}` },
      "ref"
    )
    if (!reactFileHasStateType.includes(1)) {
      reactFileHasStateType.push(1)
    }
  }
  const returnAllStateList = allStateList
  const returnCompileStack = compileStack
  const returnReactFileHasStateType = reactFileHasStateType
  return {
    returnCodeLine,
    returnAllStateList,
    returnCompileStack,
    returnReactFileHasStateType,
  }
}
```

saveState 中就写到了对复杂状态（多行）、简单状态（单行）的分别处理。

**compileStack.peek() === 5**代表 unmounted 函数编译，也就是在**compileStack.peek() === 2**中再次捕捉到的编译处理，此时的优先级 unmounted 会更高，因为需要处理 unmounted 的代码，保存，因此任务优先级栈会是[5, 2, 1]，当 unmounted 代码被保存后，则弹出栈顶，继续 useEffect 的处理。

```typescript
else if (compileStack.peek() === 5) {           //unmounted函数编译中
      if (lineItem.startsWith('}')) {         //可能unmounted结束了，需要先判断是否是块作用域
        const startIconNum = unMountedContainer.split('{').filter(item => item === '').length;
        const endIconNum = unMountedContainer.split('}').filter(item => item === '').length;
        if (startIconNum === endIconNum) {         //执行unmounted
          compileStack.shift();
          unMountedContainer = 'onUnmounted(() => {\n' + unMountedContainer + '})\n';
          if (!reactFileHasStateType.includes(5)) {
            reactFileHasStateType.push(5);
          }
        }
      } else {
        unMountedContainer += saveCodeInUnmounted(allStateList, lineItem, compileStack)
      }
    }
```

**compileStack.peek() === 6** 是在任务 3 中分配出的子任务，也就是循环遍历的解析任务，主要效果为把 react jsx 的.map 转换为 v-for 语法糖，效果是这样的：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1cfd953cc86479a956a87d055586380~tplv-k3u1fbpfcp-zoom-1.image)
实现思路是这样的：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/672a559c099f46aeb43f356148069c33~tplv-k3u1fbpfcp-zoom-1.image)
在模板编译任务中，如遇到{，则表示 jsx 语法开始，将下一行代码的任务分配至 6，将会走入我们的主体逻辑。

```typescript
else if (compileStack.peek() === 6) {            //模板中jsx语法编译中
      if (lineItem.replaceAll(' ', '') === '}') {      //jsx编译结束
        //格式化遍历项dom
        const { mapArray, mapFnParams, key, mapDomType } = jsxCompileParams;
        if (!key) {           //遍历元素的key值为必填项，否则会影响渲染
          throw new Error('please setting the map dom key!');
        }
        jsxContainer = jsxContainer.substr(0, jsxContainer.length - (jsxCompileParams.mapDomType.length + 6));
        vueTemplateList.push(`<${mapDomType} v-for='${mapFnParams} in ${mapArray}' ${key && ":key='" + key + "'"}>`);
        vueTemplateList.push(jsxContainer)
        vueTemplateList.push(`</${mapDomType}>`);
        //弹栈，将任务权交回给普通模板编译
        compileStack.shift();
      } else {          //每行代码编译完，更新编译参数集
        const returnParams = compileJsxTemplate(lineItem, jsxCompileParams)
        jsxCompileParams = { ...returnParams.jsxCompileParams };
        if (returnParams.lineItem) {
          jsxContainer += returnParams.lineItem + '\n';
        }
      }
    }
```

这里会在代码为}时认定 jsx 结束，会整理所保存到的遍历内容，而外层容器（div v-for）则会根据 jsxCompileParams 来进行手写，先看一下 jsxCompileParams 有哪些内容吧，看到这张图应该就可以明白了。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efb40a86b51b4cce8e2b979a41343c82~tplv-k3u1fbpfcp-zoom-1.image)
在 jsx 开始的前 3 行代码就可以收集到这些信息。

1.  mapArray 代表遍历数组；
2.  mapFnParams 代表遍历中用到的参数，也就是 map 函数的两个参数值；
3.  key 代表绑定元素的 key 值；

再来看一下 compileJsxTemplate API：

```typescript
const compileJsxTemplate = (lineItem, jsxCompileParams) => {
  //jsx模板编译
  lineItem = lineItem.replaceAll(" ", "")
  if (lineItem.includes(".map")) {
    //遍历渲染
    jsxCompileParams.mapArray = lineItem.split(".map")[0]
    jsxCompileParams.mapFnParams = lineItem.split(".map(")[1].split("=>")[0]
    return {
      jsxCompileParams,
    }
  } else if (lineItem === "return(") {
    return {
      jsxCompileParams,
    }
  } else if (lineItem.includes("<") && lineItem.includes("key")) {
    //存储遍历key值
    jsxCompileParams.key = lineItem.split("={")[1].split("}")[0]
    jsxCompileParams.mapDomType = lineItem.split("key")[0].split("<")[1]
    return {
      jsxCompileParams,
    }
  } else if (lineItem === ")" || lineItem === "})") {
    //jsx语法结束
    return {
      jsxCompileParams,
    }
  } else {
    //在key容器下的内层遍历子模板，保存，通常在第三行开始
    return {
      jsxCompileParams,
      lineItem: formatStateInTemplate(lineItem),
    }
  }
}
```

主要做的就是解析存储 jsxCompileParams 以及遍历中间段代码的格式，如{name}转换为{{name}}，在每行代码解析后都会返回所收集到的内容，用于 jsx 结束时进行处理。

**compileStack.peek() === 7**是在任务为 1 时（常规编译）中所捕获到的，在每段代码解析时，如遇到"const"&&"=>"的关键字或者"function"的关键字都会被解析为函数定义，会开启函数的编译，在下一段代码解析时转入函数解析任务中，具体代码：

```typescript
else if ((lineItem.startsWith('const') && lineItem.includes('=>')) || lineItem.includes('function')) {         //编译函数
  compileStack.unshift(7);
  if ((lineItem.startsWith('const') && lineItem.includes('=>'))) {       //解析出函数信息
    functionParams.functionName = lineItem.split('const')[1].split('=')[0];
    const params = lineItem.split('(')[1].split(')')[0];
    functionParams.params = params;
    if (functionParams.functionName === transformOptions.componentName) {
      compileStack.shift();
    }
  }
}
```

该方法会解析出函数名、函数参数列表，在函数定义结束时声明在 vueScript 中，在最后写入文件时插入。
完成了函数定义，也需要改变一下之前的模板渲染，在元素中遇到 onClick 时需转换成@click，如遇到 jsx 传参
onClick{() => hello('name', 12)}则需要转换成@click="hello('name', 12)"，因此需要在模板编译时新增判断。

```typescript
else if (compileStack.peek() === 3) {             //模板编译中
  lineItem = lineItem.replace('className', 'class')
  if (lineItem.replaceAll(' ', '') === '{') {            //jsx语法编译开始
    compileStack.unshift(6)                             //jsx语法编译标识符6入栈，作为最高优先级
  }
  else if (lineItem.includes(')') && lineItem.replace(/\s*/g, "") === ')') {                 //模板输出结束
    compileStack.shift();
  }
  else if (lineItem.includes('{') && lineItem.includes('}')) {        //带状态的模板
    vueTemplateList.push(formatMethodInDom(formatStateInTemplate(lineItem)));
  }
  else {
    vueTemplateList.push(formatMethodInDom(formatStateInTemplate(lineItem)));
  }
}
```

在模板编译的任务中，新增了一层函数处理——formatMethodInDom，用于处理在元素中所遇到的事件传参，代码如下：

```typescript
const formatMethodInDom = (lineItem) => {
  //改变一行代码中的onChange -> @change格式转换
  const endTagIndex = lineItem.indexOf(">")
  if (lineItem.includes("on") && endTagIndex > lineItem.indexOf("on")) {
    lineItem = lineItem.replaceAll("on", " @")
    lineItem = lineItem.split("@")
    for (let i = 1; i < lineItem.length; i++) {
      const equalIndex = lineItem[i].indexOf("=")
      const key = lineItem[i].substr(0, equalIndex)
      const value = lineItem[i].substr(equalIndex + 1)
      if (value.includes("=>")) {
        //react模板函数传参
        let _andIndex
        let endTag = value.includes("</") ? false : true //如果整个元素在一行，需要从后往前查找第二个>标签
        for (let i = value.length - 1; i >= 0; i--) {
          if (value[i] === ">" && endTag) {
            _andIndex = i
            break
          }
          if (value[i] === ">") endTag = true
        }
        const template = value.substr(_andIndex)
        const params = value.split("(")[2].split(")")[0]
        const functionName = value.split("=>")[1].split("(")[0]
        lineItem[i] = `${key.toLowerCase()}="${functionName}(${params})" `
        console.log(lineItem[i])
        if (i === lineItem.length - 1) {
          lineItem[i] += template
        }
      } else {
        lineItem[i] = key.toLowerCase() + "=" + value
      }
    }
    lineItem = lineItem.join("@")
  }
  return lineItem
}
```

这段代码的功能其实就是接收一行代码，格式化元素中所有的事件定义。

## 源码

index.js（入口文件）

```typescript
const fs = require("fs")
const path = require("path")
const { readFileFromLine } = require("./write")
const { Stack } = require("./stack")
const { computeFileLines } = require("./_utils/computeFileLines")

const {
  formatStateInTemplate,
  saveCodeInUseEffect,
  saveCodeInUnmounted,
  formatWatchToVue,
  saveState,
  compileJsxTemplate,
} = require("./compile")

const transformOptions = {
  sourcePath: path.resolve(__dirname, "sourceFile", "index.jsx"),
  outputPath: path.resolve(__dirname, "outputFile", "index.vue"),
  styleType: "less",
}
const vueTemplateList = [] //vue模板内容
const vueScriptList = [] //vue script内容
let fsContent = [] //react源文件
let compileStack = new Stack() //以栈顶值作为优先级最高的编译任务调度，1 -> 常规编译 ，2 -> 副作用函数编译中， 3 -> 模板编译中， 4 -> 引用数据类型状态编译中， 5 -> 组件销毁生命周期， 6 -> 模板中jsx语法编译中
compileStack.unshift(1)
let reactFileHasStateType = [] //1为基本数据类型ref，2为引用数据类型reactive，3为mounted，4为watch，5为unmounted
let allStateList = new Map() //所有状态的列表
let resultFileTotalLine = 0 //结果文件总行数
let mountedContainer = "" //mounted临时容器
let unMountedContainer = "" //unMounted临时容器
let stateContainer = "" //复杂state临时容器
let jsxContainer = "" //jsx模板临时容器
let jsxCompileParams = {} //jsx模板解析时的记录信息

readFileFromLine(transformOptions.sourcePath, (res) => {
  fsContent = res
  fsContent = fsContent.filter((line) => line !== "")
  fsContent.forEach((lineItem) => {
    if (compileStack.peek() !== 3) {
      lineItem = lineItem.replace(/\s*/g, "")
    }
    if (compileStack.peek() === 1) {
      //常规编译，捕捉当前行是否为特殊编译
      if (lineItem === "return(") {
        //开始输出模板
        compileStack.unshift(3)
      } else if (
        lineItem.startsWith("const[") &&
        lineItem.includes("useState")
      ) {
        //如果是状态声明
        const {
          returnCodeLine,
          returnAllStateList,
          returnCompileStack,
          returnReactFileHasStateType,
        } = saveState(
          lineItem,
          allStateList,
          compileStack,
          reactFileHasStateType
        )
        vueScriptList.push(returnCodeLine)
        allStateList = returnAllStateList
        compileStack = returnCompileStack
        reactFileHasStateType = returnReactFileHasStateType
      } else if (lineItem.startsWith("useEffect")) {
        //副作用函数
        compileStack.unshift(2)
      } else {
        //被舍弃的，跳过循环，不做编译，一些特殊react专属语法，如export default function Index()/import './index.module.less';
      }
    } else if (compileStack.peek() === 2) {
      //副作用函数编译中
      const saveCodeResult = saveCodeInUseEffect(
        allStateList,
        lineItem,
        compileStack
      )
      if (saveCodeResult.action === "unmounted") {
        //调度到unmounted
        compileStack = saveCodeResult.compileStack
      } else {
        //仍然在执行mounted任务
        if (lineItem.startsWith("},[])")) {
          //mounted结束，批量插入
          mountedContainer = "\nonMounted(() => {\n" + mountedContainer + "})\n"
          vueScriptList.push(mountedContainer)
          vueScriptList.push(unMountedContainer)
          mountedContainer = ""
          unMountedContainer = ""
          compileStack.shift()
          if (!reactFileHasStateType.includes(3)) {
            reactFileHasStateType.push(3)
          }
        } else if (lineItem.startsWith("},[")) {
          //编译成watch函数，批量插入
          const params = lineItem
            .split("[")[1]
            .split("]")[0]
            .split(",")
          vueScriptList.push(formatWatchToVue(params, mountedContainer))
          if (!reactFileHasStateType.includes(4)) {
            reactFileHasStateType.push(4)
          }
          compileStack.shift()
        } else {
          mountedContainer += saveCodeResult
        }
      }
    } else if (compileStack.peek() === 3) {
      //模板编译中
      lineItem = lineItem.replace("className", "class")
      if (lineItem.replaceAll(" ", "") === "{") {
        //jsx语法编译开始
        compileStack.unshift(6) //jsx语法编译标识符6入栈，作为最高优先级
      } else if (
        lineItem.includes(")") &&
        lineItem.replace(/\s*/g, "") === ")"
      ) {
        //模板输出结束
        compileStack.shift()
      } else if (lineItem.includes("{") && lineItem.includes("}")) {
        //带状态的模板
        vueTemplateList.push(formatStateInTemplate(lineItem))
      } else {
        vueTemplateList.push(lineItem)
      }
    } else if (compileStack.peek() === 4) {
      //复杂状态编译中
      if (lineItem.includes(")")) {
        // 编译结束
        stateContainer += `\n${lineItem}`
        vueScriptList.push(stateContainer)
        stateContainer = ""
        compileStack.shift()
      } else {
        stateContainer += `${lineItem}`
      }
    } else if (compileStack.peek() === 5) {
      //unmounted函数编译中
      if (lineItem.startsWith("}")) {
        //可能unmounted结束了，需要先判断是否是块作用域
        const startIconNum = unMountedContainer
          .split("{")
          .filter((item) => item === "").length
        const endIconNum = unMountedContainer
          .split("}")
          .filter((item) => item === "").length
        if (startIconNum === endIconNum) {
          //执行unmounted
          compileStack.shift()
          unMountedContainer =
            "onUnmounted(() => {\n" + unMountedContainer + "})\n"
          if (!reactFileHasStateType.includes(5)) {
            reactFileHasStateType.push(5)
          }
        }
      } else {
        unMountedContainer += saveCodeInUnmounted(
          allStateList,
          lineItem,
          compileStack
        )
      }
    } else if (compileStack.peek() === 6) {
      //模板中jsx语法编译中
      if (lineItem.replaceAll(" ", "") === "}") {
        //jsx编译结束
        //格式化遍历项dom
        const { mapArray, mapFnParams, key, mapDomType } = jsxCompileParams
        if (!key) {
          //遍历元素的key值为必填项，否则会影响渲染
          throw new Error("please setting the map dom key!")
        }
        jsxContainer = jsxContainer.substr(
          0,
          jsxContainer.length - (jsxCompileParams.mapDomType.length + 6)
        )
        vueTemplateList.push(
          `<${mapDomType} v-for='${mapFnParams} in ${mapArray}' ${key &&
            ":key='" + key + "'"}>`
        )
        vueTemplateList.push(jsxContainer)
        vueTemplateList.push(`</${mapDomType}>`)
        //初始化，用于下一次任务6的处理
        jsxContainer = ""
        jsxCompileParams = {}
        //弹栈，将任务权交回给普通模板编译
        compileStack.shift()
      } else {
        //每行代码编译完，更新编译参数集
        const returnParams = compileJsxTemplate(lineItem, jsxCompileParams)
        jsxCompileParams = { ...returnParams.jsxCompileParams }
        if (returnParams.lineItem) {
          jsxContainer += returnParams.lineItem + "\n"
        }
      }
    }
  })

  vueTemplateList.unshift("<template>")
  vueTemplateList.push("</template>\n")
  vueScriptList.unshift("<script setup>")
  vueScriptList.push("</script>\n")

  //处理import ..... from 'vue'  的导入配置
  if (reactFileHasStateType.length) {
    //有状态
    let importVal = ""
    if (reactFileHasStateType.includes(1)) {
      importVal = "ref"
    }
    if (reactFileHasStateType.includes(2)) {
      importVal += ",reactive"
    }
    if (reactFileHasStateType.includes(3)) {
      importVal += ",onMounted"
    }
    if (reactFileHasStateType.includes(4)) {
      importVal += ",watch"
    }
    if (reactFileHasStateType.includes(5)) {
      importVal += ",onUnmounted"
    }
    vueScriptList.splice(1, 0, `import{${importVal}}from'vue'`)
  }

  resultFileTotalLine = computeFileLines(vueTemplateList, vueScriptList)

  let resultFile = ""
  vueTemplateList.forEach((line) => {
    resultFile += line + "\n"
  })
  vueScriptList.forEach((line) => {
    resultFile += line + "\n"
  })
  resultFile += `<style lang="${transformOptions.styleType}" scoped>\n`
  readFileFromLine("./index.module.less", (res) => {
    //写入样式
    if (res) {
      res.forEach((line) => {
        resultFile += line + "\n"
      })
      resultFile += "</style>"
      resultFileTotalLine += res.length + 3
      //保存文件
      fs.writeFile(transformOptions.outputPath, resultFile, (err) => {
        if (!err) {
          console.log("转换完成，vue文件共有", resultFileTotalLine, "行代码!")
          return
        }
      })
    }
  })
})
```

compile.js（index.js 中提供的工具函数，与 index.js 交互，处理代码段）

```typescript
const { formatUseStateAction } = require("./_utils/formatuseState")

const formatStateInTemplate = (lineItem) => {
  //编译模板中的{{state}}
  let startIndex = lineItem.indexOf("{")
  let endIndex = lineItem.indexOf("}")
  lineItem = lineItem.split("")
  lineItem.splice(startIndex + 1, 0, "{")
  lineItem.splice(endIndex + 2, 0, "}")
  lineItem = lineItem.join("")
  return lineItem
}

const saveCodeInUseEffect = (allStateList, lineItem, compileStack) => {
  //在useEffect中保存代码片段
  if (lineItem.includes("return()=>")) {
    //组件销毁，把任务权先交给销毁
    compileStack.unshift(5)
    return { compileStack, action: "unmounted" }
  }
  allStateList.forEach((stateType, key) => {
    if (lineItem.includes(key.stateAction)) {
      //携带状态的特殊副作用函数语句
      let newState = lineItem.match(/\((.+?)\)/gi)[0].replace(/[(|)]/g, "")
      if (newState.includes(key.state)) {
        newState = newState.replace(key.state, `${key.state}.value`)
      }
      lineItem = `${key.state}${
        stateType === "ref" ? ".value" : ""
      } = ${newState}`
    }
  })
  return `${lineItem}\n`
}
const saveCodeInUnmounted = (allStateList, lineItem, compileStack) => {
  //在unmounted中保存代码片段
  allStateList.forEach((stateType, key) => {
    if (lineItem.includes(key.stateAction)) {
      //携带状态的特殊副作用函数语句
      const newState = lineItem.match(/\((.+?)\)/gi)[0].replace(/[(|)]/g, "")
      lineItem = `${key.state}${
        stateType === "ref" ? ".value" : ""
      } = ${newState}`
    }
  })

  return `${lineItem}\n`
}
const formatWatchToVue = (params, code) => {
  const returnCode = `watch([${params}],(${new Array(params.length)
    .fill("")
    .map((item, index) => {
      return `[${"oldValue" + index}, ${"newValue" + index}]`
    })})=>{\n${code}})`
  return returnCode
}

const saveState = (
  lineItem,
  allStateList,
  compileStack,
  reactFileHasStateType
) => {
  //保存状态
  //处理useState hook
  const stateKey = lineItem.split("[")[1].split(",")[0]
  const stateVal = lineItem.split("useState(")[1].split(")")[0] //状态值
  let returnCodeLine = ""
  if (!lineItem.includes(")")) {
    compileStack.unshift(4)
  }
  //判断state 类型，保存
  if (stateVal.startsWith("[") || stateVal.startsWith("{")) {
    returnCodeLine = `const ${stateKey}=reactive(${stateVal}${
      compileStack.peek() === 4 ? "" : ")"
    }`
    allStateList.set(
      { state: stateKey, stateAction: `set${formatUseStateAction(stateKey)}` },
      "reactive"
    )
    if (!reactFileHasStateType.includes(2)) {
      reactFileHasStateType.push(2)
    }
  } else {
    returnCodeLine = `const ${stateKey}=ref(${stateVal})`
    allStateList.set(
      { state: stateKey, stateAction: `set${formatUseStateAction(stateKey)}` },
      "ref"
    )
    if (!reactFileHasStateType.includes(1)) {
      reactFileHasStateType.push(1)
    }
  }
  const returnAllStateList = allStateList
  const returnCompileStack = compileStack
  const returnReactFileHasStateType = reactFileHasStateType
  return {
    returnCodeLine,
    returnAllStateList,
    returnCompileStack,
    returnReactFileHasStateType,
  }
}

const compileJsxTemplate = (lineItem, jsxCompileParams) => {
  //jsx模板编译
  lineItem = lineItem.replaceAll(" ", "")
  if (lineItem.includes(".map")) {
    //遍历渲染
    jsxCompileParams.mapArray = lineItem.split(".map")[0]
    jsxCompileParams.mapFnParams = lineItem.split(".map(")[1].split("=>")[0]
    return {
      jsxCompileParams,
    }
  } else if (lineItem === "return(") {
    return {
      jsxCompileParams,
    }
  } else if (lineItem.includes("<") && lineItem.includes("key")) {
    //存储遍历key值
    jsxCompileParams.key = lineItem.split("={")[1].split("}")[0]
    jsxCompileParams.mapDomType = lineItem.split("key")[0].split("<")[1]
    return {
      jsxCompileParams,
    }
  } else if (lineItem === ")" || lineItem === "})") {
    //jsx语法结束
    return {
      jsxCompileParams,
    }
  } else {
    //在key容器下的内层遍历子模板，保存，通常在第三行开始
    return {
      jsxCompileParams,
      lineItem: formatStateInTemplate(lineItem),
    }
  }
}

module.exports = {
  formatStateInTemplate,
  saveCodeInUseEffect,
  saveCodeInUnmounted,
  formatWatchToVue,
  saveState,
  compileJsxTemplate,
}
```

这里文中主要涉及到的源码就是这两个文件，具体的可以看一下 github。

## 总结

React 组件转换 Vue 组件的工具（react to vue）目前开工不久，博主在每一段大的变动时都会记录总结，也可以关注 github，更新的会比较频繁。

**github 地址：**[https://github.com/fengxinhhh/react-to-vue](https://github.com/fengxinhhh/react-to-vue)

**喜欢可以关注一下，有任何问题或者开发设计的意见也欢迎留言。**
