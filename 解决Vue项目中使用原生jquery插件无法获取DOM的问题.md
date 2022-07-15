## 问题场景

Vue 项目中有需求，是使用 jquery 的滚动插件，插件本质是在 window.onload 中获取滚动 DOM 并且对他进行一些操作。

## 发现问题

脚本中的 DOM 获取到的结果为 null，因此脚本也没有生效，如图：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7637a38a2964460db572ba015af43c25~tplv-k3u1fbpfcp-zoom-1.image)
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca4276a6b4554dd88db361c5f3ef0d14~tplv-k3u1fbpfcp-zoom-1.image)
项目目录是这样的：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e30ee4c2abc45cd9667408c92c0f4f0~tplv-k3u1fbpfcp-zoom-1.image)
我原来的理解：脚本在 app 容器下面，脚本执行中会获取到 DOM，但是仔细思考 Vue 的渲染原理，其实 Vue 在生成虚拟 DOM 之后是进行真实 DOM 的异步渲染，为了性能角度，并且在修改完状态后也是异步的方式进行渲染，想要在 Vue 中获取 DOM 可以使用 nextTick。
而这些插件其实都是同步代码，在 Vue 的源码初始化结束后，就已经开始进行脚本了，此时的应用事件队列应该是这样的：
异步队列：Vue 根据虚拟 DOM 生成真实 DOM，可以获取到某些已经挂载好的 DOM；
同步队列：我们正在逐个执行的 js 脚本。

因此会出现在脚本中无法获取 DOM 的情况。

## 解决方案

了解了原理后，解决方案其实就有了思路，既然脚本中的获取 DOM 是同步获取，那不妨将它变成异步获取 DOM，进行一个监听，如果没有 DOM，在下一次监听时再次尝试获取，获取到了以后进行实际的插件业务，插件中修改后的代码如下：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5d8c2c7a9b5416198929513f2c1f5ae~tplv-k3u1fbpfcp-zoom-1.image)

## 总结

其实项目耦合度往往很高，在一些特定情况下会提供给开发者了解项目技术栈的一些设计原理（源码），如果博主对于 Vue 的源码有了比较深刻的认识，对于此类问题也会迎刃而解，因此在开发的业余时间还是要保持学习源码，学习设计思路的习惯。
