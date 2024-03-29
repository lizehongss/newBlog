---
title: vue-响应式原理
date: 2021/10/24
categories: 
  - 源码学习
tags:
  - vue
---
# 通过object.defineProperty设置getter和setter
vue初始化调用_init方法, 在init中会执行initState方法，对props,methods,data等属性进行初始化, 主要代码如下：
```
// 省略部分代码
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```
其中主要方法作用如下：
- initProps()调用deineReactive方法使其变成响应式。
- initData()->调用observe方法使其变为响应式
- observe()给对象添加getter和setter,用于依赖收集和派发更新。代码如下:
```
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```
主要是实例化Observer实现响应式, Observer的实现如下: 通过定义getter和setter实现
```
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; 
  constructor (value: any) {
    this.value = value
  // 实例化Dep对象，Dep是一个订阅者，用来存放Watcher观察者对象
    this.dep = new Dep()
    this.vmCount = 0
    // 添加 this.__ob__ = value
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }
  // 遍历对象的 key 调用 defineReactive
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
  // 遍历数组调用observe
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```
definReactive定义一个响应式对象
```
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  // 实例化Dep对象
  const dep = new Dep()
  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }
  // 对子对象递归调用
  let childOb = !shallow && observe(val)
  //给对象添加getter和setter
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```
# 依赖收集和派发更新
## 依赖收集
- defineReactive的get方法会调用dep.depen()进行依赖收集，Dep类的主要作用是管理Watch对象
```
export default class Dep {
  // target全局唯一 Watcher
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }
  // 在 subs中添加Watcher，也就是添加属性发生改变时，要通知的watcher
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  // 移除Wathcer
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }
  // 依赖收集
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
  // 通知Watcher更新，也就是派发更新
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```
- Watcher类的定义如下:
```
let uid = 0
export default class Watcher {
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.computed = !!options.computed
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.computed = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.computed // for computed watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    if (this.computed) {
      this.value = undefined
      this.dep = new Dep()
    } else {
      this.value = this.get()
    }
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }
  // ...
}
```
```
this.deps = [] 
this.newDeps = [] 
//表示Wather实例持有的Dep实例，也就是订阅了多少dep
```
1. 组件挂载时会调用mountComponent函数，在mountComponent中实例化一个渲染Wathcer，而在实例化Wathcer时调用this.get()
    ```
      get () {
        pushTarget(this)
        let value
        const vm = this.vm
        try {
          value = this.getter.call(vm, vm)
        } catch (e) {
          // ....
        } finally {
          if (this.deep) {
            traverse(value)
          }
          popTarget()
          this.cleanupDeps()
        }
        return value
      }
    ```
2. 调用pushTarget(this)将要挂载的组件渲染Wather对象赋值给Dep.target -> 执行this.getter.call(vm,vm)，实际上是执行new Wathcer时传入的回调函数->执行updateComponent()
    ```
    new Watcher(vm, updateComponent, //...)
    ```
3. updateComponent有对组件data数据的访问-> 触发defineReactive的get方法 -> 依赖收集dep.depen()->调用Dep.target.addDep(this)->实际是调用渲染Watcher实例中的addDep
    ```
      addDep (dep: Dep) {
        const id = dep.id
        // 保证同一数据不会被添加多次
        if (!this.newDepIds.has(id)) {
          this.newDepIds.add(id)
          this.newDeps.push(dep)
          if (!this.depIds.has(id)) {
            dep.addSub(this)
          }
        }
      }
    ```
4. 将这个渲染Wather添加到在defineReactive中实例化的dep中的subs中,这就是整个依赖收集过程。
## 派发更新
- set时，派发更新的主要流程如下:
```
  set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      // ....
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
```
- observe(newVal)使新设置的值变成响应式的
- dep.notify()通知所有订阅了这个dep的Wathcher对象-> dep.notify()调用Watcher的update()方法
```
  update () {
    if (this.computed) {
      if (this.dep.subs.length === 0) {
        this.dirty = true
      } else {
        this.getAndInvoke(() => {
          this.dep.notify()
        })
      }
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }
```
- 一般组件数据更新会调用queueWatcher(this)
- queueWathcer使用了队列，先把Wather添加到队列queue，在下个nextTick执行flushSchedulerQueue() -> 执行watcher.run()方法
```
run () {
  if (this.active) {
    this.getAndInvoke(this.cb)
  }
}
```
- run()方法执行->getAndInvoke(this.cb)传入回调函数->getAndInvoke判断满足新旧值不等->执行this.cb回调函数
- 在渲染watcher中，this.cb为
```
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```
所以当修改组件相关的响应式数据时会触发组件重新渲染


# computed实现
- Vue实例初始化时调用initComputed()
在initComputed()中为每一个计算属性getter创建**computed Wather**,**computed Wathcer**初始化时会持有一个dep实例,用来管理computed Wathcer. 当update()时，要更新订阅了这个dep的watcher。
```
  if (this.computed) {
    this.value = undefined
    this.dep = new Dep()
  }
```
- 最后调用defineComputed()
defineComputed()-->调用Object.defineProperty为计算属性设置对应的key的getter和setter,一般setter为空，getter对应createComputedGetter(key)
```
function createComputedGetter (key) {
  return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      watcher.depend()
      return watcher.evaluate()
    }
  }
}
```
当render 函数访问到计算属性时-> 触发getter-> 执行watcher.depend->将渲染Wather订阅上文实例化的dep->执行watcher.evaluate()
```
  evaluate () {
    if (this.dirty) {
      this.value = this.get()
      this.dirty = false
    }
    return this.value
  }
```
watcher.evaluate()执行this.get()-> this.get()为计算属性定义的getter函数获取到this.value值
此时也会访问到计算属性所依赖的数据的getter,所以Dep.target为这个computed watcher,此时这个computed watcher订阅了所依赖数据持有的dep

当对计算属性依赖的数据做修改时->触发数据setter->通知订阅它变化的computed watcher更新-> 执行watcher.update()方法
```
  if (this.computed) {
    if (this.dep.subs.length === 0) {
      this.dirty = true
    } else {
      this.getAndInvoke(() => {
        this.dep.notify()
      })
    }
  }
```
在上文中渲染Wather订阅了this.dep, 所以渲染Wathcer通过this.getAndInvoke()的回调执行update()->重新渲染组件
this.getAndInvoke()主要是对比新旧值

在computed Watcher中this.dirty的作用时在重新渲染组件后，通过evalute()获取计算属性的值时，因为在getAndInvoke()中将this.value设为新值并置this.dirty为false,所以直接返回this.value，不触发所依赖数据的getter
```
  // 在 getAndInvoke()中
  const oldValue = this.value
  this.value = value
  this.dirty = false
```

# watch的实现
Vue实例初始化时->initWatch()->调用createWatcher(vm, key, handler[i])->返回vm.$watch(expOrFn, handler, options)
$watch在执行stateMixin时定义
实例化了Wathcer 
```
const watcher = new Watcher(vm, expOrFn, cb, options)
```
是一个 user Wathcer, options.user = true,
在实例化 user Wathcer时，调用了watcher的this.get()方法，之后的依赖收集和更新与一般组件数据相同,不同是对options属性值不同时，进入不同的分支.