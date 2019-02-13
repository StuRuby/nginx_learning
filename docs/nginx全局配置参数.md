# nginx全局配置参数
本配置只用于解释每个参数的功能，直接使用可能会报错！
``` nginx
# 使用这个参数来配置worker进程的用户和组，如果忽略group,那么group的名字等于该参数指定的用户的用户组
user  nobody;

# 指定worker进程启动的数量，这些进程用于处理客户的所有连接。选择一个正确的数量取决于服务器环境、磁盘子系统和网络基础设施。一个好的经验法则是设置该参数的值与CPU绑定的负载处理器核心的数量相同，并用1.5~2之间的数乘以这个数作为I/O密集型负载.
worker_processes  1; 

#error_log是所有错误写入的文件。如果在其他区段中没有设置其他的error_log,那么这个日志文件将会记录所有的错误。该指令的第二个参数指定了被记录错误的级别（debug、info、notice、warn、error、crit、alert、emerg）.注意，debug级别的错误只有在编译时配置了--with-debug选项才可以使用.
error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;


# 设置记录主进程ID的文件，这个配置将会覆盖编译时的默认配置
pid        logs/nginx.pid;


events {
    #该指令用于指示使用什么样的连接方法。这个配置将会覆盖编译时的默认配置，如果使用该指令，那么需要一个events区段。通常不需要覆盖，除非是当编译时的默认值随着时间的推移产生错误时才需要被覆盖设置。
    use /dev/poll

    #该指令配置一个工作进程能够接受并发连接的最大数。这个连接包括客户连接和向上游服务器的连接，但并不限于此。这对于反向代理服务器尤为重要，为了达到这个并发性连接数量，需要在操作系统层面进行一些额外调整。
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
