**在做组件库的过程中，测试项目引入线上组件发现状态的修改执行了两次，而组件项目中只执行一次（正常），分析原因后发现组件项目未开启严格模式，而测试项目开启了严格模式，代码如下：**

```typescript
ReactDOM.render(
  <React.StrictMode>
    <App />,
  </React.StrictMode>,
  document.getElementById("root")
)
```

解决方案有两种： 1.关闭严格模式

```typescript
ReactDOM.render(<App />, document.getElementById("root"))
```

2.深拷贝需要修改的状态变量

```typescript
this.setState((state) => {
  // 严格模式下, 深拷贝可解决setState被执行两次
  let temp = JSON.parse(JSON.stringify(state.aTodolist))
  temp.splice(i, 1)
  console.log(i, temp)
  return {
    aTodolist: temp,
  }
})
```

**最后贴一下官方的解释：**
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cb5c8a2e0a5426da8d110edd606c96e~tplv-k3u1fbpfcp-zoom-1.image)
