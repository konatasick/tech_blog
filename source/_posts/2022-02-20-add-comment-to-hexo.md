---
title: 给 Hexo 配置 Giscus 评论系统
tags:
  - Hexo
  - Next
  - Giscus
categories:
  - Project
date: 2022-02-20 18:14:45
---


在网上看到别人家的博客有GitHub登陆评论的功能，立刻表示：“我也要！”

一开始使用的是next自带的gitalk设置，然而在我的一番努力下依然没有配好，于是换成了Giscus，参考[Giscus的优点](https://blog.southfox.me/2022/01/%E4%B8%BA%E5%8D%9A%E5%AE%A2%E6%94%AF%E6%8C%81%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F/)。

Giscus的配置非常简单！按照上文的顺序配好了以后，卡在最后一步“嵌入到博客文章模板底部”，于是搜到了[这个插件](https://github.com/next-theme/hexo-next-giscus)，安装：

```
$ npm install hexo-next-giscus --save
```
然后在 hexo 的 _config.yml 添加：

```
giscus:
  enable: false
  repo: # Github repository name
  repo_id: # Github repository id
  category: # Github discussion category
  category_id: # Github discussion category id
  # Available values: pathname | url | title | og:title
  mapping: pathname
  # Available values: 0 | 1 
  reactions_enabled: 1
   # Available values: 0 | 1 
  emit_metadata: 1
  # Available values: light | dark | dark_high_contrast | transparent_dark | preferred-color-scheme
  theme: light
  # Available values: en | zh-C
  lang: en
  # Available value: anonymous
  crossorigin: anonymous
```
把`enable: false`改为`true`，并参考[Giscus](https://giscus.app/zh-CN)生成的脚本的元素依次填入设置。

一开始是在`_config.next.yml`不起作用，不知为何。

最后：
```
hexo clean
hexo generate
hexo deploy
```

刷新页面后就可以看到评论页面啦！

快来给我评论吧～～