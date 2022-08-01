### NGINX 简介

NGINX 是一个支持静态文件的 WEB 服务器，并发支持高。

常用作文件服务器和前端服务器的负载均衡。

有一个主进程和多个工作进程，主进程读取配置文件和维护工作进程，工作进程处理请求。

#### 配置文件

默认示例

```shell
# 用户名
user  nginx;
# 工作线程数量
worker_processes  1;
# 日志位置
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

# 
events {
    worker_connections  1024;
}

# 处理HTTP请求
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```



