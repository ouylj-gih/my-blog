---
title: Nginx
date: 2021-12-09 15:29:03
tags:
---

## 正向代理和反向代理

- 正向代理：即代理客户端, 服务端不知道实际发起请求的客户端。

- 反向代理：即代理服务端, 客户端不知道实际提供服务的服务端。

## 功能

- 主要功能是反向代理
- 可以实现集群、负载均以及热加载
- 静态资源虚拟化

## 命令

启动：`./nginx`

停止（强制结束所有连接）：`./nginx -s stop`

退出（等待客户端连接关闭后再停止）：`./nginx -s quit`

重新加载：`./nginx -s reload`

`-?/-h`: 帮助

`-v` : 显示版本并退出

`-V` : 显示版本和配置选项然后退出

`-t` : 测试配置并退出

`-T` : 测试配置，转储并退出

`-q` : 在配置测试期间抑制非错误消息

`-s` 信号：向主进程发送信号：停止、退出、重新打开、重新加载

`-p` 前缀：设置前缀路径（默认值：NONE）

`-e` 文件名：设置错误日志文件（默认：logs/error.log）

`-c` 文件名：设置配置文件（默认：conf/nginx.conf）

`-g` 指令：从配置文件中设置全局指令 

## 进程
- master进程：主进程，只有一个，接受指令，读取配置，并分发给worker进程，可以不需要中断服务，热加载。

    master进程读取配置，根据server配置监听端口，根据location配置映射路由。
    
- worker进程：工作进程，可以多个，连接并处理客户端请求、相应数据，和后端服务进行通讯。

    master进程根据配置fork出多个worker进程，worker进程抢占客户端请求，并且worker进程是异步非阻塞的，可以同时处理多个请求，可设置最大并发数。

## 配置文件

```
# 全局配置

# worker进程的用户
#user  nobody;

# worker进程数（推荐为CPU数量-1）
worker_processes  1;

# 日志（级别多到少：debug/info/notice/warn/error/crit）
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# 使用pid文件的路径
#pid        logs/nginx.pid;

# 事件配置
events {
    # 使用何种I/O模型(默认epoll)
    #use epoll;
    
    # worker进程的最大连接客户端数量
    worker_connections  1024;
}

# http配置
http {
    # 文件名; 导入其他配置文件（大量http的types配置从另外文件导入）
    include       mime.types;
    
    # 默认类型
    default_type  application/octet-stream;

    # 访问日志格式：log_format 格式名字 格式内容;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    # 访问日志：access_log 路径 格式;
    #access_log  logs/access.log  main;

    # 是否提升文件传输性能
    sendfile        on;
    # 是否数据包累计再发送
    #tcp_nopush     on;

    # 客户端保持连接服务器的时间（默认0）
    #keepalive_timeout  0;
    keepalive_timeout  65;

    # 是否压缩传输
    #gzip  on;
    # 限制最小压缩（小于1就不压缩）
    #gzip_min_length  1;
    # 定义压缩级别（gzip需开启，越高越压缩）
    #gzip_comp_level  3;
    # 压缩文件类型
    #gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript image/jpeg image/gif image/png application/json;

    # 配置上游服务器 upstream [名称] { } 
    #upstream name {
    #   server [IP1/域名1];
    #   server [IP2/域名2];
    
    # 上游长链接处理数量
    #   keepalive  65;
    
    # upstream指令参数：
    # weight=number权重
    # max_conns=number最大连接数，默认值0（不启用）
    # slow_start=time慢启动（仅商业版，需要配置权重，且两台以上的集群
    # down停止访问
    # backup备用机(当其他服务器停止才会使用)）
    # max_fails=number，fail_timeout=time：time时间内达到最大失败次数，nginx会把服务器剔除，再过time时间再尝试重新使用
    #}
    
    # 设置缓存保存目录
    #proxy_cache_path [path] 共享;


    # 虚拟主机配置
    server {
        # 监听端口号
        listen       80;
        
        # 服务器名：server_name 域名/IP;
        server_name  localhost;
        
        # 添加返回头字段
        # 允许跨域请求的域
        #add_header 'Access-Control-Allow-Origin' *;
        # 允许带上cookie请求
        #add_header 'Access-Control-Allow-Credentials' 'true';
        # 允许请求的方法，如 GET/POST/PUT/DELETE
        #add_header 'Access-Control-Allow-Methods' *;
        # 允许请求的header
        #add_header 'Access-Control-Allow-Headers' *;
        
        # 对源站点验证：valid_referers 验证规则;
        #valid_referers *;
        #if ($invalid_referer) {
        #    return 404;
        #}

        # 字符集
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # 路由规则配置：location 匹配规则
        # = 表示精确匹配，优先级最高
        # ^~ 表示uri以某个常规字符串开头，用于匹配url路径
        # ~ 表示区分大小写的正则匹配
        # ~* 表示不区分大小写的正则匹配
        # !~ 表示区分大小写不匹配的正则
        # !~ 表示区分大小写不匹配的正则
        # / 表示通用匹配，任何请求都会匹配到
        # @ 定义命名location区段，这些区段客户段不能访问，只可以由内部产生的请求来访问，如try_files或error_page等
        location / {
            # 根路径替换为设置值
            root   html;
            
            # 匹配的路径替换为设置值
            #alias /home/;
            
            # 首页
            index  index.html index.htm;
            
            # 浏览器缓存时间，expries [epoch/时长/@时刻/max];（时长可以为负和epoch为nocache，@22h30m即每天22点半过期，max永不过期）
            #expries 10s;
        }

        # 错误页面：error_page 错误码 页面路径;
        #error_page  404              /404.html;
        error_page   500 502 503 504  /50x.html;
        
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}

```

## Nginx模块

- Event modele：事件模块
- Phase handler：处理客户端请求和相应
- Output filter：过滤相应的内容
- Upstream：反向代理模块
- Load balancer：负载均衡器，包含负载均衡的算法
- Extend module：用于继承第三方模块

## 实际应用

### 静态资源防盗链

HTTP Referer是Header的一部分，当浏览器向Web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的，服务器借此可以获得一些信息用于处理，例如防止未经允许的网站盗链图片、文件等。因此HTTP Referer头信息是可以通过程序来伪装生成的，所以通过Referer信息防盗链并非100%可靠，但是，它能够限制大部分的盗链情况。

```
    [server/location] {
        ...
        
        # 对源站点验证：valid_referers [none/block/主机名称];
        valid_referers *;
        if ($invalid_referer) {
            return 404;
        }
    }
```
### 负载均衡

所谓的四到七层负载均衡，就是在对后台的服务器进行负载均衡时，依据四层的信息或七层的信息来决定怎么样转发流量。 比如四层的负载均衡，就是通过发布三层的IP地址（VIP），然后加四层的端口号，来决定哪些流量需要做负载均衡，对需要处理的流量进行NAT处理，转发至后台服务器，并记录下这个TCP或者UDP的流量是由哪台服务器处理的，后续这个连接的所有流量都同样转发到同一台服务器处理。七层的负载均衡，就是在四层的基础上（没有四层是绝对不可能有七层的），再考虑应用层的特征，比如同一个Web服务器的负载均衡，除了根据VIP加80端口辨别是否需要处理的流量，还可根据七层的URL、浏览器类别、语言来决定是否要进行负载均衡。

- 四层负载均衡：基于IP+端口的负载均衡，通过虚拟IP+端口接收请求，然后再分配到真实的服务器。
- 七层负载均衡：通过虚拟的URL或主机名接收请求，然后再分配到真实的服务器，七层就是基于URL等应用层信息的负载均衡。
- DNS地域负载均衡：基于地理的负载均衡。

### 提高上游吞吐量

```
upstream memcached_backend {
    server 127.0.0.1:8080;

    # 要保持的连接数（长连接）
    keepalive 32;
}

server {
    ...

    location /http/ {
        proxy_pass http://http_backend;
        # 长连接版本号（默认为1.0不是长连接）
        proxy_http_version 1.1;
        # 清楚Connection里的信息
        proxy_set_header Connection "";
        ...
    }
}
```

### 负载均衡策略

- 轮询和权重

- ip_hash 负载均衡

    ip_hash是根据用户请求过来的ip，然后映射成hash值，然后分配到一个特定的服务器里面；使用ip_hash这种负载均衡以后，可以保证用户的每一次会话都只会发送到同一台特定的Tomcat里面，它的session不会跨到其他的tomcat里面去的；

    原理：服务器index = hash(客户端IP) % 集群服务器数量

    使用方法：直接添加ip_hash;，后续同一ip的访问将只会请求同一个服务器。

    注意：一旦使用了ip_hash，当我们需要移除一台服务器的时候，不能直接删除这个配置项，而是需要在这台服务器配置后面加上关键字down，表示不可用；因为如果直接移除配置项，会导致hash算法发生更改，后续所有的请求都会发生混乱。需要配置一致性hash算法。

    ```
    upstream backend {
        ip_hash;
    
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com down;
        server backend4.example.com;
    }
    ```

- url_hash 负载均衡

    和ip_hash类似，url_hash是根据用户请求过来的url，然后映射成hash值，然后分配到一个特定的服务器里面；
    
    使用方法：添加hash $request_uri，表明了是按照url规则进行hash策略
    
    原理：服务器index = hash(客户端请求url) % 集群服务器数量
    ```
    upstream backend {
        # 使用Nginx内置变量;
        hash $request_uri;
    
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com down;
        server backend4.example.com;
    }
    ```
    
- least_conn 负载均衡
    
    按服务器连接数，新的请求到最少连接数的服务器
    
    使用方法：和以上类似，直接添加least_conn;
    
### 上游服务器缓存

proxy_cache_path和proxy_cache。proxy_cache_path配置缓存的存放地址和其他的一些常用配置，proxy_cache指令是为了启动缓存。
- **/path/to/cache** 本地路径，用来设置Nginx缓存资源的存放地址
- **levels** 默认所有缓存文件都放在同一个/path/to/cache下，但是会影响缓存的性能，因此通常会在/path/to/cache下面建立子目录用来分别存放不同的文件。假设levels=1:2，Nginx为将要缓存的资源生成的key为f4cd0fbc769e94925ec5540b6a4136d0，那么key的最后一位0，以及倒数第2-3位6d作为两级的子目录，也就是该资源最终会被缓存到/path/to/cache/0/6d目录中
- **key_zone** 在共享内存中设置一块存储区域来存放缓存的key和metadata（类似使用次数），这样nginx可以快速判断一个request是否命中或者未命中缓存，1m可以存储8000个key，10m可以存储80000个key
- **max_size** 最大cache空间，如果不指定，会使用掉所有
- **disk space**，当达到配额后，会删除最少使用的cache文件
- **inactive** 未被访问文件在缓存中保留时间，本配置中如果60分钟未被访问则不论状态是否为expired，缓存控制程序会删掉文件。inactive默认是10分钟。需要注意的是，inactive和expired配置项的含义是不同的，expired只是缓存过期，但不会被删除，inactive是删除指定时间内未被访问的缓存文件
- **use_temp_path** 如果为off，则nginx会将缓存文件直接写入指定的cache文件中，而不是使用temp_path存储，official建议为off，避免文件在不同文件系统中不必要的拷贝
- **proxy_cache** 启用proxy cache，并指定key_zone。另外，如果proxy_cache off表示关闭掉缓存。

```
proxy_cache_path /path/to/cache levels=1:2 keys_zone=mycache:10m max_size=10g inactive=60m use_temp_path=off;
server {
　　# ...
　　location / {
　　    # 开启并使用缓存
　　　　proxy_cache mycache;
　　    # 针对200和304状态码的缓存设置过期时间
　　    proxy_cache_valid 200 304 8h;
　　    
　　　　proxy_pass http://my_upstream;
　　}
}
```

### 使用配置SSL证书提供HTTPS访问协议

1. 上传证书和密钥
1. 安装Nginx SSL模块
1. 修改配置文件
    ```
    http {
        include       mime.types;
        default_type  application/octet-stream;
        sendfile        on;
        keepalive_timeout  65;
        
        server {
            # 监听443端口
            listen 443;
            
            # 域名
            server_name huiblog.top;
            
            # 开启ssl
            ssl on;
            
            # ssl证书的pem文件路径
            ssl_certificate  /root/card/huiblog.top.pem;
            
            # ssl证书的key文件路径
            ssl_certificate_key /root/card/huiblog.top.key;
            
            # ssl会话cache?
            ssl_session_cache shared:SSL:1m;
            
            # ssl会话超时时间?
            ssl_session_timeout 5m;
            
            # 配置加密协议
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            
            # 配置加密套件
            # 加密套件解读：
            #    ECDHE-RSA-AES128-GCM-SHA256 为例
            #    
            #    ECDHE：秘钥交换算法
            #    RSA：签名算法
            #    AES128：对称加密算法
            #    GCM-SHA256：签名算法
            ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS;
            ssl_prefer_server_ciphers on;
            
            location / {
                proxy_pass  http://公网地址:项目端口号;
            }
        }

        server {
            listen 80;
            server_name huiblog.top;
            
            # 将请求转成https
            rewrite ^(.*)$ https://$host$1 permanent;
        }
    }
    ```

## 问题解决

### nginx.pid 打开失败及无效的解决办法

    重新创建nginx.pid路径，重新设置nginx配置文件。
    
    `./nginx -c [filename]`