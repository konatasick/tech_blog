---
title: Crontab 各种运行不成功和解决方法
date: 2022-02-22 01:37:15
tags:
  - Docker
  - Crontab
  - Linux
categories:
  - Workshop
---

Crontab是一个运行定时脚本的工具，我在这里翻车了n次，记录一下debug过程。

## 常用命令
```
service cron status # 查看运行状态和报错信息
service cron reload # 重新载入 
service cron restart # 重启
vim /etc/crontab #编辑 crontab
```

## Debug
### Missing newline before EOF

报错内容：
```
(*system*) ERROR (Missing newline before EOF, this crontab file will be ignored)
```
[解决方案](https://cronitor.io/cron-reference/missing-newline-before-eof#:~:text=Fixing%20missing%20newline%20before%20EOF%20errors.%20Correct%20this,always%20added%20and%20are%20not%20%20incorrectly%20stripped.)

>Correct this error simply by **adding a blank line** at the end of your crontab. 

就是在crontab文件的末尾添加一个新的空行（回车），就离谱……

### session opened for user root by (uid=0)
报错内容：
```
pam_unix(cron:session): session opened for user root by (uid=0)
```
[解决方案](https://www.digitalocean.com/community/questions/weird-cron-logs-appearing-for-no-reasion)：

打开文件：
```
vim /etc/pam.d/common-session-noninteractive
```
找到这一行：
```
session required pam_unix.so
```
并在其上方添加：
```
session [success=1 default=ignore] pam_succeed_if.so service in cron quiet use_uid
```
然后重启：
```
service cron restart
```