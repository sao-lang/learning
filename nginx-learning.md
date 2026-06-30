# Nginx 核心学习指南
## 1. Nginx 的核心优点
 * **高并发、高性能：** 采用异步非阻塞的事件驱动模型（epoll），单机能轻松支撑数万甚至十万级别的并发连接。
 * **内存消耗极小：** 在处理大量并发时，内存占用非常低（约 10000 个非活跃连接仅消耗 2.5MB 内存）。
 * **高可靠性：** 采用 Master-Worker 多进程架构。一个 Master 管理多个 Worker，Worker 挂掉后 Master 会迅速拉起新的 Worker，服务不中断。
 * **热部署支持：** 可以在不停止服务的情况下，平滑升级 Nginx 的可执行文件或重新加载配置文件（不丢失当前连接）。
 * **模块化设计：** 扩展性好，生态丰富，支持诸如 Gzip、SSL、限流等多种功能。
## 2. 常用命令与基本使用
Nginx 的安装通常通过系统的包管理器完成（如 Ubuntu 的 apt install nginx 或 CentOS 的 yum install nginx）。安装后，主要通过命令行进行管理。
| 命令 | 说明 | 适用场景 |
|---|---|---|
| nginx -t | 测试配置文件语法是否正确 | **必用**：每次修改配置后、重启前必须执行。 |
| nginx -s reload | 平滑重启（重新加载配置） | 修改配置后使之生效，且不中断当前正在处理的请求。 |
| nginx -s stop | 快速停止 Nginx | 强制立刻关闭，不管请求是否处理完。 |
| nginx -s quit | 优雅停止 Nginx | 等待 Worker 进程处理完当前请求后再关闭。 |
| nginx -v / -V | 查看版本号 / 查看详细编译参数和模块 | 排查问题或确认是否安装了特定模块时使用。 |
## 3. 配置文件的基本结构
Nginx 的主配置文件通常位于 /etc/nginx/nginx.conf。它的结构高度层级化，主要分为三大块：
```nginx
# 1. 全局块 (Global)：配置影响 Nginx 整体运行的指令，如 worker 进程数。
worker_processes auto; 

# 2. events 块：配置与用户的网络连接。
events {
    worker_connections 1024; # 每个 worker 允许的最大连接数
}

# 3. http 块：Nginx 最核心的部分，代理、缓存、日志等绝大多数功能都在这里配置。
http {
    include       mime.types;
    default_type  application/octet-stream;

    # 3.1 server 块：配置虚拟主机（一个 HTTP 块可以有多个 Server 块）
    server {
        listen       80;          # 监听端口
        server_name  example.com; # 域名或 IP

        # 3.2 location 块：配置请求的路由，以及各种页面的处理情况
        location / {
            root   /usr/share/nginx/html; # 静态文件根目录
            index  index.html index.htm;  # 默认主页
        }
    }
}

```
## 4. 核心功能配置实战
### 4.1 反向代理 (Reverse Proxy)
**作用：** 客户端请求 Nginx，Nginx 将请求转发给后端的真实服务器（如 Tomcat、Node.js），并将结果返回给客户端。隐藏了后端真实 IP。
```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        # 将所有请求转发到本机的 8080 端口
        proxy_pass http://127.0.0.1:8080;
        
        # 将客户端真实 IP 和域名传递给后端服务器
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```
### 4.2 负载均衡 (Load Balancing)
**作用：** 当后端有多台服务器时，Nginx 可以根据配置的策略将请求分发到不同的服务器上，分摊压力。
```nginx
# 定义一个名为 backend 的服务器组
upstream backend {
    # 策略 1：轮询（默认）
    # 策略 2：weight（权重），权重越高被分配的概率越大
    server 192.168.1.101:8080 weight=3;
    server 192.168.1.102:8080 weight=1;
    
    # 策略 3：ip_hash; （将同一 IP 的请求固定打到同一台服务器，解决 Session 问题）
    # 策略 4：least_conn; （将请求分配给连接数最少的服务器）
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        # 代理到上面定义的 upstream
        proxy_pass http://backend;
    }
}

```
### 4.3 动静分离 (Static-Dynamic Separation)
**作用：** 将动态请求（如 API 接口）交给后端应用服务器处理，将静态资源（如 HTML、CSS、JS、图片）由 Nginx 直接响应，大幅提升响应速度。
```nginx
server {
    listen 80;
    server_name www.example.com;

    # 静态资源请求：利用正则表达式匹配，由 Nginx 直接读取本地文件返回
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|html)$ {
        root /var/www/static;
        expires 30d; # 设置浏览器缓存 30 天
    }

    # 动态请求：转发给后端 Tomcat 或 Spring Boot
    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
}

```
### 4.4 重定向 (Redirect)
**作用：** 网页访问路径更改或全站 HTTP 强制跳转 HTTPS 时使用。通常使用 return 或 rewrite 指令。
```nginx
server {
    listen 80;
    server_name old-domain.com;

    # 场景 1：域名废弃，永久重定向 (301) 到新域名
    return 301 http://new-domain.com$request_uri;
}

server {
    listen 80;
    server_name example.com;
    
    # 场景 2：基于正则的路径重写 (Rewrite)
    # 将 /old-path/xxx 重写为 /new-path/xxx，并继续向下匹配 location (last)
    location /old-path/ {
        rewrite ^/old-path/(.*)$ /new-path/$1 last;
    }
}

```
## 5. 进阶必备功能（强烈建议掌握）
除了你提到的功能，以下三个配置在实际生产环境中几乎是**标配**：
### 5.1 HTTPS / SSL 配置
保障数据传输安全，现在的主流网站都必须配置 HTTPS。
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    # 指定证书文件和私钥文件路径
    ssl_certificate     /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    # 推荐的 SSL 安全协议和加密套件配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        root /var/www/html;
    }
}

# 配合重定向：将所有 80 端口的 HTTP 请求强制跳转到 443 端口的 HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

```
### 5.2 Gzip 压缩
可以大幅缩小返回给浏览器的文本类资源（HTML、CSS、JS）的体积，节省带宽，加快网页首屏加载速度。通常配置在 http 块中全局生效。
```nginx
http {
    gzip on;                 # 开启 Gzip
    gzip_min_length 1k;      # 小于 1KB 的文件不压缩
    gzip_comp_level 5;       # 压缩级别 (1-9)，数字越大压缩率越高但越消耗 CPU，推荐 4-6
    gzip_types text/plain text/css application/json application/javascript text/xml; # 针对哪些类型压缩
    gzip_vary on;            # 在响应头中添加 Vary: Accept-Encoding
}

```
### 5.3 防盗链 (Access Control / Anti-hotlinking)
防止其他网站直接使用你服务器上的图片或视频链接，消耗你的服务器流量。
```nginx
location ~* \.(jpg|png|gif|mp4)$ {
    root /var/www/media;
    
    # 允许空 Referer（直接浏览器访问），允许本站域名访问
    valid_referers none blocked server_names *.example.com;
    
    # 如果来源不在上述列表内，则返回 403 拒绝访问，或者重定向到一张警告图
    if ($invalid_referer) {
        return 403;
        # rewrite ^/ http://example.com/warning.jpg;
    }
}

```
