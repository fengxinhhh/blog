**useImperativeHandle 是 react 官方为了简便我们的 ref 操作，同时还可以让子组件返回给父组件自身的状态和方法去调用**

useRef 将 ref 绑定到某个子组件标签上，用以获取整个子组件的方法和参数
useImperativeHandle: 可以自定义暴露给父组件的方法或者变量

父组件

```javascript
import React, { useState, useRef } from "react"
import ChildList from "./ChildList"

export default () => {
  let parentRef = useRef(null)
  const [name, setName] = useState("li")
  return (
    <div>
      <ChildList parentRef={parentRef} name={name}></ChildList>
      <button
        onClick={() => {
          console.log("parentRef", parentRef)
        }}
      >
        获取子组件
      </button>
    </div>
  )
}
```

子组件

```javascript
import React, { useImperativeHandle, forwardRef } from "react"

export default forwardRef((props, ref) => {
  console.log("ref", ref)
  useImperativeHandle(ref, () => {
    return {
      childFn,
    }
  })
  console.log(ref)
  const childFn = () => {
    console.log("子组件方法")
  }
  return (
    <div>
      <div ref={ref} />
    </div>
  )
})
```

用了 ref 的桥梁就这样就可以在父组件中控制子组件了，而不仅仅只是子随父渲染了。
