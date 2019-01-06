# Nginx-practice  
  
## Nginx学习实践  
  
- [什么是Nginx](#什么是nginx)  
- [Apache、Tomcat、Nginx的区别](#apachetomcatnginx的区别)  
- [安装Nginx](#安装nginx)  
- [Nginx配置](#nginx配置)  
  - [虚拟主机配置](#虚拟主机配置)  
  - [基于端口号的虚拟主机](#基于端口号的虚拟主机)  
  - [基于域名的虚拟主机](#基于域名的虚拟主机)  
  - [location](#location)  
- [Nginx模块](#nginx模块)  
- [反向代理](#反向代理)  
- [负载均衡](#负载均衡)  
  - [其他功能配置](#其他功能配置)  
- [Nginx动静分离](#nginx动静分离)  
- [缓存](#缓存)    
- [压缩](#压缩)  
- [防盗链](#防盗链)
- [跨域访问](#跨域访问)  
- [Nginx进程模型简介](#nginx进程模型简介)  
- [Nginx的高可用方案](#nginx的高可用方案)  
  - [keepalived](#keepalived)  
  - [轻量级的高可用解决方案](#轻量级的高可用解决方案)  
- [高可用实践](#高可用实践)  
  - [keepalived的配置](#keepalived的配置)  
  - [keepalived日志文件配置](#keepalived日志文件配置)  
  - [通过脚本实现动态切换](#通过脚本实现动态切换)  
- [Openresty](#openresty)  
- [网关](#网关)  
  - [网关的概念](#网关的概念)  
  - [为什么需要网关](#为什么需要网关)  
- [OpenResty实现API网关限流及登录授权](#openResty实现api网关限流及登录授权)  
  - [灰度发布的实现](#灰度发布的实现)  
  
### 什么是Nginx  
  
Nginx是一个高性能的反向代理服务器。  
Nginx是一款轻量级的Web服务器/反向代理服务器以及电子邮件代理服务器，并在一个BSD-like协议下发行。由俄罗斯的程序设计师lgor Sysoev所开发，供俄国大型的入口网站及搜索引擎Rambler使用。其特点是占有内存少，并发能力强。  
>正向代理代理的是客户端  
反向代理代理的是服务端  
  
### Apache、Tomcat、Nginx的区别  
Apache与Nginx属于静态web服务器，只返回静态资源，不具备解析jsp/servlet、php的能力。  
Tomcat是可以解析jsp/servlet的动态web服务器。  
  
### 安装Nginx  
>1.下载tar包  
>2.tar -zxvf nginx.tar.gz  
>3. ./configure [--prefix]  
>4. make && make install  
  
启动和停止
>1. sbin/nginx  
>2. sbin/nginx -s stop  
  
### Nginx配置  
  
Nginx的配置文件为nginx.conf。 主要分为三个部分  
>Main           
event  
http  
  
#### 虚拟主机配置  
```
server {
        listen       80;
        server_name  localhost;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        
        location / {
            root   html;
            index  index.html index.htm;
        }
}
```
#### 基于端口号的虚拟主机  
```
server {
       listen        8080;
       server_name   localhost;
       
       location / {
           root html;
           index index.html;
       }
}
```
#### 基于域名的虚拟主机  
www.zyf.com / order.zyf.com / user.zyf.com  
```
server {
   listen        80;
   server_name   www.zyf.com;
   location / {
       root html;
       index index.html;
   }
}
server {
   listen        80;
   server_name   order.zyf.com;
   location / {
       root html;
       index  order.html;
   }
}
server {
   listen        80;
   server_name   user.zyf.com;
   location / {
       root html;
       index  user.html;
   }
}
```
#### location  
配置语法  
```
location [= | ~* | ^~ ] /uri/ {...}
```
配置规则  
```
location = /uri 精准匹配 
location ^~ /uri 前缀匹配 
location ~ /uri 正则匹配
location / 通用匹配
```
规则的优先级  
```
1 location = /
2 location = /index
3 location ^~ /article/
4 location ^~ /article/files/
5 location ~ \.(gif|png|js|css)$
6 location /

http://192.168.11.154/ ->1
http://192.168.11.154/index ->2
http://192.168.11.154/article/files/1.txt ->4 
http://192.168.11.154/zyf.png ->5
```
>1.精准匹配是优先级最高   
2.普通匹配(最长的匹配)   
3.正则匹配  
  
实际使用建议  
```
location =/ {
}

location / {
}

location ~* \.(gif|....)${
}
```

### Nginx模块  

模块分类  
>1.核心模块 ngx_http_core_module  
2.标准模块 http模块  
3.第三方模块  

#### ngx_http_core_module
```
server{ 
    listen port 
    server_name  
    root 
    ... 
}

location { 实现uri到文件系统路径的映射
}

error_page 定义错误码页面 根据不同的错误返回不同的页面  
```

#### ngx_http_access_module  
实现基于ip的访问控制功能  
>1.allow address | CIDR | unix: | all;  
2.deny address | CIDR | unix: | all;  
  
自上而下检查，一旦匹配，将生效，条件严格的置前。  
 
#### 如何添加第三方模块  
>1.原来所安装的配置，你必在重新安装新模块的时候再次添加。  
2.不能直接make install  
  
```
configure --prefix=/data/program/nginx
```
安装方法  
```
./configure --prefix=/安装目录 --add-module = /第三方模块的目录

或者
./configure --prefix=/data/program/nginx --with-http_stub_status_module --with-http_random_index_module 

用新的nginx替换旧的
cp objs/nginx $nginx_home/sbin/nginx
```

#### http_stub_status_module  
```
location /status {
            stub_status;
}
```
>Active connections:当前状态，活动状态的连接数。   
accepts:统计总值，已经接受的客户端请求的总数。  
handled:统计总值，已经处理完成的客户端请求的总数。   
requests:统计总值，客户端发来的总的请求数。  
Reading:当前状态，正在读取客户端请求报文首部的连接的连接数。   
Writing:当前状态，正在向客户端发送响应报文过程中的连接数。  
Waiting:当前状态，正在等待客户端发出请求的空闲连接数。  
  
#### http_random_index_module
随机显示主页  
一般情况下,一个站点默认首页都是定义好的index.html、index.shtml等等，如果想站点下有很多页面想随机展示给用户浏览，那得程序上实现，很麻烦，使用nginx的random index即可简单实现这个功能，凡是以/结尾的请求，都会随机展示当前目录下的文件作为首页。  
```
1. 添加random_index on 配置，默认是关闭的。

location / {
    root   html;
    random_index on;
    index  index.html index.htm;
}

2. 在html目录下创建多个html页面。
``` 
  
### 反向代理  
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86.jpeg)  
  
nginx反向代理的指令不需要新增额外的模块，默认自带proxy_pass指令，只需要修改配置文件就可以实现反向代理。
proxy_pass 既可以是ip地址，也可以是域名，同时还可以指定端口。  
```
server {
    listen 80;
    server_name localhost;
    location / {
       proxy_pass http://192.168.11.161:8080;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    } 
}
```
#### Nginx反向代理实战
>1.启动tomcat服务器  
2.nginx配置的统一维护，将Nginx.conf文件的内容修改成如下配置  
3.在extra文件夹中添加proxy_demo.conf   
  
```
server{
    listen        80;
    server_name   localhost;
    location / {
          proxy_pass http://192.168.11.154:8080; 
    }
}
```
>4. ./nginx -s reload 重新加载  
  
### 负载均衡
网络负载均衡的大致原理是利用一定的分配策略将网络负载平衡地分摊到网络集群的各个操作单元上，使得单个重负载任务能够分担到多个单元上并行处理，使得大量并发访问或数据流量分担到多个单元上分别处理，从而减少用户的等待响应时间。  
  
upstream  
upstream是Nginx的HTTP Upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。   
语法: server address [paramters]  
  
负载均衡策略或者算法  
>1.轮询算法(默认)，如果后端服务器宕机以后，会自动踢出。  
2.ip_hash 根据请求的ip地址进行hash。   
3.权重轮询。可以对某个性能好的服务器采用较大权重。  
  
```
upstream tomcat {
  server 192.168.11.161:8080 max_fails=2 fail_timeout=60s;
  server 192.168.11.159:8080;
}
server {
    listen        80;
    server_name   localhost;
    location / {
       proxy_pass http://tomcat;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_next_upstream error timeout http_500 http_503;
       proxy_connect_timeout 60s;
       proxy_send_timeout 60s;
       proxy_read_timeout 60s;
       add_header 'Access-Control-Allow-Origin' '*';
       add_header 'Access-Control-Allow-Methods' 'GET,POST,DELETE';
       add_header 'Aceess-Control-Allow-Header' 'Content-Type,*';
    }
    location ~ .*\.(js|css|png|svg|ico|jpg)$ {
       valid_referers none blocked 192.168.11.160 www.zyf.com;
       if ($invalid_referer) {
          return 404; 
       }
       root static-resource;
       expires 1d; 
    }
}
```
  
#### 其他功能配置  
proxy_next_upstream  
语法:proxy_next_upstream [error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 | http_404 | off ];  
默认:proxy_next_upstream error timeout;  
配置块:http、server、location   
这个配置表示当向一台上有服务器转发请求出现错误的时候，继续换一台上后服务器来处理这个请求。  
默认情况下，上游服务器一旦开始发送响应数据，Nginx反向代理服务器会立刻把应答包转发给客户端。因此一旦Nginx开始向客户端发送响应包，如果中途出现错误也不允许切换到下一个上有服务器继续处理的。这样做的目的是保证客户端只收到来自同一个上游服务器的应答。  

proxy_connect_timeout  
语法: proxy_connect_timeout time;  
默认: proxy_connect_timeout 60s;  
范围: http, server, location  
用于设置nginx与upstream server的连接超时时间，比如我们直接在location中设置proxy_connect_timeout 1ms，1ms很短，如果无法在指定时间建立连接，就会报错。  
  
proxy_send_timeout  
向后端写数据的超时时间，两次写操作的时间间隔如果大于这个值，也就是过了指定时间后端还没有收到数据，连接会被关闭。
  
proxy_read_timeout  
从后端读取数据的超时时间，两次读取操作的时间间隔如果大于这个值，那么nginx和后端的链接会被关闭，如果一个请求的处理时间比较长，可以把这个值设置得大一些。  
proxy_upstream_fail_timeout  
设置了某一个upstream后端失败了指定次数(max_fails)后，在fail_timeout时间内不再去请求它，默认为10秒。  
语法 server address [fail_timeout=30s]  
```
upstream backend { #服务器集群名字
#server 192.168.218.129:8080 weight=1 max_fails=2 fail_timeout=600s;
#server 192.168.218.131:8080 weight=1 max_fails=2 fail_timeout=600s; 
}
```
  
### Nginx动静分离  
  
#### 什么是动静分离  
必须依赖服务器生存的我们称为动。不需要依赖容器的比如css/js或者图片等，这类就叫静。  
  
在Nginx的conf目录下，有一个mime.types文件，里面记录了静态资源的类型。  
  
用户访问一个网站，然后从服务器端获取相应的资源通过浏览器进行解析渲染最后展示给用户，而服务端可以返回各种类型的内容，比如xml、jpg、png、gif、flash、MP4、html、css等等，浏览器就是根据mime-type来决定用什么形式来展示的。  
服务器返回的资源给到浏览器时，会把媒体类型告知浏览器，这个告知的标识就是Content-Type，比如Content- Type:text/html。
```
location ~ .*\.(js|css|png|svg|ico|jpg)$ {
       valid_referers none blocked 192.168.11.160 www.zyf.com;
       if ($invalid_referer) {
          return 404; 
       }
       root static-resource;
       expires 1d; 
}
```
#### 动静分离的好处  
>1.Nginx本身就是一个高性能的静态web服务器。  
2.其实静态文件有一个特点就是基本上变化不大，所以动静分离以后我们可以对静态文件进行缓存、或者压缩提高网站性能。  
  
### 缓存  
  
当一个客户端请求web服务器, 请求的内容可以从以下几个地方获取：服务器、浏览器缓存中或缓存服务器。这取决于服务器端输出的页面信息。  
浏览器缓存将文件保存在客户端，好的缓存策略可以减少对网络带宽的占用，可以提高访问速度，提高用户的体验，还可以减轻服务器的负担nginx缓存配置。  

#### Nginx缓存配置  
Nginx可以通过expires设置缓存，比如我们可以针对图片做缓存，因为图片这类信息基本上不会改变。  
在location中设置expires  
格式: `expires 30s|m|h|d`  
```
location ~ .*.(jpg|jpeg|gif|bmp|png|js|css|ico)$ {
    root static; 
    expires 1d; 
}
```

### 压缩  
  
#### Gzip  
我们一个网站一定会包含很多的静态文件，比如图片、脚本、样式等等，而这些css/js可能本身会比较大，那么在网络传输的时候就会比较慢，从而导致网站的渲染速度。因此Nginx中提供了一种Gzip的压缩优化手段，可以对后端的文件进行压缩传输，压缩以后的好处在于能够降低文件的大小来提高传输效率。  
  
#### 配置信息  
```
Gzip on|off           是否开启gzip压缩。  
Gzip_buffers 4 16k    设置gzip申请内存的大小，作用是按指定大小的倍数申请内存空间。4 16k代表按照原始数据大小以16k为单位的4倍申请内存。  
Gzip_comp_level[1-9]  压缩级别，级别越高，压缩越小，但是会占用CPU资源 Gzip_disable #正则匹配UA表示什么样的浏览器不进行gzip。  
Gzip_min_length       开始压缩的最小长度(小于多少就不做压缩)，可以指定单位，比如 1k Gzip_http_version 1.0|1.1 表示开始压缩的http协议版本。  
Gzip_proxied          (nginx 做前端代理时启用该选项，表示无论后端服务器的headers头返回什么信息，都无条件启用压缩)。  
Gzip_type             对哪些类型的文件做压缩 (conf/mime.conf)。  
Gzip_vary on|off      是否传输gzip压缩标识；启用应答头"Vary: Accept-Encoding"；给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费 不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩。  
```
  
```
http {
    include       mime.types;
    default_type  application/octet-stream;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  60;
    include extra/*.conf;
    gzip  on;
    gzip_min_length 5k;
    gzip_comp_level 3;
    gzip_types application/javascript image/jpeg image/svg+xml;
    gzip_buffers 4 32k;
    gzip_vary on;
}
```
  
### 防盗链  
  
一个网站上会有很多的图片，如果你不希望其他网站直接用你的图片地址访问自己的图片，或者希望对图片有版权保护。再或者不希望被第三方调用造成服务器的负载以及消耗比较多的流量问题，那么防盗链就是你必须要做的。  
  
#### 防盗链配置  
在Nginx中配置防盗链其实很简单，
语法: valid_referers none | blocked | server_names | string ...;   
默认值: —  
上下文: server, location  
“Referer”请求头为指定值时，内嵌变量$invalid_referer被设置为空字符串，否则这个变量会被置成“1”。查找匹配时不区分大小写，其中none表示缺少referer请求头、blocked表示请求头存在，但是它的值被防火墙或者代理服务器删除、server_names表示referer请求头包含指定的虚拟主机名。配置如下  
```
location ~ .*.(gif|jpg|ico|png|css|svg|js)$ { 
    valid_referers none blocked 192.168.11.153;
    if ($invalid_referer) { 
      return 404;
    }
    root static; 
}
```
需要注意的是伪造一个有效的“Referer”请求头是相当容易的，因此这个模块的预期目的不在于彻底地阻止这些非法请求，而是为了阻止由正常浏览器发出的大规模此类请求。还有一点需要注意，即使正常浏览器发送的合法请求，也可能没 有“Referer”请求头。  

### 跨域访问  
  
什么叫跨域呢？如果两个节点的协议、域名、端口、子域名不同，那么进行的操作都是跨域的，浏览器为了安全问题都是限制跨域访问，所以跨域其实是浏览器本身的限制。  
解决办法  
```
修改proxy_demo.conf配置  
server{
   listen        80;
   server_name   localhost;
   location / {
       proxy_pass http://192.168.11.154:8080;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_send_timeout 60s;
       proxy_read_timeout 60s;
       proxy_connect_timeout 60s;
       add_header 'Access-Control-Allow-Origin' '*'; // #允许来自所有的访问地址
       add_header 'Access-Control-Allow-Methods' 'GET,PUT,POST,DELETE,OPTIONS'; //支持的请求方式
       add_header 'Access-Control-Allow-Header' 'Content-Type,*'; //支持的媒体类型
   }
   location ~ .*\.(gif|jpg|ico|png|css|svg|js)$ {
       root static;
   }
}
```
  
### Nginx进程模型简介  
  
#### 多进程    
启动Nginx会有两个进程，master进程与worker进程。其中worker进程可以有多个，全部由master进程来管理。  
```
root   7473    1 0 20:09 ? 00:00:00 nginx: master process ./nginx 
nobody 7474 7473 0 20:09 ? 00:00:00 nginx: worker process
```
每一个worker都采用多路复用处理任务。  
```
#user nobody; 用户 
worker_processes 1; 工作进程数，建议设置为 cpu 总核心数。
#error_log logs/error.log; 
#error_log logs/error.log notice; 
#error_log logs/error.log info;
#pid        logs/nginx.pid;

events {
  useepoll; io模型
  worker_connections 1024; 每一个worker进程的连接数，理论上总连接数为 processes* connections。
}
```
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/%E5%A4%9A%E8%BF%9B%E7%A8%8B.jpeg)  
  
### Nginx的高可用方案  
  
Nginx 作为反向代理服务器，所有的流量都会经过 Nginx，所以 Nginx 本身的可靠性是我们首先要考虑的问题。  
  
#### keepalived  
Keepalived 是 Linux 下一个轻量级别的高可用解决方案，Keepalived 软件起初是 专为 LVS 负载均衡软件设计的，用来管理并监控 LVS 集群系统中各个服务节点的状态，后来又加入了可以实现高可用的 VRRP 功能。因此，Keepalived 除了能够管理 LVS 软件外，还可以作为其他服务(例如:Nginx、Haproxy、MySQL 等)的 高可用解决方案软件。  
  
Keepalived 软件主要是通过 VRRP 协议实现高可用功能的。VRRP 是 Virtual Router RedundancyProtocol(虚拟路由器冗余协议)的缩写，VRRP 出现的目的就是为了解决静态路由单点故障问题的，它能够保证当个别节点宕机时，整个网络可以不间断地运行(简单来说，vrrp 就是把两台或多态路由器设备虚拟成一个设备，实现主备高可用)。  
  
所以，Keepalived 一方面具有配置管理 LVS 的功能，同时还具有对 LVS 下面节点进行健康检查的功能，另一方面也可实现系统网络服务的高可用功能。  
  
LVS 是 Linux Virtual Server 的缩写，也就是 Linux 虚拟服务器，在 linux2.4 内核以后，已经完全内置了 LVS 的各个功能模块。  
  
它是工作在四层的负载均衡，类似于 Haproxy, 主要用于实现对服务器集群的负载均衡。  
  
关于四层负载，我们知道 osi 网络层次模型的 7 层模模型(应用层、表示层、会话层、传输层、网络层、数据链路层、物理层)。  
四层负载就是基于传输层，也就是ip+端口的负载；  
而七层负载就是需要基于 URL 等应用层的信息来做负载。  
同时还 有二层负载(基于 MAC)、三层负载(IP)；  
  
常见的四层负载有:LVS、F5；  
七层负载有:Nginx、HAproxy；  
在软件层面， Nginx/LVS/HAProxy 是使用得比较广泛的三种负载均衡软件。  
  
对于中小型的 Web 应用，可以使用 Nginx、大型网站或者重要的服务并且服务比较多的时候，可以考虑使用 LVS。  
  
#### 轻量级的高可用解决方案  
LVS 四层负载均衡软件(Linux virtual server) 监控。  
lvs 集群系统中的各个服务节点的状态。  
VRRP 协议(虚拟路由冗余协议)。  
linux2.4 以后，是内置在 linux 内核中的。  
>lvs(四层) -> HAproxy(七层)  
七层 lvs(四层) -> Nginx(七层)  
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/%E9%AB%98%E5%8F%AF%E7%94%A8.jpeg)  
  
### 高可用实践  
>1.下载 keepalived 的安装包  
2.tar -zxvf keepalived-2.0.7.tar.gz  
3.在/data/program/目录下创建一个 keepalived 的文件。  
4.cd 到 keepalived-2.0.7 目录下，执行 ./configure -- prefix=/data/program/keepalived --sysconf=/etc  
5.如果缺少依赖库，则 yum install gcc; yum install openssl-devel ; yum install libnl libnl-devel  
6.编译安装 make && make install  
7.进入安装后的路径 cd /data/program/keepalived, 创建软连接: ln -s sbin/keepalived /sbin  
8.cp /data/program/keepalived-2.0.7/keepalived/etc/init.d/keepalived /etc/init.d  
9.chkconfig --add keepalived \10. chkconfig keepalived on \11. service keepalived start  
  
#### keepalived的配置  
master  
```
! Configuration File for keepalived 
global_defs {
    router_id LVS_DEVEL 运行 keepalived 服务器的标识，在一个网络内应该是唯一的
}
vrrp_instance VI_1 { #vrrp 实例定义部分
    stateMASTER #设置lvs的状态，MASTER和BACKUP两种，必须大写 
    interface ens33 #设置对外服务的接口
    virtual_router_id 51 #设置虚拟路由标示，这个标示是一个数字，同一个 vrrp 实例使用唯一标示
    priority 100 #定义优先级，数字越大优先级越高，在一个 vrrp——instance 下，master 的优先级必须大于 backup
    advert_int 1 #设定 master 与 backup 负载均衡器之间同步检查的时间间隔，单位是秒
    authentication { #设置验证类型和密码
        auth_type PASS
        auth_pass 1111 #验证密码，同一个 vrrp_instance 下 MASTER 和 BACKUP 密码必须相同
    }
    virtual_ipaddress { #设置虚拟 ip 地址，可以设置多个，每行一个
        192.168.11.100
    }
}
virtual_server 192.168.11.100 80 { #设置虚拟服务器，需要指定虚拟 ip 和服务端口
    delay_loop 6 #健康检查时间间隔
    lb_algo rr #负载均衡调度算法
    lb_kind NAT #负载均衡转发规则 
    persistence_timeout 50 #设置会话保持时间 
    protocol TCP #指定转发协议类型，有 TCP 和 UDP 两种
    real_server 192.168.11.160 80 { #配置服务器节点 1，需要指定 real server 的真实 IP 地址和端口
        weight 1 #设置权重，数字越大权重越高
        TCP_CHECK{ #realserver的状态监测设置部分单位秒 
            connect_timeout 3 #超时时间 
            delay_before_retry 3 #重试间隔
            connect_port 80 #监测端口
        } 
    }
}
```
backup  
```
! Configuration File for keepalived
global_defs {
   router_id LVS_DEVEL
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.11.100
    } 
}
virtual_server 192.168.11.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    real_server 192.168.11.161 80 {
        weight 1
        TCP_CHECK {
           connect_timeout 3
           delay_before_retry 3
           connect_port 80
        } 
    }
}
```
#### keepalived日志文件配置
1.首先看一下/etc/sysconfig/keepalived 文件  
```
vi /etc/sysconfig/keepalived  
```
```
KEEPALIVED_OPTIONS="-D -d -S 0"  
``` 
“-D” 就是输出日志的选项  
“-S 0”表示 local0.*  
具体的还需要看一下/etc/syslog.conf 文件  
2.在/etc/rsyslog.conf 里添加:local0.* /var/log/keepalived.log  
3.重新启动 keepalived 和 rsyslog 服务:  
  
```
service rsyslog restart
service keepalived restart
```
#### 通过脚本实现动态切换
1.在 master 和 slave 节点的 /data/program/nginx/sbin/nginx-ha-check.sh 目录下增加一个脚本    
```
#!bin/sh  
#指此脚本使用/bin/sh 来执行  

A=`ps -C nginx --no-header |wc -l`  
#–no-headers 不打印头文件  Wc –l  统计行数

if [ $A -eq 0 ]
   then
   echo 'nginx server is died'
   service keepalived stop
fi
```
2.修改 keepalived.conf 文件，增加如下配置  
track_script: #执行监控的服务。  
chk_nginx_service #引用 VRRP 脚本，即在 vrrp_script 部分指定的名字。定期运行它们来改变优先级，并最终引发主备切换。  
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/keepalived.jpeg)  
  
### Openresty  
  
OpenResty 是一个通过 Lua 扩展 Nginx 实现的可伸缩的 Web 平台，内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超 高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。 
  
#### 安装
1.下载安装包  
```
https://openresty.org/cn/download.html  
```
2.安装软件包  
```
tar -zxvf openresty-1.13.6.2.tar.gz  
cd openrestry-1.13.6.2  
./configure [默认会安装在/usr/local/openresty 目录] --prefix= 指定路径  
make && make install  
```
3.可能存在的错误，第三方依赖库没有安装的情况下会报错  
```
yum install readline-devel / pcre-devel /openssl-devel  
```
安装过程和 Nginx 是一样的，因为他是基于 Nginx 做的扩展。  
  
#### HelloWorld
开始第一个程序，HelloWorld  
```
cd /data/program/openresty/nginx/conf
```
```
location / {
   default_type  text/html;
   content_by_lua_block {
      ngx.say("helloworld");
   }
}
```
在 sbin 目录下执行.nginx 命令就可以运行，看到 helloworld。  

#### 建立工作空间  
创建目录  
为了不影响默认的安装目录，我们可以创建一个独立的空间来练习，先到在安装目录下创建 demo 目录,安装目录为/data/program/openresty/demo  
  
mkdir demo  
然后在 demo 目录下创建两个子目录，一个是 logs，一个是 conf。  
  
创建配置文件  
```
worker_processes 1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8888;
        location / {
            default_type text/html;
            content_by_lua_block {
                ngx.say("Hello world")
            } 
        }
    }
}
```
执行:
```
./nginx -p /data/program/openresty/demo 【-p 主要是指明 nginx 启动时 的配置目录】
```
  
刚刚通过一个 helloworld 的简单案例来演示了 nginx+lua 的功能，其中用到了 ngx.say 这个表达式，通过在 contentbyluablock 这个片段中进行访问。这个表达式属于 ngxlua 模块提供的 api，用于向客户端输出一个内容。  

#### 库文件使用  
通过上面的案例，我们基本上对 openresty 有了一个更深的认识，其中我们用到了自定义的 lua 模块。实际上 openresty 提供了很丰富的模块，让我们在实现某些场景的时候更加方便。可以在 /openresty/lualib 目录下看到。比如在 resty 目录下可以看到 redis.lua、mysql.lua 这样的操作 redis 和操作数据库的模块。  
  
使用 redis 模块连接redis  
```
worker_processes 1;
error_log  logs/error.log;
events {
    worker_connections 1024;
}
http {
    lua_package_path '$prefix/lualib/?.lua;;'; 添加”;;”表示默认路径下的 lualib
    lua_package_cpath '$prefix/lualib/?.so;;';
    server {
        location /demo {
            content_by_lua_block {
            local redisModule=require "resty.redis";
            local redis=redisModule:new(); # lua 的对象实例
            redis:set_timeout(1000);
            ngx.say("=======begin connect redis server"); 
            localok,err=redis:connect("127.0.0.1",6379); #连接redis
            if not ok then
              ngx.say("==========connection redis failed,error message:",err);
            end
            
            ngx.say("======begin set key and value"); 
            ok,err=redis:set("hello","world");
            if not ok then
              ngx.say("set value failed");
              return; 
            end
            
            ngx.say("===========set value result:",ok); 
            redis:close();
        } 
    }
}
```
到 nginx 路径下执行 
```
./nginx -p /data/program/openresty/redisdemo 
```
在浏览器中输入:http://192.168.11.160/demo 即可看到输出内容。  
连接到 redis 服务器上以后，可以看到 redis 上的结果。  

redis 的所有命令操作，在 lua 中都有提供相应的操作。比如 redis:get(“key”)、redis:set()等。  
  
### 网关  
  
####  网关的概念  
从一个房间到另一个房间，必须必须要经过一扇门，同样，从一个网络向另一个网络发送信息，必须经过一道“关口”，这道关口就是网关。顾名思义，网关(Gateway) 就是一个网络连接到另一个网络的“关口”。  
  
那什么是 api 网关呢?  
在微服务流行起来之前，api 网关就一直存在，最主要的应用场景就是开放平台，也就是 open api，比如阿里的开放平台。当微服务流行起来以后，api 网关就成了上层应用集成的标配组件。  
  
#### 为什么需要网关  
对微服务组件地址进行统一抽象。  
API 网关意味着你要把 API 网关放到你的微服务的最前端，并且要让 API 网关变成由应用所发起的每个请求的入口。这样就可以简化客户端实现和微服务应用程序之间的沟通方式。  
  
Backends for frontends  
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/%E7%BD%91%E5%85%B3.jpeg)  
  
当服务越来越多以后，我们需要考虑一个问题，就是对某些服务进行安全校验以及用户身份校验。甚至包括对流量进行控制。我们会对需要做流控、需要做身份认证的服务单独提供认证功能，但是服务越来越多以后，会发现很多组件的校验是重复的。这些东西很明显不是每个微服务组件需要去关心的事情。微服务组件只需要负责接收请求以及返回响应即可。可以把身份认证、流控都放在 API 网关层进行控制。
  
灰度发布  
在单一架构中，随着代码量和业务量不断扩大，版本迭代会逐步变成一个很困难的事情，哪怕是一点小的修改，都必须要对整个应用重新部署。但是在微服务中，各个模块是是一个独立运行的组件，版本迭代会很方便，影响面很小。 
同时，为服务化的组件节点，对于我们去实现灰度发布(金丝雀发布:将一部分流量引导到新的版本)来说，也会变的很简单。  
所以通过 API 网关，可以对指定调用的微服务版本，通过版本来隔离。如下图所示  
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/%E7%81%B0%E5%BA%A6%E5%8F%91%E5%B8%83.jpeg)  
  
### OpenResty实现API网关限流及登录授权  
  
#### OpenResty 为什么能做网关  
前面我们了解到了网关的作用，通过网关，可以对 api 访问的前置操作进行统一的管理，比如鉴权、限流、负载均衡、日志收集、请求分片等。所以 API 网关的核心是所有客户端对接后端服务之前，都需要统一接入网关，通过网关层将所有非业务功能进行处理。  
  
OpenResty 为什么能实现网关呢? OpenResty 有一个非常重要的因素是，对于每一个请求，Openresty 会把请求分为不同阶段，从而可以让第三方模块通过挂载行为来实现不同阶段的自定义行为。而这样的机制能够让我们非常方便的设计 api 网关。  
![](https://github.com/YufeizhangRay/image/blob/master/Nginx/OpenResty.jpeg)  
  
Nginx 本身在处理一个用户请求时，会按照不同的阶段进行处理，总共会分为 11 个阶段。而 openresty 的执行指令，就是在这 11 个步骤中挂载 lua 执行脚本实现 扩展。每个指令的作用：
```
initbylua : 当 Nginx master 进程加载 nginx 配置文件时会运行这段 lua 脚本，一般用来注册全局变量或者预加载 lua 模块。  
initwokerby_lua: 每个 Nginx worker 进程启动时会执行的 lua 脚本，可以用来做健康检查。  
setbylua:设置一个变量。  
rewritebylua:在 rewrite 阶段执行，为每个请求执行指定的 lua 脚本。  
accessbylua:为每个请求在访问阶段调用 lua 脚本。  
contentbylua:前面演示过，通过 lua 脚本生成 content 输出给 http 响应。  
balancerbylua:实现动态负载均衡，如果不是走 contentbylua，则走 proxy_pass,再通过 upstream 进行转发。  
headerfilterby_lua: 通过 lua 来设置 headers 或者 cookie。  
bodyfilterby_lua:对响应数据进行过滤。  
logbylua : 在 log 阶段执行的脚本，一般用来做数据统计，将请求数据传输到后端进行分析。  
```
  
#### 灰度发布的实现  
1.文件件目录，/data/program/openresty/gray [conf、logs、lua]  
2.编写 Nginx 的配置文件 nginx.conf
```
worker_processes 1;
error_log  logs/error.log;
events{
    worker_connections 1024;
} 
http{
    lua_package_path "$prefix/lualib/?.lua;;"; 
    lua_package_cpath "$prefix/lualib/?.so;;"; 
    upstream prod {
           server 192.168.11.156:8080;
    }
    upstream pre {
           server 192.168.11.156:8081;
    } 
    server {
           listen 80;
           server_name localhost;
           location /api {
              content_by_lua_file 
              lua/gray.lua; 
           }
           location @prod {
              proxy_pass http://prod;
           }
           location @pre {
              proxy_pass http://pre;
           }
    } 
    server {
           listen 8080;
           location / {
              content_by_lua_block {
                 ngx.say("I'm prod env");
              }
           } 
    }
    server {
           listen 8081;
           location / {
              content_by_lua_block {
                 ngx.say("I'm pre env");
              }
           } 
    }
}
```
3.编写 gray.lua 文件 
```
local redis=require "resty.redis"; 
local red=redis:new(); 
red:set_timeout(1000);
local ok,err=red:connect("192.168.11.156",6379);
if not ok then
  ngx.say("failed to connect redis",err); 
  return;
end 
local_ip=ngx.var.remote_addr; 
local ip_lists=red:get("gray");
if string.find(ip_lists,local_ip) == nil then 
  ngx.exec("@prod");
  else
    ngx.exec("@pre");
  end
local ok,err=red:close();
```
4. 
```
1. 执行命令启动 nginx: [./nginx -p /data/program/openresty/gray]  
2. 启动 redis，并设置 set gray 192.168.11.160  
3. 通过浏览器运行: http://192.168.11.160/api 查看运行结果  
```
修改 redis gray 的值，将客户端的 ip 存储到 redis 中 set gray 1. 再次运行结果，即可看到访问结果已经发生了变化。
