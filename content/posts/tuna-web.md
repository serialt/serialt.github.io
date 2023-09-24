+++
title = 'Tuna Web'
date = 2023-09-24T17:01:26+08:00
draft = false
tags = ["mirrors"]
+++

## 清华镜像网页搭建镜像站

tuna mirror-web地址：https://github.com/tuna/mirror-web.git

tuna mirror-web基于jekyll开发，由于ruby环境安装复杂，因此采用docker编译，但在build镜像的时候，出现安装包依赖安装失败，在多次测试后，无法build成镜像。前段时间，在查看mirror-web段issues时，有人询问mirror-web段README.md文档的下一步，官方给了一点提示，有关于基于nginx的第三方模块来实现目录第渲染的提示说明，再次尝试后，经历各种困难和折磨，终于摸索出。


### 1、下载mirror-web的jekyll编译环境

tuna的编译镜像：tunathu/mirror-web

在下载tunathu/mirror-web时发生了一个小问题，由于在国内使用的docker镜像加速，下载的镜像是旧版本的，但境外的服务器下载的镜像是最新的，在踩坑后果断推到自己的dockerhub上，新的下载地址：serialt/tuna-mirror-web。



### 2、下载github仓库

```
git clone https://github.com/tuna/mirror-web.git /opt/mirror-web
```



### 3、下载额外资源和编译

```
cd /opt/mirror-web
wget https://mirrors.tuna.tsinghua.edu.cn/static/tunasync.json -O static/tunasync.json
wget https://mirrors.tuna.tsinghua.edu.cn/static/tunet.json -O static/tunet.json
mkdir -p static/status
wget https://mirrors.tuna.tsinghua.edu.cn/static/status/isoinfo.json -O static/status/isoinfo.json

docker run -it -v  /opt/mirror-web/:/data serialt/tuna-mirror-web:20211006
```

编译的后静态文件在_site里



### 4、编译nginx

需要安装第三方模块

* modules/ngx_http_js_module.so
* modules/ngx_http_fancyindex_module.so

```
[root@serialt nginx]# ll
总用量 1068
drwxr-xr-x  9 sonar sonar     186 10月  5 22:27 nginx-1.20.1
-rw-r--r--  1 root  root  1061461 5月  25 23:34 nginx-1.20.1.tar.gz
drwxrwxr-x  3 root  root      217 10月 27 2020 ngx-fancyindex-0.5.1
-rw-r--r--  1 root  root    25148 10月 27 2020 ngx-fancyindex-0.5.1.tar.xz
drwxr-xr-x 10 root  root      228 10月  6 15:49 njs
```

```
# njs下载
git clone https://github.com/nginx/njs

# ngx-fancyindex 下载
wget https://github.com/aperezdc/ngx-fancyindex/releases/download/v0.5.1/ngx-fancyindex-0.5.1.tar.xz
```

编译：

```shell
[root@serialt nginx]# ls
nginx-1.20.1  nginx-1.20.1.tar.gz  ngx-fancyindex-0.5.1  ngx-fancyindex-0.5.1.tar.xz  njs
[root@serialt nginx]# cd nginx-1.20.1/

[root@serialt nginx-1.20.1]# ./configure --prefix=/usr/local/nginx --with-pcre --with-http_auth_request_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module  --with-mail --with-mail_ssl_module  --with-stream --with-stream_ssl_module --with-stream_realip_module --add-dynamic-module=/root/nginx/ngx-fancyindex-0.5.1 --add-dynamic-module=/root/nginx/njs/nginx

# add m
 3997  [2022-02-25 00:49:21] [root] [10.5.0.10] ./configure --with-compat --add-dynamic-module=/root/github/tuna-mirror-web/njs-0.6.2/nginx
 3998  [2022-02-25 00:49:31] [root] [10.5.0.10] make modules
```

nginx配置文件内容

```shell
[root@serialt mirrors]# cat /usr/local/nginx/conf/nginx.conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

load_module modules/ngx_http_js_module.so;
load_module modules/ngx_http_fancyindex_module.so;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

map $http_user_agent $isbrowser {
 default 0;
 "~*validation server" 0;
 "~*mozilla" 1;
}

    
        js_path /opt/mirror-web/_site/static/njs;
        js_include /opt/mirror-web/_site/static/njs/all.njs;
    #js_path /opt/mirror-web/static/njs;
    #js_include /opt/mirror-web/static/njs/all.njs;
    server {
        listen       8007;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        #root         /opt/mirror-web/_site;


        fancyindex_header /fancy-index/before;
        fancyindex_footer /fancy-index/after;
        fancyindex_exact_size off;
        fancyindex_time_format "%d %b %Y %H:%M:%S +0000";
        fancyindex_name_length 256;

        error_page 404 /404.html;

        location /fancy-index {
         internal;
         root /opt/mirror-web/_site;
         subrequest_output_buffer_size 100k;
         location = /fancy-index/before {
           js_content fancyIndexBeforeRender;
         }
         location = /fancy-index/after {
           js_content fancyIndexAfterRender;
         }
        }


        location / {
         root /opt/mirror-web/_site;
         index index.html index.htm;
         #try_files /_site/$uri $uri/ /_site/$uri;

         fancyindex on;
        }

#        location / {
#            root   html;
#            index  index.html index.htm;
#        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }



}
[root@serialt mirrors]#
```

### 5、镜像资源暴露



方法1：以软链接形式存放在`/opt/mirror-web/_site`中

```
[root@serialt _site]# pwd
/opt/mirror-web/_site
[root@serialt _site]# ll
总用量 160
-rw-r--r--   1 root root 16415 10月  6 15:37 404.html
drwxr-xr-x   2 root root     6 10月  6 17:14 cc
drwxr-xr-x   2 root root    19 10月  6 17:14 centos
drwxr-xr-x   2 root root    94 10月  6 09:25 fancy-index
-rw-r--r--   1 root root 36650 10月  6 15:37 feed.xml
drwxr-xr-x 103 root root  4096 10月  6 09:25 help
-rw-r--r--   1 root root 27679 10月  6 15:37 index.html
-rw-r--r--   1 root root 20728 10月  6 15:37 legacy_index.html
-rw-r--r--   1 root root 18092 10月  6 15:37 LICENSE
drwxr-xr-x  47 root root  4096 10月  6 09:25 news
-rw-r--r--   1 root root    58 10月  6 15:37 robots.txt
-rw-r--r--   1 root root 19134 10月  6 15:37 sitemap.xml
drwxr-xr-x   8 root root   115 10月  6 15:37 static
drwxr-xr-x   2 root root    24 10月  6 09:25 status
```

方法二：把`/opt/mirror-web/_site`里的文件以软链接的方式链接到镜像的跟目录（建议使用）

```
ln -snf /opt/mirror-web/_site/* /opt/imau
```



目录描述文件：_data/options.yml

站点资源显示控制：static/tunasync.json

tunasync.json

```
[
    {
        "name": "ant",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "book",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "centos",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "dev",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "frp",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "git",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "go",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "grafana",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "iso",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "jdk",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "jmeter",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "kubernetes",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "mac",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "monitor",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "node",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "root-ca",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "other",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "printer",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "prometheus",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "pycharm",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "python",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "repo",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "script",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "test",
        "is_master": true,
        "status": "success"
    },
    {
        "name": "tool",
        "is_master": true,
        "status": "success"
    }
]
```

```
        "last_update": "2022-01-11 16:39:37 +0800",
        "last_update_ts": 1641890377,
        "last_started": "2022-01-11 16:39:21 +0800",
        "last_started_ts": 1641890361,
        "last_ended": "2022-01-11 16:39:37 +0800",
        "last_ended_ts": 1641890377,
        "next_schedule": "2022-01-11 22:39:37 +0800",
        "next_schedule_ts": 1641911977,
        "upstream": "rsync://msync.centos.org/CentOS/",


  - status: 'success'
    last_update: '-'
    name: "AUR"
    url: 'https://aur.tuna.tsinghua.edu.cn/'
    upstream: 'https://aur.archlinux.org/'
    is_master: true
```



options.yml

```yaml
# Content Related
mirror_desc:
  - name: git 
    desc: git 二进制编译版本
  - name: go
    desc: golang 开发环境
  - name: grafana
    desc: grafana 的安装包
  - name: helm
    desc: helm 的二进制发行包
  - name: ios
    desc: 镜像文件，如centos, ubuntu, rocky等 
  - name: jdk
    desc: java 开发环境安装包

 

 
new_mirrors:
  - hugging-face-models
  - endeavouros
  - ubuntukylin
  - putty
  - postmarketOS
  - postmarketOS-images
  - obs-studio
  - stellarium

unlisted_mirrors:
  - status: 'success'
    last_update: '-'
    name: "AUR"
    url: 'https://aur.tuna.tsinghua.edu.cn/'
    upstream: 'https://aur.archlinux.org/'
    is_master: true
  - link_to: 'osdn'
    name: "manjaro-cd"
    url: '/osdn/storage/g/m/ma/manjaro/'
  - link_to: 'osdn'
    name: "manjaro-arm-cd"
    url: '/osdn/storage/g/m/ma/manjaro-arm/'
  - link_to: 'osdn'
    name: "mxlinux-isos"
    url: '/osdn/storage/g/m/mx/mx-linux/ISOs/'
  - link_to: 'osdn'
    name: "garuda-linux"
    url: '/osdn/storage/g/g/ga/garuda-linux/'
  - link_to: 'osdn'
    name: "linuxlite-cd"
    url: '/osdn/storage/g/l/li/linuxlite/'
  - link_to: 'github-release'
    name: "prometheus"
    url: '/github-release/prometheus/prometheus/'
  - link_to: 'github-release'
    name: "iina"
    url: '/github-release/iina/iina/'
  - link_to: 'github-release'
    name: "VSCodium"
    url: '/github-release/VSCodium/vscodium/'
  - link_to: 'github-release'
    name: "FreeCAD"
    url: '/github-release/FreeCAD/FreeCAD/'
  - link_to: 'github-release'
    name: "git-for-windows"
    url: '/github-release/git-for-windows/git/'
  - link_to: 'github-release'
    name: "llvm-binary"
    url: '/github-release/llvm/llvm-project/'
  - link_to: 'github-release'
    name: "miniforge"
    url: '/github-release/conda-forge/miniforge/'
  - link_to: 'github-release'
    name: "stellarium"
    url: '/github-release/Stellarium/stellarium/'
  - link_to: 'github-release'
    name: "cmder"
    url: '/github-release/cmderdev/cmder/'
  - link_to: 'github-release'
    name: "googlefonts"
    url: '/github-release/googlefonts/'
  - link_to: 'github-release'
    name: "minikube"
    url: '/github-release/kubernetes/minikube/'
  - link_to: 'github-release'
    name: "obs-studio"
    url: '/github-release/obsproject/obs-studio/'
  - link_to: 'github-release'
    name: "orchestrator"
    url: '/github-release/openark/orchestrator/'
  - link_to: 'github-release'
    name: "texstudio"
    url: '/github-release/texstudio-org/texstudio/'
  - link_to: 'github-release'
    name: "rust-analyzer"
    url: '/github-release/rust-analyzer/rust-analyzer/'
  - link_to: 'github-release'
    name: "thuthesis"
    url: '/github-release/tuna/thuthesis'


force_redirect_help_mirrors:
  - AOSP
  - lineageOS
  - homebrew
  - linux.git
  - linux-next.git
  - linux-stable.git
  - git-repo
  - gentoo-portage.git
  - chromiumos
  - weave
  - CocoaPods
  - llvm
  - llvm-project.git
  - openthos-src
  - qemu.git
  - linux-firmware.git
  - gcc.git
  - crates.io-index.git
  - binutils-gdb.git
  - glibc.git
  - flutter-sdk.git
  - julia-general.git

force_show_help_mirrors:
  - hugging-face-models

label_map:
  unknown: label-default
  syncing: label-info
  success: label-success
  fail: label-warning
  failed: label-warning
  paused: label-warning

```



### 6、docker镜像使用

#### 静态文件编译

构建 Jekyll 的 docker 镜像环境复杂，建议直接使用官方或者已经存在的镜像`tunathu/mirror-web`或者`serialt/tuna-mirror-web`

```shell
[root@tc ~]# docker run -it -v /path/to/mirror-web/:/data  serialt/tuna-mirror-web
```

**一些动态数据已经下载，若需要最新的，可以就行以下操作,然后在构建**

下载最新的动态数据文件

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/static/tunasync.json -O static/tunasync.json
wget https://mirrors.tuna.tsinghua.edu.cn/static/tunet.json -O static/tunet.json
mkdir -p static/status
wget https://mirrors.tuna.tsinghua.edu.cn/static/status/isoinfo.json -O static/status/isoinfo.json
```



#### 运行服务

* docker镜像网页根目录: /opt/mirror-web

* 镜像站资源根目录: /opt/mirror

#### 启动服务

```shell
docker run -tid  -v /opt/tuna-mirror-web/_site:/opt/mirror-web  -v /opt/mirror:/opt/mirror -p 8099:80 --name=tuna-mirror-nginx serialt/tuna-mirror-web-nginx:7b0c89d

```



```yaml
version: "3"
networks:
  tuna-mirror-nginx:
    external: false

services:
  tuna-mirror-nginx:
    image: serialt/tuna-mirror-web-nginx:latest
    container_name: mirror-nginx
    hostname: mirror-nginx
    restart: always
    networks:
      - tuna-mirror-nginx
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/opt/mirror-web/_site:/opt/mirror-web"
      - "/opt/mirror:/opt/mirror"
    ports:
      - "80:80"
    dns:
      - 223.5.5.5
      - 223.6.6.6
```

### 新镜像站版本发布

```shell
docker run -it --rm  -v /opt/tuna-mirror-web:/data serialt/tuna-mirror-web

docker restart tuna-mirror-nginx
```

### 编写说明

```text
_data/options.yml: 是显示在镜像站主页对各个目录的说明
static/tunasync.json: 是对当前repo的同步信息的描述，可以自行编辑，也可以从tuna上下载
help: help目录里存有各个repo的帮助信息，在主页上会显示有个"?"
news: 镜像站的新闻信息
```

### help 说明

permalink 是表示help的首页的路径，必须要有

```text
---
layout: help
category: help
mirrorid: app
permalink: /help/app/
---
```





