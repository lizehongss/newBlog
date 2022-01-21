---
title: 使用preload远程页面调用electron接口
tags: electron
categories: electron
date: 2020/10/15
---
## 主要原理
主要是通过electron中的preloa在本地中预先加载一个指定脚本js文件，这个文件可以使用node APIs调用electron中的相关接口如ipcRenderer等，所以在electron中使用远程网页的窗口中定义加载js文件，并开放webviewTag,就可以在预加载文件中通过定义window中的方法调用electron接口，然后在远程网页中直接调用预加载文件中定义的方法就可以实现与electron的相关通信。主要实现过程如下:
1. 在BrowserWindow中定义如下：(__注意preload的文件路径需要绝对路径__)
```
win = new BrowserWindow({
    webPreferences: {
        webviewTag: true,
        perload: path.join(__dirname, './preload.js')
    }
})
```
2. 在preload.js中定义相关事件，这里定义一个与electron主进程进行通信的方法
```
// preload.js
const { ipcRenderer } = require("electron")
window.senSomeToMain = function () {
    ipcRenderer.send('senSomeTomain')
}
// 监听electron主进程返回
ipcRenderer.on('returnSometh', (event, data) => {
    //该方法在远程网页中定义
    window.getReturn(data)
})

```
3. 在远程页面中的使用如下:
```
// index.html

// 与electron主进程通信
window.sendSomeToMain()
// 定义返回信息的回调
window.getReturn = function (data) {
    console.log(data)
}
```
## 主要用途
方便网页资源部署在服务器，使远程网页可以通过本地的electron访问相关的硬件资源，如打印机等。也方便使网页开发功能业务和electron资源调用业务分开,使部署和开发相对简单。

