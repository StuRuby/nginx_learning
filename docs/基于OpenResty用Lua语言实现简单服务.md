# 基于OpenResty用Lua语言实现简单服务
`OpenResty`是一款基于nginx和LuaJIT的web平台,可以在[OpenResty官网](https://openresty.org/cn/)详细了解一下。

## 下载OpenResty

我们这里以macos为例,为了便于后期定制`OpenResty`的模块，我们这里使用编译的方式进行安装。

安装前的准备，安装`pcre`,`openssl`。

``` shell 
brew install pcre openssl
```
安装好`pcre`和`openssl`之后，即可进行`OpenResty`的安装.

``` shell
wget https://openresty.org/download/openresty-1.13.6.2.tar.gz

tar -xzvf openresty-1.13.6.2.tar.gz

cd openresty-1.13.6.2

./configure \
   --with-cc-opt="-I/usr/local/opt/openssl/include/ -I/usr/local/opt/pcre/include/" \
   --with-ld-opt="-L/usr/local/opt/openssl/lib/ -L/usr/local/opt/pcre/lib/" \
   -j8


make 

sudo make install 


```

# Hello World

## 创建项目目录

``` shell 
mkdir ~/helloWorld

cd ~/helloWorld

mkdir logs/ conf/
```
logs目录用于放置log文件,conf目录用于放置我们的配置文件


## 创建`nginx.conf`配置文件

创建`conf/nginx.conf`文件，然后写入以下内容

``` nginx
worker_processes 1;
error_log logs/error.log;
events {
    worker_connections 1024;
}

http {
    server {
        listen 8099;
        location /lua {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
            ';
        }

        location / {
            root /Users/k/Sites/arcgis_js_api/sdk;
        }
    }
}

```

## 启动nginx服务

首先，设置`environment path`

``` shell 
PATH=/usr/local/openresty/nginx/sbin:$PATH
export PATH
```

然后启动nginx 服务

``` shell 
/usr/local/openresty/nginx/sbin/nginx -p `pwd`/ -c conf/nginx.conf

```

## 测试服务

使用`curl`访问我们的web服务

``` shell 
curl  http://localhost:8099/lua
```
如果正常的话，会返回 <p>hello,World</p>

当然，我们也可以在浏览器中访问 http://localhost:8099/lua 进行测试



