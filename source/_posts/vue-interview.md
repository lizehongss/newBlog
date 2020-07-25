---
title: vue相关原理面试题整理
date: 2019/8/4
categories:
  - 面试
tags: 
  - vue
---

## vue响应式原理实现 

主要通过Object.defineProperty实现，通过它对对象的属性进行getter和setter,从而实现访问和修改这个属性时，通过getter和setter实现依赖注入和派发更新。依赖注入和派发更新通过订阅-观察者模式实现，具体如下, 在访问属性时，通过getter给dep添加watcher类，setter时触发dep.notify()方法，使订阅了这个属性所实例化的dep的所有Watcher触发update方法

## vue监测数组变化
vue为了监测数组的变化，重新实现了Array的原型方法，包括push,prop等方法,重写的方法会执行本身原有的逻辑，对能增加数组本身长度的方法获取到插入的值，成一个响应式对象，并且再调用 ob.dep.notify() 手动触发依赖通知

## nextTick实现原理
在下次 DOM 更新循环结束之后执行延迟回调。nextTick主要使用了宏任务和微任务。根据执行环境分别尝试采用
PromiseMutationObserversetImmediate如果以上都不行则采用setTimeout
定义了一个异步方法，多次调用nextTick会将方法存入队列中，通过这个异步方法清空当前队列。

# Vue的生命周期
主要有created, mounted, updated, destory等， created时Vue实例并没有挂载，无法访问到组件的data,method，mountd是组件开始挂载能访问到data,method等数据,当组件DOM更新时触发update，组件销毁时触发destory

## computed和watcher
computed是计算属性，计算组件中data的值返回一个结果,当data的值改变时，计算属性的结果值也会改变
wather用来监听data数据的变化

## 组件中的data为什么是一个函数
一个组件被复用多次的话，也就会创建多个实例。本质上，这些实例用的都是同一个构造函数。如果data是对象的话，对象属于引用类型，会影响到所有的实例。所以为了保证组件不同的实例之间data不冲突，data必须是一个函数。

## v-model的原理
v-model本质上是一个语法糖,当表单元素上使用时，实质上是分别定义了data数据和input事件
```
<input v-bind="message" v-on:input="message = $event.targe.value" >
```
在组件上使用时同上,不过事件input通过子组件$emit触发
```
<child :value="message" @input="message=arguments[0]"></child>
```

## Vue事件绑定原理
Vue事件绑定分为原生事件和自定义事件，原生事件本质上还是通过addEvent和removeEvent实现, 自定义事件，Vue会维护一个事件总线，当组件绑定自定义事件时，通过$on push到事件总线，当通过$emit调用时，在事件总线中查找并触发事件

##　Vue模版编译原理
将Vue模板通过编译生成AST(一种用JavaScript对象的形式来描述整个模板),再对AST进行优化，主要是对数据不会改变的节点标记为静态节点，最后生成render函数

##　Vue2.x的diff算法
diff算法主要判断vnode和oldVnode生成新的Vnode节点，主要过程如下:
同级比较，再比较子节点
先判断一方有子节点一方没有子节点的情况(如果新的children没有子节点，将旧的子节点移除)
比较都有子节点的情况(核心diff)
递归比较子节点
Vue2的核心Diff算法采用了双端比较的算法，同时从新旧children的两端开始进行比较，借助key值找到可复用的节点，再进行相关操作。相比React的Diff算法，同样情况下可以减少移动节点次数，减少不必要的性能损耗，更加的优雅。

##　虚拟Dom以及key属性
在浏览器中对DOM操作是很昂贵的。频繁的操作DOM，会产生一定的性能问题，Virtual DOM本质就是用一个原生的JS对象去描述一个DOM节点。是对真实DOM的一层抽象。
「key的作用是尽可能的复用 DOM 元素。」
新旧 children 中的节点只有顺序是不同的时候，最佳的操作应该是通过移动元素的位置来达到更新的目的。
需要在新旧 children 的节点中保存映射关系，以便能够在旧 children 的节点中找到可复用的节点。key也就是children中节点的唯一标识。

## 组件通信
父子组件通过$parnet和$refs来访问数据，兄弟组件主要通过Vuex和实现一个 Event Bus
```
  var Event=new Vue();
  Event.$emit(事件名,数据);
  Event.$on(事件名,data => {});
```

##　hash路由和history路由实现原理
hash模式的原理是 onhashchange 事件，可以在 window 对象上监听这个事件
history实际采用了HTML5中提供的API来实现，主要有history.pushState()和history.replaceState()。
