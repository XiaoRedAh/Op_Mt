Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。

Nginx 启动特别容易，几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。并且可以在不间断服务的情况下进行软件版本的升级。

Nginx代码完全用C语言写成。官方数据测试表明能够支持高达 50,000 个并发连接数的响应。

利用Nginx，可以很方便地实现反向代理，动静分离，负载均衡

[入门文档](https://www.yuque.com/wukong-zorrm/cql6cz)

# 安装Nginx

## 直接安装

>不同Linux发行版的安装方式略有不同

**Centos**

```sh
# 1. 安装前确保系统已经更新
sudo yum update
# 2. 安装EPEL仓库
sudo yum install epel-release
# 3. 安装nginx
sudo yum install nginx
# 4. 验证安装
sudo nginx -V
```

**Debian/Ubuntu**

```sh
# 1. 更新仓库信息
sudo apt update
# 2. 安装nginx
sudo apt install nginx
# 3. 验证安装
sudo nginx -V
```

## 从源码编译安装

>从源码编译安装的⽅式可以⾃定义Nginx的安装⽬录、模块等，但是安装过程⽐较繁琐，需要安装⼀些依赖库

[详细教程以及可能遇到的问题](https://blog.csdn.net/qq1195566313/article/details/123965492?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168862580416782427472708%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=168862580416782427472708&biz_id=0&spm=1018.2226.3001.4450)

① 安装gcc
编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装

```sh
yum install gcc-c++
```

② 安装PCRE pcre-devel
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库

```sh
yum install -y pcre pcre-devel
```

③ 安装zlib
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip 

```sh
yum install -y zlib zlib-devel
```

④ 安装OpenSSL 
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```sh
yum install -y openssl openssl-devel
```

⑤ 下载并解压Nginx源码
```sh
wget https://nginx.org/download/nginx-1.19.9.tar.gz
tar -zxvf nginx-1.19.9.tar.gz
``` 

⑦ 执行nginx-configure文件

```sh
cd nginx-1.19.9
./configure
```
⑧ make命令编译

执行完后会有一个MakeFile文件夹
make 是一个命令工具，它解释 Makefile 中的指令。在 Makefile文件中描述了整个工程所有文件的编译顺序、编译规则
```sh
make
make install
```
⑨ 查询nginx安装目录，进入安装目录执行nginx

```sh
whereis nginx
//前往安装目录找到sbin 执行nginx
./nginx
```

# Nginx常用命令

添加环境变量，使得可以全局使用nginx命令

```sh
//找到nginx的安装目录
whereis nginx 
//前往根目录，找到etc文件夹
//打开配置文件
vim profile

//在配置文件中添加如下内容：
export PATH=$PATH:/node-v14.19.1-linux-x64/bin:/usr/local/nginx/sbin
```

查看nginx版本号
```sh
nginx -v
//详细一点
nginx -V
```

启动/停止nginx
```sh
//启动nginx
nginx
//立即停止nginx
nginx -s stop
//优雅地关闭，Nginx在退出前完成已经接受的请求处理
nginx -s quit
```

检查配置⽂件是否正确，也可⽤来定位配置⽂件的位置
```sh
nginx -t
```

每次修改完配置文件后，需要重载nginx配置文件
```sh
nginx -s reload
```

查看nginx进程
```sh
ps -ef | grep nginx
```

# Nginx配置文件

Nginx的配置⽂件是 nginx.conf ，⼀般位于 /etc/nginx/nginx.conf 。
```sh
//检查配置⽂件是否正确，也可⽤来定位配置⽂件的位置
nginx -t
//打开配置文件
vim nginx.conf
```

nginx.conf的结构如下：
* 全局块
* events块
* http块
  * http全局块
  * 多个server块：每个server块中，可以包含server全局块和多个location块
```nginx
# 全局块
worker_processes 1;
events {
    # events块
}
http {
    # http块
    server {
        # server块
        location / {
            # location块
        }
    }
}
```

## 全局块

全局块是默认配置文件从开始到events块之间的一部分内容，主要设置一些**影响Nginx服务器整体运行的配置指令**。【主要包括配置运行Nginx服务器的用户（组）、允许生成的worker_process数、进程PID存放路径、日志存放路径和类型以及配置文件引⼊等】

因此，这些指令的作用域是Nginx服务器全局。

* **user [user] [group]**  指定可以运行nginx服务的用户和用户组，只能在全局块配置 。user指令在Windows上不生效，如果制定具体用户和用户组会报警告
* **worker_processes** nginx进程数量。比如设置为2，nginx将会开启一个master进程和两个worker进程
* **pid  logs/nginx.pid** 存放pid文件
* **error_log  logs/error.log;**  全局错误日志类型 debug info warn error 存放地址

```nginx
# 指定运⾏Nginx服务器的⽤户，只能在全局块配置
# 将user指令注释掉，或者配置成nobody的话所有⽤户都可以运⾏
# user [user] [group]
# user nobody nobody;
user nginx;

# 指定⽣成的worker进程的数量，也可使⽤自动模式，只能在全局块配置
worker_processes 1;

# 错误⽇志存放路径和类型
error_log /var/log/nginx/error.log warn;

# 进程PID存放路径
pid /var/run/nginx.pid;
```

## events块

events块涉及的指令**主要影响Nginx服务器与用户的网络连接**。常用到的设置包括是否开启对多worker process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型处理连接请求，每个worker process可以同时支持的最大连接数等

* **accept_mutex** 默认开启。开启之后nginx 的多个worker将会以串行的方式来处理，只会有一个worker将会被唤起，其他的worker继续睡眠。如果不开启，会将多个worker全部唤起，但只有一个Worker能获取新连接，其它的Worker会重新进入休眠状态

* **worker_connections** 单个进程最大连接数（最大连接数=连接数+进程数）

```nginx
events {
# 指定使⽤哪种⽹络IO模型，只能在events块中进⾏配置
# use epoll
# 每个worker process允许的最⼤连接数
 worker_connections 1024;
}
```

## http块
http块是Nginx服务器配置中的重要部分，包括http全局块和server块。**代理、缓存和日志定义等绝大多数的功能**和**第三方模块的配置**都可以放在这个模块中。

* **include**指令，用于引入其他的配置文件
* **default_type** 如果Web程序没设置，Nginx也没对应文件的扩展名，就用Nginx 里默认的 default_type定义的处理方式。
* **log_format**指令，用于定义日志格式，只能在http块中进行配置
* **sendfile** 简单来说就是启用sendfile()系统调用来替换read()和write()调用，**减少系统上下文切换从而提高性能**，当 nginx 是静态文件服务器时，能极大提高nginx的性能表现
* **keepalive_timeout** HTTP协议有一个 KeepAlive 模式，它告诉 webserver 在处理完一个请求后保持这个 TCP 连接的打开状态。若接收到来自客户端的其它请求，服务端会利用这个未被关闭的连接，而不需要再建立一个连接。
* **gzip** 开启Gzip压缩功能， 可以使网站的css、js 、xml、html 文件在**传输时进行压缩，提高访问速度**, 进而优化Nginx性能

```nginx
http {
    # nginx 可以使⽤include指令引⼊其他配置⽂件
    include /etc/nginx/mime.types;

    # 默认类型，如果请求的URL没有包含⽂件类型，会使⽤默认类型
    default_type application/octet-stream; # 默认类型

    # 开启⾼效⽂件传输模式
    sendfile on;

    # 连接超时时间
    keepalive_timeout 65;

    # access_log ⽇志存放路径和类型
    # 格式为：access_log <path> [format [buffer=size] [gzip[=level]][flush=time] [if=condition]];
    access_log /var/log/nginx/access.log main;

    # 定义⽇志格式
    log_format main '$remote_addr - $remote_user[$time_local]"$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" "$http_x_forwarded_for"';

    # 设置sendfile最⼤传输⽚段⼤⼩，默认为0，表示不限制
    # sendfile_max_chunk 1m;

    # 每个连接的请求次数
    # keepalive_requests 100;

    # keepalive超时时间
    keepalive_timeout 65;

    # 开启gzip压缩
    # gzip on;

    # 开启gzip压缩的最⼩⽂件⼤⼩
    # gzip_min_length 1k;

    # gzip压缩级别，1-9，级别越⾼压缩率越⾼，但是消耗CPU资源也越多
    # gzip_comp_level 2;

    # gzip压缩⽂件类型
    # gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

    # upstream指令⽤于定义⼀组服务器，⼀般⽤来配置反向代理和负载均衡
    upstream www.example.com {
        # ip_hash指令⽤于设置负载均衡的⽅式，ip_hash表示使⽤客户端的IP进⾏hash，这样可以保证同⼀个客户端的请求每次都会分配到同⼀个服务器，解决了session共享的问题
        ip_hash;
        # weight ⽤于设置权重，权重越⾼被分配到的⼏率越⼤
        server 192.168.50.11:80 weight=3;
        server 192.168.50.12:80;
        server 192.168.50.13:80;
    }
     server {
        # 参考server块的配置
    }
}
```

## server块和location块

**server块是配置虚拟主机的**，⼀个http块可以包含多个server块，每个server块就是⼀个虚拟主机。它内部可有多台主机联合提供服务，一起对外提供在逻辑上关系密切的一组服务

* **listen**指令的配置非常灵活，可以单独制定ip，单独指定端口或者同时指定ip和端口
* **server_name**  nginx允许一个虚拟主机有一个或多个名字，也可以使用通配符" * "来设置虚拟主机的名字。支持ip、域名、通配符、正则等

如果多个server的端口重复，那么根据域名或者主机名去匹配 server_name 进行选择。

```nginx
# curl http://localhost:80 会访问这个
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

# curl http://nginx-dev:80 会访问这个
server{
    listen 80;
    server_name nginx-dev;#主机名
    
    location / {
        root /home/AdminLTE-3.2.0;
        index index.html index2.html index3.html;
    }
  
}
```

***
每个server块中可以包含多个location块。location块在整个Nginx配置文档中起着重要的作用，Nginx服务器在许多功能上的灵活性往往在location指令的配置中体现出来

location 指令可以分为以下 3 类：
* 前缀字符串匹配
* 正则表达式匹配
* 用于内部跳转的命名location

前缀字符串匹配
>精确匹配 =
前缀匹配 ^~（如果匹配成功，立刻停止后续的正则搜索）
按文件中顺序的正则匹配 ~ 或 ~*
匹配不带任何修饰的前缀匹配。


root 指定目录的上级目录，并且该上级目录要含有locatoin指定名称的同名目录。
```nginx
/按照以下配置，则访问/img/目录下的文件时，nginx会去/var/www/image/img/目录下找文件
location /img/ {
	root /var/www/image;
}
```
***
```nginx
server {
    # 监听IP和端⼝
    # listen的格式为：
    # listen [ip]:port [default_server] [ssl] [http2] [spdy][proxy_protocol] [setfib=number][fastopen=number] [backlog=number];
    # listen指令⾮常灵活，可以指定多个IP和端⼝，也可以使⽤通配符,下⾯是几个实际的例⼦：
    # listen 127.0.0.1:80; # 监听来自127.0.0.1的80端⼝的请求
    # listen 80; # 监听来自所有IP的80端⼝的请求
    # listen *:80; # 监听来自所有IP的80端⼝的请求，同上
    # listen 127.0.0.1; # 监听来自127.0.0.1的80端⼝，默认端⼝为80
    listen 80;

    # server_name ⽤来指定虚拟主机的域名，可以使⽤精确匹配、通配符匹配和正则匹配等⽅式
    # server_name example.org www.example.org; # 精确匹配
    # server_name *.example.org; # 通配符匹配
    # server_name ~^www\d+\.example\.net$; # 正则匹配
    server_name localhost;

    # location块⽤来配置请求的路由，⼀个server块可以包含多个location块，每个location块就是⼀个请求路由
    # location块的格式是：
    # location [=|~|~*|^~] /uri/ { ... }
    # = 表示精确匹配，只有完全匹配上才能⽣效
    # ~ 表示区分⼤⼩写的正则匹配
    # ~* 表示不区分⼤⼩写的正则匹配
    # ^~ 表示普通字符匹配，如果匹配成功，则不再匹配其他location
    # /uri/ 表示请求的URI，可以是字符串，也可以是正则表达式
    # { ... } 表示location块的配置内容
    location / {
        # root指令⽤于指定请求的根目录，可以是绝对路径，也可以是相对路径
        root /usr/share/nginx/html; # 根⽬录
        # index指令⽤于指定默认⽂件，如果请求的是目录，则会在目录下查找默认⽂件
        index index.html index.htm; # 默认⽂件
    }
    # 下⾯是⼀些location的示例：
    location = / { # 精确匹配请求
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
    location ^~ /images/ { # 匹配以/images/开头的请求
        root /usr/share/nginx/html;
    }
    location ~* \.(gif|jpg|jpeg)$ { # 匹配以gif、jpg或者jpeg结尾的请求
        root /usr/share/nginx/html;
    }
    location !~ \.(gif|jpg|jpeg)$ { # 不匹配以gif、jpg或者jpeg结尾的请求
        root /usr/share/nginx/html;
    }
    location !~* \.(gif|jpg|jpeg)$ { # 不匹配以gif、jpg或者jpeg结尾的请求
        root /usr/share/nginx/html;
    }
    # error_page ⽤于指定错误⻚⾯，可以指定多个，按照优先级从⾼到低依次查找
    error_page 500 502 503 504 /50x.html; # 错误⻚⾯
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

## vue路由404问题

>问题：vue项目代理到nginx上，进行路由跳转时，会报错404
原因：服务器根据页面路由，按路径寻找资源。vue项目是一个单页面应用，打包好的web站点只有一个html页面，不存在其他资源目录下的html，服务器找不到对应页面，所以才报404。

解决方法：location块中添加`try_files`
`try_files`指令可用于检查指定的文件或目录是否存在;如果不存在，则重定向到指定位置。
```nginx
try_files $uri $uri/ /index.html;
```

***
**`try_files`几个用法**

例1：如果原始URI对应的文件不存在，NGINX将内部重定向到/www/data/images/default.gif

```nginx
server {
    root /www/data;

    location /images/ {
        try_files $uri /images/default.gif;
    }
}
```

例2：如果指令的所有参数都无法解析为现有文件或目录，则会返回404错误
```nginx
location / {
    try_files $uri $uri/ $uri.html =404;
}
```

例3：如果原始 URI 和附加尾随斜杠的 URI 都没有解析到现有文件或目录中，则请求将重定向到命名位置，该位置会将其传递到代理服务器。
```nginx
location / {
    try_files $uri $uri/ @backend;
}

location @backend {
    proxy_pass http://backend.example.com;
}
```


# HTTP反向代理

>在客户端代理转发请求称为正向代理。例如VPN。
在服务器端代理转发请求称为反向代理。例如nginx

Nginx反向代理的两个常用配置语法
```sh
//用来设置被代理服务器地址，可以是主机名称、IP地址加端口号形式。
proxy_pass 
//更改Nginx服务器接收到的客户端请求的请求头信息，然后将新的请求头发送给代理的服务器
proxy_set_header 
```
**proxy_pass配置说明**
如果proxy-pass的地址只配置到端口，不包含/或其他路径，那么location将被追加到转发地址中
```nginx
//访问http://localhost/some/path/page.html 将被代理到http://localhost:8080/some/path/page.html 
location /some/path/ {
    proxy_pass http://localhost:8080;
}
```
如果proxy-pass的地址包括/或其他路径，那么/some/path将会被替换
```nginx
//访问http://localhost/some/path/page.html 将被代理到http://localhost:8080/zh-cn/page.html。
location /some/path/ {
    proxy_pass http://localhost:8080/zh-cn/;
}
```

**proxy_set_header配置说明**

用户可以重新定义或追加header信息传递给后端服务器。可以包含文本、变量及其组合。默认情况下，仅重定义两个字段
```nginx
proxy_set_header Host       $proxy_host;
proxy_set_header Connection close;
```
由于使用反向代理之后，后端服务无法获取用户的真实IP，所以，一般反向代理都会设置以下header信息。
```nginx
location /some/path/ {
    #nginx的主机地址
    proxy_set_header Host $http_host;
    #用户端真实的IP，即客户端IP
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_pass http://localhost:8088;
}
```

常用变量的值：
`$host`：nginx主机IP，例如192.168.56.105
`$http_host`：nginx主机IP和端口，192.168.56.105:8001
`$proxy_host`：localhost:8088，proxy_pass里配置的主机名和端口
`$remote_addr`:用户的真实IP，即客户端IP。

***

**例：代理到百度**
访问/就会被转到百度
```nginx
location / {
   root   html;
   index  index.html index.htm;
   proxy_pass http://www.baidu.com;
}
```

**例2：nginx反向代理解决跨域**
[详见此文章](https://blog.csdn.net/qq1195566313/article/details/124486764?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168862708316800211542009%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=168862708316800211542009&biz_id=0&spm=1018.2226.3001.4450)

# 动静分离

在Web开发中，动态资源其实就是指那些后台资源，而静态资源就是指HTML，JavaScript，CSS，img等文件。

Apache Tocmat 严格来说是一款java EE服务器，主要是用来处理 servlet请求。处理css、js、图片这些静态文件的IO性能不够好。

因此，一般来说，都需要将动态资源和静态资源分开，**将静态资源部署在Nginx上**，如果是静态资源的请求，就直接到nginx配置的静态资源目录下面获取资源，如果是动态资源的请求，nginx利用反向代理的原理，把请求转发给后台应用去处理，实现动静分离。从而提高系统的访问速度，减少tomcat的请求次数，给后端服务器降压。

# 负载均衡

>跨多个应用程序实例的负载平衡是一种常用技术，用于优化资源利用率、最大化吞吐量、减少延迟和确保容错配置。nginx作为非常有效的HTTP负载平衡器，将流量分配到多个应用程序服务器，提升Web应用程序的性能，提高扩展性和可靠性。

使用upstream定义一组服务（upstream 位于 http上下文中，与server 并列，不要放在server中。）

```nginx
upstream ruoyi-apps {
    #不写，默认采用轮循机制
    server localhost:8080;
    server localhost:8088;
  
}

server {
  
  listen 8003;
  server_name ruoyi.loadbalance;
  
  location / {
    proxy_pass http://ruoyi-apps;
  }

}
```
***
**负载均衡策略**

**轮循机制round-robin**：默认机制，以轮循机制方式分发。
**权重weight**：权重越大服务器承载的并发就越高，默认是1
```nginx
upstream my-server {
    server performance.server weight=3;
    server app1.server weight=2;
    server app2.server weight=1;
}
```

**健康检查**：在反向代理中，如果后端服务器在某个周期内响应失败次数超过规定值，nginx会将此服务器标记为失败，并在之后的一个周期不再将请求发送给这台服务器。
* 通过fail_timeout来设置检查周期，默认为10秒。
* 通过max_fails来设置检查失败次数，默认为1次。
* backup是备用服务器参数，在生产server全部都出问题之后，可以自动切换到备用server上，为恢复服务争取时间

在以下示例中，如果NGINX无法向服务器发送请求或在30秒内请求失败次数超过3次，则会将服务器标记为不可用30秒。

```nginx
upstream backend {
  server backend1.example.com;
  server backend2.example.com max_fails=3 fail_timeout=30s; 
  server backend3.example.com backup;
} 
```

**最小连接least-connected**：将下一个请求分配给活动连接数最少的服务器
```nginx
upstream backend {
    least_conn; # 采用把最小连接策略
    server backend1.example.com;
    server backend2.example.com;
}
```

**ip-hash**：客户端的 IP 地址将用作哈希键，来自同一个ip的请求会被转发到相同的服务器。
```nginx
upstream backend {
    ip_hash; # 采用ip-hash策略
    server backend1.example.com;
    server backend2.example.com;
}
```

基于 IP 的哈希算法存在一个问题，那就是当有一个上游服务器宕机或者扩容的时候，会引发大量的路由变更，进而引发连锁反应，导致大量缓存失效等问题。

**hsah**：允许用户自定义hash的key，key可以是字符串、变量或组合。
例如，key可以是配对的源 IP 地址和端口，也可以是 URI，如以下示例所示：
```nginx
upstream backend {
    hash $request_uri consistent; 
    server backend1.example.com;
    server backend2.example.com;
}
```

`consistent`参数启用ketama一致哈希算法，如果在上游组中添加或删除服务器，只会重新映射部分键，从而最大限度地减少缓存失效。

**随机**：每个请求都将传递到随机选择的服务器。
```nginx
upstream backend {
    random two least_conn;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
    server backend4.example.com;
}
```

`two`是可选参数，NGINX在考虑服务器权重的情况下随机选择两台服务器，然后使用指定的方法选择其中一台，默认为选择连接数最少（least_conn）的服务器。

# 配置HTTPS

>HTTPS 协议是由HTTP 加上TLS/SSL 协议构建的可进行加密传输、身份认证的网络协议，主要通过数字证书、加密算法、非对称密钥等技术完成互联网数据传输加密，实现互联网传输安全保护。

① 生成ssl证书

② 打开编辑配置文件

```sh
sudo vim /etc/nginx/sites-enabled/default
```

把80端口注释掉，将ssl的443端口的注释去掉，然后按如下格式添加证书：

```nginx
server {
    listen              443 ssl;
    server_name         xxxx;
    ssl_certificate     ssl_certificate存放的位置;
    ssl_certificate_key ssl_certificate_key存放的位置;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
  
    location / {
        ...
    }
}
```

***

**https优化**
SSL 操作会消耗额外的 CPU 资源。CPU 占用最多的操作是 SSL 握手。有两种方法可以最大程度地减少每个客户端的这些操作数：

* 使保持活动连接能够通过一个连接发送多个请求
* 重用 SSL 会话参数以避免并行连接和后续连接的 SSL 握手

会话存储在工作进程之间共享并由 ssl_session_cache 指令配置的 SSL 会话缓存中。一兆字节的缓存包含大约 4000 个会话。默认缓存超时为 5 分钟。可以使用 ssl_session_timeout 指令增加此超时。以下是针对具有 10 MB 共享会话缓存的多核系统优化的示例配置：

```sh
ssl_session_cache   shared:SSL:10m;
ssl_session_timeout 10m;
```

# 优化写法

[看这篇文章](https://www.yuque.com/wukong-zorrm/cql6cz/ld0pzi)