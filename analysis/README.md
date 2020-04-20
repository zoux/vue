# vue-analysis

## vue 实例化流程总结

1. $mount 流程开始
2. 获取 render，或将 template 转换成 render
3. 调用 render 生成 VNode
4. 调用 update 触发 patch 生成组件，并 insert DOM

![](https://ustbhuangyi.github.io/vue-analysis/assets/new-vue.png)



## 相关资料

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

mountComponent 方法的核心是先实例化一个渲染 Watcher，在它的回调函数中会调用 updateComponent 方法。
在此方法中调用 vm._render 方法先生成虚拟 Node，最终调用 vm._update 更新 DOM。

> Watcher 在这里起到两个作用，一是初始化的时候会执行回调函数，二是当 vm 实例中的监测的数据发生变化的时候执行回调函数。

#### render

vm._render 最终是通过执行 createElement 方法并返回的是 VNode。

#### Virtual DOM

在 Vue.js 中，Virtual DOM 是用 VNode 这个 Class 去描述的。VNode 借鉴了一个开源库 snabbdom 的实现，然后加入了一些 Vue.js 特色的东西。

VNode 是对真实 DOM 的一种抽象描述，它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等，其它属性都是用来扩展 VNode 的灵活性以及实现一些特殊 feature 的。

由于 VNode 只是用来映射到真实 DOM 的渲染，不需要包含操作 DOM 的方法，因此它是非常轻量和简单的。

映射到真实的 DOM 实际上要经历 VNode 的 create、diff、patch 等过程。

#### createElement

createElement 会对参数 tag 进行判断：
* 如果是一个普通的 html 标签，像上一章的例子那样是一个普通的 div，则会实例化一个普通 VNode 节点。
* 否则通过 createComponent 方法创建一个组件 VNode。

#### update

_update 的作用是把 VNode 渲染成真实的 DOM。它被调用的时机有 2 个，一个是首次渲染，一个是数据更新的时候。

其核心就是调用 `vm.__patch__` 方法：
1. patch 主要调用了 createElm(通过虚拟节点创建真实的 DOM 并插入到它的父节点中)。
2. createElm 主要调用了 createComponent(尝试创建子组件)、createChildren(创建子元素)、insert(把 DOM 插入到父节点中)。

> createChildren 实际上是遍历子虚拟节点，递归调用 createElm，这是一种常用的深度优先的遍历算法。

> 因为是递归调用，子元素会优先调用 insert，所以整个 vnode 树节点的插入顺序是先子后父。


### 组件化

#### createComponent

createComponent 后返回的是组件 vnode，它也一样走到 vm._update 方法，进而执行了 patch 函数。

createComponent 的关键步骤：
1. 构造子类构造函数。`Ctor = baseCtor.extend(Ctor)`
2. 安装组件钩子函数。用于在 VNode 执行 patch 的过程中执行相关的钩子函数。`installComponentHooks(data)`
3. 实例化 vnode。
    ```javascript
    const vnode = new VNode(
      `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
      data, undefined, undefined, undefined, context,
      { Ctor, propsData, listeners, tag, children },
      asyncFactory
    )
    return vnode
    ```

> Vue.extend 的作用就是构造一个 Vue 的子类。

#### patch

组件同样从 `vm.__patch__` 开始，经过 createElm、createComponent、init(利用 createComponentInstanceForVnode 创建一个 Vue 的实例，然后调用 child.$mount 方法挂载子组件)。

> createComponentInstanceForVnode 中的 vnode.componentOptions.Ctor 对应的就是子组件的构造函数；
> activeInstance 为当前激活的 vm 实例；vm.$vnode 为组件的占位节点的 VNode；vm._vnode 为组件的真实渲染 VNode。

> 创建实例时通过触发 Vue.prototype._init 中的 initInternalComponent 来给组件的 $options 赋值。

> createComponent 遍历子节点，若为非组件节点则 createChildren 创建子元素，若为组件节点则递归调用 createComponent。

#### 合并配置

options 的合并有 2 种方式，子组件初始化过程通过 initInternalComponent 方式要比外部初始化 Vue 通过 mergeOptions 的过程要快，合并完的结果保留在 vm.$options 中。

#### 生命周期


