---
title: css布局
tags: css
categories: css
date: 2018/6/23
---


# css布局（持续更新）
## 将浮动元素围住
1. 为父元素添加  **overflow:hidden** 属性演示如下：

2. 同时浮动父元素 **为父元素添加float:left**演示如下：

3. 添加非浮动元素的清除元素 代码如下
```
.clear:after{
    content:".";
    display:block;
    height:0;
    visibility:hidden;
    clear:both;
}
```

## 三栏一固定宽度布局

1. 父元素wrapper 设定一固定宽度，水平外边距为auto

2. 子元素nav和artice设置浮动，宽度相加为wrapper的宽度（再加以此类推）

3. header和footer 默认与布局同宽 **footer要清除浮动**
html代码如下：
```
<div id="wrapper">
<header>
<!-- 标题 -->header</header>
<nav><!-- 无序列表 -- nav</nav>
<article><! -- 文本 --> article</article>
<aside><! -- 文本 -->aside</aside>
<footer><!-- 文本 --> footer</footer>
```
css代码如下：
```
#wrapper{
  margin: 0 auto;
  width:960px;
  height:700px;}
header{
  background:#dcd9c0;
  height:30px;}
nav{
  width:150px;
  float:left;
  background:#dcd9c0;
  height:400px;}
  article{
  width:600px;
  float:left;
  background:#ffed53;
  height:400px;}
aside{
  width:210px;
  float:left;
  background:#3f7ccf;  
  height:400px;}
footer{
  clear:both;
  background:#000;
  height:40px;}
```
- 各栏边界分开的解决方法：
 1.  为子元素里的内容加一个div,为div设置一个内边距
 2.  为浮动栏设置 **box-sizing:border-box** 以及内边距和边框即可--  **IE7和IE8不支持** 

 
## 三栏--中栏流动布局
- 方法一
1.  用一个div class为threecolarap 包围全部三栏并为其设置浮动
2.  用一个div class为twocolarap包围左栏和中栏并为其设置浮动以及加上 **margin-right:210px** 把右栏拉到区块外边距腾出的位置上
3. 中栏加上 **margin-right:210px** 在流动居中的栏右测腾出空间
- 方法二

1. 为每一栏display属性设定为table-cell (**IE7不支持**)
## 补充
### 元素居中
1. 为父元素应用 **text-algin:center** 
2. 为要居中的元素设定 **display:inline-block**