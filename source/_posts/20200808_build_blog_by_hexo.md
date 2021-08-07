---
title: 使用 Hexo + GitHub Page 自建博客
date: 2021-08-08 04:14:38
tags: Hexo
---

# 创建页面

参考资料：https://zhuanlan.zhihu.com/p/70240127

使用Hexo自建博客，Hexo会自动生成静态页面并部署到GitHub repo上，不需要直接git push。而在GitHub repo里也只会有public目录下的内容。与此同时public文件夹在hexo的目录下是在.gitignore里的，所以不要直接push整个目录，这样会导致网页404。

关于网站信息的修改在`_config.yml`进行，例如修改标题在Site下。

## 修改域名

主要步骤上面的资料已经很详细了，但关于这段

>注：如果你用了 hexo clean 命令删除本地静态资源，CNAME文件也会被删掉，所以你可以在本地生成的public文件夹中直接创建好，再上传。

也许是typo，应该把CMAKE文件放在source里，才不会被删。

# 安装主题

本站采用next主题：https://github.com/next-theme/hexo-theme-next

因为hexo是5.0+版本，所以安装主题只需：

```
$ cd hexo-site
$ npm install hexo-theme-next
```
再把_config.yml的theme改成next。但这种安装方法比较新，next的中文文档没有更新，需要参考英文文档进行定制：https://theme-next.js.org/docs/getting-started/configuration.html 。

以改scheme为例，如果按照上面的教程走，那就是npm安装的，运行：
```
$ cp themes/next/_config.yml _config.next.yml
```
然后修改Schemes的部分即可。