# vue-analysis

[源码资料](https://ustbhuangyi.github.io/vue-analysis)

## 各章总结

### 准备工作

#### Vue.js 源码目录设计

```
src
├── compiler        # 编译相关
├── core            # 核心代码
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码
```

#### Vue.js 源码构建

* `Runtime Only` 需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。
* `Runtime + Compiler` 没有对代码做预编译，在 Vue.js 2.0 中，最终渲染 template 都是通过 render 函数，那么这个编译过程会发生运行时，所以需要带有编译器的版本。很显然，这个编译过程对性能会有一定损耗。

#### Vue 的入口

Vue 本质上就是一个用 Function 实现的 Class，通过各种 mixin 扩展 prototype、通过 initGlobalAPI 扩展 static。


### 数据驱动

#### new Vue 发生了什么

Vue 初始化主要就干了几件事情：合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化 data、props、computed、watcher 等。

在初始化的最后，检测到如果有 el 属性，则调用 vm.$mount 方法挂载 vm。

#### Vue 实例挂载的实现（即 $mount 的实现）

1. 首先对 el 做了限制，Vue 不能挂载在 body、html 这样的根节点上。
2. 如果没有定义 render 方法，则会把 el 或者 template 转换成 render 方法。因为在 Vue 2.x 中，所有 Vue 的组件的渲染最终都需要 render 方法。
3. 调用 mountComponent 方法完成渲染工作。

> mountComponent 方法的核心是先实例化一个渲染 Watcher，在它的回调函数中会调用 updateComponent 方法。
> 在此方法中调用 vm._render 方法先生成虚拟 Node，最终调用 vm._update 更新 DOM。

> Watcher 在这里起到两个作用，一是初始化的时候会执行回调函数，二是当 vm 实例中的监测的数据发生变化的时候执行回调函数。

#### 
