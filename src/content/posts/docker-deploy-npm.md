---
title: Docker 部署 Nginx Proxy Manager：极简的可视化反代神器
published: 2026-03-30
description: 'Nginx Proxy Manager 专为简化 Nginx 反向代理管理而设计，提供美观的 Web 界面，让你无需手搓配置就能轻松搞定反代和免费 SSL 证书。'
image: ''
tags: [Docker, 教程, Nginx]
category: '技术'
draft: false
lang: 'zh-CN'
---

Nginx Proxy Manager (NPM) 是一个开源的 **Docker 容器项目**，专为简化 Nginx 反向代理管理而设计。它提供了一个漂亮、安全且易用的 Web 界面，让你无需编写复杂的 Nginx 配置，就能轻松管理代理主机、SSL 证书、重定向等功能。

## 🌟 核心亮点

- **免费 SSL**：内置 Let’s Encrypt 支持，证书自动申请 + 自动续期（也可使用自定义证书）。
- **超级简单**：一键创建转发域名（Proxy Hosts）、重定向、Stream 转发、404 主机等，可以屏蔽常见探测，支持 WebSocket，强制 HTTPS，使用 HTTP/2 等功能，都只需要在面板上点点点！
- **安全与控制**：支持访问列表、HTTP 基础认证、多用户权限管理、审计日志，以及高级 Nginx 配置选项。
- **保护主机安全**：通过使用 Host 网络模式，NPM 不仅可以反向代理 Docker 容器部署的项目，也可以轻松反向代理其它（如 Node.js, Rust 等）以二进制可执行文件运行且仅监听在 `127.0.0.1` 上的程序。无需将这些端口暴露在外网，极大提升了 VPS 的安全性。

## 💬 个人使用感悟：NPM 还是裸 Nginx？

在我刚接触 VPS、还未深入了解 Nginx 配置语法的时候，NPM 这个项目让我得以快速上手。它帮我极其简单地实现了服务的反向代理，让我能优雅地通过域名访问各项服务，而不用再去死记硬背每个应用跑在哪个端口。

更重要的是，**使用 Nginx 进行反向代理，本身就是保护 VPS 安全的最佳实践之一**。很多新手不知道的是，**Docker 映射端口时会直接修改底层的 iptables 转发规则，这意味着系统常用的 UFW 防火墙默认是管不住 Docker 暴露的端口的**。如果你习惯性地在配置中直接映射端口（比如只写 `8080:80`），Docker 默认会将端口绑定在 `0.0.0.0`，导致你的服务直接绕过防火墙暴露在公网上，极易被 FOFA 等资产测绘引擎无差别扫描并记录。

因此，我强烈建议的安全做法是：**将所有内部服务的端口仅绑定监听在 `127.0.0.1`**（即只有本机可访问，必须显式写成 `127.0.0.1:8080:80`），然后让 Nginx 或 NPM 作为机器上唯一的公网入口（仅开放 `80/443` 端口），将流量安全地“反向代理”到后端的 `127.0.0.1` 服务上。NPM 的极简可视化操作，让新手小白也能极其轻松地实现这种安全防护机制。

不过，等你折腾多一点，你会发现：**其实手搓原生 Nginx 配置文件并没有想象中那么难！** 通过提取和复用配置片段，你可以很轻易地写出每个站点需要的 Nginx 配置文件。详情可以看我的另一篇博客：👉 **<a href="/posts/nginx-install-and-usage-practice/" target="_blank">《Nginx 的安装与个人使用实践总结》</a>**。

**到底该怎么选呢？**
- **选 NPM**：适合手头宽裕（内存 ≥ 1G）的机器，主打一个“点点点”的省心和优雅的可视化管理。
- **选裸 Nginx**：毕竟安装 NPM，就意味着你需要安装 Docker。Docker 引擎本身的内存开销，相比于极致轻量的裸 Nginx 还是大了不少。在一些只有 256M 或 128M 内存的“低配小鸡”上，使用 Docker 显然不是一个明智的选择，这个时候还是裸 Nginx 更加合适。

## 🛠️ Docker Compose 部署示例 (Host 网络模式)

如同我上文所说，为了实现最高效且安全的反向代理（包括代理其它如 Node.js, Rust 等以二进制可执行文件运行且仅监听在 `127.0.0.1` 上的程序），这里我们强烈建议配置使用 **`host` 网络模式**。

使用 `network_mode: host` 后，NPM 容器将直接与宿主机共享网络栈，这意味着：
1. 它会自动占用宿主机的 `80` (HTTP)、`443` (HTTPS) 和 `81` (管理面板) 端口，你**不需要也不可以**再写 `ports` 端口映射。
2. 在 NPM 的管理面板中反向代理本机服务时，目标的 IP 地址可以直接写 `127.0.0.1`，配置起来极其符合直觉。

以下是我正在使用的 `docker-compose.yml` 示例，你可以直接复制使用：

```yaml
version: '3.8'

services:
  nginx-proxy-manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped

    # === 核心修改 ===
    # 使用 host 模式，容器直接共用宿主机网卡
    # 因此不需要（也不能）写 ports 映射，它会自动占用宿主机的 80, 443, 81 端口
    network_mode: host

    environment:
      # 建议修改为你所在的时区，例如 Asia/Shanghai
      TZ: "Asia/Shanghai"

      # 如果你的宿主机不支持 IPv6，建议取消下面这行的注释
      # DISABLE_IPV6: 'true'

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

### 🚀 初始访问与登录

使用 `docker compose up -d` 启动容器后，打开浏览器访问：
`http://你的VPS_IP:81`

**默认初始账密：**
- Email: `admin@example.com`
- Password: `changeme`

> ⚠️ **注意**：首次登录后，系统会强制要求你修改邮箱和密码，请务必设置一个强密码。同时，由于我们使用的是 `host` 模式，NPM 将直接使用宿主机网络，此时 **UFW 等本地防火墙是生效的**。所以请务必确保在 UFW 或系统的防火墙中放行了 `80`、`443` 和 `81` 端口！如果你的 VPS 服务商（如阿里云、腾讯云、AWS 等）有外部安全组，也同样记得去网页控制台进行放行！

## 🎯 基础使用指南：反代一个服务

要使用 NPM 将你的内部服务通过域名暴露到公网，通常分为两步：**配置域名解析**和**在面板中添加反代规则**。

### 第一步：在 Cloudflare 中解析域名

在你开始在面板中“点点点”之前，首先需要确保你的域名已经指向了你这台 VPS 的 IP 地址。这里以大多数人都在使用的 Cloudflare 为例：

1. 登录 Cloudflare 控制台，进入你的域名管理页面。
2. 点击左侧的 **DNS** 设置。
3. 添加一条 **A 记录** 或 **CNAME 记录**，将你想要使用的子域名（比如 `nginx.yourdomain.com`）解析到你 VPS 的公网 IP 上。

![Cloudflare 解析示例](https://imgbed.cetaceang.qzz.io/file/博客/1774426572871_cf解析.webp)

### 第二步：在 Nginx Proxy Manager 中添加 Proxy Host

完成域名解析后，回到 Nginx Proxy Manager 的管理面板进行反代配置：

1. **进入 Proxy Hosts 页面**
   在控制台首页，点击顶部导航栏的 **Hosts** -> **Proxy Hosts**，然后点击页面右上角绿色的 **Add Proxy Host** 按钮。
   ![Proxy Hosts 页面](https://imgbed.cetaceang.qzz.io/file/博客/1774880674553_npm_addproxyhost.png)

2. **配置**
   在弹出的窗口中，填写基本转发信息：
   - **Domain Names**：填写你刚刚解析的域名（例如 `nginx.yourdomain.com`），输入后按回车键确认。
   - **Scheme**：保持默认的 `http`。
   - **Forward Hostname / IP**：填写你要反向代理的目标 IP。由于我们使用了 `host` 网络模式，若反代本机服务直接填写 `127.0.0.1` 即可；NPM 不仅可以反向代理本机，也可以作为其它源站的访问入口，此时在这里直接填写对应源站的 IP 即可。
   - **Forward Port**：填写目标服务的端口号。例如我们要反代 NPM 自身的管理面板，填写 `81`。
   - **选项配置（Options）**：建议开启 **Cache Assets**（缓存静态资源）、**Block Common Exploits**（拦截常见漏洞探测）和 **Websockets Support**（支持 WebSocket 协议）。
   ![配置 Details](https://imgbed.cetaceang.qzz.io/file/博客/1774880673377_npm_addhost1.png)

3. **配置 SSL（自动申请证书）**
   切换到 **SSL** 选项卡，NPM 会自动帮你申请免费的 Let's Encrypt 证书：
   - **SSL Certificate**：下拉选择 `Request a new SSL Certificate`（申请新证书）。
   - **开启安全选项**：建议勾选 **Force SSL**（强制 HTTPS）、**HTTP/2 Support**（开启 HTTP/2 支持）和 **HSTS Enabled**（启用 HSTS 提升安全性）。
   - 勾选同意 Let's Encrypt 的服务条款（I Agree...）。
   ![配置 SSL](https://imgbed.cetaceang.qzz.io/file/博客/1774880669979_npm_addhost2.png)

最后点击右下角的 **Save** 按钮。NPM 会自动进行证书申请并保存配置，稍等片刻，当列表中出现带有绿色 `Online` 状态的记录时，就说明配置成功了！现在，你就可以通过 `https://nginx.yourdomain.com` 安全优雅地访问你的服务了。

## 🎉 总结

Nginx Proxy Manager 极大降低了反向代理和构建 HTTPS 站点的门槛，非常适合不想深陷复杂 Nginx 配置语法折磨的小白。特别是配合 Docker 的 `host` 网络模式，NPM 不仅能代理 Docker 内部的容器，更能轻松反向代理那些监听在本机回环地址（如 `127.0.0.1`）上的非 Docker 服务。

除了基础的反代功能，在 NPM 的 **Advanced（高级设置）** 中，你还可以灵活配置访问黑白名单（Access Lists）、自定义重定向规则（Redirection Hosts）以及编写自定义的 Nginx 参数。随着你的不断探索与折腾，NPM 一定会成为你玩转服务器的绝佳利器！
