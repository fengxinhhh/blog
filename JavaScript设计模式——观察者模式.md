# 前言

观察者模式（也称发布订阅模式）是 JavaScript 中非常常见的设计模式，可以实现页面中的消息机制的监听，也是 Vue、React 主流框架实现的数据响应手段，解决了主体对象之间的解耦，今天来实现一下。

# Dep 发布者

```javascript
class Dep {
  //发布者（商店）
  constructor(shopMoney, shopName) {
    this.shopMoney = shopMoney
    this.shopName = shopName
    this.observers = []
  }
  setState(state) {
    //状态改变，通知订阅
    this.shopMoney = state
    this.noticy()
  }
  addObserver(ob) {
    //添加订阅者
    this.observers.push({
      name: ob.name,
      constructor: ob,
    })
  }
  noticy() {
    //状态改变，发布
    this.observers.forEach((ob) => {
      ob.constructor.update(
        `${ob.name}你好，${this.shopName}价格更新了，价格为${this.shopMoney}`
      )
    })
  }
}
```

这里是发布者构造函数，包含商品初价，商品名称，订阅者列表，之后的订阅者创造实例时会用到 addObserver 方法，noticy 方法用于通知每一个订阅者（observers）

# Observer 订阅者

```javascript
class Observer {
  //订阅者（顾客）
  constructor(name, dep) {
    this.name = name
    this.dep = dep
    this.dep.forEach((d) => {
      d.addObserver(this)
    })
  }
  update(val) {
    //改变通知
    console.log(val)
  }
}
```

这里订阅者可以同时订阅多个 dep，并且在实例化时调用 dep 实例的 addObserver 方法。
这样就形成 dep 和 observer 的关联性。

# 案例

现在我们创造一家猪肉店和一家鞋店，并且可以通过 input 改变价格，页面如下：

![![在这里插入图片描述](https://img-blog.csdnimg.cn/db7c0cf0dd454458b179ef34f4ebce65.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29770bee355241d1b17b5b74f97dfe77~tplv-k3u1fbpfcp-zoom-1.image)
我们通过一个 input 函数来作为发布者改变商品价格的入口。

```javascript
<span>鞋子价格</span><input type="text" oninput="changeMoney(event, 1)" value="500">
<span>猪肉价格</span><input type="text" oninput="changeMoney(event, 2)" value="30">

<script>
function changeMoney(e, id) {
  id === 1
  ?
  dep1.setState(e.target.value)
  :
  dep2.setState(e.target.value)
}
</script>
```

同时构建两个 dep 实例和两个 observer 实例：

```javascript
const dep1 = new Dep(500, "鞋子")
const dep2 = new Dep(30, "猪肉")
const ob1 = new Observer("顾客1", [dep1, dep2])
const ob2 = new Observer("顾客2", [dep1])
```

输入 input，结果如下：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b557ceb62034582a81f80ec1a1a3f29~tplv-k3u1fbpfcp-zoom-1.image)

这样，一个简易的发布订阅模式就完成了。

# 源码

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <span>鞋子价格</span><input type="text" oninput="changeMoney(event, 1)" value="500">
  <span>猪肉价格</span><input type="text" oninput="changeMoney(event, 2)" value="30">
</body>
<script>
  class Dep {         //发布者（商店）
    constructor(shopMoney, shopName) {
      this.shopMoney = shopMoney;
      this.shopName = shopName;
      this.observers = [];
    }
    setState(state) {       //状态改变，通知订阅
      this.shopMoney = state;
      this.noticy();
    }
    addObserver(ob) {       //添加订阅者
      this.observers.push({
        name: ob.name,
        constructor: ob
      });
    }
    noticy() {            //状态改变，发布
      this.observers.forEach(ob => {
        ob.constructor.update(`${ob.name}你好，${this.shopName}价格更新了，价格为${this.shopMoney}`)
      })
    }
  }
class Observer {        //订阅者（顾客）
  constructor(name, dep) {
    this.name = name;
    this.dep = dep;
    this.dep.forEach(d => {
      d.addObserver(this);
    })
  }
  update(val) {       //改变通知
    console.log(val)
  }
}

const dep1 = new Dep(500, '鞋子');
const dep2 = new Dep(30, '猪肉');
const ob1 = new Observer('顾客1', [dep1, dep2]);
const ob2 = new Observer('顾客2', [dep1]);

function changeMoney(e, id) {
  id === 1
  ?
  dep1.setState(e.target.value)
  :
  dep2.setState(e.target.value)
}
</script>
</html>
```

# 总结

设计模式真的很有意思，可以感受到 JavaScript 的灵活性，也可以为我们平时写业务逻辑提供更多的思维和解决方式。
