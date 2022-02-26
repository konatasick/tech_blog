---
title: Windows 下配置 Anaconda3 + vscode 基础环境
date: 2022-02-26 19:23:16
tags: 
- vscode
- python
- Anaconda3
categories: 
- Workshop
---
之前写过一个[vscode安利帖](https://tech.konata.co/2020-01-16-vscode/)，最近因为重新配置环境，顺便把更基础的部分补充一下。

## 安装必要软件
### Anaconda3

在[官网](https://www.anaconda.com/products/individual)下载，安装时勾选`add anaconda3 to my PATH enviroment variable`。

### VScode
在[官网](https://code.visualstudio.com/)下载，安装时勾选`添加到PATH`，重启使其生效。

### git
在[官网](https://git-scm.com/downloads)下载，并配置好个人信息：

```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
## 配置 VScode

重启电脑，打开 vscode，下载 python 插件，有需要可以改成中文，参考[vscode安利帖](https://tech.konata.co/2020-01-16-vscode/)。

在 vscode 的菜单中选择：Terminal-new terminal

在出现的终端界面右边选择默认terminal为cmd（重要）：

![](https://upload-images.jianshu.io/upload_images/12583080-3334c9e7ec5b3655.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再点+出现新的cmd命令窗口，输入
```
Conda activate base
```
激活conda，如果最左边有个（base）说明成功了.
## 配置清华镜像
如果 conda 和 pip 太慢，可以配置清华镜像：
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --setshow_channel_urls yes
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U 
Pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
