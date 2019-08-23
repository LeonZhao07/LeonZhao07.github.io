---
title: nginx 反向代理
author: leon
layout: post
---
# Nginx 解决跨域问题
最近有个项目，本地前端项目直接调用Dev环境的API是会存在跨域问题，那么只要简单配置一个Nginx转发就可以解决这个问题。  
## 下载安装
首先去[下载地址](https://nginx.org/en/download.html)下载安装最新版Nginx。  
安装完成后，安装目录下便有了nginx.exe文件，直接双击打开就可以使用默认配置启动nginx。  
## 配置
但是想要Nginx达到我们的要求，还需要对配置文件进行一定的更改。默认配置文件为 $AppPath$\nginx-1.16.1\conf\nginx.conf 。  
打开配置文件，首先要修改 Server节点：
```
server {
        listen       233;
        server_name  localhost;

        #charset koi8-r;
        
        #access_log  logs/host.access.log  main;
        location / {
            add_header Access-Control-Allow-Origin *;
            if ($request_method = OPTIONS ) {
                add_header Access-Control-Allow-Origin *;
                add_header Access-Control-Allow-Methods "GET,HEAD,PUT,PATCH,POST,DELETE";
                add_header Access-Control-Allow-Headers "*";
                return 204;
            }
            proxy_pass http://testurl.com/api/;
        }   

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```
当request_method为options时，直接在header中加上一些信息返回204。浏览器再次请求Get或Post时，则正常转发。

## 问题
当我修改nginx.conf后，再次双击nginx.exe，可浏览器发送OPTIONS请求时，nginx并未返回我设置的header，导致浏览器一直报405 error。我以为是我配置的问题，一直在修改。后来发现我的修改似乎并未生效。  
这是我发现任务浏览器中存在10多个nginx.exe进程...原来我之前的操作，nginx并未重启，而是每次都是启动一个全新的进程。导致转发有问题。  
在任务浏览器中手动结束任务，发现有些nginx.exe进程始终结束不了，结束了旧的，又出现个新的。使用 *nginx.exe -s stop* 停止nginx，报错*nginx: [error] OpenEvent("Global\ngx_quit_16180") failed (2: The system cannot find the file specified)* 。应该是我之前手动结束了几个nginx进程的原因。  
只好*tasklist /fi "imagename eq nginx.exe"* kill掉所有nginx进程。再次启动nginx，对OPTIONS的Response已经正常。  
可是转发正常的POST/GET请求，缺仍有问题 *An existing connection was forcibly closed by the remote host) while reading response header from upstream*，是因为默认的keepalive_timeout太短了，将keepalive_timeout的值调大以后，一切转发正常

## 常用命令
- ngnix -v
- nginx.exe -s stop
- start nginx
- nginx.exe -s reload




