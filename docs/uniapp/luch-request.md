# luch-request

- 基于 Promise 对象实现更简单的 request 使用方式，支持请求和响应拦截
- 支持全局挂载
- 支持多个全局配置实例
- 支持自定义验证器
- 支持文件上传/下载
- 支持 task 操作
- 支持自定义参数
- 支持多拦截器
- 对参数的处理比 uni.request 更强

## 安装

```bash
npm i luch-request -S
```

> vue.config.js

```js
// vue.config.js
module.exports = {
  transpileDependencies: ['luch-request'],
}
```

## 使用

> 封装 request.js

```js
import Request from 'luch-request'

const METHOD = {
  GET: 'GET',
  POST: 'POST',
  PUT: 'PUT',
  DELETE: 'DELETE',
  CONNECT: 'CONNECT',
  HEAD: 'HEAD',
  OPTIONS: 'OPTIONS',
  TRACE: 'TRACE',
}

const http = new Request({
  // 请求地址 根据环境自动注入
  baseURL: process.uniEnv.BASE_API,
  // 默认请求方式
  method: METHOD.POST,
  header: {
    PLATFORM: process.env.VUE_APP_PLATFORM,
  },
  dataType: 'json',
  // #ifndef MP-ALIPAY
  responseType: 'text',
  // #endif
  // 注：如果局部custom与全局custom有同名属性，则后面的属性会覆盖前面的属性，相当于Object.assign(全局，局部)
  // custom: {}, // 全局自定义参数默认值
  // #ifdef H5 || APP-PLUS || MP-ALIPAY || MP-WEIXIN
  timeout: 60000,
  // #endif
  // #ifdef APP-PLUS
  // sslVerify: true,
  // #endif
  // #ifdef H5
  // 跨域请求时是否携带凭证（cookies）仅H5支持（HBuilderX 2.6.15+）
  withCredentials: false,
  // #endif
  // #ifdef APP-PLUS
  firstIpv4: false, // DNS解析时优先使用ipv4 仅 App-Android 支持 (HBuilderX 2.8.0+)
  // #endif
  // 局部优先级高于全局，返回当前请求的task,options。请勿在此处修改options。非必填
  // getTask: (task, options) => {
  // 相当于设置了请求超时时间500ms
  //   setTimeout(() => {
  //     task.abort()
  //   }, 500)
  // }
})

http.interceptors.request.use(
  (config) => {
    // 可使用async await 做异步操作
    config.header = {
      ...config.header,
    }

    return config
  },
  (config) => {
    // 可使用async await 做异步操作
    return Promise.reject(config)
  }
)

http.interceptors.response.use(
  (response) => {
    /* 对响应成功做点什么 可使用async await 做异步操作*/
    uni.hideLoading()
    const code = response.data.code
    if (code !== 200) {
      // 服务端返回的状态码不等于200，则reject()
      if (code === 401) {
        // 登录失效操作
        return Promise.reject(response)
      }
      uni.showToast({
        icon: 'none',
        title: response.data?.msg,
      })
      return Promise.reject(response) // return Promise.reject 可使promise状态进入catch
    }
    return response.data.data
  },
  (response) => {
    /*  对响应错误做点什么 （statusCode !== 200）*/
    console.error(response)
    uni.hideLoading()
    uni.showToast({
      icon: 'none',
      title: JSON.stringify(response.data) || '服务器开小差了..',
    })
    return Promise.reject(response)
  }
)

export { http, METHOD }
```

> api.js

```js
import { http } from './request.js'

export function getTest(data = {}) {
  return http.request({
    url: `/xxx/xxx/test`,
    data,
  })
}
```

> index.vue

```vue
<script>
import { getTest } from './api'

export default {
  methods: {
    async query() {
      try {
        const result = await getTest({ a: 1, b: 2 })
        console.log(result)
      } catch (error) {}
    },
  },
}
</script>
```
