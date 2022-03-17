---
title: Docker 部署的 mastodon 在容器内 debug
date: 2022-02-23 13:54:28
tags:
  - Mastodon
  - Docker
categories:
  - Project
---

TL; DR:

进入container改代码：

```
docker exec -it mastodon_web_1 /bin/bash 
```

预编译确保不报错：

```
RAILS_ENV=production bundle exec rails assets:precompile
```

退出并重启容器：

```
docker-compose restart
```

感谢小柠檬 @lemon@nya.lemonade.moe 用自己的实例测试研究出这套可行方案！

## 非docker的魔改方法

可参考这篇文章：[进阶魔改：修改字数上限、媒体上限、投票上限、添加自定义主题、界面用语、非登陆用户有限显示，附阻止本站嘟文流入某站点方法](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/14/mastodon-modify.html)

一般的修改逻辑是：

- 修改对应代码
- 预编译

```
RAILS_ENV=production bundle exec rails assets:precompile
```

- 重启服务

```
systemctl restart mastodon-sidekiq
systemctl reload mastodon-web
systemctl restart mastodon-streaming
```
## docker 魔改方法

参考这篇文章：[如何利用Docker搭建Mastodon实例（二）：进阶魔改篇](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/01/Mastodon-on-Docker-2.html)

在其中的“如何在Docker上保留你的魔改”部分，提到了三种方法：

1. 进入容器修改；
2. 用`docker-compose build`本地编译；
3. 利用GitHub编译镜像，直接拉取最新镜像后重启。

对于较为简单的改动，第三种方法已经够用，但有时候我们需要对改动进行调试，以确保网站按照我们预想的效果运行，此时如果每次改动代码都要走提交-编译镜像-拉取镜像-重启的流程，调试起来就会非常不方便。本文就提供了第一种方法的补充，进入容器修改代码并实时生效。

建议的workflow是：

- 进入容器修改代码并确保调试完成
- 在容器外修改代码并走传镜像的流程进行正式更新

### 在容器内debug

先进入容器，如果是按照官方的docker-compose.yml，容器名应该是`mastodon_web_1`，可用`docker ps`确认：

```
docker exec -it mastodon_web_1 /bin/bash 
```
进入container改代码，然后预编译：
```
RAILS_ENV=production bundle exec rails assets:precompile
```

这步不报错之后exit退出，然后：
```
docker-compose restart
```
即可在站点实时看到效果。

本方法适合进入docker进行一些调试并且可以快速在网页端看到修改结果，确认无误后还是推荐打包成镜像再`docker-compose down/up -d`，参考下文。

### 利用GitHub编译镜像

参考这篇文章：[如何利用Docker搭建Mastodon实例（二）：进阶魔改篇](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/01/Mastodon-on-Docker-2.html)

如果有500之类的报错可以参考：[数据库适应](https://blog.tantalum.life/posts/how-to-run-your-mastodon-by-docker/#%E6%95%B0%E6%8D%AE%E5%BA%93%E9%80%82%E5%BA%94)

对数据进行migrate，之后需要再重新`docker-compose down/up -d`一次，即：
```
docker-compose run --rm web rails db:migrate
docker-compose down
docker-compose up -d
```