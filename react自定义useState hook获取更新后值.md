**在业务中有比较多的场景需要在 setState 中获取更新后的值从而进行下一步的业务操作,在 Class 组件中可以通过：**

```typescript
this.setState(
  {
    name: "123",
  },
  (newVal) => {
    console.log(newVal)
  }
)
```

而 react 自身的 useState hook 并不支持这样做，通常是这样获取新值：

```typescript
useEffect(() => {
  console.log(name)
}, [name])
```

自定义 useState 主要思路其实是基于 useState 和 useEffect，在 useEffect 获取到状态变化时回调 callback 函数，从而拿到新值，直接上代码~

```typescript
import { SetStateAction, useCallback, useState, useEffect, useRef } from "react"

const useStateCallback = (defaultVal: any) => {
  const [state, setState] = useState(defaultVal)
  const listenRef = useRef<any>() //监听新状态的回调器
  const _setState = useCallback(
    (newVal: SetStateAction<any>, callback: Function) => {
      //更新业务
      listenRef.current = callback
      setState(newVal)
    },
    []
  )
  useEffect(() => {
    listenRef.current && listenRef.current(state) //回调新状态
  }, [state])

  return [state, _setState]
}

export default useStateCallback
```

其实 hook 本身就是多了一步在更新状态时把自定义的 callback 丢给 useRef，让 useRef 长久保留在内存中，并且监听状态，在状态更新时调用内存中（useRef）中的回调函数，从而实现。

hooks 其实本身并非高深的东西，其实 hooks 就是函数，而自定义 hook 其实就是基于 react 核心的一些 hooks 去根据我们的业务实现一些和状态有关的工具函数。
