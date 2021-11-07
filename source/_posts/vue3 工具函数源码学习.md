---
title: vue3 工具函数源码学习
date: 2021/11/07
categories: 
  - 源码学习
tags: 
  - vue3
---
## vue3工具函数目录结构
vue3工具函数所在目录如下所示
```
vue-nex/packages/shared/src
```
src目录下有以下文件:**​**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1022616/1635670732893-e86882a3-292d-4b84-b787-9bc1c4ac4a77.png#clientId=uf59860a1-0cb7-4&from=paste&height=301&id=u8ebe0df7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=301&originWidth=447&originalType=binary&ratio=1&size=19079&status=done&style=none&taskId=ue3e0b117-2885-4ec7-a0c7-8e2c3370b55&width=447)
其中index.ts是入口文件，其它文件在index.ts中引入， index中定义了常用的工具函数
```
import { makeMap } from './makeMap'

export { makeMap }
export * from './patchFlags'
export * from './shapeFlags'
export * from './slotFlags'
export * from './globalsWhitelist'
export * from './codeframe'
export * from './normalizeProp'
export * from './domTagConfig'
export * from './domAttrConfig'
export * from './escapeHtml'
export * from './looseEqual'
export * from './toDisplayString'
```
## 主要工具函数解析
这里主要学习理解index里的工具函数，部分源码和解析如下:
```javascript
export const EMPTY_OBJ: { readonly [key: string]: any } = __DEV__
  ? Object.freeze({})
  : {}
export const EMPTY_ARR = __DEV__ ? Object.freeze([]) : []
export const NOOP = () => {}
```

- EMPTY_OBJ和EMPTY_ARR都使用**Object.freeze()**, 它的作用是冻结一个对象，使其不能被修改。
- EMPTY_OBJ和EMPTY_ARR返回一个空对象和空数组，并且在开发环境是被冻结， 主要作用应该是在开发环境修改空对象时使其报错明显。
- NOOP返回一个空函数，方便判断和压缩代码，每个地方都写 ()=> {} 明显代码量会增多
```javascript
export const NO = () => false

const onRE = /^on[^a-z]/
export const isOn = (key: string) => onRE.test(key)

export const isModelListener = (key: string) => key.startsWith('onUpdate:')

export const extend = Object.assign

export const remove = <T>(arr: T[], el: T) => {
  const i = arr.indexOf(el)
  if (i > -1) {
    arr.splice(i, 1)
  }
}
```

- NO 返回false，主要也是压缩代码量
- isOn 判断通过正则判断字符串是否是on开头，并且on 后首字母不是小写字母，如 **onReg**
- isModelListener 通过判断字符串是否是**onUpdate:** 开头
- extend 提供**Object.assign**方法的缩写， 应该主要也是为了压缩代码
- remove 通过**splice方法**删除数组中的一项，传入数组和索引
```javascript
const hasOwnProperty = Object.prototype.hasOwnProperty
export const hasOwn = (
  val: object,
  key: string | symbol
): key is keyof typeof val => hasOwnProperty.call(val, key)

export const isArray = Array.isArray
export const isMap = (val: unknown): val is Map<any, any> =>
  toTypeString(val) === '[object Map]'
export const isSet = (val: unknown): val is Set<any> =>
  toTypeString(val) === '[object Set]'

export const isDate = (val: unknown): val is Date => val instanceof Date
export const isFunction = (val: unknown): val is Function =>
  typeof val === 'function'
export const isString = (val: unknown): val is string => typeof val === 'string'
export const isSymbol = (val: unknown): val is symbol => typeof val === 'symbol'
export const isObject = (val: unknown): val is Record<any, any> =>
  val !== null && typeof val === 'object'

export const isPromise = <T = any>(val: unknown): val is Promise<T> => {
  return isObject(val) && isFunction(val.then) && isFunction(val.catch)
}
export const objectToString = Object.prototype.toString
export const toTypeString = (value: unknown): string =>
  objectToString.call(value)
```
上述的主要函数的主要作用都是判断是否为ES的某一些内置类型

- hasOwn 通过Object.prototype.hasOwnProerty判断对象本身是否有key对应的属性
- isArray 通过Array.isArray方法判断,使用instanceof判断并不准确
- isMap isSet通过Object.prototype.toString判断是否为**Map**,**Set**
- isDate 通过instanceof判断
- isFunction, isString,isSymbol,isObject通过**typeof判断**
- isPromise通过val是否为对象，then和catch是否为函数判断
```javascript
export const toRawType = (value: unknown): string => {
  // extract "RawType" from strings like "[object RawType]"
  return toTypeString(value).slice(8, -1)
}

export const isPlainObject = (val: unknown): val is object =>
  toTypeString(val) === '[object Object]'

export const isIntegerKey = (key: unknown) =>
  isString(key) &&
  key !== 'NaN' &&
  key[0] !== '-' &&
  '' + parseInt(key, 10) === key

export const isReservedProp = /*#__PURE__*/ makeMap(
  // the leading comma is intentional so empty string "" is also included
  ',key,ref,' +
    'onVnodeBeforeMount,onVnodeMounted,' +
    'onVnodeBeforeUpdate,onVnodeUpdated,' +
    'onVnodeBeforeUnmount,onVnodeUnmounted'
)

const cacheStringFunction = <T extends (str: string) => string>(fn: T): T => {
  const cache: Record<string, string> = Object.create(null)
  return ((str: string) => {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }) as any
}
```

- toRawType截取toTypeString中的一部分，主要是_string,object_这些，如[object Set]返回Set
- isPlainObject 判断是不是纯粹的对象， 如isPlainObject([]) // false
- isIntegerKey 判断是不是数字型的字符串key值
- cacheStringFunction 缓存函数， 实现一个单例
```javascript
const camelizeRE = /-(\w)/g
/**
 * @private
 */
export const camelize = cacheStringFunction((str: string): string => {
  return str.replace(camelizeRE, (_, c) => (c ? c.toUpperCase() : ''))
})

const hyphenateRE = /\B([A-Z])/g
/**
 * @private
 */
export const hyphenate = cacheStringFunction((str: string) =>
  str.replace(hyphenateRE, '-$1').toLowerCase()
)

/**
 * @private
 */
export const capitalize = cacheStringFunction(
  (str: string) => str.charAt(0).toUpperCase() + str.slice(1)
)

/**
 * @private
 */
export const toHandlerKey = cacheStringFunction((str: string) =>
  str ? `on${capitalize(str)}` : ``
)
```
上述函数都使用**cacheStringFunction函数**包裹，确保返回第一次所创建的那唯一的一个实例。

- camelize 连字符 - 转驼峰， 将on-click中的-c匹配并替换为C
- hyphenate 将驼峰转连字符， 将onClick中的C匹配到并替换为-C再通过toLowerCase转为小写
- capitalize 首字母大写
- toHandlerKey将click转化onClick这种
```javascript
// compare whether a value has changed, accounting for NaN.
export const hasChanged = (value: any, oldValue: any): boolean =>
  !Object.is(value, oldValue)

export const invokeArrayFns = (fns: Function[], arg?: any) => {
  for (let i = 0; i < fns.length; i++) {
    fns[i](arg)
  }
}

export const def = (obj: object, key: string | symbol, value: any) => {
  Object.defineProperty(obj, key, {
    configurable: true,
    enumerable: false,
    value
  })
}

export const toNumber = (val: any): any => {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}
```

- hasChanged: 通过**Object.is()**判断两个值是否严格相等
- invokeArrayFns：执行数组里的函数
- def: 定义对象属性，使其可删除和不可枚举
- toNumber : 将值转换为数字
```javascript
let _globalThis: any
export const getGlobalThis = (): any => {
  return (
    _globalThis ||
    (_globalThis =
      typeof globalThis !== 'undefined'
        ? globalThis
        : typeof self !== 'undefined'
        ? self
        : typeof window !== 'undefined'
        ? window
        : typeof global !== 'undefined'
        ? global
        : {})
  )
}
```

- getGlobalThis: 获取全局 this 指向, 依次查找__globalThis, self ,window, global
