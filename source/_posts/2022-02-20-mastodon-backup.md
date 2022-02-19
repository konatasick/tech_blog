---
title: Mastodon 媒体存储和数据库备份
date: 2022-02-20 00:19:10
tags:
  - Mastodon
  - postgres
  - Scaleway
categories:
  - Project
---
**写在前面：**
本文备份方法我还在验证中，请谨慎使用，最好以防万一多导出一份`backup.dump` 。

## Why：为什么要备份

定期备份是好习惯，自从体验过一次wordpress搬家以后深刻感受到，有（异地）备份走遍天下都不怕。因为各种教备份的资料都比较全，本文主要是个人记录+学习用，如果想深入了解备份究竟在做什么也可以参考这篇文章。

当你有一份长毛象的异地备份，你可以：

- 迁移站点到其他服务器
- 在服务器完全瘫痪时或不小心重装后，依然可以还原所有数据库内容

## What：备份哪些内容

俗话说得好，没有经过检验的备份不是好备份。那么最快让我们清楚需要备份什么的方式就是看还原/迁移站点需要什么。

这里参考了[非docker站点迁移到docker](https://pullopen.github.io/%E7%AB%99%E7%82%B9%E7%BB%B4%E6%8A%A4/2020/10/21/migrate-Mastodon-to-Docker.html) 和 [docker迁移到docker](https://blog.tantalum.life/posts/migrate-mastodon-to-docker/) 两篇资料，大致上分为三步：

### 1. 转移配置文件

配置文件也就是你的mastodon安装目录（按照pullopen的教程的话应该是`/home/mastodon/mastodon`）下的：

```
.env.production 
docker-compose.yml 
```

也就是在更改站点设置中最常用的两个文件。

### 2. 转移数据库

这里有两个方法：

- 通过备份文件`backup.dump` 进行转移，也就是备份的最关键内容，通常通过`pg_dump`导出备份，再通过`pg_restore`进行还原。

- 如果是docker，可以直接复制`./postgres/`和`./redis/`，并对`./postgres/`赋权，参考[将站点从Docker迁移至Docker](https://pullopen.github.io/%E7%AB%99%E7%82%B9%E7%BB%B4%E6%8A%A4/2020/10/21/migrate-Mastodon-to-Docker.html) 

### 3. 转移媒体文件

这里分为两种情况，如果你已经配置了媒体文件在云存储，这一步是不需要的。如果你没有配置过，那么你的所有文件都在`./public/system/`目录下。具体而言有：

```
$ cd /home/mastodon/mastodon/public/system
$ du -h --max-depth=1
9.6M    ./media_attachments
3.3G    ./cache
20M     ./custom_emojis
13M     ./accounts
3.4G    .
```

其中`./cache`是外站存储，其他都是本站的数据。我们当然希望除了外站存储以外，对本站的所有数据都做好备份。

关于定期清除外站存储，请参考[这个嘟文](https://moe.cat/@AstroProfundis/104527119128844099)，我的crontab设置：

```
30 4    * * *   root    docker exec mastodon_web_1 tootctl media remove --days=14 >> /home/mastodon/log/mastodon/all.log 2>&1
30 4    * * *   root    docker exec mastodon_web_1 tootctl media remove-orphans >> /home/mastodon/log/mastodon/all.log 2>&1
30 4    * * *   root    docker exec mastodon_web_1 tootctl statuses remove --days=180 >> /home/mastodon/log/mastodon/all.log 2>&1
```

我是直接启动容器执行tootctl命令，更多相关内容可以通过[使用脚本的tootctl](https://mantyke.icu/2022/mastodon-media-file-cleanup/)或者[非docker安装的tootctl](https://www.notion.so/3f645c4a2ab14f34aef37703ee286d3a#d9a3235086fc44ceb3293cf26100a5ae)了解。

**总结：如果是docker的话，直接对除`./public/system/cache`以外整个文件夹和个别配置文件进行备份即可。**

## Where：备份放在哪里

### 本地备份

本文将备份文件和备份脚本都放在`/opt/mastodon-backup` 下，你也可以选择任何你喜欢的安全位置。

### 异地备份

最好的备份一定是异地备份。[pullopen](https://pullopen.github.io/%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA/2020/10/19/Mastodon-on-Docker.html#%E5%88%A9%E7%94%A8scaleway%E5%A4%87%E4%BB%BD%E6%95%B0%E6%8D%AE%E5%BA%93) 和 [o3o](https://www.notion.so/0f154999939e44109a5827e6c542fb53) 都推荐了Scaleway云备份，并附上了详细的资料。

## When：什么时候备份

建议设置每天晚上自动备份一次，可以使用crontab创建定时任务。

## Who：谁来备份

当然是站长啊还有谁！

## How：如何备份

啰嗦了这么久终于到了这一步，我只是希望在我真正操作之前能够明白我在做什么。

这一节脚本主要参考[利用Scaleway备份数据库](https://pullopen.github.io/%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA/2020/10/19/Mastodon-on-Docker.html#%E5%88%A9%E7%94%A8scaleway%E5%A4%87%E4%BB%BD%E6%95%B0%E6%8D%AE%E5%BA%93) 。

### 本地备份

只看脚本的前半部分：

```
#!/bin/bash
source /etc/profile
now=$(date "+%Y%m%d-%H%M%S")
origin="/home/mastodon/mastodon"
target="scaleway:backup.retirenow.top"
echo `date +"%Y-%m-%d %H:%M:%S"` " now starting export"
/usr/bin/docker exec mastodon_db_1 pg_dump -U postgres -Fc mastodon_production > ${origin}/backup.dump &&
echo `date +"%Y-%m-%d %H:%M:%S"` " succeed and upload to s3 now"
/usr/bin/zip -P back-up-password ${origin}/backup_${now}.zip ${origin}/backup.dump 
```

前面定义了一些路径和时间等，先通过`pg_dump`导出`backup.dump`，再将其加密压缩为`backup_${now}.zip`，但从上文可见，对于docker安装的情况，直接copy文件夹即可，所以我对这部分做了一些改动并参考[o3o的配置备份脚本](https://www.notion.so/0f154999939e44109a5827e6c542fb53#59063f5382174e638fc529602d810d32)，增加了一些关键信息的备份，我的完整**本地备份**脚本如下：

```
#!/bin/bash

# Save this file at  /opt/mastodon-backup/backup_local.sh
# Set a cronjob:  0 5 * * * root /bin/bash /opt/mastodon-backup/backup_local.sh > /opt/mastodon-backup/logs/backup_local.log 2>&1

#Loading /etc/profile
# source /etc/profile

#Set backup_folder and name
now=$(date "+%Y%m%d-%H%M%S")
origin_folder="/home/mastodon/mastodon"
backup_folder="/opt/mastodon-backup/dbbackup"

#Clean up old backup file
rm -r ${backup_folder}/*

#Generating a mastodon backup exclude cache
echo `date +"%Y-%m-%d %H:%M:%S"` " now starting export"
/usr/bin/zip -P 密码 -rqx=${origin_folder}/public/system/cache/* ${backup_folder}/backup_${now}.zip ${origin_folder}

#Copying important files
cp -r /etc/nginx/sites-available ${backup_folder}
echo `date +"%Y-%m-%d %H:%M:%S"` " done!"
```

可以单独运行一次脚本看看是否正确备份，运行后查看备份目录：

```
$ ls /opt/mastodon-backup/dbbackup/ -lh
total 76M
-rw-r--r-- 1 root root  76M Feb 20 00:09 backup_20220220-000931.zip
drwxr-xr-x 2 root root 4.0K Feb 20 00:09 sites-available
```

我将这个脚本存为`backup_local.sh`，在我配置好Scaleway之前守护我的数据，将其放在crontab下定期运行：

```
0 5 * * * root /bin/bash /opt/mastodon-backup/backup_local.sh.sh > /opt/mastodon-backup/logs/backup_local.sh.log 2>&1
```

备份时间设在清除外部缓存半小时后。

### 异地备份

再看原脚本的后半部分

```
/usr/bin/rclone copy ${origin}/backup_${now}.zip ${target} &&
echo `date +"%Y-%m-%d %H:%M:%S"` " ok all done"
rm -f ${origin}/backup.dump ${origin}/backup_${now}.zip
/usr/bin/rclone --min-age 7d  delete ${target}
```

`rclone copy`是将压缩后的备份文件按照配置好的设置上传至Scaleway，之后`rm -f`删除本地的备份文件，最后设置rclone删除大于7天的备份文件。这一步也可以直接在Scaleway删除，参考[循环销毁备份文件](https://www.notion.so/0f154999939e44109a5827e6c542fb53#59063f5382174e638fc529602d810d32)。

在我更改后的脚本，我采取的方法是每次先删除上一次的备份文件，而不是在上传后删除，这样可以在本地和Scaleway都有一个备份。

Scaleway和`rclone`的设置请直接参考[利用Scaleway备份数据库](https://pullopen.github.io/%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA/2020/10/19/Mastodon-on-Docker.html#%E5%88%A9%E7%94%A8scaleway%E5%A4%87%E4%BB%BD%E6%95%B0%E6%8D%AE%E5%BA%93) ，等我配好Scaleway会更新这个部分。



