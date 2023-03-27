# 通过npm手脚架创建第一个vue项目
# 认识目录
- vue在哪里路由让我访问到那个界面的？
# 一些基础概念
vue项目中``src/HelloWorld.vue``等文件进行熟悉。
此节主要介绍语法。
## 应用实例
每个 Vue 应用都是通过 createApp 函数创建一个新的 应用实例：
```js
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})
```
- where is 'vue' imported from?

## 根组件
我们传入 createApp 的对象实际上是一个组件，每个应用都需要一个“根组件”，其他组件将作为其子组件。

如果你使用的是单文件组件，我们可以直接从另一个文件中导入根组件。
```js

import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
```

## 挂载应用
应用实例必须在调用了 .mount() 方法后才会渲染出来。该方法接收一个“容器”参数，可以是一个实际的 DOM 元素或是一个 CSS 选择器字符串：
```html

<div id="app"></div>
```
```js

app.mount('#app')
```
- where is the '#app'?

## 多个应用实例

应用实例并不只限于一个。createApp API 允许你在同一个页面中创建多个共存的 Vue 应用，而且每个应用都拥有自己的用于配置和全局资源的作用域。
```js

const app1 = createApp({
  /* ... */
})
app1.mount('#container-1')

const app2 = createApp({
  /* ... */
})
app2.mount('#container-2')
```

# 模板语法
## 文本插值

最基本的数据绑定形式是文本插值，它使用的是“Mustache”语法 (即双大括号)：
```template

<span>Message: {{ msg }}</span>
```

以msg为变量，js中的msg 属性更改时html中的它也会同步更新。

双大括号会将数据解释为纯文本，而不是 HTML。若想插入 HTML，你需要使用 v-html 指令：
```template

<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
**但是在网站上动态渲染任意 HTML 是非常危险的，因为这非常容易造成 XSS 漏洞。**

双大括号中支持JavaScript表达式：
```js
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>
```
在 Vue 模板内，JavaScript 表达式可以被使用在如下场景上：
- 在文本插值中 (双大括号)
- 在任何 Vue 指令 (以 v- 开头的特殊 attribute) attribute 的值中

每个绑定仅支持单一表达式，也就是一段能够被求值的 JavaScript 代码。一个简单的判断方法是是否可以合法地写在 return 后面。

因此，下面的例子都是无效的：
```js

<!-- 这是一个语句，而非表达式 -->
{{ var a = 1 }}

<!-- 条件控制也不支持，请使用三元表达式 -->
{{ if (ok) { return message } }}

```
双括号内也可以调用函数：
```js
<span :title="toTitleDate(date)">
  {{ formatDate(date) }}
</span>
```
**绑定在表达式中的方法在组件每次更新时都会被重新调用**。

[v-bind的使用场景](https://zhuanlan.zhihu.com/p/360230036)

## Attribute 绑定
#

双大括号不能在 HTML attributes 中使用。想要响应式地绑定一个 attribute，应该使用 v-bind 指令：
```template

<div v-bind:id="dynamicId"></div>
```

v-bind 指令指示 Vue 将元素的 id attribute 与组件的 dynamicId 属性保持一致。如果绑定的值是 null 或者 undefined，那么该 attribute 将会从渲染的元素上移除。

因为 v-bind 非常常用，简写语法：
```template

<div :id="dynamicId"></div>
```

## 布尔型 Attribute

布尔型 attribute 依据 true / false 值来决定 attribute 是否应该存在于该元素上。disabled 就是最常见的例子之一。

v-bind 在这种场景下的行为略有不同：
```template

<button :disabled="isButtonDisabled">Button</button>
```

当 isButtonDisabled 为真值或一个空字符串 (即 ``<button disabled="">``) 时，元素会包含这个 disabled attribute。而当其为其他假值时 attribute 将被忽略。


## 动态绑定多个值

如果你有像这样的一个包含多个 attribute 的 JavaScript 对象：
```js

data() {
  return {
    objectOfAttrs: {
      id: 'container',
      class: 'wrapper'
    }
  }
}
```
通过不带参数的 v-bind，你可以将它们绑定到单个元素上：
```template

<div v-bind="objectOfAttrs"></div>
```
## 指令 Directives
指令是带有 v- 前缀的特殊 attribute。Vue 提供了许多内置指令，包括上面我们所介绍的 v-bind 和 v-html。

指令 attribute 的期望值为一个 JavaScript 表达式 (除了少数几个例外，即之后要讨论到的 v-for、v-on 和 v-slot)。一个指令的任务是在其表达式的值变化时响应式地更新 DOM。以 v-if 为例：
```template

<p v-if="seen">Now you see me</p>
```

这里，v-if 指令会基于表达式 seen 的值的真假来移除/插入该 <p> 元素。

另一个例子是 v-on 指令，它将监听 DOM 事件：
```template

<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

这里的参数是要监听的事件名称：click。v-on 的缩写是 @ 字符。


###  指令完整语法
最后，Vue中的指令语法是：
![](https://cn.vuejs.org/assets/directive.69c37117.png)

# 响应式基础
## 声明响应式状态 
vue用 data 选项来声明组件的响应式状态，所以会经常看到
```js
export default {
  data() {
    return {
      count: 1
    }
  }
```
这些返回值仅在首次创建时被返回。若所需的值还未准备好，在必要时也可以使用 null、undefined 或者其他一些值占位。

虽然也可以不用上述方法，直接向组件实例添加新属性，但这个属性也就不是响应式了。

## 声明方法

要为组件添加方法，我们需要用到 methods 选项。这样声明methods，如下方``increment``：
···js

export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  mounted() {
    // 在其他方法或是生命周期中也可以调用方法
    this.increment()
  }
}
···
很多method在模板中它们常常被用作事件监听器：
···js
<button @click="increment">{{ count }}</button>
···
## DOM 更新时机

当你更改响应式状态后，DOM 会自动更新。然而，你得注意 DOM 的更新并不是同步的：Vue 将缓冲它们直到更新周期的 “下个时机” 以确保无论你进行了多少次状态更改，每个组件都只更新一次。

# 推荐写法
