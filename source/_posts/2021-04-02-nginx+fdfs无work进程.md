---
title: nginx+fdfs无work进程
tags: 
  - nginx
  - FastDFS
toc: true
abbrlink: 28017
categories:
  - Nginx
date: 2021-04-02 22:00:00
---









### 问题描述：

nginx 按装fastdfs-nginx-module模块后，启动nginx，然后发现有两个master进程，没有work进程。

搜索了一圈，也没发现有价值的信息。重装了几次nginx且为安装fdfs模块，这个问题依旧存在。。

查看nginx的错误日志发现，work进程因为某种原因异常退出了。

```
2021/04/02 22:20:33 [alert] 34290#0: worker process 44752 exited on signal 11 (core dumped)
ngx_http_fastdfs_process_init pid=44772
2021/04/02 22:20:36 [alert] 34290#0: worker process 44772 exited on signal 11 (core dumped)
ngx_http_fastdfs_process_init pid=44799
2021/04/02 22:20:40 [alert] 34290#0: worker process 44799 exited on signal 11 (core dumped)
ngx_http_fastdfs_process_init pid=44819
2021/04/02 22:20:43 [alert] 34290#0: worker process 44819 exited on signal 11 (core dumped)
ngx_http_fastdfs_process_init pid=44839
```

又搜索一圈，终于遇到到了和我一样的问题。[传送门](https://cychan811.gitee.io/2020/08/21/nginx-fastfds%E9%85%8D%E7%BD%AE%E5%AE%8C%E6%88%90%E5%90%8Eworker%E6%8A%A5%E9%94%99exited-on-signal-11-core-dumped-%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/)

### 错误原因：

因为nginx已经按装了fdfs模块，但是fdfs的tracker没有启动，导致nginx的work进程启动出错并退出了，接着master进程重启新的work进程，从日志文件中也可以看出，nginx循环打印这个错误。

### 解决方法：

在`/etc/init.d`目录下执行这条命令，启动tracker和storage：

```shell
sudo fdfs_trackerd /etc/fdfs/tracker.conf
sudo fdfs_storaged /etc/fdfs/storage.conf
```

并将tracker 和 storage加入开机启动：

```shell
sudo update-rc.d fdfs_trackerd defaults 99
sudo update-rc.d fdfs_storage defaults 99
```

ps：最后的数字代表启动顺序（0~99），数字越小，优先级越高。

