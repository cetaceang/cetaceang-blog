---
title: 我的 Nginx 安装与使用实践
published: 2026-03-03
description: 记录我在 Ubuntu/Debian 服务器上安装、配置和使用 Nginx 的个人实践与踩坑总结。
image: ''
tags: [Linux]
category: '技术'
draft: false
lang: 'zh-CN'
---

## 1）安装 Nginx 和 Certbot

```bash
sudo apt update
sudo apt install -y nginx certbot
sudo systemctl enable --now nginx
```

检查是否启动成功：

```bash
systemctl status nginx --no-pager
```

## 2）写 Nginx 配置文件

我把这一阶段的配置放在同一个点完成，核心是三部分：

- 主配置：`/etc/nginx/nginx.conf`
- ACME 验证入口：`/etc/nginx/conf.d/acme.conf`
- 可复用片段：`/etc/nginx/snippets/*.conf`

先把运行时需要的目录一次性创建好：

```bash
sudo mkdir -p /var/cache/nginx/assets_cache
sudo chown -R www-data:www-data /var/cache/nginx/assets_cache

sudo mkdir -p /var/www/acme/.well-known/acme-challenge
sudo chown -R www-data:www-data /var/www/acme
```

### 2.1 主配置 `/etc/nginx/nginx.conf`

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    server_tokens off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }

    proxy_cache_path /var/cache/nginx/assets_cache
        levels=1:2
        keys_zone=assets_cache:200m
        max_size=1g
        inactive=7d
        use_temp_path=off;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    gzip on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### 2.2 ACME 验证入口 `/etc/nginx/conf.d/acme.conf`

```nginx
server {
    listen 80 default_server;
    server_name _;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/acme;
        try_files $uri =404;
    }

    location / { return 404; }
}
```

### 2.3 复用 snippets（按需 include）

我这里一共维护 7 个片段，按职责拆开，站点里按需 `include`：

- `acme-webroot.conf`（ACME challenge）
- `redirect-https-308.conf`（80 -> HTTPS 308）
- `security-headers.conf`（HSTS / 基础安全头）
- `proxy-common.conf`（反代通用头 + WebSocket + timeout）
- `cache-assets.optional.conf`（浏览器缓存头，可选）
- `proxy-cache-assets.optional.conf`（Nginx `proxy_cache`，可选）
- `block-common-exploits.optional.conf`（常见探测拦截，可选）

`/etc/nginx/snippets/acme-webroot.conf`：

```nginx
location ^~ /.well-known/acme-challenge/ {
    root /var/www/acme;
    try_files $uri =404;
}
```

`/etc/nginx/snippets/redirect-https-308.conf`：

```nginx
location / {
    return 308 https://$host$request_uri;
}
```

`/etc/nginx/snippets/security-headers.conf`：

```nginx
add_header Strict-Transport-Security "max-age=63072000" always;
add_header X-Content-Type-Options nosniff always;
add_header X-Frame-Options DENY always;
add_header Referrer-Policy no-referrer always;
```

`/etc/nginx/snippets/proxy-common.conf`：

```nginx
proxy_http_version 1.1;

proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $connection_upgrade;

proxy_read_timeout  300;
proxy_send_timeout  300;
```

`/etc/nginx/snippets/cache-assets.optional.conf`：

```nginx
expires 30d;
add_header Cache-Control "public, max-age=2592000, immutable" always;
access_log off;
```

`/etc/nginx/snippets/proxy-cache-assets.optional.conf`：

```nginx
proxy_cache assets_cache;
proxy_cache_key "$scheme$request_method$host$request_uri";
proxy_cache_valid 200 206 301 302 30m;
proxy_cache_valid 404 1m;
proxy_cache_lock on;
proxy_cache_revalidate on;
proxy_cache_min_uses 1;
proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
add_header X-Proxy-Cache $upstream_cache_status always;
```

`/etc/nginx/snippets/block-common-exploits.optional.conf`：

```nginx
location ~ /\.(?!well-known) {
    deny all;
}
location ~* \.(?:bak|conf|dist|ini|log|old|orig|save|sql|swp)$ {
    deny all;
}
```

配置写完后，先检查语法再重载：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## 3）创建反向代理站点配置（`/etc/nginx/sites-available/example`）

先写站点文件，但这一步先不启用：

```bash
sudo nano /etc/nginx/sites-available/example
```

写入以下内容（示例上游是 `127.0.0.1:3001`，按你的实际服务端口调整）：

```nginx
upstream example_upstream {
    server 127.0.0.1:3001;
    keepalive 32;
}

server {
    listen 80;
    server_name www.example.com;

    include /etc/nginx/snippets/acme-webroot.conf;
    include /etc/nginx/snippets/redirect-https-308.conf;
}

server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate     /etc/letsencrypt/live/www.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    include /etc/nginx/snippets/security-headers.conf;
    include /etc/nginx/snippets/acme-webroot.conf;

    location / {
        proxy_pass http://example_upstream;
        include /etc/nginx/snippets/proxy-common.conf;
    }

    # Optional:
    # include /etc/nginx/snippets/block-common-exploits.optional.conf;
}
```

先申请证书（这里不要先启用站点）：

```bash
sudo certbot certonly --webroot -w /var/www/acme -d www.example.com
```

证书申请成功后，再启用站点并重载 Nginx：

```bash
sudo ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/example
sudo nginx -t
sudo systemctl reload nginx
```

最后做一次续期演练，确认自动续期流程没有问题：

```bash
sudo certbot renew --dry-run
```

到这里，整套流程就完成了。只要你的反向代理后端服务正常运行（在我的例子里是 `127.0.0.1:3001`），就可以直接访问你的域名，验证服务是否已经正常对外提供。
