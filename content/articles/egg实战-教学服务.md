---
title: "egg实战-教学服务"
date: 2021-07-13T13:52:57+08:00
draft: false
---

### 文件结构
```
---config
-----plugin.js  引入插件
-----config.default.js  默认配置文件，所有环境都会加载这个文件，一般作为开发环境的默认配置文件配置
-----config.prod.js  生产环境配置（prod 是env，指）
---app
-----router api和controller的映射
-----controller 解析用户输入，处理后返回相应结构，利用service和数据库交互
-----service 进行复杂业务逻辑处理
-----model  数据库表映射
-----middleware  中间件，注册在配置文件config,或者在框架和插件中使用
-----router.js  （必须）框架统一以此作为所有路由入口
---run egg框架启动时会把合并后的最终配置dump到agent_config中
-----agent_config_meta.json
-----agent_config.json   
---package.json 依赖和script命令
---app.js 统一的入口文件，进行启动过程自定义，可以利用框架提供的生命周期函数
---pkg-build.js 是执行build-win和build-linux时会执行，配置在package.json内的bin字段下
---preload.js 执行start命令时的配置文件，主要是负责读取配置环境env
```

### 插件
egg-view-nunjucks 渲染html的模版插件
1. 写入plugin
2. config.default.js 中配置
```javascript
config.view = {
  mapping: {
     '.html': 'nunjucks'
  }
}
```
egg-jwt  生成token的插件
egg管理token的插件，可以通过config里的match来匹配需要验证的token的路由
egg-sequelize  辅助我们将定义好的model对象加载到app和cxt上

### middleware 中间件写法
```javascript
module.exports = (options, app) = {
    return async function auth(ctx, next) {
        cxt.set('Access-Control-Allow-Credentials', 'true')
        const token = ctx.header['token']
        const result = awiat ctx.service.auth.vertify(token)
        if (result) {
            await next()
        } else {
            ctx.body = {
                result: '99',
                message: '您的请求不合法'
            }
        }
    }
}
module.exports = () = {
  return async function(ctx, next) {
    const startTime = Date.now()
    await next();
    const endTime = Date.now()
    reportTime(endTime - startTime)
  }
}
// 在app.js中加入
module.exports = app = {
    app.config.coreMiddleware.unshif('report')
}
```
### router
```javascript
module.export = app = {
    const { router, controller } = app;
    router.redirect('/', '/public/index.html', 302)
    router.get('/user/:id', controller.user.info)
}
```

### controller
```javascript
const controller = require('egg').Controller;
class UserController extends Controller {
    async login() {
        const ctx = this.ctx;
        ctx.validate(user_rules.loginRule);
        const {
            account,
            email,
            password,
            customerCode,
            yzm
        } = ctx.request.body;
        let body = {
            result: 0,
            message: '操作成功'
        }
        let {
            login_code
        } = ctx.session
        if (login_code) {
            if (yzm !== login_code) {
                body.result = 1;
                body.message = '验证码错误'，
                ctx.body = body;
                return
            }
        } else {
            login_code = ctx.cookies.get('login_code');
            if (login_code && yzm !== login_code) {
                body.result = 1;
                body.message = '验证码错误‘
                ctx.body = body;
                return;
            } else if (!login_code) {
                body.result = 1;
                body.message = '您的浏览器禁用了cookie'
                cxt.body = body;
                return;
            }
        }
    }
}
```
### service
```javascript
const Service = require('egg').Service

class UserService extends Service {
  async find(field) {
    const w = {
      $or: [{
        uid: fields.uid
      }, {
        email: fields.email
      }]
    }
    return await this.ctx.model.User.findOrcreat({
      where: w,
      defaults: fields
    })
  }
}
```
###  model

### schedule 定时任务
```javascript
const Subscription = require('egg').Subscription

class UpdateCache extends Subscription {
  static get schedule() {
    return {
      interval: '1m',
      type: 'all'
    }
  }

  async subscribe() {
    const res = await this.cxt.curl('http://www.api.com/cache', {
      dataType: 'json'
    })
    this.ctx.app.cache = res.data;
  }
}
module.exports = UpdateCache
```
### 注意事项
1.开发环境没有注册auth的中间件，生产环境注册了ac
2.MVC整个结构通过router.j串联起来，router.js文件位置固定app/router.js
3.应用层中间件：app.config.appMiddleware 框架默认层中间件:app.config.coreMiddleware
4.所有controller必须放在app/controller目录下，可多级目录
5.ctx.body 实际上是ctx.response.body的简写