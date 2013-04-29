---
layout: post
title: '使用Google PageSpeed加速Nginx'
description: '使用Google PageSpeed加速Nginx'
date: 2013-04-29
comments: true
categories:
- Linux
tags:
- Google
- PageSpeed
- Nginx
- Liunx

---

Page Speed是谷歌提供的一个Web优化工具，它可以对网站的Web服务器配置和前端代码执行若干测试，并提供优化建议。

主要特性包括

* Image optimization: stripping meta-data, dynamic 
* resizing, recompression
* CSS & JavaScript minification, concatenation, inlining, and outlining
* Small resource inlining
* Deferring image and JavaScript loading
* HTML rewriting
* Cache lifetime extension 

在此工具的基础上，谷歌针对Apache、nginx服务器提供了一个傻瓜式的优化工具mod_pagespeed、 ngx_pagespeed，这些工具可以自动执行网页优化，比如对网络传输的HTML字节、图像、CSS、JavaScript进行压缩优化等。

Nginx刚刚发布了1.4.0，Google也刚刚更新了它最新的ngx_pagespeed。

Github地址：https://github.com/pagespeed/ngx_pagespeed

# 下载ngx_pagespeed
```
git clone https://github.com/pagespeed/ngx_pagespeed.git
```

# 下载Nginx
```
wget http://nginx.org/download/nginx-1.4.0.tar.gz
```

# 编译安装
```
$ tar xvzf nginx-1.4.0.tar.gz 
$ cd nginx-1.4.0
$ ./configure --prefix='/usr/local/nginx' --with-http_ssl_module --with-http_gzip_static_module --with-cc-opt='-Wno-error' --add-module='/opt/source/ngx_pagespeed' --user=www --group=www --with-google_perftools_module --with-http_realip_module --with-http_spdy_module
$ make -j12
$ sudo make install
```

# 配置
```
在http中加入
pagespeed on;
pagespeed FileCachePath /var/ngx_pagespeed_cache;

在server中加入
location ~ "\.pagespeed\.([a-z]\.)?[a-z]{2}\.[^.]{10}\.[^.]+" { add_header "" ""; }
location ~ "^/ngx_pagespeed_static/" { }
location ~ "^/ngx_pagespeed_beacon$" { }
location /ngx_pagespeed_statistics { allow 127.0.0.1; deny all; }
location /ngx_pagespeed_message { allow 127.0.0.1; deny all; }
```

新建缓冲文件夹，并配置权限

```
sudo mkdir /var/ngx_pagespeed_cache
sudo chown -R www:www /var/ngx_pagespeed_cache
```

# 测试
启动nginx后，检测模块是否安装正确

```
$ curl -I -p http://www.somesite.com/ |grep X-Page-Speed 
X-Page-Speed: 1.5.27.1-2845
```
显示上面的信息就说明安装正确了，再看看你的网站，是不是快了很多呢？

