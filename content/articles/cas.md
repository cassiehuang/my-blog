---
title: "Cas单点登录"
date: 2021-07-13T13:52:57+08:00
draft: false
---

https://www.cnblogs.com/Eleven-Liu/p/10336181.html
### CAS
1. CAS是企业级的单点登陆解决方案
2. 大概流程是
（1）浏览器向www发起请求，无ST
（2）www返回一个重定向到cas的请求，带上返回地址
（3）浏览器向cas发起请求
（4）cas检查TGC，第一次访问，不带TGC，则返回登陆页面
（5）浏览器填写好登陆信息后发送给cas
（6）cas验证登陆信息，在cookie中写入TGC，下次浏览器访问cas会带上这个TGC，写入重定向，并将ST写入www地址链接
（7）浏览器重定向到www网站，携带ST
（8）www根据ST去cas验证登陆是否有效
（9）cas根据ST验证，通过后，告诉www该ST有效，www在session中记录登录状态
（10）www返回浏览器资源
（11）浏览器第二次访问www
（12）www从session中知道已经登陆，直接返回资源
（13）浏览器访问mail
（14）mail返回一个重定向请求，重定向到cas
（15）浏览器访问cas，会带上TGC
（16）cas验证TGC，返回ST（token），并让浏览器重定向到mail
（17）重定向到mail
（18）mail根据ST去cas验证是否有效
（19）cas验证有效后，返回mail，mail通过后在session中设置登录状态
（20）返回浏览器资源