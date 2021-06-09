---
title: 将镜像发布到DockerHub
tags: Docker
toc: true
abbrlink: 53659
categories:
  - Docker
date: 2021-01-10 00:00:00
---























步骤1：在DockerHub上注册帐号。

步骤2：本地登录DockerHub帐号

> docker login

步骤3：使用`docker tag`命令将镜像重新命令并设置版本号

docker tag命令使用方式：

> docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

如果要发布镜像到DockerHub，则TARGET_IMAGE的格式为 <font color="red">namespace/imagename</font>。

namespace就是DockerHub的用户名

步骤4：使用命令提交镜像

> docker push TARGET_IMAGE[:TAG]