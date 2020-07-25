---
title: vue学习笔记(一)————基础知识
tags: vue
categories: vue
date: 2018/2/6
---

# 前言
学习vue的笔记，应该会试着写一个系列，目的还是加深印象，毕竟感觉自己的记忆力真的不ok,也是给自己的学习留下一点东西

# 创建vue实例和数据

```
var app = new Vue({
	el:'#app'，//指定挂载vue实例的元素，一般为id
	data:{
		//数据
		a:1
	}

	})

```
在html中使用{{a}}来显示数据， **是双向绑定的** 

# 生命周期

vue 常用的生命周期钩子为


    语法      |  简介 
---------      |  ------------
created        | 实例创建完成后调用
mounted        | el挂载到实例上后调用
beforeDestroy  | 实例销毁之前调用

用法示例：
```
ar app = new Vue({
	el:'#app'，//指定挂载vue实例的元素，一般为id
	data:{
		//数据
		a:1
	}
	created: function(){
		//函数
	}

	})

```

# 过滤器
 过滤器应该写在filters中,主要对数据进行过滤，如格式化文本，字母全部大写

```
ar app = new Vue({
	el:'#app',
	data:{
	}
	filters:{
		b: function (){
			//函数
		}
	}
	})
```

html中写法为：

```
{{a|b}}
```

**过滤器也可以串联接收数据**

```
{{a|b|c}}
```

#计算属性
计算属性应该写在computed中，应用在过长的逻辑运算
```
{{d}}

ar app = new Vue({
	....
	computed:
	d:function(){
		return //返回结果
	}
	})

```
**注意:**

1. 计算属性不仅可以依赖当前Vue实例的数据，也可以依赖其它实例的数据
2. 计算属性是基于它的依赖缓存的，只要数据不改变，计算属性不更新

# 基本指令
## v-bind
动态更新HTML元素上的属性
```
<a v-bind:href="url"></a>

ar app = new Vue({
	...
	data:{
		url:...
	}

	})
```
语法糖为： 如 **：href="url"**
## v-on 
绑定事件监听器
```
<button v-on:click="a"></button>

ar app = new Vue({
	....
	methods:{
		a: function(){
			//函数
		}
	}

		})
```
在供click调用的方法，可以不加括号，可用$event访问原生DON事件，如：
```
v-on:click="a(mes,$event)"

```
语法糖为@ 如 **@click="a"**
在vue中提供修饰符来实现各种事件方法的绑定，其中包括

    语法      |  简介 
---------     |  ----------------
@click.stop   | 阻止事件冒泡
@click.prevent| 提交事件不再重载页面
@click.once   | 只触发一次
@keyup.13     | 监听键盘事件，键值为13

其中@keyup还提供其它键值的快捷名称，可在vue文档查找，**如@keyup.down**
## 其它指令

    语法      |  简介 
---------    |  ----------------
v-html                  | 输出html 
v-cloak                 | 在vue实例结束编译时从绑定的HTML元素上移除
v-once                  | 定义它的元素或组件只渲染一次
v-if  v-eles-if v-else  | 与if等相同，后加判断式
v-show                  | 改变元素的CSS属性display，false时会隐藏
v-for                   |与for类似，与in搭配使用






