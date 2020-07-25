---
title: vue学习笔记(二)————组件
tags: vue
categories: vue
date: 2018/2/10
---
# 前言
主要是有关vue组件的笔记，然后也不想为了定blog而写blog,所以应该会写一些真的自己想记下来的东西，也希望可以尽量精简的写
# 组件用法
需要注册才能使用组件，注册分为局部注册和全局注册，代码示例如下：

**必须在实例注册，组件才可以使用**
```
//全局注册
Vue.component("组件名",{

    //选项
    
});
//局部注册
var 组件名 ={
    //选项
}
html模板如下：
<组件名></组件名
```
# 组件选项
## template
temlplate后面是要渲染的内容，必须有一个元素包围它，代码如下：

**注意v-show不能用在template中，可以使用v-if**
```
Vue.component("组件名",{
template: '<div></div>';
    
});
```
## data
data的用法与实方法相同，不同的是要将数据return出去，代码如下：

```
Vue.component("组件名",{
.....
data : function{
    return  {
        mws: "text"
    }
}
    
    
});
```
## props
props主要是用来接收来自父组件的数据，可以是字符串数组和对象
**注意数据是单向的，只能父组件传到子组件**
代码如下：
```
html:

<mytext message="text"></mytext>

js：
Vue.component("mytext",{
    props:['message']
    
});
```
props的数据可以进行验证，代码如下：
```
props: {
    message : {
        type:Number， //类型
        default:""text //如果没有定义的默认值
        
    }
    }
}
```
## wath 
 主要是用来监听props的值的改变，从而通知父组件或者更新props值
 代码如下
 ```
 watch:{
     message: function(val){
         //当message发生改变时触发的函数 
         //val 为更新后的message值
     }
 }
 ```
# 组件通信

## 父组件向子组件通信

使用上面的props即可

## 子组件向父组件通信
使用自定义事件，子组件用$emit()来触发向父组件通信的事件代码如下：
```
html:
<mytext @send="handle"></mytext>
//其中send为子组件发送过来的参数，为自定义事件的名称
Vue.component("mytext",{
template:'<button @click="hadles">',
methods:{
    hadles: function(){
        this.$emit('handle',数据);
        //第一个参数为事件名称，第二个为发送的数据
    }
}
    
});
```
*可以使用v-model来简化，不过个人感觉没有必要，所以不想写在这里做总结。。。。。。*
## 非父子组件通信
在vue.js2.x中使用一个空的vu实例来作中介，组件把自定义事件名称和数据发送到这个空实例，其它实例或组件通过监听空实例的自定义事件来刷新数据 
代码如下：
```
var bus =new Vue();
bus.$emit('clicks',mes);
bus.$on('clicks',function(){});
```
补充：
当子组件较多时，可以用ref属性为组件指定一个索引名称：代码如下:
```
html代码如下：
<mytext ref="A></mytext>
//调用指定的子组件的message
var msg=this.$refs.comA.message
```
# slot 
slot是内容分发，用在将父组件的dom结构挂载，它与props,events触发事件构成组件的3个APi来源.
**slot的作用域是在父组件上**
代码如下：
```
<mytext>
<p></p>//挂载内容 
</mytext>
Vue.component("mytext",{
template:'
    <div>
    <slot>
    <p></p>//没有内容时的默认内容
    </slot>//父组件的内容将挂载在slot里面
    </div>
'
    
});
```
可以给slot指定一个name，使用分发指定内容代码如下：
```
<mytext>
<p slot="one"></p>//挂载内容,slot为具名slot
</mytext>
Vue.component("mytext",{
template:'
    <div>
    <slot>
    <p></p>//没有内容时的默认内容
    </slot>//父组件的内容将挂载在slot里面
    <slot name="one">//具名slot将分发在这里
    </div>
'
```
也可以指定数据传递到父组件的挂载里，使用scope,代码如下：
```
<mytext>
<p slot="one" scope="props">
    {{props.msg}} //显示子组件的msg
</p>//挂载内容,slot为具名slot,props为临时变量
</mytext>
Vue.component("mytext",{
template:'
    <div>
    <slot>
    <p></p>//没有内容时的默认内容
    </slot>//父组件的内容将挂载在slot里面
    <slot name="one" msg="数据">//具名slot将分发在这里
    //msg为向父组件传递的数据
    </div>
'
```
可以使用$slots访问分发的内容 
```
var header =this.$slots.one;
```

# 结语
    还有许多关于组件的高级用法，不过我不是大佬，等以后用到再写吧。写完收工睡觉
    