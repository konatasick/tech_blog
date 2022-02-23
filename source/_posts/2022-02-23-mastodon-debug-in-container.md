---
title: Docker 部署的 mastodon 在容器内 debug
date: 2022-02-23 13:54:28
tags:
  - Mastodon
  - Docker
categories:
  - Project
---

感谢小柠檬 @lemon@nya.lemonade.moe 用自己的实例测试研究出这套可行方案！

一个简短的解释，晚点我写详细一点。

非docker的魔改方法：

https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/14/mastodon-modify.html

修改后运行：
```
RAILS_ENV=production bundle exec rails assets:precompile
systemctl restart mastodon-sidekiq
systemctl reload mastodon-web
systemctl restart mastodon-streaming
```
对应的docker 魔改方式：
```
docker exec -it mastodon_web_1 /bin/bash 
```
进入container改代码，然后编译：
```
RAILS_ENV=production bundle exec rails assets:precompile
```

这步不报错之后exit退出，然后：
```
docker-compose restart
```
即可代替那几句systemctl restart。

如果有500之类的报错可以参考：

https://blog.tantalum.life/posts/how-to-run-your-mastodon-by-docker/#%E6%95%B0%E6%8D%AE%E5%BA%93%E9%80%82%E5%BA%94

我暂时没有遇到。

本方法适合进入docker进行一些调试并且可以快速在网页端看到修改结果，确认无误后还是推荐打包成镜像再`docker-compose down/up -d`。

建议的workflow是：

- 在docker中热修代码并restart
- 确认代码ok之后在容器外修改代码并走传镜像的流程进行正式更新，即：

https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/01/Mastodon-on-Docker-2.html

