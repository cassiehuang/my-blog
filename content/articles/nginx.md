---
title: "安装和配置nginx服务器"
date: 2021-07-02T10:06:56+08:00
draft: false
---

#### 配置vue history模式支持
server {
    try_files $uri $uri/ /erp-farm-admin/index.html;
}
#### 配置反向代理
loaction ^~ /api/ {
  proxy_pass http://127.0.0.1:8081/
}

```
worker_processes 1;
events {worker_connections 1024;}
http {include mime.types;default_type application/octet-stream;
sendfile on;
keepalive_timeout 65;
server {
listen 80;server_name localhost;
#gzip服务开启，对大于100k的文件进行gzip压缩，提高前端资源的响应速度
gzip on;
gzip_static on;
gzip_min_length 100;
gzip_comp_level 6;
gzip_types application/javascript text/css text/xml;
gzip_disable "MSIE [1-6]\.";
gzip_vary on;
gzip_buffers 32 4K;
#上传文件大小限制
client_max_body_size 100M;
#charset koi8-r;
#access_log logs/host.access.log main;
location / {
#静态资源的根路径
root /usr/share/nginx/html/cbkc-web/;index index.html index.htm;
#支持vue history路由模式
try_files $uri $uri/ /index.html;
#html文件不缓存，可以解决每次上线后，由于浏览器缓存造成的错误
if ($request_filename ~* .*\.(?:htm|html)$) {
add_header Cache-Control "private, no-store, no-cache, must-revalidate, proxy-revalidate";
}
#js和css文件设置过期时间
if ($request_filename ~* .*\.(?:js|css)$) {expires 7d;}
#图片、视频等文件设置过期时间
if ($request_filename ~* .*\.(?:jpg|ipeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$) {expires 7d;}}
location /manager/ {alias /usr/share/nginx/html/cbkc-web-admin/;index index.html index.htm;}
#反向代理，解决跨域问题
location ^~ /api/ {
#后台开启了请求来源检测
proxy_set_header host $host;
proxy_pass http://112.7.35.26:8082/;
}
```
