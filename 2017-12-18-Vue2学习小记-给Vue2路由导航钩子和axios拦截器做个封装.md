### 1.写在前面
最近在学习Vue2，遇到有些页面请求数据需要用户登录权限、服务器响应不符预期的问题，但是总不能每个页面都做单独处理吧，于是想到axios提供了拦截器这个好东西，再于是就出现了本文。

### 2.具体需求
* 用户鉴权与重定向：使用Vue提供的路由导航钩子
* 请求数据序列化：使用axios提供的请求拦截器
* 接口报错信息处理：使用axios提供的响应拦截器

### 3.简单实现

#### 3.1 路由导航钩子层面鉴权与重定向的封装
>  路由导航钩子所有配置均在router/index.js，这里是部分代码

```
import Vue from 'vue'
import Router from 'vue-router'
import { getUserData } from '@/script/localUserData'

const MyAddress = r => require.ensure([], () => r(require('@/views/MyAddress/MyAddress')), 'MyAddress')

Vue.use(Router)

const routes = [
  {
    path: '/profile/address',
    name: 'MyAddress',
    component: MyAddress,
    meta: {
      title: '我的地址',
      requireAuth: true
    }
  },
  // 更多...
]

const router = new Router({
  mode: 'history',
  routes
})

```

> 我们主要来看下面逻辑处理部分的代码

```
let indexScrollTop = 0
router.beforeEach((to, from, next) => {
  // 路由进入下一个路由对象前，判断是否需要登陆
  // 在路由meta对象中由个requireAuth字段，只要此字段为true，必须做鉴权处理
  if (to.matched.some(res => res.meta.requireAuth)) {
    // userData为存储在本地的一些用户信息
    let userData = getUserData()
    // 未登录和已经登录的处理
    // getUserData方法处理后如果userData.token没有值就是undefined，下面直接判断
    if (userData.token === undefined) {
      // 执行到此处说明没有登录，君可按需处理之后再执行如下方法去登录页面
      // 我这里没有其他处理，直接去了登录页面
      next({
        path: '/login',
        query: {
          redirect: to.path
        }
      })
    } else {
      // 执行到说明本地存储有用户信息
      // 但是用户信息是否过期还是需要验证一下滴
      let overdueTime = userData.overdueTime
      let nowTime = +new Date
      // 登陆过期和未过期
      if (nowTime > overdueTime) {
        // 登录过期的处理，君可按需处理之后再执行如下方法去登录页面
        // 我这里没有其他处理，直接去了登录页面
        next({
          path: '/login',
          query: {
            redirect: to.path
          }
        })
      } else {
        next()
      }
    }
  } else {
    next()
  }
  if (to.path !== '/') {
    indexScrollTop = document.body.scrollTop
  }
  document.title = to.meta.title || document.title
})

router.afterEach(route => {
  if (route.path !== '/') {
    document.body.scrollTop = 0
  } else {
    Vue.nextTick(() => {
      document.body.scrollTop = indexScrollTop
    })
  }
})
export default router

```

> 至此，路由钩子层面的鉴权处理完毕，不过细心的你可能注意到：导航去登录页面调用的next方法里面有个query对象，携带了目标路由的地址，这是因为在登录成功后我们需要重定向到目标页面。

#### 3.2 对axios拦截器封装

>  axios所有配置均在件script/getData.js文件，这里是本文件公共代码部分

 ```
import qs from 'qs'
import { getUserData } from '@/script/localUserData'
import router from '@/router'
import axios from 'axios'
import { AJAX_URL } from '@/config/index'
axios.defaults.baseURL = AJAX_URL

```

> axios请求拦截器代码

 ```
/**
 * 请求拦截器，请求发送之前做些事情
 */
axios.interceptors.request.use(
  config => {
    // POST || PUT || DELETE请求时先格式化data数据
    // 这里需要引入第三方模块qs
    if (
      config.method.toLocaleUpperCase() === 'POST' ||
      config.method.toLocaleUpperCase() === 'PUT' ||
      config.method.toLocaleUpperCase() === 'DELETE'
    ) {
      config.data = qs.stringify(config.data)
    }
    // 配置Authorization参数携带用户token
    let userData = getUserData()
    if (userData.token) {
      config.headers.Authorization = userData.token
    }
    return config
  },
  error => {
    // 此处应为弹窗显示具体错误信息，因为是练手项目，劣者省略此处
    // 君可自行写 || 引入第三方UI框架
    console.error(error)
    return Promise.reject(error)
  }
)

```

> axios响应拦截器代码

```
/**
 * 响应拦截器，请求返回异常统一处理
 */
axios.interceptors.response.use(
  response => {
    // 这段代码很多场景下没用
    if (response.data && response.data.success === false) {
      // 根据实际情况的一些处理逻辑...
      return Promise.reject(response)
    }
    return response
  },
  error => {
    // 此处报错可能因素比较多
    // 1.需要授权处用户还未登录，因为路由段有验证是否登陆，此处理论上不会出现
    // 2.需要授权处用户登登录过期
    // 3.请求错误 4xx
    // 5.服务器错误 5xx
    // 关于鉴权失败，与后端约定状态码为500
    switch (error.response.status) {
      case 403:
        // 一些处理...
        break
      case 404:
        // 一些处理...
        break
      case 500:
        let userData = getUserData()
        if (userData.token === undefined) {
          // 此处为未登录处理
          // 一些处理之后...再去登录页面...
          // router.push({
          //   path: '/login'
          // })
        } else {
          let overdueTime = userData.overdueTime
          let nowTime = +new Date
          if (overdueTime && nowTime > overdueTime) {
            // 此处登录过期的处理
            // 一些处理之后...再去登录页面...
            // router.push({
            //   path: '/login'
            // })
          } else {
            // 极端情况，登录未过期，但是不知道哪儿错了
            // 按需处理吧...我暴力回到了首页
            router.push({
              path: '/'
            })
          }
        }
        break
      case 501:
        // 一些处理...
        break
      default:
        // 状态码辣么多，按需配置...
        break
    }
    return Promise.reject(error)
  }
)

```

> 想了解更多关于axios的信息？请移步[这里](https://github.com/axios/axios#request-config)。

这个封装很简单，面对复杂的业务肯定还需要更多的考量，但是对于一般的小项目或边缘业务也差不多够用了。最后希望这篇文章能对有需要的同学提供一些帮助。
