# Nginx 作为反向代理

[返回首页](https://sturuby.github.io/nginx_learning/)

反向代理时一个web服务器，它终结了客户端连接，并且生成了另一个新连接，新连接代表客户端向上游服务器生成连接。

下面，我们将主要介绍以下内容
- 反向代理介绍
- 上游服务器类型
- 负载均衡
- 转换“if”配置为更现代的处理
- 使用错误文档处理上游问题
- 确定客户端真实的IP地址


## 反向代理简介

Nginx在作为一个反向代理使用的过程中，可以根据请求的uri、客户机参数或者一些其他的逻辑进行拆分，更好的响应客户端请求。

### proxy_pass

`proxy_pass`指令有一个参数，url请求将会被转换，带有uri部分的proxy_pass指令将会使用该uri替代`request_uri`部分。例如：

``` nginx
location /uri {
    proxy_pass http://localhost:8080/newuri;
}
```

然而，这个规则有两个例外的情况。第一，如果location定义了一个正则表达式，那么uri部分将不会发生转换。这种情况下,uri `/local`将会直接传递到上游服务器，而不会如期转换为`/foreign`
。

``` nginx
location ~ ^/local {
    proxy_pass http://localhost:8080/foreign;
}
```

第二种情况，如果在location内哟rewrite规则改变了uri，那么Nginx使用这个uri处理请求，不在发生转换。如下所示，uri传递到上游服务器的将会是`/index.php?page=<match>`，而不是预期的`/index`。

``` nginx
location / {
    rewrite /(.*)$ /index.php?page=$1 break;
}
```

### 代理模块

代理模块常用指令可以参考 Nginx官网 [Module ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)

配置文件`proxy.conf`可以参考以下配置：

``` nginx
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size 10m;
client_body_buffer_size 128k;
proxy_connect_timeout 15;
proxy_send_timeout 15;
proxy_read_timeout 15;
proxy_send_lowat 12000;
proxy_buffer_size 4k;
proxy_buffers 32 4k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;

```

我们设置了一些常用指令的值
- 大多数情况下没有必要重写`location`头，将`proxy_redirect`指令设置为off;
- 因为设置了Host头，所以上游服务器能够将请求映射到一个虚拟服务器，否则就使用用户输入的URL中的主机部分。
- `X-Real-IP`头和`X-Forwarded-For`头有相似的目的，都用于转发连接客户端IP地址到上游服务器得到信息。
- `$remote_addr`变量在`X-Real-IP`头内使用，就是Nginx接受客户端请求的IP地址。
- `$proxy_add_x_forwarded_for`变量包含在`X-Forwarded-For`头中，它来源于客户端请求，跟随者`$remote_addr`变量。
- `client_max_body_size`指令，不是严格的代理模块指令，但是它与代理配置也有很大的关系，如果这个值设置的太低，将不能上传文件到上游服务器。
- 在建立与上游服务器初始连接的过程中，`proxy_connect_timeout`指令表明了Nginx将会等待的时间长度。
- `proxy_read_timeout`和`proxy_send_timeout`指令定义了Nginx同上游服务器连接成功的两次操作等待的时间。
- `proxy_send_lowat`指令只在FreeBSD系统下有效，并且在该协议下传输数据之前指定套接字发送缓冲应该容纳的字节数。
- `proxy_buffer_size`、`proxy_buffers`和`proxy_busy_buffers_size`指令控制了Nginx如何快速的响应用户的请求。
- `proxy_temp_file_write_size`指令控制worker进程阻塞后台数据的时间，值越大，处理阻塞的时间越长。

这些指令可以包含在一个文件中`proxy.conf`，然后可以在同一个配置文件中多次使用。

``` nginx
location / {
    include proxy.conf;
    proxy_pass http://localhost:8080;
}
```
如果这些指令中存在一个不同于include文件中的值，也可以通过在`location`部分明确设置覆盖掉原有的值。
``` nginx
location /uploads {
    include proxy.conf;
    client_max_body_size 500m;
    proxy_connect_timeout 75;
    proxy_send_timeout 90;
    proxy_read_timeout 90;
    proxy_pass http://localhost:8080;
}
```

### 带有cookie的遗留应用程序

这里演示一下如何重写cookie的域和路径，以便匹配新的应用端点。

``` nginx
server {
    server_name app.example.com;
    location /url1 {
        proxy_cookie_domain url1.example.com app.example.com;
        proxy_cookie_path $uri /url1$uri;
        proxy_redirect default;
        proxy_pass http://url1.example.com/;
    } 
    location /url2 {
        proxy_cookie_domain url2.example.org app.example.com;
        proxy_cookie_path $uri /url2$uri;
        proxy_redirect default;
        proxy_pass http://url2.example.org/;
    }
    location / {
        proxy_pass http://localhost:8080;
    }
}

```

### upstream模块

与proxy模块紧密搭配的是upstream模块。upstream模块将会启用一个新的配置区段，在该区段定义了一组上游服务器，这些服务器可能被设置了不同的权重，也可能是不同的类型，也可能出于需要对服务器进行维护，故而标记为down.

这里列一下upstream区段中的有效指令

|upstream模块指令|说明|
|--|--|
|ip_hash|该指令通过IP地址的哈希值确保客户端均匀的连接所有服务器，键值基于C类地址|
|keepalive|该指令指定每一个worker进程缓存到上游服务器的连接数。在使用HTTP连接时，proxy_http_version应该设置为1.1，并且将proxy_set_header设置为Connection ""|
|least_conn|该指令激活负载均衡算法，将请求发送到活跃连接数最少的那台服务器|
|server|该指令为upstream定义一个服务器地址（带有TCP端口号的域名、IP地址，或者是Unix域套接字）和可选参数。

`server`参数如下。

- weight ：该参数设置一个服务器的优先级优于其他服务器
- max_fails:该参数设置在fail_timeout时间之内尝试对一个服务器连接的最大次数，如果超过这个次数，那么就会被标记为down。
- fail_timeout:在这个指定的时间内服务器必须提供响应，如果在这个时间内没有收到响应，那么服务器将会被标记为down状态。
- backup ： 一旦其他服务器宕机，那么仅有该参数标记的机器才会接收请求。
- down：该参数标记为一个服务器不再接受任何请求。


### 保持活动连接

Nginx服务器将会为每一个worker进程保持同上游服务器的连接。在Nginx需要同上游服务器持续保持一定数量的打开连接时，连接缓存非常有用。如果上游服务器通过HTTP进行“对话”，那么Nginx将会使用HTTP/1.1协议的持久连接机制维护这些打开的连接。

配置示例如下：

``` nginx
upstream apache {
    server 127.0.0.1:8080;
    keepalive 32; 
}

location / {
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_pass http://apache;
}

```

这种机制也可以被用于代理非HTTP连接中，下面演示一下Nginx与两个memcached实例保持64个连接。

``` nginx
upstream memcaches {
    server 10.0.100.10:11211;
    server 10.0.100.20:11211;
    keepalive 64;
}
```

如果从默认轮询负载均衡算法切换为ip_hash或者least_conn，那么我们需要在keepalive之前指定负载均衡算法。

``` nginx
upstream apaches {
    least_conn;
    server 10.0.200.10:80;
    server 10.0.200.20:80;
    keepalive 32;
}
```

### 上游服务器的类型

上游服务器是Nginx代理连接的一个服务器，他可以是不同的物理机器，也可以是虚拟机，但是并不是必须如此。

### 单个上游服务器

这里以单个apache服务器为例，假设Apache已配置为在localhost:8080上监听。下面是一个最简单的配置。
``` nginx
server {
    location / {
        proxy_pass http://localhost:8080;
    }
}
```

这个最简单的配置,Nginx将会终止所有的客户端连接，然后将代理所有请求到本地主机的TCP协议的8080端口上。一般情况下，这种配置还需要进行再次扩展，以便让Nginx直接提供任何静态文件，然后代理服务器将剩余的请求发送到Apache。

``` nginx
server {
    location / {
        try_files $uri @apache;
    }

    location @apache {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

### 多个上游服务器

有时候我们也可以需要使用Nginx代理请求到多个上游服务器，这可以通过upstream来声明，定义多个server可以参考upstream中的proxy_pass指令。

``` nginx
upstream app {
    server 127.0.0.1:9000;
    server 127.0.0.1:9001;
    server 127.0.0.1:9002;
}

server {
    location / {
        proxy_pass http://app;
    }
}
```
使用这个配置，Nginx将会通过轮询的方式将连续的请求传递给3个上游服务器。在一个应用程序仅处理一个请求时，这个配置很有用。


## 负载均衡

Nginx在作为反向代理的同时，还提供了负载均衡器的功能，他提供了三种不同的负载均衡算法，你可以依据项目的使用环境，选择其中的一种。

- 轮询（round-robin）：轮询算法是基于在队列中谁是下一个的原理确保将访问量均匀的分配给每一个上游服务器的。
- IP哈希（IP hash）：Nginx通过IPV4地址的前三个字节或者整个IPV6地址作为哈希键来实现，同一个IP地址池地址总是被映射到同一个上游服务器，所以这个机制的目的不是要确保公平分配给每一台上游服务器，而是在客户端和上游服务器之间实现一致映射。
- 最少连接数 （Least Connection）：该算法的目的是通过选择一个活跃的最少连接数服务器，然后将负载均匀分配给上游服务器，如果上游服务器的处理能力不相同，那么可以为server指令使用weight参数来指示。


