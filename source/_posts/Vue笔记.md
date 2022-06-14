---
title: Vue 3 基础
date: 2022/6/14 20:00:00
tags: 
  - vue
  - js
categories: 
  - Web前端
---

> Vue.js 是一个基于 **MVVM** 架构的渐进式框架，可以动态构建用户界面，解耦视图和数据
>
> - 官方 Doc ： [ Vue.js ](https://v3.cn.vuejs.org/guide/introduction.html) 

# 一、概述

## MVVM

- M（model）：构成页面内容的模型数据
- V（view）：展示数据的页面
- VM（view model）：将 `model` 中的数据解析到 `view` 中

开发者只需关心页面和数据的构成，无需关心页面和数据的状态关系，几乎不用操作 `DOM` 就可以完成页面和数据的交换

## 基本使用

1. 获取 `Vue` 核心库的 js 文件

```bash
npm install vue
```

2. 在页面中引入 `Vue` 

```html
<script src="js/vue.global.js"></script>
```



# 二、Vue 3 模板语法



## Vue 插值语法

### 

### **mustache**

使用 `mustache` 语法把 data、method 中的文本数据插入到 HTML 代码中，形如 `{{ exp }}` 

```html
<body>
    <div id="app">
        {{ name }}
    </div>
</body>
<script>
    //mustache语法
    const app = Vue.createApp({
        data() {
            return {
                name:"hello world"
            }
        }
    }).mount('#app')
</script>
```



### v-once

该指令后不需要跟表达式，表示元素和组件至渲染一次，不会随着数据改变而改变

```html
<div id="vonce" v-once>
    {{ name }}
    <button @click="change">切换</button>
</div>
```



### v-html

有时，我们从服务器请求到的数据是html代码，这时候用mustache语法会把html代码输出，但我们希望得到的是按照html格式来进行解析的内容，此时要用到 `v-html` 

```html
<body>
    <div id="vhtml" >
        <div v-html="url"></div>
    </div>
</body>
<script>
    //v-html 语法
    const vhtml = Vue.createApp({
        data() {
            return {
                url:"<p>v-html test</p>"
            }
        }
    }).mount('#vhtml')
</script>
```



### v-pre

类似转义符，将内容原封不动的显示出来，不做任何解析



### v-bind

网页中的属性、网址等，大多是从服务器请求得来的，这些数据暂存在 vue 中，在 HTML 中利用 `v-bind` 来实现动态绑定，该指令有一个语法糖，只留下一个 `:` 

```html
<body>
    <div id="vbind">
        <h3 :class="{active : isActive, line : isLine}">
            {{ name }}
        </h3>
    </div>
</body>
<script>
    //v-bind 语法
    const vhtml = Vue.createApp({
        data() {
            return {
                url:"<p>v-html test</p>"
            }
        }
    }).mount('#vbind')
</script>
```



## 事件处理



### v-on

前端开发需要经常与用户进行交互，此时就需要监听用户事件，比如点击、拖拽等。在Vue中，我们使用 `v-on` 来监听事件，该指令同样有一个语法糖，直接使用 `@` 绑定属性，当事件监听调用的方法没有参数时，可直接写方法名，不用加 `()` 

```html
<body>
	<div id="von">
        <p>{{ message }}</p>
        <button @click="reversemsg">反转字符串</button>
    </div>
</body>
<script>
    //v-on 语法
    const von = Vue.createApp({
        data() {
            return {
                message: "12345678"
            }
        },
        methods:{
            reversemsg: function(){
                this.message = this.message.split('').reverse().join('')
            }
        }
    }).mount('#von')
</script>
```



### v-on 修饰符

- stop：点击某一块区域时，不想让其中其他的子区域也触发
- prevent：防止表单进行提交，阻止事件的默认行为
- once：事件只触发一次
- keyup/keydown：监听某个按键的录入

```html
<body>
	<div id="von">
        <p>{{ message }}</p>
        <button @click.once="reversemsg">反转字符串</button>
        <input v-model="message" type="text" v-on:keyup.enter="keyclick">
    </div>
</body>
<script>
    const von = Vue.createApp({
        data() {
            return {
                message: "12345678"
            }
        },
        methods:{
            reversemsg() {
                this.message = this.message.split('').reverse().join('')
            },
            keyclick() {
                alert(this.message)
            }
        }
    }).mount('#von')
</script>
```



## 条件渲染与列表渲染



### v-if

使用 `v-if` 、 `v-else` 、`v-else-if` 做条件判断

```html
<body>
	<div id="vif">
        <h3 v-if="score >= 90">v-if 90</h3>
        <h3 v-else-if="score >= 80">v-if 80</h3>
        <h3 v-else>v-if else</h3>
    </div>
</body>
<script>
    //v-if 语法
    const vif = Vue.createApp({
        data() {
            return {
                score: 86
            }
        }
    }).mount("#vif")
</script>
```



### v-show

`v-show` 和 `v-if` 类似，都能决定元素是否显示，区别在于条件为 false 时：

- `v-if` 修饰的元素根本不会在 DOM 中
- `v-show` 是新增一个行内样式 `display:none` 

而条件为 true 时：

- `v-if` 会再创建一个元素
- `v-show` 只需要把 `display` 属性去掉即可

因此，当需要频繁切换时 `v-show` 更好，反之 `v-if` 更好



### v-for

使用 `v-for` 遍历数组和对象，绑定 `key` 值避免出错（`key` 只能用字符串或数字）

```html
<body>
    <div id="vfor">
        <table style="border-collapse: collapse;">
            <thead>
                <tr>
                    <th v-for="item in items">{{ item }}</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item,index) in info" :key="index">
                    <th style="border: 1px solid #000;">{{ item.name }}</th>
                    <td style="border: 1px solid #000;">{{ item.age }}</th>
                </tr>
            </tbody>
        </table>
    </div>
</body>
<script>
	//v-for 语法
    const vfor = Vue.createApp({
        data(){
            return {
                items: ['name','age'],
                info:[{
                    name: "Judy",
                    age: 31
                },{
                    name: "Tom",
                    age: 24
                }]
            }
        }
    }).mount("#vfor")
</script>
```



## 数据双向绑定



### v-model

`v-model` 用以实现表单元素和数据的双向绑定，实际上就是两个语法的结合（`v-bind` 绑定一个`value` 属性，`v-on` 给当前元素绑定`input` 事件），可参考 `v-on` 修饰符代码实例



### v-model 中的修饰符

- `lazy` ：默认情况下，`v-model` 是在 input 事件中同步输入框的数据，一旦数据有变，data 里面的数据也会自动改变；但有时我们不想让 data 中的数据立刻发生改变，就可以用 `lazy` 修饰符，让数据失去焦点或回车时才更新

```html
<div>
    <input type="text" v-model.lazy="message">
</div>
```

- `number` ：当我们往 input 里输入数字时，Vue 会自动把这些数字识别成字符串，如果想以数字的格式存储它，那么就需要添加 `number` 修饰符

```html
<div>
    <input type="text" v-model.number="age">
</div>
```

- `trim` ：如果想要去除 input 框中输入的空格，需要添加 `trim` 修饰符

```html
<div>
    <input type="text" v-model.trim="message">
</div>
```



# 三、数据控制



## 计算属性 computed



当需要对 data 里面的数据进行处理并显示时，需要用到 `computed` 属性。

相比 `methods` ，从最终的实现结果来看，两种方式是完全相同。然而，不同的是，**计算属性将基于它们的响应依赖关系进行缓存**，即计算属性只会在相关响应式依赖发生改变时重新求值，只要相关响应式依赖没有改变，计算属性会立即返回之前的计算结果，不必再重新执行函数。

因此，对于任何包含响应式数据的复杂逻辑，你都应该使用 **计算属性** 。

```html
<body>
    <div id="computed">
        <h3>
           {{ fullname }}
        </h3>
    </div>
</body>
<script>
	//computed 语法
    const computed = Vue.createApp({
        data(){
            return {
                firstname: "Stephen",
                lastname: "Curry"
            }
        },
        computed: {
            fullname(){
                return "computed-test: "+ this.firstname + " " + this.lastname
            }
        }
    }).mount("#computed")
</script>
```

#### 

### 计算属性的 Setter

计算属性默认只有 getter，不过在需要的时候你也可以提供一个 setter，更新相应的依赖。

```html
<body>
    <div id="computed">
        <h3>
           {{ fullname }}
        </h3>
    </div>
</body>
<script>
	//computed 语法
    const computed = Vue.createApp({
        data(){
            return {
                firstname: "Stephen",
                lastname: "Curry"
            }
        },
        computed: {
            fullname: {
                get(){
                    return "computed-test: "+ this.firstname + " " + this.lastname
                },
                set(fullname){
                    const names = fullname.split(' ')
                    this.firstname = name[names.length - 2]
                    this.lastname = name[names.length - 1]
                }
            }
        }
    }).mount("#computed")
</script>
```



## 侦听器 watch



Vue 通过 `watch` 自定义侦听器用于对数据进行监控，当监视的变量发生变化，监视器定义的响应操作会自动执行。

当需要在数据变化时执行异步操作或开销较大的操作时，这个方式是最有用的。

```html
<body>
    <div id="watch">
        <p>
            Do an add calculation:
            <input v-model="expr" />
        </p>
        <p>{{ answer }}</p>
    </div>
</body>
<script>
	//watch 语法
    const watchExampleVM = Vue.createApp({
        data() {
            return {
                expr: '1+1?',
                answer: '2'
            }
        },
        watch: {
            // 每当 question 发生变化时，该函数将会执行
            expr(newValue, oldValue) {
                if (newValue.indexOf('?') > -1) {
                    this.getAnswer()
                }
            }
        },
        methods: {
            getAnswer() {
                exprs = this.expr.split("?")[0].split("+")
                this.answer = eval(exprs[0])+eval(exprs[1])
            }
        }
    }).mount('#watch')
</script>
```



# 四、生命周期

## 图示

![https://v3.cn.vuejs.org/images/lifecycle.svg](../blog-assets/Vue笔记/lifecycle.svg)