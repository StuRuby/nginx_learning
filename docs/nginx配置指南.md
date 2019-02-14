# nginx配置指南

[返回首页](https://sturuby.github.io/nginx_learning/)

本配置只用于解释每个参数的功能，直接使用可能会报错！

## nginx全局配置参数
``` nginx
# 使用这个参数来配置worker进程的用户和组，如果忽略group,那么group的名字等于该参数指定的用户的用户组
user  nobody;

#指定worker进程启动的数量，这些进程用于处理客户的所有连接。选择一个正确的数量取决于服务器环境、磁盘子系统和网络基础设施。
#一个好的经验法则是设置该参数的值与CPU绑定的负载处理器核心的数量相同，并用1.5~2之间的数乘以这个数作为I/O密集型负载.
worker_processes  1; 

# error_log是所有错误写入的文件。如果在其他区段中没有设置其他的error_log,那么这个日志文件将会记录所有的错误。
# 该指令的第二个参数指定了被记录错误的级别（debug、info、notice、warn、error、crit、alert、emerg）。
# 注意，debug级别的错误只有在编译时配置了--with-debug选项才可以使用.
error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;


# 设置记录主进程ID的文件，这个配置将会覆盖编译时的默认配置
pid        logs/nginx.pid;


events {
    # 该指令用于指示使用什么样的连接方法。这个配置将会覆盖编译时的默认配置，如果使用该指令，那么需要一个events区段。
    # 通常不需要覆盖，除非是当编译时的默认值随着时间的推移产生错误时才需要被覆盖设置。
    use /dev/poll;

    # 该指令配置一个工作进程能够接受并发连接的最大数。这个连接包括客户连接和向上游服务器的连接，但并不限于此。
    # 这对于反向代理服务器尤为重要，为了达到这个并发性连接数量，需要在操作系统层面进行一些额外调整。
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    proxy_cache_path /Users/k/Softwares/nginx/cache 
                     levels=1:2
                     keys_zone=my_cache:10m
                     max_size=10g
                     inactive=60m
                     use_temp_path=off;    

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;
    gzip_min_length 1;
    gzip_comp_level 2;
    gzip_types text/plain application/javascript  application/x-javascript text/css application/xml  text/javascript application/x-httpd-php image/jpeg image/gif image/png;
  
    include       conf.d/*.conf;

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


## 使用include文件
在nginx配置文件中，`include`文件可以在任何地方，以便增强配置文件的可读性，并且能够使得部分配置文件重新使用。使用`include`文件，要确保被包含的文件自身有正确的Nginx语法，即配置指令和块(blocks),然后指定这些文件的路径

``` nginx
include /opt/local/etc/nginx/mime.types;
```

在路径中也可以使用通配符，来表示可以配置多个文件。
``` nginx
include /opt/local/etc/nginx/vhost/*.conf;
```

如果没有给定全路径，那么Nginx将会依据它的主配置文件路径进行搜索。Nginx测试配置文件很容易，通过下面的命令来完成。

``` shell
nginx -t -c <path_to_nginx.conf>
```
该命令将测试Nginx的配置文件，包括`include`文件，但是它只检查语法错误。

## HTTP的server部分

HTTP的server部分控制了HTTP模块的方方面面，是使用最多的一个部分。该部分的配置主要用于处理HTTP连接，因此该模块提供了相当数量的指令。下面我们来具体看一下这部分的内容。

### 客户端指令
 
http 客户端指令

|http客户端指令|说明|
|--|--|
|chunked_transfer_encoding|在发送给客户端的响应中,该指令允许禁用 http/1.1 标准的块传输编码|
|client_body_buffer_size|为了阻止临时文件写到磁盘， 可以通过该指令为客户端请求 体设置缓存大小 ， 默认的缓存大小为两个内存页面|
|client_body_ in_ file一only|用于调试或者是进一步处理客户端请求体。 该指令设置为 “ on ”能够将客户端请求体强制写入到磁盘文件|
|client_body_in_single_buffer|为了减少复制的操作， 使用该指令强制 Ng i nx 将整个客户端 请求体保存在单个缓存中|
|client_body_temp_path|该指令定义一个命令路径用于保存客户端请求体|
|client_body_timeout|该指令指定客户体成功读取的两个操作之间的时间间隔|
|client header buffer size|该指令为客户端请求头指定 一个缓存大 小 ，当请求头大于 lKB 时会用到这个设置|
|client header timeout|该超时是读取整个客户端头的时间长度|
|client_max body_size|该指令定义允许最大的客户端请求头， 如果大于这个设置， 那么 客户端将会是 413 (Request Entity Too Large ）错误|
|keepalive_disable|该指令对某些类型的客户端禁用 keep-alive 请求功能|
|keepalive_requests|该指令定义在一个 keep-alive 关闭之前可以接受多少个请求|
|keepalive_timeout|该指令指定 keep-alive 连接持续多久 。 第二个参数也可以 设置， 用于在响应头中设置“ keepalive ”头|
|large_client_header_buffers|该指令定义最大数量和最大客户端请求头的大小|
|msie_padding|为了填充响应的大小至 5 1 2 字节， 对于 MSIE 客户端， 大于 400 的状态代码会被添加注释以便满足 5 1 2 字节， 通过启用该 命令可以阻止这种行为|
|msie refresh|对于 MSIE 客户端， 该指令可启用发送一个 refresh头，而不是redirect|


**文件I/O指令**

这些指令用于控制 Nginx 如何投递静态文件以及如何管理文件描述符。

|http文件I/O指令|说明|
|---|---|
|aio|该指令启用异步文件I/O。该指令对于现代版本的 FreeBSD 和所有 Linux 发行版都有效。 在 FreeBSD 系统下， aio 可能被用于 sendfile 预加载数据。 在 Linux 下，则需要 directio 指令，自动禁用 sendfile|
|directio|该指令用于启用操作系统特定的标志或者功能提供大于给定参数的文件。 在 Linux 系统下， 使用aio时需要使用该指令|
|directio_alignment|该指令设置 directio 的算法。 默认值为 5 1 2 ，通常足够了，但是在 Linux 的XFS 下推荐增加为4KB|
|open_file_cache|该指令配置一个缓存用于存储打开的文件描述符、目录查询和文件查询错误|
|open_file_cache_errors|该指令按照 open_file_cache ，启用文件查询错误缓存|
|open_file_cache_min_uses|open_file_cache 缓存的文件描述符保留在缓存中， 使用该指令配置最少使用文件描述符的次数|
|open_file_cache_valid|该指令指定对 open_file_cache 缓存有效性检查的时间间隔|
|postpone_output|该指令指定 Nginx发送给客户端最小的数值,如果可能的话,没有数据会发送,直到达到此值|
|read ahead|如果可能的话， 内核将预读文件到设定的参数大小。目前支持FreeBSD 和 Linux (Linux 会忽略大小）|
|sendfile|该指令使用sendfile(2)直接复制数据从一个到另一个文件描述符|
|sendfile_max_chunk|该指令设置在一个sendfile(2)中复制最大数据的大小，这是为了阻止worker“贪婪”|


**Hash指令**

Hash指令控制Nginx分配给某些变量多大的静态内存，在启动和重新配置时，Nginx会计算需要的最小值。

|HTTP Hash指令|说明|
|--|--|
|server_names_hash_bucket_size|该指令指定用于保存 server_name 散列表大小的“桶”|
|server_names_hash_max_size|该指令指定 server_name 散列表的最大大小|
|types_hash_bucket_size|该指令指定用于存储散列表的“桶”的大小|
|types_hash_max_size|该指令指定散列类型表的最大大小|
|variables_hash_bucket_size|该指令指定用于存储保留变量“桶 ”的大小|
|variables_hash_max_size|该指令指定存储保留变量最大散列 值的大小|



**Socket指令**

Socket指令描述了Nginx如何设置创建TCP套接字的变量选项

|HTTP Socket指令|说明|
|--|--|
|lingering_close|该指令指定如何保持客户端的连接 ，以便用于更多数据的传输|
|lingering_ time|在使用lingering_close 指令的连接中， 该指令指定客户端连接为了处理更多的数据需要保持打开连接的时间|
|lingering_timeout|结合lingering_close ，该指令显示Nginx在关闭客户端连接之前,为获得更多数据会等待多久|
|reset_timedout_connection|使用这个指令之后,超时的连接将会被立即关闭,释放相关的内存。默认的状态是处于FIN_WAITl,这种状态将会一直保持连接|
|send_lowat|如果非零,Nginx将会在客户端套接字尝试减少发送操作|
|send_timeout|该指令在两次成功的客户端接收响应的写操作之间设置－个超时时间|
|tcp_nodelay|启用或者禁用TCP_NODELAY选项,用于keep-alive连接|
|tcp_nopush|仅依赖于sendfile 的使用。它能够使得Nginx在－个数据包中尝试发送响应头以及在数据包中发送一个完整的文件.|


## 示例配置文件

这些配置在`nginx.conf`中，会跟随在全局配置指令之后。
``` nginx
http {
    include /opt/local/etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    server_names_hash_max_size 1024;
}
```


