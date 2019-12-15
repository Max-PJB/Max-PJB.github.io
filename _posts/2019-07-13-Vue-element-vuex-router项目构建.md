---
title: vue 开发入门
categories:
  - vue
  - 前端
  - 前后端分离
tags:
  - js
  - vue
---

## 先安装 node.js

一路安装下去，就好了。

## 打开命令行工具，安装 npm

1. 设置源： `npm config set registry https://registry.npm.taobao.org `

2. 验证是否成功： `npm config get registry`

3. 安装cnpm： `npm install -g cnpm –registry=https://registry.npm.taobao.org`

## 安装 vue-cli 开始构建项目

1. 安装vue-cli： `cnpm install -g @vue/cli`

2. 创建 vue 项目，进入到你的 workspace，需要存放项目的地方：`vue create my-project` ,项目名必须全部小写，否则报错 `Warning: name can no longer contain capital letters`

然后就会进入项目配置的选择： 根据自身需要选择，最新的vue-cli 可以直接在这里就安装 vuex 和 vue-router，那就把他选上吧

```
    Successfully created project nlp-web.
👉  Get started with the following commands:
 $ cd my-project
 $ npm run serve
 ```

看到这个就开心了啊， npm run serve 和 3.0之前的 npm run dev 是一个意思，看看 config 里面的配置就知道了，只是换个名字而已

3. 按照指令， `cd my-project` & `npm run serve` 试试看咯

```
 DONE  Compiled successfully in 10677ms                                                                         10:09:02

  App running at:
  - Local:   http://localhost:8080/
  - Network: http://10.33.247.31:8080/

  Note that the development build is not optimized.
  To create a production build, run npm run build.

```

打开浏览器，访问 `localhost:8080` 就可以看到啦

## 安装 vue-router , 配置 （如果使用的是最新的 vue-cli ，这里可以在上面自动选择的哦！）

(router官网)[https://router.vuejs.org/zh/]

> vue-router 是页面级别的导航，到底还是在 App.vue 下，我一直把他当做功能更强大的 tab 页
>

`cnpm install vue-router --save`

在 src 下新建 router/router.js
```
import Vue from 'vue'
import VueRouter from 'vue-router'

// 引入组件
import hello from "@/components/HelloWorld";
import routerTest from "@/components/RouterTest"


// 要告诉 vue 使用 vueRouter
Vue.use(VueRouter);

const routes = [
	/** 当首次进入页面的时候，页面中并没有显示任何内容。
		这是因为首次进入页面时，它的路径是 '/'，我们并没有给这个路径做相应的配置。
		一般，页面一加载进来都会显示home页面，我们也要把这个路径指向home组件。
		但是如果我们写{ path: '/', component: Home },vue 会报错，因为两条路径却指向同一个方向。,
		这怎么办？这需要重定向，所谓重定向，就是重新给它指定一个方向，它本来是访问 / 路径，我们重新指向‘/home’,
		它就相当于访问 '/home', 相应地, home组件就会显示到页面上。
		vueRouter中用 redirect 来定义重定向。 */
	{
		path: "/index",
		component: hello
	},
	{
		path: "/home",
		redirect: "/index",
	},
	{
		path: "/",
		redirect: "/index",
	},
	{
		path: "/test",
		// 这里采用命名路由
		name: "pageTest",
		component: routerTest,
	}
]

var router = new VueRouter({
	routes
})

export default router;
```

`main.js` 中配置

```
// 导入 vue-router 的配置注入
// import router 的router 一定要小写， 不要写成Router, 否则报 can't match的报错
import router from './router/router.js'

new Vue({
	router,
	store,
  render: h => h(App),
}).$mount('#app')
```

`App.vue` 中测试，啥也不需要，直接在 template 里面加，其中 `<router-view></router-view>` 就是每个路由被启动后，页面渲染到这里，默认是进入到 home

```
<template>
  <div id="app">

		<router-link to="/test">test</router-link>
		<router-link to="/home">home</router-link>

		<!-- 对应的组件内容渲染到router-view中 -->
		<router-view></router-view>
  </div>

</template>
```

## vuex ， persistedstate 安装

(vuex官网)[https://vuex.vuejs.org/zh/installation.html]

> vuex 是缓存
>
> persistedstate 是持久化，vuex 在使用的时候页面刷新，数据就都没有了，所以需要这个 persistedstate 插件

进入到项目目录，`cd my-project `
安装命令： `cnpm install vuex-persistedstate --save` 和 `cnpm install vuex --save`

src 下新建文件 `store/store.js`

```
import Vue from 'vue'
import Vuex from 'vuex'
import createPersistedState from "vuex-persistedstate"

Vue.use(Vuex)

const store = new Vuex.Store({
	state: {
		todos: [{id: 1,text: '...',done: true},
			{id: 2,text: '...',done: false},
			{id: 3,text: '...',done: true},
				{id: 4,text: '...',done: true},
		],
		myname:"pengjb",
		yourname:"hehe"
	},
	getters: {
		doneTodos: state => {
			return state.todos.filter(todo => todo.done)
		}
	},
	mutations: {
		add(state,n) {
			state.todos.push({
				id: n,
				text: '...',
				done: true
			})
		}
	},
	actions: {
		ppp1(context,arg){
			context.commit("add",arg)
		},
		// 使用参数结构，{commit} 获取 context中的 commit 函数
		//Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，
		//因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。
		//但是 context 却不等同于 store
		ppp2 ({ commit }	) {
			commit('add',7)
		}
	},
	// 使用 vuex 的持久化插件  vuex-persistedstate，默认持久化所有state
	plugins: [createPersistedState()]
})

export default store
```

`main.js` 中需要配置
```
// 导入 vuex 的 根配置注入
import store from './store/store.js'

new Vue({
	router,
	store,
  render: h => h(App),
}).$mount('#app')
```









