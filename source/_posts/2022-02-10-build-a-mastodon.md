---
title: build-a-mastodon
date: 2022-02-10 04:38:33
tags: 
- nginx
- certbot
- mastodon
categories: 
- Project
---

## Prerequisites

参考 [这个教程](https://www.howtoforge.com/how-to-install-mastodon-social-network-with-docker-on-ubuntu-1804/) 或 [蓝盒子站长的教程](https://pullopen.github.io/基础搭建/2020/10/19/Mastodon-on-Docker.html) ，一直进行到把 docker 和 docker-compose 装好。

Checklist：

- ssh-key设置并关闭密码登录
- 开启防火墙
- 域名映射到服务器ip
- 安装成功 docker 和 docker-compose 

## Postgres database

在设置mastodon的时候，运行：

```bash
sudo docker-compose run --rm web bundle exec rake mastodon:setup
```

我在这个地方卡住了非常久，在postgres那里尝试了各种版本教程的排列组合最后的结果都是：

```bash
Database connection could not be established with this configuration, try again.
FATAL:  role "mastodon" does not exist
```

后来我认真看了一下这一步，它的输出是：

```bash
Starting mastodon_db_1    ... done
Starting mastodon_redis_1 ... done
```

查看docker container也是这两个docker在跑：

```bash
$ docker container list -a
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                    PORTS     NAMES
bd7debf13a49   redis:6-alpine         "docker-entrypoint.s…"   31 minutes ago   Up 31 minutes (healthy)             mastodon_redis_1
30c6e3b74c40   postgres:12.5-alpine   "docker-entrypoint.s…"   31 minutes ago   Up 31 minutes (healthy)             mastodon_db_1
```

于是我尝试把教程中4. 初始化PostgreSQL的这句话：

```bash
docker exec -it postgres12 psql -U postgres
```

改成

```bash
docker exec -it mastodon_db_1 psql -U postgres
```

之后再添加mastodon用户：

```bash
> CREATE USER mastodon WITH PASSWORD 'your_password' CREATEDB;
> exit
```

完整操作看起来是这样的：

```bash
$ sudo docker exec -it mastodon_db_1 psql -U postgres
psql (12.5)
Type "help" for help.

postgres=# CREATE USER mastodon WITH PASSWORD 'mastodon-db' CREATEDB;
CREATE ROLE
postgres=# exit
```

这时候再运行：

```bash
sudo docker-compose run --rm web bundle exec rake mastodon:setup
```

按照下列输出就可以通过验证了：

```bash
Domain name: example.com

Single user mode disables registrations and redirects the landing page to your public profile.
Do you want to enable single user mode? No

Are you using Docker to run Mastodon? Yes

PostgreSQL host: mastodon_db_1
PostgreSQL port: 5432
Name of PostgreSQL database: mastodon
Name of PostgreSQL user: mastodon
Password of PostgreSQL user: （这里写上面你给mastodon设置的your_passwords）
Database configuration works! 🎆

Redis host: mastodon_redis_1
Redis port: 6379
Redis password: （这里是直接回车，没有密码）
Redis configuration works! 🎆

Do you want to store uploaded files on the cloud? No

Do you want to send e-mails from localhost? （这一步参考原教程配置，我配坏了）

This configuration will be written to .env.production
Save configuration? Yes
Below is your configuration, save it to an .env.production file outside Docker:
```

把接下来的部分一直到

```
It is also saved within this container so you can proceed with this wizard.
```

两行绿字中间的所有内容都复制下来。

接下来依然按照蓝盒子教程走，两个问题选yes，输入管理员账号。

## 配置nginx

这边又出了问题，按照教程走下来在这一步报错：

```bash
$ sudo systemctl reload nginx
Job for nginx.service failed.
See "systemctl status nginx.service" and "journalctl -xe" for details.
```

查看报错信息：

```bash
$ sudo nginx -t
nginx: [emerg] no "ssl_certificate" is defined for the "listen ... ssl" directive in /etc/nginx/sites-enabled/example.com.conf:25
nginx: configuration file /etc/nginx/nginx.conf test failed
```

原来是证书没有设置好。这边我把soft link先取消了（否则会报错），运行

```bash
sudo rm /etc/nginx/sites-enabled/example.com.conf
sudo certbot certonly --nginx -d example.com
sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
```

然后把example.com.conf 里这两行取消注释（#删掉）

```
  # Uncomment these lines once you acquire a certificate:
  ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
```

记得上面的example.com实际上应该是你的域名。







参考资料：

[https://pullopen.github.io/%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA/2020/10/19/Mastodon-on-Docker.html](https://pullopen.github.io/基础搭建/2020/10/19/Mastodon-on-Docker.html)

https://gist.github.com/TrillCyborg/84939cd4013ace9960031b803a0590c4

https://peterbabic.dev/blog/running-mastodon-with-docker-compose/

https://www.howtoforge.com/how-to-install-mastodon-social-network-with-docker-on-ubuntu-1804/

https://www.alibabacloud.com/blog/how-to-install-mastodon-using-docker-on-alibaba-cloud-ecs_595049

https://www.linode.com/docs/guides/install-mastodon-on-ubuntu-2004/

https://www.notion.so/Mastodon-042a05ee29a048df8b2c1afd49e4c49b

