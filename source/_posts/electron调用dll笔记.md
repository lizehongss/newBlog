---
title: electron调用dll笔记
tags:  electron dll
categories: electron
date: 2021/09/29
---
## electron安装相关依赖
1. electron调用dll需要安装node-gyp,用于编译node-ffi-napi,ref-arrray-di, ref-struct-di, ref-napi等库。
node-gyp在window下运行需要python2.7环境和 Visual Studio Build Tools.([详情在这里](https://github.com/nodejs/node-gyp))
在这里可以使用 windows-build-tools进行安装，命令如下(__注意需要管理员模式__)：
```
npm install --global --production windows-build-tools
```
2. 为方便开发调用32位和64位dll,建议使用nvm管理node版本,[可以在这里下载安装](https://github.com/coreybutler/nvm-windows/releases)

## 关键库调用说明
ref-napi, ref-struct,ref-array,在下面引用的是TooTallNate原作者的仓库链接，因为说明文档更加具体和详细,但使用时建议使用node-ffi-napi仓库中的版本，因为它们能够在node 10.0 以上版本使用
### node-ffi-napi
[node-ffi-napi](https://github.com/node-ffi-napi/node-ffi-napi):用于使用js调用c++动态链接库(dll)，主要用法如下:
```
var ffi = require('ffi-napi');

// 声明要调用的函数
var libm = ffi.Library('libm', {
  //第一个参数为该函数的输出类型, 第二个参数为参数数组，定义该函数的传参类型
  'ceil': [ 'double', [ 'double' ] ]
});
libm.ceil(1.5); // 2

```
[详细用法可以查看这里](https://github.com/node-ffi/node-ffi/wiki/Node-FFI-Tutorial)
其中需要注意的是，如果是调用相互动态链接的dll，需要调用window的keilDay32设置目录
如下所示:
```
const kernel32 = new ffi.Library("kernel32", {
            SetDllDirectoryA: ["bool", ["string"]]
})
let result = kernel32.SetDllDirectoryA(path)
```
### ref-napi
[ref-napi](https://github.com/node-ffi-napi/ref-napi): 用于将node缓冲区的实例转换为指针，方便传参入dll函数进行调用,[具体调用实例在这](https://tootallnate.github.com/ref)
重要类型调用如下:

```
// 在Buffer区创建了一个4个字节的int类型内存空间并将指向该空间的指针赋值给outNumber,这样就可以直接将outNumber作为传参给C++函数，C++函数可以通过该指针赋值到内存区，js使用deref()函数可以拿到实际的值
var outNumber = ref.alloc('int'); 
libmylibrary.manipulate_number(outNumber);
var actualNumber = outNumber.deref();
```
### ref-struct
[ref-struct](https://github.com/TooTallNate/ref-struct)提供一个C++的struct接口在node缓冲区的实现，具体实现方法如下:
```
// 定义一个Struct类型
// 引用node-ffi-napi中的版本
var StructType = require('ref-struct-di')(ref)
let structText =StructType({
  x: ref.types.int,
  y: ref.types.int,
  width: ref.types.int,
  height: ref.types.height
})
// 在C++函数中声明, 
someDllMethod: ['int', [ref.refType(structText)]],
// 如果是struct有传参默认值, 调用时如下:
let hasValue = new structText({
  x: 0,
  y: 0,
  width: 0,
  height: 0
})
someDllMethod(hasValue.ref())
// 如果要拿到回参的值，直接调用hasValue即可
console.log(hasValue.x)
```
### ref-array
[ref-array](https://github.com/TooTallNate/ref-array)用于在node缓冲区实现一个C++的数组定义，用法如下:
```
// 引用node-ffi-napi中的版本
var ArrayType = require('ref-array-di')(ref)
// 定义一个int类型，长度为5的数组类型
let textArray = ArrayType(ref.types.int, 5)
```
### 其它特殊使用说明
1. 如果C++需要传入一个类型为void的二级指针句柄，建议如下使用:
```
someDllMethods: ['long', [ref.refType(ref.refType(ref.types.void))]]

// 在使用时, 定义一个二级int型指针
// 这里如果使用type为void时，无法正确取值
// void indirection 为2
let void = ref.alloc(ref.refType(ref.types.int))
// 解析一次
void.deref()
// 解析二次
void.deref().deref()

```
2. c++ 内部使用GBK编码, 如果回参是一个有指定字节数的char数组,需要拿到这个数组的值，并使用[iconv-lite](https://www.npmjs.com/package/iconv-lite)将其解析。

```
// char数组在buffer区中的数据
let textBuffer = new Buffer.form([12,52,25,2,52])
var iconv = require("iconv-lite");
str = iconv.decode(textBuffer, "GBK");
```
## 参考文档
1. [node-ffi 食用指南（难吃](https://www.v2ex.com/t/474611)
2. [ref文档](http://tootallnate.github.io/ref/)
3. [electron对接Dll](https://blog.csdn.net/zxl761303248/article/details/108051680)