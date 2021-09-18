---
title: 解决 docker no space left on device
tags:
  - docker
categories:
  - Workshop
date: 2021-09-16 15:14:57
---


操作系统为 CentOS Linux release 7.2，试过网上的一些方法例如软链接，增加graph路径都没有用，自己摸索找到了配置路径的位置，记录一下。

1. 使用`df -h` 查看磁盘状况：

```bash
$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/vda1                      99G   91G  3.4G  97% /
/dev/vdb1                     493G   62G  406G  14% /data
overlay                        99G   91G  3.4G  97% /var/lib/docker/overlay2/xxx
```

可以看到`/var/lib/docker/`放满了，可以移到`data`目录下。

同时可以先确认一下目前的 docker 目录：

```bash
$ docker info
...
Docker Root Dir: /var/lib/docker
...
```

2. 清理无用的内容

```bash
$ docker system prune
```

可添加`-a`清除得更彻底。之后查看磁盘状况：

```bash
$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/vda1                      99G   83G   12G  88% /
/dev/vdb1                     493G   62G  406G  14% /data
overlay                        99G   83G   12G  88% /var/lib/docker/overlay2/xxx
```

有所缓解，但12G依然不够用，需要更改 docker 目录到 `/data`下。

3. 停止docker服务

```bash
$ systemctl stop docker
```

4. 创建新的docker工作目录

```bash
$ mkdir -p /data/dockerlib
```

5. 迁移/var/lib/docker

```text
rsync -avz /var/lib/docker /data/dockerlib
```

6. 改动配置文件

```bash
$ vim /etc/docker/daemon.json
```

在文件中更改

```properties
"data-root": "/data/dockerlib/docker",
```

7. 重启docker服务

```bash
systemctl daemon-reload
 
systemctl restart docker
 
systemctl enable docker
```

8. 确认是否配置成功

```bash
$ docker info
...
Docker Root Dir: /data/dockerlib/docker
...
```

9. 确认 docker 已经成功转移

修改原 docker 路径名，再进行一些操作

```bash
$ mv /var/lib/docker /var/lib/docker_temp
$ docker image list
$ docker container list
```

在这一步我发现之前的 container 进去会有问题，但是重启后可以重新进入：

```bash
$ docker stop xxx
$ docker start xxx
$ docker exec -it xxx /bin/bash
```

确认无误后可以删除，为了防止出错可以用一阵子再删

```bash
$ rm -rf /var/lib/docker_temp
```

