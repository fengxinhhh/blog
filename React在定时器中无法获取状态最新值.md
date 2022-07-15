**在做轮播图组件时发现了一个问题，在setInterval中无法通过状态直接获取最新值，如：**

```typescript
const [rightTransform, setRightTransform] = useState(pictureSize);

const autoPlay = () => {        //普通轮播自动播放
    timer.current = setInterval(() => {
      let oldState = rightTransform;
      console.log('老状态', oldState)			//始终只会打印初始的pictureSize
      setRightTransform(o => {
        const newState = JSON.parse(JSON.stringify(o));
        return newState >= (renderImgList.length) * pictureSize ? 0 : newState + pictureSize
      })
	}, 1000);
}
```
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68fd285d310c46979d57007a6864d5e7~tplv-k3u1fbpfcp-zoom-1.image)
## 问题原因
定时器一直都没有被清除，因此获取的状态始终是定时器被创建时候的状态。

## 问题解决
从代码和图片可以看到，定时器中打印的状态永远都是初始值，后面所改变的值虽然更新了，页面也发生了变化，但是我们从log中无法获取到实时状态。


**解决办法很简单，第一种，就是在setState中获取上一次状态，因为useState hooks提供了记录上一次状态的缓存回调，可以在这个回调中获取上一轮状态，如图：**
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/504801ebe3ce4e0d9c90a437890f7d3a~tplv-k3u1fbpfcp-zoom-1.image)

```typescript
timer.current = setInterval(() => {
      let oldState = rightTransform;
      setRightTransform(o => {
        console.log('在setState中的老状态', o)
        const newState = JSON.parse(JSON.stringify(o));
        return newState >= (renderImgList.length) * pictureSize ? 0 : newState + pictureSize
      })
      console.log('直接打印老状态', oldState)
}
```
