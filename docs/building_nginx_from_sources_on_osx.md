# macos 源码编译nginx

## 安装依赖的包
- Openssl
- Pcre

Openssl macos默认会自带

Pcre是一个perl库，可以用于提供nginx配置文件中地址正则表达式功能

``` shell
brew search pcre
brew install pcre

```

## 源码编译nginx

### 1.获取nginx

打开[nginx官网]('https://nginx.org/en/download.html')
上面有几栏选项。
- Mainline 主线版本
- Stable 稳定版本
- Legacy 传统版本

我们这里选择稳定版本，获取nginx下载地址。定位到对应的文件夹进行下载.
``` shell 
wget https://nginx.org/download/nginx-1.14.2.tar.gz

```
解压安装包

``` shell 
tar -xzf nginx-1.14.2.tar.gz
```

解压完成后删除压缩包
``` shell 
rm -rf nginx-1.14.2.tar.gz
```

进入目录
``` shell 
cd nginx-1.14.2
```
安装 nginx

``` shell 
./configure --prefix=/<your_nginx_install_path>
make 
make install 
```
启动 nginx

```
cd <your_nginx_install_path>
sudo sbin/nginx

```
如果遇到因80端口被占用而导致无法启动的情况，一般是因为mac默认启动了apache，关掉即可

``` shell
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist

```
关闭 nginx
``` shell
sudo sbin/nginx -s reload
```





