# 学点VUE：术语清单

# VUE Glossary

## 2021 年 6 月 5 日

## Jun 5 2021

### 前言

本人并未打算深入学习VUE，所以这篇博客只是列清单式的学习，仅作了解。

[参考链接](https://www.bilibili.com/video/BV15741177Eh?p=23)。

### 一、Hello VUEjs！

**1.1 认识VUE**

- VUE的读音
- VUE的“渐进式”
- VUE的特点

**1.2 下载VUE**

- CDN引入
- 下载引入
- npm引入

**1.3 初识VUE**

- Mustache：VUE的响应式
- `v-for`：列表，遍历数组和对象的不同方式
- `v-on`：事件监听，缩写为@，比如@click

**1.4 VUE中的MVVM**

- VUE中的View、ViewModel和Model

**1.5 Options**

- el：绑定对象
- data：数据
- methods：方法
- 生命周期函数
- computed：计算方法

### 二、插值语法

- mustache语法
- `v-once`
- `v-html`
- `v-text`
- `v-pre`
- `v-cloak`

### 三、动态绑定

**3.1 基础属性**

- `vbind:src`
- `vbind:href`

**3.2 动态绑定class**

- 对象语法

这样写html，

```html
<div :class="{ active: isActive }"></div>
```

这样写vue，

```vue
data() {
  return {
    isActive: true,
  }
}
```

- 数组语法

**3.3 动态绑定style**

- 对象语法
- 数组语法

> 关于class和style的绑定，更多参考[Vue官方文档](https://v3.cn.vuejs.org/guide/class-and-style.html#%E7%BB%91%E5%AE%9A-html-class)。

### 四、计算属性

- 计算属性的本质：对象
- 计算属性的get和set方法
- 对比methods和计算属性

计算属性只调用一次，性能更高，计算属性会储存缓存，只有监听到变化时才会再调用。

### 五、VUE指令

- VUE基础指令
- `v-html`
- `v-text`
- `v-show`
- `v-if`

条件判断。

- `v-else-if`
- `v-else`
- `v-for`

循环遍历：遍历数组/对象，有无索引值，拿key/value。

- `v-on`（语法糖？）

事件监听的参数问题，以及如何传递event。

- `v-bind`（语法糖？）

> 参考[VUE3官方中文文档：指令](https://vue3js.cn/docs/zh/api/directives.html#%E6%8C%87%E4%BB%A4)

### 六、v-DOM（虚拟DOM）

- v-DOM的优化和复用
- v-DOM的key和diff算法
- 数组哪些方法是响应式的？

### 七、双向绑定

- v-model：表单元素和数据的双向绑定
- 双向绑定原理：v-bind`:`绑定一个value，v-on`@`绑定浏览器事件
- v-mode表单：input的两个个type（radio或checkbox）
- v-model表单：结合select和option
- 值绑定
- v-model的修饰符：lazy, number, trim

> 参考：[VUE3中文文档-表单输入绑定](https://v3.cn.vuejs.org/guide/forms.html#number)

### 八、组件化开发

- 组件化思想
- VUE组件化开发步骤：创建组件构造器、注册组件、使用组件
- VUE组件基础：[链接](https://v3.cn.vuejs.org/guide/component-basics.html#%E5%9F%BA%E6%9C%AC%E5%AE%9E%E4%BE%8B)
- 全局组件和局部组件
- 父组件和子组件

> 看到这里的时候，突然发现这么多的东西，竟然只是VUE基础中的基础，官方文档后面还有一大堆深入理解的东西，真是不知道什么时候才能学完。
>
> —— 吐槽

- 组件注册：模板语法与Component
- 语法糖：`Vue.component()`直接跳过`Vue.extend()`
- 进一步：模板抽离
- 组件内部的数据和方法

> 思考：组件中的data为什么必须是一个函数？

```javascript
const comp = Vue.component('comp', {
	template: '#template-id',
  data() {
    return {
      title: 'abc',
      count: 0,
    }
  },
  methods: {
    increment() {
      ++this.counter;
    },
    decrement() {
      --this.counter;
    }
  }
})
```

```html
<template id="tempale-id">
  <div>
    <p>当前计数： {{counter}}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>
```

### 九、组件通信

- 父传子：props

可以定义props的类型和默认值，当类型是引用变量是，default必须是一个函数返回。

```javascript
props: {
  str: {
    type: String,
    default: 'abc',
    required: true,
  }
  arr: {
    type: Array,
    default() {
      return [];
    },
  }
}
```

> 详见[VUE3中文文档之props](https://vue3js.cn/docs/zh/guide/component-props.html#props)

- props 的大小写命名 (camelCase vs kebab-case)

HTML 中的 attribute 名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。这意味着当你使用 DOM 中的模板时，camelCase (驼峰命名法) 的 prop 名需要使用其等价的 kebab-case (短横线分隔命名) 命名：

```js
const app = Vue.createApp({})

app.component('blog-post', {
  // camelCase in JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

```html
<!-- kebab-case in HTML -->
<blog-post post-title="hello!"></blog-post>
```

重申一次，如果你使用字符串模板，那么这个限制就不存在了。

- 子传父：自定义事件

> React的数据子传父是通过父组件的回调函数通信。

子组件发射（`this.$emit()`）事件，父组件通过`v-bind:`监听事件。注意，父子组件都需要在`methods`里做声明，数据也在methods里的方法传递。

> 参考[VUE3中文文档之自定义事件](https://vue3js.cn/docs/zh/guide/component-custom-events.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BA%8B%E4%BB%B6)

- 项目实例：[Github地址](https://github.com/coderwhy/HYMall)
- 父子间通信：结合双向绑定。

把`v-model`拆解为`v-bind`和`v-on`，在父组件实现方法来接受子组件`v-on`发出函数的`this.$emit()`。

- `watch`监听数据改变，产生副作用

> [VUE3中文文档：watch、computed和watchEffect](https://vue3js.cn/docs/zh/api/computed-watch-api.html#computed)

- 组件的直接访问方式
  - 父组件访问子组件：`$children`或`$refs`，后者用得更多；
  - 子组件访问父组件：`$parent`和`$root`；

### 十、插槽

> 参考：[VUE3中文文档之插槽](https://vue3js.cn/docs/zh/guide/component-slots.html#%E6%8F%92%E6%A7%BD)

- 可复用性和可扩展性：让使用者决定组件内容
- 具名插槽：`name`和`v-slot`属性
- 编译作用域：插槽里数据的使用要特别注意作用域
- 作用域插槽：父组件替换子组件插槽里面的标签，**内容**由子组件决定。

用法：`slot-scope="slot"`。

> 说起来有点抽象，给一例子，比如我们有一个子组件是一个列表，需要展示，就可以通过父组件来告诉子组件怎样展示：竖着，横着或者其他格式。

### 十一、Webpack、Vite与VUE组件

我们先来看看vite初始化时的一个VUE组件`HelloWorld.vue`，

```vue
<template>
  <div>
    <h1>{{ msg }}</h1>

    <p>
      <a href="https://vitejs.dev/guide/features.html" target="_blank">
        Vite Documentation
      </a>
      |
      <a href="https://v3.vuejs.org/" target="_blank">Vue 3 Documentation</a>
    </p>

    <button @click="state.count++">count is: {{ state.count }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test hot module replacement.
    </p>
  </div>
</template>

<script setup>
import { defineProps, reactive } from "vue";

defineProps({
  msg: String,
});

const state = reactive({ count: 0 });
</script>

<style scoped>
a {
  color: #42b983;
}
</style>
```

分为三个部分，

- <`template>`：定义html+data+state的模板；
- `<script setup>`：定义数据、状态和响应式，以及import各种东西；
- `<style scoped>`：定义本组件的css样式；

我们需要webpack或者vite配置`vue-loader`，以至于能够加载vue组件。

我们先来看看Vite的plugins写法，（现在还看不懂）

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()]
})
```

再来看看webpack的写法，

```js
// webpack.config.jsconst { VueLoaderPlugin } = require('vue-loader')

module.exports = {
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      // this will apply to both plain `.js` files
      // AND `<script>` blocks in `.vue` files
      {
        test: /\.js$/,
        loader: 'babel-loader'
      },
      // this will apply to both plain `.css` files
      // AND `<style>` blocks in `.vue` files
      {
        test: /\.css$/,
        use: [
          'vue-style-loader',
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    // make sure to include the plugin for the magic
    new VueLoaderPlugin()
  ]
}
```

详见[链接](https://vue-loader.vuejs.org/guide/#manual-setup)。

> `vue-loader`并不在webpack官方提供的loader里，而是vue自己实现的。

### 十二、VUE-CLI脚手架

请直接学习**3**版本。vue-cli 3是基于webpack4的，原则是“零配置”。

安装，

```shell
npm install @vue/cli
```

创建项目，

```shell
vue create hello-world
```

**🌟插件和preset**

**插件**可以修改 webpack 的内部配置，也可以向 `vue-cli-service` 注入命令。在项目创建的过程中，绝大部分列出的特性都是通过插件来实现的。一个 **Vue CLI prese**t 是一个包含创建新项目所需预定义选项和插件的 JSON 对象，让用户无需在命令提示中选择它们。

这里有一个 preset 的示例：

```json
{
  "useConfigFiles": true,
  "cssPreprocessor": "sass",
  "plugins": {
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-eslint": {
      "config": "airbnb",
      "lintOn": ["save", "commit"]
    },
    "@vue/cli-plugin-router": {},
    "@vue/cli-plugin-vuex": {}
  }
}
```

其他的看[中文文档](https://cli.vuejs.org/zh/guide/)。

### 十三、Vue-Router

**🌟路由与前端路由**

路由是通过互联的网络把信息从源地址传输到目的地址的活动。

- 后端路由：后端来处理URL和页面的映射关系，通常应用在服务器端渲染中。

步骤如下，url发送到服务器，服务器通过正则匹配该url，然后交给一个controller来处理，controller生成页面数据，返回到前端，最后由端（浏览器）完成渲染。

- AJAX与前后端分离

后端只提供API返回数据，前端通过AJAX获取数据，然后由JS操作将数据渲染到view层。这样做使得前端可以专注于交互和可视化，后端专注于写接口，这样不同的view层（比如native、web或客户端应用）可以使用同一套API。

后端不负责任何界面内容。

并且，服务器也可以分离，一个服务器是静态资源服务器（html+css+js，html和css可以直接挂载，js代码需要执行，在执行过程中可以进行AJAX操作），另一个是提供API接口的服务器，以接受AJAX的返回，这里通常要提供大量数据。

**这个时候，浏览器的渲染过程基本可以将给前端写js处理，这就叫前端渲染。**

- 单页面富应用（SPA，Single Page Application）

SPA最主要的特点就是在前后端分离的基础上加上了一层**前端路由**，也就是由前端来维护一套路由规则。这个时候，虽然url变化了，但是没有向后端发出请求，只是从已经获取的js中判断是哪个页面，找出需要渲染的页面组件，然后渲染到网页上。

**这个时候，就可以由前端接管url到页面的映射关系了。前端路由的核心是，url改变时，整个页面不会进行刷新。**

**🌟history**

在h5之前，要改变页面的url而不引起刷新，则需要`location.hash='someStr'`，号

提供了[history](https://developer.mozilla.org/en-US/docs/Web/API/History)的Web API，通过

```js
history.pushState(state, title [, url])
```

可以改变页面的url而不引起刷新。

> The **`History`** interface allows manipulation of the browser *session history*, that is the pages visited in the tab or frame that the current page is loaded in.

history提供很多方法， 比如前进（`.forward()`）、后退（`.back()`）、到（`.go()`）等方法。

`.pushState()`方法和push的区别是，replace是不能返回的，而push是压入栈中，这样历史栈数据都被保存下来的。

**🌟vue-router**

来看示例，

```html
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 出口：路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

来看js，

```javascript
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义 (路由) 组件。
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes // (缩写) 
})

// 4. 创建和挂载根实例。
const app = new Vue({
  router
}).$mount('#app')
```

**🌟动态路由**

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 `User` 组件，对于所有不同的用户，都要使用这个组件来渲染。那么，我们可以在 `vue-router` 的路由路径中使用“动态路径参数”(dynamic segment) 来达到这个效果：

```js
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

在组件内使用动态参数，

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

动态路由可以方便地创建**404页面**，常规参数只会匹配被 `/` 分隔的 URL 片段中的字符。如果想匹配任意路径，我们可以使用通配符 (`*`)：

```js
{
  path: '*'
}
```

当使用*通配符*路由时，请确保路由的顺序是正确的，也就是说含有*通配符*的路由应该放在最后。路由 `{ path: '*' }` 通常用于客户端 404 错误。如果你使用了*History 模式*，请确保[正确配置你的服务器](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。

**🌟嵌套路由**

要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id',
      component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

此时，基于上面的配置，当你访问 `/user/foo` 时，`User` 的出口是不会渲染任何东西，这是因为没有匹配到合适的子路由。如果你想要渲染点什么，可以提供一个 空的子路由` { path: '', component: UserHome }`。

**🌟参数传递**

动态路由就是最基本的参数传递的方法，通过`params`，

还有一种方式是`query`参数。

**🌟导航守卫**

“导航”表示路由正在发生改变。正如其名，`vue-router` 提供的导航守卫主要用来通过跳转或取消的方式守卫导航。有多种机会植入路由导航过程中：全局的, 单个路由独享的, 或者组件级的。

完整的导航流程如下，

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。

**🌟路由懒加载**

当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

结合 Vue 的[异步组件](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#异步组件)和 Webpack 的[代码分割功能](https://doc.webpack-china.org/guides/code-splitting-async/#require-ensure-/)，轻松实现路由组件的懒加载。

首先，可以将异步组件定义为返回一个 Promise 的工厂函数 (该函数返回的 Promise 应该 resolve 组件本身)：

```js
const Foo = () =>
  Promise.resolve({
    /* 组件定义对象 */
  })
```

第二，在 Webpack 2 中，我们可以使用[动态 import ](https://github.com/tc39/proposal-dynamic-import)语法来定义代码分块点 (split point)：

```js
import('./Foo.vue') // 返回 Promise
```

**结合这两者，这就是如何定义一个能够被 Webpack 自动代码分割的异步组件。**

```js
const Foo = () => import('./Foo.vue')
```

B有时候我们想把某个路由下的所有组件都打包在同个异步块 (chunk) 中。只需要使用 [命名 chunk ](https://webpack.js.org/guides/code-splitting-require/#chunkname)，一个特殊的注释语法来提供 chunk name (需要 Webpack > 2.4)。

```js
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

这些`Foo`，`Bar`什么的最后都写在router的component属性中，

```js
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar },
  ]
})

// 或者直接这样写，
const router = new VueRouter({
  routes: [
    { path: '/foo', component: () => import('./Foo.vue') },
    { path: '/bar', component: () => import('./Bar.vue') },
  ]
})
```

**🌟keep-alive**

keep-alive是Vue的内置组件，类似React的useMemo()。

> `<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 `<transition>` 相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在组件的父组件链中。当组件在 `<keep-alive>` 内被切换，它的 `activated` 和 `deactivated` 这两个生命周期钩子函数将会被对应执行。
>
> 
>
> `<transition>` 元素作为**单个**元素/组件的过渡效果。`<transition>` 只会把过渡效果应用到其包裹的内容上，而不会额外渲染 DOM 元素，也不会出现在可被检查的组件层级中。

基本用法如下，

```vue
<!-- 基本 -->
<keep-alive>
  <component :is="view"></component>
</keep-alive>
```

### 十四、Vuex

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的**状态 (state)**。Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

使用方式如下，

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

**现在，你可以通过 `store.state` 来获取状态对象，以及通过 `store.commit` 方法触发状态变更：**

```js
store.commit('increment')

console.log(store.state.count) // -> 1
```

为了在 Vue 组件中访问 `this.$store` property，你需要为 Vue 实例提供创建好的 store。Vuex 提供了一个从根组件向所有子组件，以 `store` 选项的方式“注入”该 store 的机制：

```js
new Vue({
  el: '#app',
  store: store,
})
```

现在我们可以从组件的方法提交一个变更：

```js
methods: {
  increment() {
    this.$store.commit('increment')
    console.log(this.$store.state.count)
  }
}
```

再次强调，我们通过提交 mutation 的方式，而非直接改变 `store.state.count`，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutation。

> 为什么需要一个状态管理器，而不是所有组件都共享一个变量呢？答案也很简单：**响应式！**

### 十五、axios之网络请求封装

> 略，要学的话去看文档。



这次VUE就看到这里吧！已经想去做其他事了，不看了，以后有机会再补充。
