---
title: "Single Spa"
date: 2021-07-13T11:30:47+08:00
draft: false
---

### systemjs
运行于浏览器端的模块加载器，将我们整个应用的所需要的js文件，都以imports的形式引入进来
   
使用方法一：
System.import('./test.js');

使用方法二：
1. 写一个配置文件，给每个资源定义一个key
 {
  "imports": {
    "vue": './public/vue.js',
    "single-spa": './public/single-spa.js'
  }
}
2. 引入配置文件
```<script type="systemjs-importmap" src=""></script>```
3. 引入文件
System.import(key)

## single-spa实践
### 结构目录设计
```
---common
---projects
---root_html_file
-----index.html 引入index.js,  systemjs，single-spa, 
-----index.js
-----registry 
--------public_dependents.json
--------singleSpa_project.json
```
### index.js
```javascript
(async function() {
  var System_imports = [
    System.import('./single-spa.js'),
    System.import('./frame.js')
  ];
  Promise.all(System_imports).then((modules) => {
    var SingleSpa = modules[0];
    var Frame = modules[1];

    SingleSpa.registerApplication(
      'frame',
      Frame,
      () => true
    )
    SingleSpa.start();

    window.addEventListener('INIT_FRAME', function() {
      const arr = Object.keys(SINGLE_SPA_PROJECTS);
      arr.forEach((project) => {
        if (project !== 'frame') {
          SingleSpa.registerApplication(
            project,
            () => System.import(SINGLE_SPA_PROJECTS[project]),
            location => location.pathname.indexOf('/' + project) === 0
          )
        }
      })
    })
  })
})()

```
### projects/app1代码分析
1. vue.config.js
```javascript
module.exports = {
  chainWebpack: config => {
    config.devServer.set('inline', false)
    config.devServer.set('hot', true)
    config.output.filename(`js/[name].js`)
    config.externals(['vue', 'vue-router', 'vuex', 'element-ui'])
  },
  filenameHashing: false,
  productionSourceMap: false
}
```
2. set-public-path.js
```
import { setPublicPath } from ''systemjs-webpack-interop'
setPublicPath('app1', 2)
```
是为了解决webpack构建时，资源设置publicPath，加载时会认为是当前页面的路径，实际上对于app1而言，路径应该是相对于app1的路径，通过这个设置，可以设置为正确的路径
在main.js开头引入
3、main.js
```javascript
import './set-public-path'
import Vue from 'vue'
import App from './App.vue'
import I18n from './i18n'
import SingleSpaVue from 'single-spa-vue'

const vueLifecycles = new SingleSpaVue({
  Vue,
  appOptions: {
    el: '#single-spa-projects',
    i18n: I18n,
    store: Store,
    router: Router,
    render: h => h(App),
  }
})
// singleSpa生命周期有两种写法
// 应用初始
export const bootstrap = vueLifecycles.bootstrap
//应用挂载
export function mount(props) {
  return vueLifecycles.mount
}
//应用卸载
export const unmount = VueLifecycles.unmount
```
