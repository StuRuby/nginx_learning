# 用nginx搭建一个可用的静态资源web服务器

[返回首页](https://sturuby.github.io/nginx_learning/)

这里使用`arcgis js api sdk`作为静态资源服务器的资源。[下载地址](https://developers.arcgis.com/downloads/apis-and-sdks?product=javascript)

## 配置nginx

``` nginx

server {
    listen 8082;
    server_name  localhost;

    location / {
      root  <your_arcgis_js_api_sdk_path>;
      # alias <your_alias_path>
      index index.html index.htm;
    }

    error_page  500 502 503 504 /50x.html;

    location = /50x.html {
        root html;
    }
}

```

## 设置gzip压缩

``` nginx
http {
    # gzip配置
    gzip on;
    gzip_min_length 1k; #小于阈值指定大小的文件即不进行压缩
    gzip_comp_level 2; # 压缩级别
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
}

```

重载nginx

``` nginx
sudo nginx -s reload
```

使用gzip之后，我们的静态资源web服务的传输效率会提升很多。

## 使用autoindex模块展示本地文件路径

`autoindex`的官网文档[Module ngx_http_autoindex_module](https://nginx.org/en/docs/http/ngx_http_autoindex_module.html)

`autoindex`的功能很简单，开启后当访问以`/`结尾的地址时展示当前文件目录。配置方式如下：
``` nginx
location / {
    autoindex on;
}
```

## nginx 其余配置

### 配置响应访问速度
`set $limit_rate 1k` 设置响应访问速度，每秒传输速率，用于当公网带宽有限时，限制大文件访问速度，给`js`,'css'等小文件访问机会。
[Module ngx_stream_core_module](https://nginx.org/en/docs/http/ngx_http_core_module.html) > Embedded Variables >`limit_rate`


### 记录nginx访问日志

nginx配置

``` nginx

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
}

server {
    access_log <your_log_file_path>  main #<your_log_name>
}

```
此处日志命名为`main`。命名的原因主要为**我们可能会对不同域名下作不同格式的日志记录，或者不同url作反向代理时作不同的记录。不同用途记录不
同的日志格式**



