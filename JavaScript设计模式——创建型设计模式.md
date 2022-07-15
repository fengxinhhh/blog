# 简单工厂模式

抽象各个对象的共同点，加工出最初形态，对于不同点进行独立设计。

```javascript
function createBook(name, time, type) {
  //创建一个对象，并对对象拓展属性和方法
  const o = new Object()
  //共同参数
  o.name = name
  o.time = time
  o.getName = function() {
    console.log(this.name)
  }
  //差异性
  if (type === "js") {
    //js类书差异处理
  } else if (type === "css") {
    //css类书差异处理
  } else if (type === "html") {
    //html类书差异处理
  }
  return o
}

const book1 = createBook("js book", 2014, "js")
const book2 = createBook("css book", 2013, "css")
const book3 = createBook("html book", 2012, "css")

book1.getName()
book2.getName()
book3.getName()
```

# 安全工厂模式

安全工厂模式可以屏蔽对类的错误使用，如忘记写 new 实例化对象避免产生一些非预期的行为。

```javascript
function Factory(type, content) {
  //如果忘记写new实例化了，加工重新执行一次
  if (this instanceof Factory) {
    this[type](content)
  } else {
    new Factory(type, content)
  }
}
Factory.prototype = {
  JavaScript: (content) => {
    console.log(content)
  },
  Java: (content) => {
    console.log(content)
  },
}

Factory("JavaScript", "JavaScript哪家强")
Factory("Java", "Java哪家强")
```

# 建设者模式

顾名思义，结合多个类，创造出结合出来的终极人物（对象）

```javascript
//创建人类
function Human(param) {
  //技能
  this.skill = (param && param.skill) || "保密"
  //兴趣爱好
  this.hobby = (param && param.hobby) || "保密"
}
Human.prototype.getSkill = function() {
  return this.skill
}
Human.prototype.getHobby = function() {
  return this.hobby
}
//创建姓名类
function Name(name) {
  ;(function(_this, name) {
    _this.wholeName = name
    _this.firstName = name.slice(0, name.indexOf(" "))
    _this.secordName = name.slice(name.indexOf(" "))
  })(this, name)
}
//创建职位类
function Work(work) {
  ;(function(_this, work) {
    switch (work) {
      case "code":
        _this.work = "工程师"
        _this.workDescript = "每天沉醉于编程"
        break
      case "UI":
      case "UE":
        _this.work = "设计师"
        _this.workDescript = "设计更是一种艺术"
        break
      case "teach":
        _this.work = "教师"
        _this.workDescript = "分享页是一种快乐"
        break
      default:
        _this.work = work
        _this.workDescript = "对不起，我们还不清楚您所选择职位的相关描述"
    }
  })(this, work)
}
//创建建设者类
function Person(name, work) {
  var person = new Human()
  person.name = new Name(name)
  person.work = new Work(work)
  return person
}

const person = new Person("xiao ming", "code")
console.log(person.skill) //保密
console.log(person.work.workDescript) //每天沉醉于编程
console.log(person.name.firstName) //xiao
```

这里结合了 Humer、Name、Work，最后在 Person 中构造出了一个应聘者。

# 原型模式

用原型实例指向创建对象的类，适用于创建新的对象的类共享原型对象的属性以及方法。
简而言之，就是根据一个基类（原型类）构造出多个子类，将公用方法和属性保存在原型类中。

```javascript
//图片轮播基类
function LoopImages(imgArr, container) {
  this.imgArr = imgArr
  this.container = container
}
LoopImages.prototype = {
  //创建轮播图片
  createImage: function(img) {
    this.imgArr.push(img)
    console.log("LoopImages createImage function")
  },
  //切换下一张图片
  changeImage: () => {
    console.log("LoopImages changeImage function")
  },
}
//上下滑动切换类
function SlideLoopImg(imgArr, container) {
  LoopImages.call(this, imgArr, container)
}
SlideLoopImg.prototype = new LoopImages()
//重写继承的切换下一张图片方法
SlideLoopImg.prototype.changeImage = () => {
  console.log("SlideLoopImg changeImage function")
}

//渐隐切换类
function FadeLoopImg(imgArr, container, arrow) {
  LoopImages.call(this, imgArr, container)
  this.arrow = arrow
}
FadeLoopImg.prototype = new LoopImages()
//重写继承的切换下一张图片方法
FadeLoopImg.prototype.changeImage = () => {
  console.log("FadeLoopImg changeImage function")
}

const fade = new FadeLoopImg(["01.jpg", "02.jpg", "03.jpg", "04.jpg"], "div", [
  "left.jpg",
  "right.jpg",
])
console.log(fade.arrow) //['left.jpg','right.jpg']
console.log(fade.imgArr) //['01.jpg', '02.jpg', '03.jpg', '04.jpg']
fade.createImage("05.jpg") //LoopImages createImage function
console.log(fade.imgArr) //['01.jpg', '02.jpg', '03.jpg', '04.jpg', '05.jpg']
fade.changeImage() //FadeLoopImg changeImage function
```

可以看到，原型模式可以让多个对象分享同一个原型对象的属性与方法，这也是一种继承方式。
原型对象更加适合在创建复杂的对象时，对于那些需求一直在变化而导致对象结构不停改变时，将那些比较稳定的属性与方法公共提取，实现继承，代码复用。

# 单例模式

单例模式，顾名思义，一个类只能有一个实例，重复实例化只会返回第一次实例的对象。

## 静态变量

通过一个立即执行函数，将所有方法挂载，并且为静态变量，无法改变。

```javascript
const React = (function() {
  var react = {
    useState: () => {},
    useEffect: () => {},
    useRef: () => {},
    useMemo: () => {},
    useCallback: () => {},
    useReducer: () => {},
    useContext: () => {},
  }
  return {
    get: function(name) {
      return react[name] || null
    },
  }
})()

console.log(React.get("useState"))
```

## 实例化单例

创建一个立即执行函数，利用闭包，使记录值常驻在内存中，每次实例化的时候判断是否为第一次实例化，实现单例。

```javascript
const React = (function() {
  let instance = null
  return function(params) {
    if (instance) {
      return instance
    }
    this.params = params
    return (instance = this)
  }
})()

console.log(
  new React({
    useState: () => {},
    useEffect: () => {},
    useRef: () => {},
    useMemo: () => {},
    useCallback: () => {},
    useReducer: () => {},
    useContext: () => {},
  }) === new React("hh")
) //true
```

## 惰性单例

非实例化创建方式，也是一种延迟创建的方式。

```javascript
function React(fns) {
  this.fns = fns
}
React.instance = null
React.getFn = function() {
  console.log(this.fns)
}
React.getInstance = function(name) {
  if (!this.instance) {
    return (this.instance = new React(name))
  }
  return this.instance
}

console.log(
  React.getInstance({
    useState: () => {},
    useEffect: () => {},
    useRef: () => {},
    useMemo: () => {},
    useCallback: () => {},
    useReducer: () => {},
    useContext: () => {},
  }) === React.getInstance("xx")
) //true
```

# 总结

本文介绍了 JavaScript 中创建型设计模式，希望看过之后对读者开发中代码质量有所帮助，有所感悟。
