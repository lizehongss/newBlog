---
title: element源码样式学习
date: 2020/4/20
categories: 
  - 源码学习
tags: 
  - element
---

# element源码样式学习
# 目录结构
element的样式存放在element的**packages/theme-chalk**中,目录结构如下所示:

```
│  alert.scss //组件样式
│  aside.scss
│  autocomplete.scss
│  avatar.scss
│  backtop.scss
│  badge.scss
│  base.scss
│  breadcrumb-item.scss
│  breadcrumb.scss
│  button-group.scss
│  button.scss
|  // 省略部分组件样式
├─common // 组件共用样式
│      popup.scss //弹层类组件共用样式
│      transition.scss //主要定义了element组件中使用vue的 transition 组件时用到的动态效果样式
│      var.scss // 定义了element各组件UI的基本样式的变量,包括颜色,文本大小,边框大小,组件不同尺寸对应的不同样式
│      
├─date-picker // date组件使用样式
│      date-picker.scss
│      date-range-picker.scss
│      date-table.scss
│      month-table.scss
│      picker-panel.scss
│      picker.scss
│      time-picker.scss
│      time-range-picker.scss
│      time-spinner.scss
│      year-table.scss
│      
├─fonts // 字体文件
│      element-icons.ttf
│      element-icons.woff
│      
└─mixins // scss复用函数
       config.scss // 样式名的全局配置
       function.scss // 组件样式使用到的sccss函数
       mixins.scss // 共用的mixins函数
       utils.scss // 工具类函数样式, 如禁用用户选择
       _button.scss // 按钮基本样式
```
# 主要文件分析
## config.scss
**config.scss** 文件定义了element样式的全局配置,如样式名前缀,,样式名分割符等

```css
// 这四个样式名配置是element所有样式名定义的基础
// 如: el-button, el-select, is-disabled等样式名
$namespace: 'el';
$element-separator: '__';
$modifier-separator: '--';
$state-prefix: 'is-';
```

## mixins.scss
**mixins.scss** 文件定义了element各组件样式使用的基本mixins,这里对最主要的mixinx做分析

1. [@mixin](#) b() 混入el
```
@mixin b($block) {
	// 假如 $block为button
  $B: $namespace+'-'+$block !global;  $namespace在config.scss中定义为el,故$B为el-blutton
  .#{$B} {    // .#{} 为scss变量插值,编译后为.el-button
    @content;
  }
}
// @content`用在mixin里面的，当定义一个mixin后，并且设置了@content
// @include的时候可以传入相应的内容到mixin里面
```

2. [@mixin](#) e() 混入__

```css
@mixin e($element) {
  /**假设$element为disabled **/
  $E: $element !global;
  $selector: &;  // 父选择器
  $currentSelector: ""; // 要生成的选择器
  /** 遍历$element //可能有多个
  /* $B 为 mixin b()混入中的变量名
  /* 这里使用$B是因为在element组件样式中e的混入必定是在b混入下的
  /* 如果是在el-button下使用e混入,则生成 .el-button__disabked
  **/
  @each $unit in $element {
    $currentSelector: #{$currentSelector + "." + $B + $element-separator + $unit + ","};
  }
  /** hitAllSpecialNestRule 判断$elector是否包含--, is-, 在function.scc 中定义
  /* 这里判断是否包含is--,--是因为使用了 @at-root
  /* @at-root指令可以使一个或多个规则被限定输出在文档的根层级上，而不是被嵌套在其父选择器下
  /* 包含有is--等前缀的样式名,在组件中一般是可移除的,所以在输出在文档的根层级上时要加上父选择器
  **/
  @if hitAllSpecialNestRule($selector) {
    @at-root {
      #{$selector} {
        #{$currentSelector} {
          @content;
        }
      }
    }
  } @else {
    @at-root {
      #{$currentSelector} {
        @content;
      }
    }
  }
}
```

3. [@mixin](#) m() 混入--

```css
/** 
/* 该方法与混入e基本相同,不同的是没有判断hitAllSpecialNestRule
/* 因为混入 -- 一般的是组件的尺寸样式,如 el-radio--medium,el-radio--small等,父选择器一般为el-radio等,
/* 故不需要判断
**/
@mixin m($modifier) {
  $selector: &;
  $currentSelector: "";
  @each $unit in $modifier {
    $currentSelector: #{$currentSelector + & + $modifier-separator + $unit + ","};
  }

  @at-root {
    #{$currentSelector} {
      @content;
    }
  }
}
```

4. [@mixin](#) w() 混入 is

```css
/** 假设$state为check
/* 则编译后为在文档的根层级(在组件中使用时,为组件根样式)为.el-radio.is-checked
**/
@mixin when($state) {
  @at-root {
    &.#{$state-prefix + $state} {
      @content;
    }
  }
}
```
## functon.scss
**function.scss** 主要是定义了 **hitAllSpecialNestRule** 函数方法，主要用来对选择器是否含有'is-', '--', ':'主要代码如下:

```css
@import "config";

/* BEM support Func
 -------------------------- */
@function selectorToString($selector) {
  //字符化
  $selector: inspect($selector);
  /** str-slice 
  /* 从 $string 中截取子字符串，通过 $start-at 和 $end-at 
  /* 设置始末位置，未指定结束索引值则默认截取到字符串末尾。
  /* 这里主要是去除第一个字符和最后一个字符,避免如 --el, el：等的干扰
  **/
  $selector: str-slice($selector, 2, -2);
  @return $selector;
}

@function containsModifier($selector) {
  $selector: selectorToString($selector);
    /** str-index($string, $substring)
    /* 返回一个下标，标示 $substring 在 $string 中的起始位置。没有找到的话，则返回 null 值。
    /* $modifier-specarator --
  	**/
  @if str-index($selector, $modifier-separator) {
    @return true;
  } @else {
    @return false;
  }
}

@function containWhenFlag($selector) {
  $selector: selectorToString($selector);
/** $state-prefix: 'is-'; **/
  @if str-index($selector, '.' + $state-prefix) {
    @return true
  } @else {
    @return false
  }
}

@function containPseudoClass($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, ':') {
    @return true
  } @else {
    @return false
  }
}
//** 对上述四种情况进行判断，有一种存在就返回true **/
@function hitAllSpecialNestRule($selector) {

  @return containsModifier($selector) or containWhenFlag($selector) or containPseudoClass($selector);
}
```

