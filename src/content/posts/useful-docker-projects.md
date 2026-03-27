---
title: 值得部署的实用 Docker 项目推荐
published: 2026-03-25
pin: true
description: '整理并推荐一些个人在使用并觉得不错，有价值的 Docker 项目，涵盖基础运维、个人效率与数据安全、资源聚合与娱乐等多个方面。'
image: ''
tags: [Docker]
category: '技术'
draft: true
lang: 'zh-CN'
---

## 💡 写在前面

个人使用过程中，我觉得 **Docker Compose** 的使用体验是非常不错的，它可以同时管理生命周期绑定的多个 service（服务）。因此，在日常折腾中我很少直接使用 `docker run` 命令。

基于此，在后文中我所推荐的每一个项目，**都会附上我个人正在使用的 `docker-compose.yml` 示例**，方便大家直接抄作业。如果您的确更偏爱单行的 `docker run` 指令，那这篇文章可能无法为您提供最直接的参考，或许可以先将本页暂时关闭了~（笑）

**🛡️ 关于安全性的一点重要说明：**
经常折腾 Linux 的朋友应该知道，常用的防火墙（如 UFW）默认是“管不住” Docker 的。因为 Docker 在映射端口时，会直接越过 UFW 修改底层 iptables 转发规则。这意味着如果你在配置中直接映射端口（例如：`ports: - "8080:80"`），你的服务就会直接暴露在公网上。任何人只要访问你的 `IP:端口` 就能直接看到你的服务，这也导致服务极易被 FOFA 等资产测绘引擎扫描并记录，留下安全隐患。

为了保护 VPS 的安全和隐藏后台资产，**我个人的习惯是将所有容器暴露的端口强行绑定在本地回环地址上**，例如：
`ports: - "127.0.0.1:25774:25774"`
这样一来，服务就只能被本机访问，随后再通过 Nginx（比如后文推荐的 Nginx Proxy Manager）将其反向代理出去供公网访问。因此，后文列出的所有 `docker-compose.yml` 示例，也都会遵循这一安全习惯。

---

## 🛠️ 环境准备：安装 Docker

如果你还没有安装 Docker，可以访问 <a href="https://docs.docker.com/engine/install/" target="_blank">Docker 官方文档</a>。这里我也推荐你使用官方的一键安装脚本，只需在终端运行下面两行命令即可：

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

安装完成 Docker 以后，建议将当前用户加入 `docker` 用户组，这样以后运行 docker 命令就不需要每次都加 `sudo` 了：

```bash
sudo usermod -aG docker $USER
```

最后，运行以下测试命令：

```bash
docker run hello-world
```

如果你看到终端打印出了一段欢迎消息，那么恭喜你，Docker 环境就已经准备就绪了！

---

## 📦 项目推荐列表

### 1. Nginx Proxy Manager：告别手搓配置的反代神器

在折腾各种 Docker 容器时，对新手小白来说，最让人头疼的往往是端口的暴露、域名的绑定以及 SSL 证书的申请。

**Nginx Proxy Manager (NPM)** 完美解决了这个痛点。它提供了一个基于 Web 的图形化界面，让你只需在网页上“点点点”，就能轻松完成反向代理、WebSocket 支持、强制 HTTPS 等操作。而且它原生集成了 Let’s Encrypt，可以自动为你申请并续期免费的 SSL 证书。对于不擅长手搓nginx配置的新手小白来说，它绝对是装机必备的第一个容器。

👉 **<a href="/posts/docker-deploy-npm/" target="_blank">点击这里查看 Nginx Proxy Manager 的详细介绍与 Docker 部署教程</a>**

### 2. Komari 探针：纯粹而优雅的轻量级监控

作为一个 MJJ，必不可少的项目那肯定是探针。毕竟有一个笑话这么说：一个程序员买了 20 台 VPS，朋友问他：“你拿这些干嘛？”他说：“跑项目。” “什么项目？” “监控其它 19 台 VPS 有没有掉线。”

**Komari** 是一款开源的轻量级、自托管服务器状态监控工具，与功能日益臃肿的哪吒探针不同，Komari 放弃了复杂功能，回归探针本质。它的资源占用极低（128M内存的小鸡都能轻松扎针），且原生支持长达 30 天的历史数据追踪，拥有极高颜值的 Web 面板。

👉 **<a href="/posts/docker-deploy-komari/" target="_blank">点击这里查看 Komari 的详细介绍与 Docker 部署教程</a>**

