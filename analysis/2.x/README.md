# vue-analysis for 2.x

## vue 实例化流程总结

1. $mount 流程开始
2. 获取 render，或将 template 转换成 render
3. 调用 render 生成 VNode
4. 调用 update 触发 patch 生成组件，并 insert DOM

![](https://ustbhuangyi.github.io/vue-analysis/assets/new-vue.png)


## 相关资料

[源码资料](https://ustbhuangyi.github.io/vue-analysis)
