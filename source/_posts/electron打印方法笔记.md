---
title: Electrond打印方法笔记
tags: electron
categories: electron
date: 2021/8/22

---
# 前言
主要是总结最近在工作所使用到的electron方法总结，总结的主要是静默打印的方法.
# webview打印
- webview主要是electron提供的在一个独立的 frame 和进程里显示外部 web 内容的标签，详细介绍可以[看这里](https://www.electronjs.org/docs/api/webview-tag).需要注意的是在electron>=5的版本里webview标签是禁用的，需要在主进程`BrowserWindow`里设置``webviewTag``为`true`启用:
```
    mainWindow = new BrowserWindow({
        title: '桌面应用程序',
        webPreferences: 
            webviewTag: true
        }
    });
```
- 打印主要使用到是webview的isLoading()和print()方法，webview打印主要适用于打印远程的网页内容.在webview标签里设置好要打印的远程网页url，直接获取到webview元素并调用print方法即可打印.
```
<webview id="webview" src="https://github.com"></webview>
```

```
let webview = document.getElementById('webview')
webview.print({
    silent: true, // 是否静默打印
    deviceName: 'print' // 打印机名称
})
```
- 关于打印机名称的获取可以在主进程中获取并通过IPC传输到渲染进程
```
ipcMain.on('getPrintDeviceList', (event) => {
    event.return.value = mainwindow.webContents.getPrinters()
)
```
- 需要注意的是远程网页资源加载需要时间，这里最好使用webview.isLoading()确认页面资源加载完成后再调用打印方法， 具体调用如下:
```
webview.addEventListener('dom-ready', checkCanPrint)

function checkCanPrint() {
    // 还在加载中，则setTimeout秒后再检查
    if (webview.isLoading) {
    setTimeOut(() => {checkCanPrint}, 1 * 1000)
   } else {
       webview.print() // 打印
   }
}
```
# 在主进程中调用webContents.print()打印
- 如果需要打印渲染进程里的本地网页内容,可以在主进程中调用webContents.print()，在要打印的渲染进程页面中触发主进程调用打印.
```
    // main.js
    ipcMain.on('print', (event, deviceName) => {
        mainWindow.webContent.print( {
            silent: true,
            deviceName: deviceName
        } )
    })
    // print.js
    const {ipcRenderer} = require('electron')
    ipcRender.send('print', deviceName)

```
- 使用webContents.print()打印时，与在浏览器使用window.onprint()一样，在css中可以使用媒体查询控制要打印的内容和样式
```
//print.css

@media print {
 .print-dom {
     display: block;
 }
 .not-print-dom {
     display: none;
}
}
```
# 总结
使用webview和webContents.print()方法打印是目前在业务使用的两种方法，网上文章其实好多，但在打印远程网页时如何确保网页内容加载完成再打印是这篇文章形成的主要原因。如果有其它更好的方法也希望各位大佬留言.