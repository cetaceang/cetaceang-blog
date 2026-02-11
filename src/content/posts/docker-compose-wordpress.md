---
title: docker compose部署WordPress，我如何按照个人喜好构建？
published: 2026-02-11
description: '数据库与WordPress分离部署，MariaDB替代MySQL降低资源占用，按自己的喜好来'
image: ''
tags: [Docker, WordPress, 自部署]
category: '技术'
draft: false
lang: ''
---

# docker compose部署WordPress，我如何按照个人喜好构建？

最近折腾了一下用 Docker Compose 部署 WordPress，踩了一些坑，也摸索出了一套比较舒服的实践。虽然最终我的个人博客选择了静态方案，但 WordPress 依然是很多场景下的好选择——丰富的生态、可视化编辑、上手门槛低。在实践中我发现，如果使用 MariaDB 而不是 MySQL 8.0，WordPress 的资源占用远没有想象中那么高。这篇文章就来分享一下最符合我个人喜好的部署方式。

## 准备工作

我用的系统是debian13，之前已经部署过docker。如果你没有docker，可以访问<a href="https://docs.docker.com/engine/install/" target="_blank">docker官方文档</a>。我也推荐你使用官方的安装脚本，运行下面两行命令

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

安装docker以后，将当前用户加入docker用户组

```bash
sudo usermod -aG docker $USER
```

运行

```bash
docker run hello-world
```

如果你看到一个消息被打印，那么docker就安装好了

## 部署WordPress

<a href="https://hub.docker.com/_/wordpress" target="_blank">WordPress官方的docker hub</a>里的docker-compose.yml已经属于开箱即用。不过，想要让自己用的舒服，我还是决定按照个人喜好做一些修改。首先，我希望这个数据库可以复用，我不想要数据库和WordPress的生命周期绑定，所以我将数据库和WordPress拆成了两个docker-compose.yml。先创建一个docker网络shared-net

```bash
docker network create shared-net
mkdir mariadb
cd mariadb
vim docker-compose.yml
```

复制以下内容

```yaml
services:
  mariadb:
    image: mariadb:10.6
    container_name: mariadb
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: your-password-here
    volumes:
      - ./data:/var/lib/mysql
    networks:
      - shared-net
    # ports:
    #   - "127.0.0.1:3306:3306"

networks:
  shared-net:
    external: true
```

一般情况下，不建议映射端口，由于docker的问题，这会将你的数据库端口直接暴露在公网。你可以选择将数据库和WordPress加入同一个docker网络shared-net。
运行`docker compose up -d`，然后进入mariadb命令行。

```bash
docker exec -it mariadb mysql -uroot -p
```

回车后输入你在`docker-compose.yml`里设置的`MYSQL_ROOT_PASSWORD`。

成功后你会看到：

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.6.x-MariaDB ...

MariaDB [(none)]>
```

创建数据库

```sql
CREATE DATABASE wp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

创建专用用户

```sql
CREATE USER 'wp_user'@'%' IDENTIFIED BY 'your_password_here';
```
把your_password_here换成你的密码

授权

```sql
GRANT ALL PRIVILEGES ON wp_db.* TO 'wp_user'@'%';
```

刷新并退出

```sql
FLUSH PRIVILEGES;
EXIT;
```

验证是否成功

```bash
docker exec -it mariadb mysql -uwp_user -p -e "SHOW DATABASES;"
```

输入wp_user的密码后，能看到wp_db就说明成功了

部署好数据库，继续部署WordPress

```bash
cd
mkdir wordpress
cd wordpress
vim docker-compose.yml
```

复制以下内容

```yml
services:
  wordpress:
    image: wordpress
    container_name: my-wordpress
    restart: unless-stopped
    ports:
      - "127.0.0.1:8080:80"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password_here
      WORDPRESS_DB_NAME: wp_db
    volumes:
      - ./data:/var/www/html
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
    networks:
      - shared-net

networks:
  shared-net:
    external: true
```

WORDPRESS_DB_PASSWORD环境变量使用你刚才设置的密码。这里多挂载了一个 `uploads.ini`，用来覆盖 PHP 默认的上传限制。WordPress 默认的上传大小很小，上传主题或稍大的图片就会被拦住。在 wordpress 目录下创建 `uploads.ini`，内容如下

```ini
file_uploads = On
memory_limit = 256M
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 600
```

如果你用了 Nginx 做反向代理，还需要在对应的站点配置里加上 `client_max_body_size 100m;`，否则大文件会在 Nginx 这一层就被拒掉。

都准备好之后，运行`docker compose up -d`

## 访问方式

这里我把 WordPress 绑定在 `127.0.0.1:8080`，只允许本机访问。  
如果要通过域名对外访问，还需要在宿主机前面加一层反向代理（Nginx/Caddy/Traefik 都可以）。  
我个人用的是 Docker 部署的 Nginx Proxy Manager（`host` 网络），把域名转发到 `127.0.0.1:8080`，这里就不展开具体配置了。

配置好反向代理并完成 DNS 解析后，访问你的域名，你应该能看到 WordPress 的安装引导页面。选择语言，设置站点标题、管理员账号和密码，点击安装，几秒钟后你的 WordPress 就正式上线了。

## 最后

这套方案我自己用下来，比较满意的地方是数据库和 WordPress 的生命周期解耦了，数据库可以给其他项目复用，WordPress 坏了不用担心数据库跟着丢；MariaDB 的内存占用也确实比 MySQL 8.0 低不少，适合小鸡；数据库没有映射端口到宿主机，只走 docker 内网通信，安全上也省心一些。不太方便的地方在于，两个 compose 项目意味着备份和迁移时要分别处理，建库建用户也是手动操作，步骤多了出错的概率就高。另外这篇文章没有涉及数据备份，如果是认真在用的站点，这一块还是要自己补上。如果你也在考虑自建 WordPress，希望这篇文章能帮你少踩几个坑。

