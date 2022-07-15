## useEffect 介绍

useEffect 时 reactHook 中最重要，最常用的 hook 之一。

useEffect 相当于 react 中的什么生命周期呢？
这个问题在 react 官网中有过介绍，在使用的过程中，容易被忽略，在面试的时候却经常被问及，（面试造航母，上班拧螺丝？），开个玩笑这个问题并不难回答，下面是 react 官方的原话：
如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

componentDidMount 组件挂载
componentDidUpdate 组件更新
componentWillUnmount 组件将要摧毁

useEffect 需要传递两个参数，第一个参数是逻辑处理函数，第二个参数是一个数组
用法

```javascript
useEffect(() => {
  /** 执行逻辑 */
}, [])
```

**重要理解**
一、第二个参数存放变量，当数组存放变量发生改变时，第一个参数，逻辑处理函数将会被执行
二、第二个参数可以不传，不会报错，但浏览器会无线循环执行逻辑处理函数。

```javascript
useEffect(() => {
  /** 执行逻辑 */
})
```

三、第二个参数如果只传一个空数组，逻辑处理函数里面的逻辑只会在组件挂载时执行一次 ，不就是相当于 componentDidMount

```javascript
useEffect(() => {
  /** 执行逻辑 */
}, [])
```

四、第二个参数如果不为空数组，如下

```javascript
const [a, setA] = useState(1)
const [b, setB] = useState(2)
useEffect(() => {
  /** 执行逻辑 */
}, [a, b])
```

逻辑处理函数会在组件挂载时执行一次和(a 或者 b 变量在栈中的值发生改变时执行一次) 这是不是相当于 componentDidMount 和 componentDidUpdate 的结合
五、useEffect 第一个参数可以返回一个回调函数，这个回调函数将会在组件被摧毁之前和再一次触发更新时，将之前的副作用清除掉。这就相当于 componentWillUnmount。

useEffect 去除副作用。我们可能会在组件即将被挂载的时候创建一些不断循环的订阅（计时器，或者递归循环）。在组件被摧毁之前，或者依赖数组的元素更新后，应该将这些订阅也给摧毁掉。

比如以下的情况（没有去除计时器，增大不必要的开销和代码风险)

```javascript
const [time, setTime] = useState(0)

useEffect(() => {
  const InterVal = setInterval(() => {
    setTime(time + 1)
  }, 1000)
}, [])
```

利用第五点，在组件被摧毁前去除计时器。

```javascript
const [time, setTime] = useState(0)

useEffect(() => {
  const InterVal = setInterval(() => {
    setTime(time + 1)
  }, 1000)
  return () => {
    clearInterval(InterVal)
  }
}, [])
```

## useEffect 常见跳坑

**1、useEffect 执行函数被循环执行。**
出现这种情况可能有两种原因。

没传第二个参数

```javascript
useEffect(() => {
  /** 执行逻辑 */
})
```

**2、你在 useEffect 执行函数里面改变了 useEffect 监测的变量**

```javascript
const [a, setA] = useState(1)
useEffect(() => {
  /** 执行逻辑 */
  setA(a + 1)
}, [a])
```

解决的方法 不要在 useEffect 第一参数执行函数中去改变第二参数依赖元素的地址的值。当依赖元素的地址的值发生改变，又会执行一次执行函数，这不是无限循环么。

**3、useEffect 监测不到依赖数组元素的变化。**
只有一种可能，依赖数组元素的地址的值根本就没变，比如:

```javascript
const [a, setA] = useState({
  b: "dx",
  c: "18",
})

const changeA = () => {
  setA((old) => {
    old.b = "yx"
    return old
  })
}

useEffect(() => {
  /** 当组件挂载时执行一次changeA */
  changeA()
}, [])

/**当changeA执行却没有打印 a*/
useEffect(() => {
  /** 执行逻辑 */
  console.log(a)
}, [a])
```

是因为 changeA 没有真正的改变 a 在栈中的值（地址的值），只是改变了 a 在堆中的值。
useEffect 监测不到堆中值得变化，所有引用类型数据都应该注意这一点。
解决的办法：

```javascript
const [a, setA] = useState({
  b: "dx",
  c: "18",
})

const changeA = () => {
  setA((old) => {
    const newA = { ...old }
    newA.b = "yx"
    return newA
  })
}

useEffect(() => {
  /** 当组件挂载时执行一次changeA */
  changeA()
}, [])

/**当changeA执行打印  {b:'yx',c:'18'}  */
useEffect(() => {
  /** 执行逻辑 */
  console.log(a)
}, [a])
```
