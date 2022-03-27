# Nginx

> nginx 推荐使用 `yum` 在宿主机直接安装，不建议 Docker 安装。

## nginx 安装

[Nginx 官方文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

```shell
vim /etc/yum.repos.d/nginx.repo

# 添加以下内容
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1

# 安装 nginx
yum install -y nginx
```

* 静态文件目录：/usr/share/nginx/html
* 配置文件目录：/etc/nginx
* 日志文件目录：/var/log/nginx

## nginx 代理

> Nginx 代理建议按项目划分，一个项目一个文件，不要全部放在一起。

配置文件目录： `/etc/nginx/conf.d/*.conf`, 下面给出一个[样例配置模板](./example.conf)：

```conf
upstream example {
    server 172.16.211.155:16600;
    server 172.16.211.156:16600;
    server 172.16.211.153:16600;
}

# 普通反向代理
server {
    listen       80;
    server_name  example.shiduai.com;
    
    # 307 可以转发 POST 请求，301 不可以
    return 307 https://example.shiduai.com$request_uri;
    # rewrite ^(.*)$  https://$host$1 permanent;
}

# HTTPS 反向代理
server {
    listen       443 ssl;
    server_name  example.shiduai.com;
    ssl_certificate   /home/cert/5257519__shiduai.com.pem;
    ssl_certificate_key  /home/cert/5257519__shiduai.com.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    
    # 需要自定义专属的请求日志和错误日志
    access_log  /var/log/nginx/example/access.log;
    error_log /var/log/nginx/example/error.log;

    location /example/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://example/example/;    
    }
}

# 配置内网服务器访问
server {
    listen       80;
    server_name  example-inner.shiduai.com;

    # 需要自定义专属的请求日志和错误日志
    access_log  /var/log/nginx/example/access.log;
    error_log /var/log/nginx/example/error.log;

    location /example/ {
        proxy_pass http://example/example/;
    }
}
```

## nginx stream tcp 模块

```shell
# 安装 stream 模块
yum install -y nginx-mod-stream
```

添加 stream 目录

```shell
# 创建 tcp 代理配置目录
mkdir -p /etc/nginx/tcp.d

# 编辑 nginx 主配置文件
vim /etc/nginx/nginx.conf

# 在文件最后追加如下内容

# stream config.
stream {
    # tcp/ip proxy
    include /etc/nginx/tcp.d/*.conf;
}
```

在 tcp.d 目录下创建一个 [mysql.conf](./mysql.conf) 配置文件，用来代理 mysql。

```shell
upstream mysql-server {
    # localhost  可修改为对应的 IP 地址
    # 3306 可修改为对应的数据库端口
    # weight 权重
    server localhost:3306 weight=1 max_fails=3 fail_timeout=30s;
}

server {
    # 监听的端口
    listen 3306;
    proxy_connect_timeout 5s;
    proxy_timeout 30s;
    proxy_pass mysql-server;
}
```