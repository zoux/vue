# vue-analysis for 2.x

## new Vue 初始化过程

1. $mount 流程开始
2. 获取 render，或将 template 编译(compile)成 render
3. 调用 render 生成 VNode
4. 调用 update 触发 patch 生成组件，并 insert DOM

![](https://ustbhuangyi.github.io/vue-analysis/assets/new-vue.png)


## 响应式原理

![](https://ustbhuangyi.github.io/vue-analysis/assets/reactive.png)


## 相关资料

[源码资料](https://ustbhuangyi.github.io/vue-analysis)
