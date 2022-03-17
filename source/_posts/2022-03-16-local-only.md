---
title: Mastodon | 以 local-only 代码合并为例浅谈如何合并特定 commit
tags:
  - mastodon
  - git
categories:
  - Project
date: 2022-03-16 20:43:31
---


## 前言

本文会尽我可能解释涉及到的所有概念，但还是推荐有一定的 git 使用基础后阅读。推荐阅读的前置内容：[《如何利用Docker搭建Mastodon实例（二）：进阶魔改篇》](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/01/Mastodon-on-Docker-2.html) 的升级章节。


最初是有一位大神写了 local-only 的功能，可以让嘟文限制在本实例而不流出外部。此 [pr](https://github.com/mastodon/mastodon/pull/8427) 并没有被官方采纳，但是一些站点都有增加这个魔改功能。

本篇以我的 [local-only commit](https://github.com/retirenow/mastodon/commit/8ca207b9e3b45fc4e3962385572bd37e243c3e36) 为例，说明如何合并这种代码量较大的魔改（代码量小的通常找到位置复制粘贴即可，参考文章开头的[教程](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/01/Mastodon-on-Docker-2.html) ）。



## 从远程仓库拉取代码

首先将**你的**代码仓库 `git clone` 到本地，或直接 `cd` 到已有的仓库目录：
```
$ git clone https://github.com/yourname/mastodon.git 
$ cd mastodon
```
此时如果查看远程连接情况，只会看到一个名为 `origin` 的远程仓库链接，也就是你自己的远程仓库的名字。
```
$ git remote -v
origin  https://github.com/yourname/mastodon.git (fetch)
origin  https://github.com/yourname/mastodon.git (push)
```
此时如果查看分支情况，一般而言默认是只 `clone` 了 `main` 分支（即 `origin/main`）。
```
$ git branch -vv
* main 4bdce2c51 [origin/main] 你最新的 commit 内容 (#17618)
```


### （可选）创建一个专门用于处理此次合并的分支
如果你熟悉 `git` 的分支管理，此时可以新建一个专门的分支，最后再并入主干，本文采取直接合入主干的方式，故这部分不展开讲。

## 设置远程仓库连接
一个基础的 `git` 概念是，在本地的这个是你的本地代码仓库，而 GitHub 只是代码的远程托管，本地和远程仓库可以理解为一个仓库的两个副本，你可以把你的代码发送给远程仓库，也可以从远程仓库拉取，而且一个本地仓库可以连接不止一个远程仓库。

现在添加我的 [local-only 的仓库](https://github.com/retirenow/mastodon.git) 作为 `upstream` ：
```
$ git remote add upstream https://github.com/retirenow/mastodon.git
```
此时再查看远程连接：
```
$ git remote -v
origin  https://github.com/yourname/mastodon.git (fetch)
origin  https://github.com/yourname/mastodon.git (push)
upstream        https://github.com/retirenow/mastodon.git (fetch)
upstream        https://github.com/retirenow/mastodon.git (push)
```
就会有两个远程，名字分别为 `origin` 和 `upstream`。

## 拉取分支并合并代码
设置好远程以后，我们就可以从 `upstream` 拉取含有 local-only 的分支：
```
$ git fetch upstream main:instance_only_statuses
```
这边因为分支都叫 `main`，为了区分我给从 `upstream` 拉取的本地分支重命名了一下。上面这句话的意思是，从 `upstream` 拉取它的 `main` 分支，并命名为本地的 `instance_only_statuses`。

这时候查看分支情况，会有两个分支：
```
$ git branch -vv
  instance_only_statuses d8c03d962 retirenow 最新的 commit 内容
* main                   4bdce2c51 [origin/main] 你最新的 commit 内容 (#17618)
```
星号代表你现在处于 `main` 分支上。

现在 cherry-pick 这个 commit：
```
$ git cherry-pick 8ca207b9e3b45fc4e3962385572bd37e243c3e36
```
那一长串编码是这个 `commit` 的哈希值，可以唯一指定找到这串 `commit` ，可以在 `commit` 页面找到：
![](https://s2.loli.net/2022/03/16/iXBq4wMYpsR2mZa.png)

`cherry-pick` 用于只合并特定 `commit` ，和 `merge` 类似，`merge` 是合并两个分支（包含所有 `commit`）。

## 处理冲突
在 `cherry-pick` 之后，如此大的改动量通常会提示代码有冲突，推荐用 [vscode](https://tech.konata.co/2022-02-26-anaconda-vscode/) 处理。

从侧边的 `git` 点进去看：

![](https://s2.loli.net/2022/03/16/fOcUHQFNBJW8LVw.png)

其中暂存的更改（Staged Changes）就是已经被自动合并好的代码，不需要处理，而合并更改（Merge Changes）则是需要处理的冲突文件，具体数量因人而异。

点开合并更改中一个文件，在冲突的地方，会有如下所示：
![](https://s2.loli.net/2022/03/16/aRDdFfQykV3qzXL.png)

Accept Current Change 是保持现在（main分支）的代码，Accept Incomming Change 是用 local-only 的代码，Accept Both Changes 则是保留两者。

这里的处理其实需要一点灵性和连蒙带猜，可以结合[原始pr的file change](https://github.com/mastodon/mastodon/pull/8427/files)理解一下它增加代码的逻辑然后去选择如何更改。以上图为例，显然是因为我们main分支增加了一个叫 "ordered_media_attachment_ids" 的东西，而 local only 也增加了 "local_only"，所以我们的选择就是 Accept Both Changes。

## 处理完冲突后

一般而言，如果你是参考[这个教程](https://pullopen.github.io/%E8%BF%9B%E9%98%B6%E9%AD%94%E6%94%B9/2020/11/01/Mastodon-on-Docker-2.html)进行代码魔改，此时你只需要提交你的改动：

```
$ git add .
$ git commit -m "local-only"
$ git push -u origin main
```
然后静待 GitHub 编译好镜像，你再拉取使用即可。

我在这一步遇到了报错500，参考[这篇文章](https://blog.tantalum.life/posts/how-to-run-your-mastodon-by-docker/#%E6%95%B0%E6%8D%AE%E5%BA%93%E9%80%82%E5%BA%94)对数据库进行 migrate 再重启即可。
### 在 docker 容器中 debug
但是通常来说如此大的代码改动量，有一定风险不能一次性跑通，如果直接使用镜像遇到问题，我推荐参考[《Docker 部署的 mastodon 在容器内 debug》](https://tech.konata.co/2022-02-23-mastodon-debug-in-container/)。或者如果不放心也可以先在容器内部编译通过再拉取镜像。

首先进入 docker 容器:
```
$ docker exec --user root -it mastodon_web_1 /bin/bash
```

容器里默认是没有 `git` 的，需要先安装 `git`：
```
$ apt-get update
$ apt-get install git
```
这时候可能会遇到报错
```
E: Archives directory /var/cache/apt/archives/partial is missing
```
解决方案：
```
$ mkdir -p /var/cache/apt/archive/partial
```
继续：
```
$ apt-get install git
```
即可正确安装。

进入到我们的代码目录：
```
$ cd /mastodon
$ git init
```
然后添加你的远程仓库并拉取最新的代码：
```
$ git remote add origin https://github.com/yourname/mastodon.git
$ git fetch origin main:main
$ git checkout -f main
```
也可以一开始就直接在 docker 容器里合代码，如果你会用 vscode 连接 docker 容器，原理都是一样的。

接下来 precompile:
```
$ RAILS_ENV=production bundle exec rails assets:precompile
```
通过后退出并 `docker-compose restart`，就可以直接在网页上预览效果了。
