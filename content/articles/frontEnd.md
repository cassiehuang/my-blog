---
title: "前端开发规范梳理"
date: 2021-07-07T09:58:04+08:00
draft: false
---

文件目录规范
1、/src/pages文件夹内只能存放真正的一级导航页面，如果是页面分为多个组件模块，在pages内建同名文件夹下存放组件
2、常量采用全大写的方式命名，且存放在constant/index.js 文件内统一进行管理
3、接口文件统一在api里进行管理
4、图片、文件名不能用中文命名
5、较大的图片，如背景图等放在static文件夹下（图片会单独打包，利于缓存命中）
6、较小的图片，如各种图标，放在asset文件夹下（图片会以base64的方式打包进文件，减少请求次数）
7、一个vue文件代码行数尽量不要超过500行，超过的500行的考虑拆分为组件
css规范
1、html内类名以-连接，如“container-wrap”，补充一个id和class命名规则（BEM（Block， Element， Modifier）命名规范）：
•	Block：独立有意义的实体，eg：header、container、menu、checkbox、footer等在页面布局时，划分单独模块
•	Element：元素，是Block下的子元素，其没有独立的意义，属于Block的一部分，eg：menu-item，list-item，header-title等
•	Modifier：Block或者Element上的标志标识，用来改变外观、状态、行为、标记等，eg：disabled、checked、color yellow，size big，highlighted等
2、类选择器比元素选择器更好，尽量减少元素选择器的使用
3、为组件样式添加scope作用域，如果不能使用scope作用域的样式，需要添加具有组件内唯一性的类名，避免对其他组件造成影响
4、公用样式放在common.less文件内
js规范
1、vue组件命名以PascalCase的方式命名（ufp框架内用cameCase方式命名）
2、组件不能和html5的标签重复，如“Footer”、“Header”不能存在
3、使用组件时，组件全小写以-的方式连接，如“<date-pick></date-pick>”
4、文件的引入不使用“import { parseTime } from '../../utils/index'”这种方式，采用“import { parseTime } from '@/utils/index'”,@路径指向src路径，采用前者经常会导致路径错误，且不利于文件移动
5、js文件内变量采用cameCase命名法，如“resultList”
6、使用v-for指令要设置key，尽量不用index作为key值
7、避免v-if和v-for同时作用于同一个元素上
举例：<li v-for="(value, key) in arr" v-if="value.show"></li> 错误
          <li v-for="(value, key) in filterArr"></li> 正确
8、优先使用vuex管理全局状态，而不是通过this.$root或者全局事件总线
9、指令缩写：（用：表示v-bind，用@表示v-on）
10、组件模板（template）内应该只包含简单的表达式，复杂的表达式则应该重构为计算属性或者方法
11、不同逻辑、不同业务、不同语义之间打代码插入一个空行分隔来提升可读性
12、不使用匿名函数
13、不使用var，使用let、const
14、数组循环时需要分清forEach、map、filter的不同使用场景
15、函数的功能要尽可能单一
其他
1、因为目前使用的是git进行版本管理，任何时候的代码都能跟踪找回，所以对于无用代码需要及时删除，比如一些调试的console语句、无用的弃用功能代码
工具
1、使用prettier、eslint格式化代码，处理换行、缩进、分号、单引号、双引号、空格等问题，保存文件时自动保持一致，规范采用eslint:recommend，具体项可参考https://eslint.bootcss.com/docs/rules (工具使用：基于vscode建立vue项目前端规范)
