## 有半年没有复习 js 的一些基础了，今天写一下 js 中的改变 this 指向的方法，更好的复习原理

## 毕竟...咱们是工程师呀，不能天天调用~

call 方法：

```javascript
var obj = { name: "fx" }
function fn(a, b, c) {
  console.log(this.name)
  console.log(a, b, c)
}
Function.prototype.myCall = function() {
  //call
  const fn = this || window //取方法
  const params = [...arguments].length > 1 ? [...arguments].slice(1) : "" //取参数列表
  const obj = [...arguments][0] //取代理对象
  obj.fn = fn
  ;(params && obj.fn(...params)) || obj.fn() //执行
  delete obj.fn
}
Function.prototype.myApply = function() {
  //apply
  const fn = this || window //取方法
  const arrayParams = [...arguments].length > 1 ? [...arguments][1] : "" //取参数
  const obj = [...arguments][0] //取代理对象
  obj.fn = fn
  ;(arrayParams && obj.fn(...arrayParams)) || obj.fn() //执行
  delete obj.fn
}
Function.prototype.myBind = function() {
  //bind
  const fn = this || window //取方法
  const _arg = [].slice.call(arguments)
  const obj = _arg[0] //取对象
  const _argParams = _arg.length > 1 && _arg.slice(1) //取第一轮参数
  return (...p) => {
    const params = [...p].length ? [...p] : [] //取第二轮参数
    const allParams = [..._argParams, ...params] //合并两次调用的参数
    return fn.apply(obj, allParams)
  }
}
fn.myCall(obj, 1, 2, 3)
fn.myApply(obj, [1, 2, 3])
fn.myBind(obj, 1, 2)(3)
```

call 和 apply 其实很类似，只是处理参数列表的时候有微差，而 bind 其实就是注意一下要取两次调用的执行参数合并以后传给调用宿主即可。
