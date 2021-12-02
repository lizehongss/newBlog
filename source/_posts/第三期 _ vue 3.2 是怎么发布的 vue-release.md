---
title: vue3.2是怎么发布vue-release的
date: 2021/12/02
categories: 
  - 源码学习
tags: 
  - vue3
---
# 相关源码文件及依赖
vue3 跟发布相关的文件主要目录如下:
```javascript
vue-next/scripts/release.js
```
所使用到的引入依赖如下：
```javascript
const args = require('minimist')(process.argv.slice(2))
const fs = require('fs')
const path = require('path')
const chalk = require('chalk')
const semver = require('semver')
const currentVersion = require('../package.json').version
const { prompt } = require('enquirer')
const execa = require('execa')
```
各依赖作用如下

- [minimist](https://github.com/substack/minimist) 主要是获取命令行参数, 这里会将所有的命令行参数添加进args对象中
- fs, path 都是node常用的自带模块，主要是操作文件和路径
- [chalk](https://github.com/chalk/chalk) 主要用于终端美化
- [semver](https://www.npmjs.cn/misc/semver/) 主要作用是对版本校验比较
- currentVersion 获取当前package.jsonr的version信息
- [enquirer](https://github.com/enquirer/enquirer): 主要用来创建命令行交互式询问用户输入，这里引用的prompt给用户提供输入，选择等功能
- [execa](https://github.com/sindresorhus/execa): 主要是用来执行命令行命令
<a name="VR10z"></a>
# 主要流程
<a name="U5zw5"></a>
## 流程图
![vue3发布流程图.drawio (1).png](https://s3.bmp.ovh/imgs/2021/12/ee6e02c0e482ff2c.png)
<a name="ddal6"></a>
## 重要函数分析
```javascript
const inc = i => semver.inc(currentVersion, i, preId)
const bin = name => path.resolve(__dirname, '../node_modules/.bin/' + name)
const run = (bin, args, opts = {}) =>
  execa(bin, args, { stdio: 'inherit', ...opts })
const dryRun = (bin, args, opts = {}) =>
  console.log(chalk.blue(`[dryrun] ${bin} ${args.join(' ')}`), opts)
const runIfNotDry = isDryRun ? dryRun : run
const getPkgRoot = pkg => path.resolve(__dirname, '../packages/' + pkg)
const step = msg => console.log(chalk.cyan(msg))
```

- **inc**函数主要是通过semver库的inc方法生成一个版本号,如：semver.inc('3.2.4', 'prerelease', 'beta') => 3.2.5-beta.0
- **bin**函数主要使用是传入name，返回node_modules/.bin/${name} ,方便获取 node_modules/.bin/ 目录下的命令
- **run**函数主要通过execa执行命令行命令, 如: run('yarn', ['build', '--release']) => yarn build --release
- **dryRun**函数是空跑命令， 通过console.log打印传入的命令，方便调试
- **runIfNotDry**通过一开始用户传入的命令行参数**isDryRun**来判断是空跑还是真的执行命令
- **getPKgRoot**获取包路径
- **step函数**通过chalk打印传入的信息
<a name="o8UBA"></a>
# 总结
大致明白了Vue3是如何发布包的，整个流程其实也可以应用在公司前端项目的发布中，例如通过输入的版本号修改pack.json信息，自动执行git命令，校验版本号等。
<a name="Le0Dk"></a>
# 参考资料

- [Vue 3.2 发布了，那尤雨溪是怎么发布 Vue.js 的？](https://juejin.cn/post/6997943192851054606#heading-24)
- [第二期 | vue3 工具函数 | 源码共读公告【必看】](https://www.yuque.com/ruochuan12/notice/p2)​
