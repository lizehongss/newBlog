---
title: vue数据驱动原理
date: 2019/5/15
categories:
  - 源码学习
tags: 
  - vue
---

Vue的定义
```
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
```
function Vue (options) {
  //...
  this._init(options)
}
```
new Vue()-> 调用this._init(options)
this._init初始化各种配置，包括初始化生命周期，初始化data,props, computed, watcher等等-> 最后调用vm.$mout(vm.$options.el)挂载vm.
带 compiler 版本的 $mount 实现
```
Vue.prototype.$mount = function(//...){
  // 如this.$options.render不存在，调用compileToFunctions方法转换成render方法
        const { render, staticRenderFns } = compileToFunctions(template, {
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns
  // 调用原先原型上的 $mount 方法挂载。
  return mount.call(this, el, hydrating)
}
```
调用原先原型上的 $mount 方法定义如下:
```
// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
调用mountComponent(this, el, hydrating)
```
// ....
// 实例化渲染Wathcher
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  // 完成整个渲染工作
  return vm

```
new Wathcer 在初始化时会执行回调函数updateComponent且 vm 实例中的监测的数据发生变化的时候执行回调函数
```
// 在watcher定义中, this.getter为传入的updateComponent
get () {
  // ...
  value = this.getter.call(vm, vm)
}
```
在整个过程中核心是vm._render和vm._update
```
  updateComponent = () => {
    vm._update(vm._render(), hydrating)
  }
```
vm._render()用来把实例渲染成一个虚拟 Node，也就是VNode->调用vnode = render.call(vm._renderProxy, vm.$createElement)
vm.$createElement定义如下：
```
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```