# 用nginx搭建一个具备缓存功能的反向代理服务

[返回首页](https://sturuby.github.io/nginx_learning/)

在我们的开发过程中，由于上游服务要处理非常复杂的业务逻辑，而且需要考虑开发效率，所以他的性能有时并不会太好，我们这里会演示如何使用`nginx`作为反向代理，由一台nginx根据负载均衡算法代理给多台上游服务，这样就实现了水平扩展，在用户无感知的情况下，我们添加更多的上游服务器，来提升我们的处理性能。
当上游服务器出现问题时，nginx可以把请求从有问题，出现灾难的服务器转交给正常的服务器。

我们这里使用2台nginx服务器作为上游服务器，一台nginx作为反向代理服务器。

上游服务器配置

上游服务器A

``` nginx
server {
    listen 127.0.0.1:8080;
    # the same config
}

```

上游服务器B

``` nginx
server {
    listen 127.0.0.1:8081;
    # the same config
}

```

反向代理nginx的配置文件

``` nginx 

upstream local {
    server 127.0.0.1:8080;
    serevr 127.0.0.1:8081;
}

server {
    server_name nginx_proxy_demo;
    listen 80;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://local;
    }
}

```

反向代理设置的所有配置都可以在[Module ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)中找到。

这里有一个很重要的特性，叫做`proxy_cache`。因为，当我们的nginx作为反向代理时，通常只有一些动态的请求，也就是说不同用户访问同一个url，返回的结果是不同时，才会交由nginx上游服务器去做处理。但是有一些内容，可能是一段时间不会发生变化的，为了减轻上游服务的压力，nginx会将上游服务的内容，缓存一段时间，因为nginx的性能远远大于上游服务器的性能，所以在使用了nginx缓存功能之后，对于一些小的站点，可能会带来非常大的性能提升。

下面演示一下如何配置缓存服务器

``` nginx
http {
    proxy_cache_path <your_cache_file_path>  levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off
}

server {
    location / {
        #proxy_settings
        proxy_cache my_cache;
        proxy_cache_key $host$uri$is_args$args;
        proxy_cache_valid 200 304 302 1d;
        proxy_pass  http://local;       
    }
}

```

