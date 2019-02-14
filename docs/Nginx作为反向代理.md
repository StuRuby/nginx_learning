# Nginx 作为反向代理

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
