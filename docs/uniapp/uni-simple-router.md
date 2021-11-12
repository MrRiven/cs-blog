# uni-simple-router

- H5 端 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 uni-app 过渡系统的视图过渡效果
- 细粒度的导航控制
- HTML5 历史模式或 hash 模式
- H5 端自定义的滚动条行为
- 更丰富的传参方式，优雅的加载进度
- app 端独有启动页生命周期/过渡页
- nvue/vue 全套支持，APP 端 tabbar 拦截有效

## 安装

> npm

```bash
npm i uni-simple-router -S
```

> 自动读取 page.json 作为路由表

```bash
npm install uni-read-pages -S
```

> vue.config.js

```js
const TransformPages = require('uni-read-pages')
const { webpack } = new TransformPages()
module.exports = {
  configureWebpack: {
    plugins: [
      new webpack.DefinePlugin({
        ROUTES: webpack.DefinePlugin.runtimeValue(() => {
          const tfPages = new TransformPages({
            includes: ['path', 'name', 'aliasPath'],
          })
          return JSON.stringify(tfPages.routes)
        }, true),
      }),
    ],
  },
}
```

> router.js

```js
import { RouterMount, createRouter } from 'uni-simple-router'

const router = createRouter({
  platform: process.env.VUE_APP_PLATFORM,
  keepUniOriginNav: false,
  debugger: false,
  routerBeforeEach: (to, from, next) => {
    next()
  },
  routerAfterEach: (to, from) => {},
  routerErrorEach: (error, router) => {
    console.error(error)
    err(error, router, true)
  },
  detectBeforeLock: (router, to, navType) => {},
  resolveQuery: (jsonQuery) => jsonQuery,
  parseQuery: (jsonQuery) => jsonQuery,
  h5: {
    paramsToQuery: false,
    vueRouterDev: false,
    vueNext: false,
    mode: 'hash',
    base: '/',
    linkActiveClass: 'router-link-active',
    linkExactActiveClass: 'router-link-exact-active',
    scrollBehavior: (to, from, savedPostion) => ({ x: 0, y: 0 }),
    fallback: true,
  },
  APP: {
    registerLoadingPage: true, //v2.0.6+
    loadingPageStyle: () => JSON.parse('{"backgroundColor":"#FFF"}'),
    loadingPageHook: (view) => {
      view.show()
    },
    launchedHook: () => {
      plus.navigator.closeSplashscreen()
    },
    animation: {},
  },
  applet: {
    //v2.0.6+
    animationDuration: 300,
  },
  routes: [
    ...ROUTES,
    {
      path: '*',
      redirect: (to) => {
        return { name: '404' }
      },
    },
  ],
})

export { router, RouterMount }
```

## 使用

> main.js

```js
import Vue from 'vue'
import App from './App'
import { router, RouterMount } from './router.js' //路径换成自己的
Vue.use(router)

App.mpType = 'app'
const app = new Vue({
  ...App,
})

//v1.3.5起 H5端 你应该去除原有的app.$mount();使用路由自带的渲染方式
// #ifdef H5
RouterMount(app, router, '#app')
// #endif

// #ifndef H5
app.$mount() //为了兼容小程序及app端必须这样写才有效果
// #endif
```

> index.vue

```
// router.push() 等同于 uni.navigateTo()
// 字符串
this.$Router.push('/pages/router/router1')

// 对象
this.$Router.push({path:'/pages/router/router1'})

// 命名的路由
this.$Router.push({ name: 'router1', params: { userId: '123' }})

// 带查询参数，变成 /router1?plan=private
this.$Router.push({ path: 'router1', query: { plan: 'private' }})

// router.replace() 等同于 uni.redirectTo()
this.$Router.replace(...)

// router.replaceAll() 等同于 uni.reLaunch()
this.$Router.replaceAll(...)

// router.pushTab() 等同于 uni.switchTab()
this.$Router.pushTab(...)

// router.back(n,{...}) 等同于 uni.navigateBack()
// 后退 2 步记录
this.$Router.back(2,{
    success:(...arg)=>{
        console.log(arg)
    }
})

// 如果 history 记录不够用，那就默默地失败呗
this.$Router.back(100,{
    success:(...arg)=>{
        console.log(arg)
    }
})
```

> [更多用法](https://hhyang.cn/v2/)
