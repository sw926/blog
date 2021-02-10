---
title: vps安装Ngrok和Nginx
date: 2019-02-20 10:54:43
category: Other
tags: 
    - Ngrok
    - Nginx
---

使用的是 vultr 的 vps，系统是 Ubuntu，ssh 登录直接是 root 用户，根据这篇文章安装 Ngrok
<https://blog.csdn.net/yjc_1111/article/details/79353718>

## 准备工作

首先安装 git 和 go:

``` sh
apt-get install build-essential golang mercurial git
```

载源码，当然也可以不安装git，但是需要手动上传代码到需要的位置。
此处使用非官方地址，修复了部分包无法获取（摘自网络）：

``` sh
git clone https://github.com/tutumcloud/ngrok.git ngrok
```

代码在 `/root/ngrok` 目录。

## 编译

需要一个域名，比如 sw926.com，

``` sh
cd ngrok

NGROK_DOMAIN="sw926.com"

openssl genrsa -out base.key 2048

openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem

openssl genrsa -out server.key 2048

openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr

openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt
```

执行完替换证书：

``` sh
cp base.pem assets/client/tls/ngrokroot.crt
```

编译：

``` sh
make release-server release-client
```

## 启动服务端

``` sh
./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain="sw926.com" -httpAddr=":8000" -httpsAddr=":4443"
```

## 客户的

编译客户端：

``` sh
GOOS=darwin GOARCH=amd64 make release-client
```

将编译好的客户端下载到本地：

``` sh
scp root@{vps地址}:/root/ngrok/bin/darwin_amd64/ngrok ./
```

我们可以把这个文件放到 bin 目录，之后需要新建一个 ngrok.conf 文件，输入以下内容

```
server_addr: "sw926.com:4443"
trust_host_root_certs: false
```

然后启动客户端：

``` sh
ngrok -config=./ngrok.conf -subdomain="ngrok" localhost:3000
```

如果配置成功，会看的以下内容：

``` sh
Tunnel Status                 online
Version                       1.7/1.7
Forwarding                    http://ngrok.sw926.com:8000 -> localhost:3000
Forwarding                    https://ngrok.sw926.com:8000 -> localhost:3000
Web Interface                 127.0.0.1:4040
# Conn                        0
Avg Conn Time                 0.00ms
```

`127.0.0.1:4040` 是本地管理的界面，`localhost:3000` 是本地启动的服务，我们访问 `ngrok.sw926.com:8000`，其实就是访问的 `localhost:3000`

## ngrok 在服务的开机启动

<https://zhuanlan.zhihu.com/p/25167530>

新建一个启动脚本，目录为 `/root/ngrok.sh`，输入以下内容

``` sh
cd /root/ngrok
nohup ./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain="sw926.com" -httpAddr=":8000" -httpsAddr=":443"
exit 0
```

在 `/etc/init.d` 下新建一个 `ngrok` 文件，输入以下内容：

``` 
#!/bin/sh
#chkconfig:2345 70 30
#description:ngrok

ngrok_path=/root
case "$1" in
    start)
        echo "start ngrok service.."
        sh ${ngrok_path}/ngrok.sh
        ;;
    *)
    exit 1
    ;;
esac
```

赋予权限：

``` sh
sudo chmod 755 /etc/init.d/ngrok
```

然后注册服务：

``` sh
cd /etc/init.d
sudo update-rc.d ngrok defaults
sudo update-rc.d start 70 2 3 4 5
```

启动服务：

```
sudo service ngrok start
```

## 使用 nginx 转发

安装：

``` sh
sudo apt-get install nginx
```

启动：

``` sh
sudo /etc/init.d/nginx start
```

访问 vps，就会看到 niginx 的 welcome 页面

## 配置转发

<http://www.wxapp-union.com/portal.php?mod=view&aid=1148>
<https://segmentfault.com/q/1010000004277269>

在服务器中新建/etc/nginx/conf.d/ngrock.sw926.com.conf文件，添加以下内容：

```
server {    
    listen  80;  
    server_name *.ngrok.up2m.win;  
 
    ## send request back to apache ##
    location / {
        proxy_pass  http://127.0.0.1:8000;
        #Proxy Settings
        proxy_redirect     off;
        #proxy_set_header Host downloads.openwrt.org;
        proxy_set_header   Host             $host:8000;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_max_temp_file_size 0;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
   }
}
```

重载 nginx：

``` sh
nginx -s reload  
```

## 客户端快捷方式

我使用的是 zsh，可以在 `~/.zshenv` 文件添加一个函数：

``` sh
function ngrok_start
{
    ngrok -config=/Users/xxx/.config/ngrok/ngrok.conf -subdomain="$1" $2
} 
```

启动的时候就简单一些了：

``` sh
ngrok_start web localhost:3000
```