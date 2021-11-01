---
title: "从无到有的 Node.js+Vue.js+MongoDB 前后端分离项目(二)"
catalog: true
date: 2018-11-03 15:40:27
subtitle: "一个自制的旅游网站"
header-img: "code.jpg"
tags:
- Site
- Blog
- Code
catagories:
- Hexo
---
>前面已经完成了后台 Node + Express 框架的搭建，以及成功连接上 MongoDB 数据库。如果中间没有什么问题的话，我们就可以开始下一步啦，那就是完成前台搭建，至于为什么不直接写接口，我觉得还是有部分的前台支持，写起来会更加舒服一点。我们前台使用的是 Vue.js，这是我接触的第二个前端框架，第一个是 AngularJS，其实前者对后者有很多借鉴的地方，所以学起来不是很难，加上 Vue.js 的中文文档也很方便阅读，所以更加事半功倍了。话不多说，我们开始手把手搭建吧。

>如果你对 Vue.js 不够了解，请移步[这里](https://cn.vuejs.org/v2/guide/).

# 搭建前台
---
首先初始化一个 Vue 脚手架，官方提供了很多种，具体请看[这里](https://github.com/vuejs/vue-cli/tree/master)，我是选用了其中的[webpack](https://github.com/vuejs-templates/webpack)，也许我的目录结构与你们的不同，这是因为新版的已经升级到 vue-cli 3，些许不同没有关系。大致目录如下：

```
.
├── build/                      # webpack config files
│   └── ...
├── config/
│   ├── index.js                # main project config
│   └── ...
├── src/
│   ├── main.js                 # app entry file
│   ├── App.vue                 # main app component
│   ├── components/             # ui components
│   │   └── ...
│   └── assets/                 # module assets (processed by webpack)
│       └── ...
├── static/                     # pure static assets (directly copied)
├── test/
│   └── unit/                   # unit tests
│   │   ├── specs/              # test spec files
│   │   ├── eslintrc            # config file for eslint with extra settings only for unit tests
│   │   ├── index.js            # test build entry file
│   │   ├── jest.conf.js        # Config file when using Jest for unit tests
│   │   └── karma.conf.js       # test runner config file when using Karma for unit tests
│   │   ├── setup.js            # file that runs before Jest runs your unit tests
│   └── e2e/                    # e2e tests
│   │   ├── specs/              # test spec files
│   │   ├── custom-assertions/  # custom assertions for e2e tests
│   │   ├── runner.js           # test runner script
│   │   └── nightwatch.conf.js  # test runner config file
├── .babelrc                    # babel config
├── .editorconfig               # indentation, spaces/tabs and similar settings for your editor
├── .eslintrc.js                # eslint config
├── .eslintignore               # eslint ignore rules
├── .gitignore                  # sensible defaults for gitignore
├── .postcssrc.js               # postcss config
├── index.html                  # index.html template
├── package.json                # build scripts and dependencies
└── README.md                   # Default README file
```
这是官方的目录结构详解，我就不一一阐述了。了解了大概目录以后，我们开始下一步。

通过 package.json 文件可以看到项目中引入了哪些插件，可以一个个的装上：
```
"dependencies": {
    "axios": "^0.18.0",   
    "babel-polyfill": "^6.26.0", 
    "element-ui": "^2.4.8",    
    "es6-promise": "^4.2.5",     
    "moment": "^2.22.2",
    "sass": "^1.14.3",
    "vue": "^2.5.2",
    "vue-cropper": "^0.4.6",
    "vue-router": "^3.0.1",
    "vuex": "^3.0.1"
},
  ```
本项目使用的 UI 框架是Element-UI，因为在实习过程中的公司使用的是这个 UI 框架，用起来感觉还不错，对里面的 api 等也有些了解，所以采用了这个框架。样式方面用的是 sass，比普通的 css 方便很多，不了解的可以去学习一下，点击[这里](https://www.sass.hk/docs/)。

## 构建项目
---
项目 npm run dev 跑起来以后，就可以看到 Vue 的欢迎页面，接下来我们把项目变成自己的。
首先引入 Element-UI。
打开 src 目录下的main.js,加入这么一行代码：
` import Element from 'element-ui'`
然后在下面加上：
```
Vue.use(Element, {
  size: 'medium', // set element-ui default size
})
```
这样 element-ui 就可以在全局使用啦。
顺便引入其他我们需要的插件，main.js代码如下：
```
import Vue from 'vue'
import 'babel-polyfill'
import App from './App'
import router from './router'
import store from './store'

import Element from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
import './assets/fonts/iconfont.css'

import '@/styles/index.scss'

Vue.config.productionTip = false

Vue.use(Element, {
  size: 'medium', // set element-ui default size
})


/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
})
```
这时你的项目应该会报错，没错，因为有很多文件你并没有，现在我们来解释一下上面的部分文件，router 是我们的路由文件，需要创建一个文件夹。store 使我们的"仓库",是Vue的状态管理仓库，也需要创建相关文件夹。iconfont.css 使我们后面会用到的，iconfont 图标文件，暂时可以注释掉下面的 index.scss 是我们的样式公共文件夹，然后把需要用到的东西挂载在根节点上就OK了。具体目录如下：

## axios配置
下面我们主要讲一下对于请求，也就是 axios 的配置，具体代码如下：
```
import Vue from 'vue'
import axios from 'axios'
import qs from 'qs'
import {
  Message
} from 'element-ui'
import store from '@/store'

const service = axios.create({
  baseURL: 'http://192.168.1.105:3333/', //api的base_url
  // baseURL: 'http://192.168.0.104:3333/',
  timeout: 120000,
  method: 'post',
  headers: {
    'Content-Type': 'application/json; charset=utf-8'
  }
})

service.interceptors.request.use(config => {
  if (store.getters.token) {
    config.headers['_TK'] = store.getters.token //让每个请求携带token
  }
  return config
}, error => {
  console.log(error)
  Promise.reject(error)
})

service.interceptors.response.use(
  response => {
    if (response.data.code == 200) {
      return Promise.resolve(response)
    } else if (response.data.code == 401 || response.data.code == 402) {
      // this.$store.dispatch('LogOut').then(() => this.$store.push('./login'))
        Message({
          message: response.data.message,
          type: 'error',
          duration: 3000
        })
      return Promise.reject(response)
    } else{
      Message({
        message: response.data.message,
        type: 'error',
        duration: 3000
      })
      return Promise.reject(response)
    }
  },
  error => {
    this.$store.commit('REM_TOKEN');
    console.log('err' + error) //for  DEBUG
    Message({
      message: error.data.message,
      type: 'error',
      duration: 3000
    })
    return Promise.reject(error)
  }
)

export default service
```
对于上面这么多代码，其实很简单，第一部分就是引入相关文件，然后初始化一个 axios 请求，并配置好相关请求信息。最重要的是下面两个方法，第一个是请求拦截器，主要的目的是让每次请求都在请求头中携带 token，这样每次都可以在后台进行 token 验证。后面的是一个响应拦截器，对于返回的数据进行判断，如果返回code是200，就获取相关返回信息，如果是其他的错误code，就提醒相关bug。

## 配置路由
---
接下来我们看看项目路由如何配置，先打开app.vue文件，设置路由：
```
<template>
  <div id="app">
    <keep-alive>
      <router-view v-if="$route.meta.keepAlive"></router-view>
    </keep-alive>
    <router-view v-if="!$route.meta.keepAlive"></router-view>
  </div>
</template>
```
最外层的keep-alive是用于页面缓存，是否应用缓存可以在router中进行设置，下面的router-view就是表示该项目使用vue-router进行页面跳转，我们去看看router该如何配置：
```
import Vue from 'vue'
import Router from 'vue-router'

const _import = file => () => import('@/views/' + file + '.vue')
Vue.use(Router)
```
引入必要文件，然后定义一个文件路径，这个根据个人喜好定义了，然后use(Router)。我展示一个基本的路由配置，其实相关用法官方文档都很详细：
```
{
      path: '/flight',
      name: 'flight',
      component: _import('flight/index'),
      meta: {
        title: '航班信息',
        keepAlive: true
      },
      children: [
        {
          path: 'flightInfo',
          name: 'flightInfo',
          component: _import('flight/flightInfo/index'),
          // meta: { keepAlive: true },
        },
        {
          path: 'flightPay',
          name: 'flightPay',
          component: _import('flight/flightPay/index'),
          meta: { keepAlive: true },
        },
      ]
    },
```
如上，很简单的路由配置，可以嵌套路由，并且可以自己决定是否使用页面缓存。

## Vuex
---
最后我们来讲讲 Store，其实如果你的项目够简单，或者页面之间关联并不大的话，你就该考虑考虑是否有必要使用Vuex。引用官方的一段话：
虽然 Vuex 可以帮助我们管理共享状态，但也附带了更多的概念和框架。这需要对短期和长期效益进行权衡。
如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的。确实是如此——如果您的应用够简单，您最好不要使用 Vuex。一个简单的 store 模式就足够您所需了。但是，如果您需要构建一个中大型单页应用，您很可能会考虑如何更好地在组件外部管理状态，Vuex 将会成为自然而然的选择。

当然我选择使用了，对于学习Vue的前端人员，这是技能。

因为我的项目中分为几个业务模块，所以我相对应的设置了几个 module,就拿用户模块来说吧，先看看store目录下的index.js：
```
import Vue from 'vue'
import Vuex from 'vuex'
import user from './modules/user'
import getters from './getters'

Vue.use(Vuex);

const store = new Vuex.Store({
    modules: {
        user
    },
    getters
})

export default store
```
引入相关文件，然后应用相关模块，需要提到getter.js，官方文档中也有相关介绍，用于提供一个公共从state中获取数据的方法，例子如下：
```
const getters = {
    token: state => state.user.token,
    loginTime: state => state.user.loginTime,
    userphone: state => state.user.userphone,
    userInfo: state => state.user.userInfo
}

export default getters
```
这样在其他页面就可以这样获取需要的内容：this.$store.getters.userphone。
在页面上需要改动store中的内容需要dispatch相关方法，然后触发commit，去修改相关内容。以登录为例：
```
LoginByUserPhone({
    commit
}, userInfo) {
return new Promise((resolve, reject) => {
loginByUserPhone(userInfo).then(response => {
    const data = response.data
    commit('SET_USER', data.body.userphone)
    commit('SET_USERINFO', data.body.userInfo)
    commit('SET_TIME', data.body.loginTime)
    commit('SET_TOKEN', data._TK)
    setToken(response.data._TK)
    setAdmin(data.body.userAccount)
    if (data.code == 200) {
    resolve()
    }
}).catch(error => {
    reject(error)
})
})
},
```
大概就是这么一个工作流程，更多的用法还是要在项目中自己琢磨。
以上就是前端框架的大概搭建，如有不详尽的敬请谅解。等绘画完相关页面以后，就可以进行与后台的通信了。慢慢来，一切都会很完美。
下一节我们学习如何进行后台通信，以及如何写接口，如何访问数据库。