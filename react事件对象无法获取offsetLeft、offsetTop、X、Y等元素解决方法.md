## react 获取到的事件缺少了部分一些属性，和原生事件对象不同，如图：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f89ac1f8ab51488b9b3cef4eb01cd4dd~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd5b6f1ea42c4b898fd88aa521c39e6b~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb47d3d83dd246738a67380330130e8c~tplv-k3u1fbpfcp-zoom-1.image) \*

## 解决方法

事件中使用：

```typescript
e.nativeEvent
```

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c05adcade4a94fc5b8ed4e7825bebf06~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f56a0a6e216e4fda9a6e75d950d61a6d~tplv-k3u1fbpfcp-zoom-1.image)
这是一个比较坑的地方，应该是 react 的事件对象没有包含一些原生 eventDom 的属性。
