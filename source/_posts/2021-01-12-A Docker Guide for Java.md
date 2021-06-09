---
title: A Docker Guide for Java
tags: Docker
toc: true
abbrlink: 59967
categories:
  - Docker
date: 2021-01-12 00:00:00
---













参考资料：https://www.baeldung.com/docker-java-api

## 1.概述

在这篇文章中，我们看一下另一个建立已久的平台的具体的API----Java API Client for Docker

通过这篇文章，我们理解如何连接一个正在运行的Docker守护进程的方式，还有这个API为Java开发人员提供了什么样的重要智能。

## 2.Maven Dependency

首先，我们需要添加一个主要的依赖到我们的pom.xml文件中：

```shell
<dependency>
    <groupId>com.github.docker-java</groupId>
    <artifactId>docker-java</artifactId>
    <version>3.0.14</version>
</dependency>
```

写这文章的时候，这个API的最新版本是3.0.14。每个发行版可以从[the github release page](https://github.com/docker-java/docker-java/releases)或者 [the maven repository](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.github.docker-java%22%20AND%20a%3A%22docker-java%22)浏览。

## 3.使用Docker Client

DockerClient 是一个我们在Docker 引擎/守护进程和我们应用程序之间建立连接的地方。

默认情况下，Docker 守护进程只能在<font color="red">*unix:///var/run/docker.sock* file</font>访问。在本地，我们可以通过监听**the Unix socket**与Doocker引擎交流，并且这样无需任何配置。

在这，我可使用*DockerClientBuilder*类去创建一个连接，这个连接使用的是默认的配置：

> <font color="blue">DockerClient dockerClient = DockerClientBuilder.getInstance().build();</font>

简单的，我们可以使用以下两步打开一个连接：

> <font color="blue">
> DefaultDockerClientConfig.Builder config 
> = DefaultDockerClientConfig.createDefaultConfigBuilder();</font>
>
> <font color="blue">DockerClient dockerClient = DockerClientBuilder
> .getInstance(config)
> .build();
> </font>

由于引擎可以依赖其他特征，客户端也可以在不同条件下配置。

例如，the builder 接受一个server URL，如果引擎在端口2375可用的话，我们能更新连接值。

> <font color="blue">
> DockerClient dockerClient
> = DockerClientBuilder.getInstance("tcp://docker.baeldung.com:2375").build();
> </font>

注意，我们需要在<font color="red">连接字符串 (connection string)</font>前加上<font color="purple">unix://</font> 或者 <font color="purple">tcp://</font>，这依赖于<font color="red">连接类型 (connection type)</font>。

进一步，我们能以一个更高级的配置，这个配置使用*DefaultDockerClientConfig*类。

> DefaultDockerClientConfig config
>  = DefaultDockerClientConfig.createDefaultConfigBuilder()
>    .withRegistryEmail("info@baeldung.com")
>     .withRegistryPassword("baeldung")
>     .withRegistryUsername("baeldung")
>     .withDockerCertPath("/home/baeldung/.docker/certs")
>     .withDockerConfig("/home/baeldung/.docker/")
>     .withDockerTlsVerify("1")
>     .withDockerHost("tcp://docker.baeldung.com:2376").build();
>    
> DockerClient dockerClient = DockerClientBuilder.getInstance(config).build();
> 

同样地，我们可以使用Properties执行相同的方法。

> 
> Properties properties = new Properties();
> properties.setProperty("registry.email", "info@baeldung.com");
> properties.setProperty("registry.password", "baeldung");
> properties.setProperty("registry.username", "baaldung");
> properties.setProperty("DOCKER_CERT_PATH", "/home/baeldung/.docker/certs");
> properties.setProperty("DOCKER_CONFIG", "/home/baeldung/.docker/");
> properties.setProperty("DOCKER_TLS_VERIFY", "1");
> properties.setProperty("DOCKER_HOST", "tcp://docker.baeldung.com:2376");
> 
> DefaultDockerClientConfig config
>   = DefaultDockerClientConfig.createDefaultConfigBuilder()
>     .withProperties(properties).build();
> 
> DockerClient dockerClient = DockerClientBuilder.getInstance(config).build();
> 

除非我们在源代码里面配置引擎设置，否则另一个选择是设置想符合的环境变量，以至于我们只考虑项目中DockerClient的默认的实例化。

> export DOCKER_CERT_PATH=/home/baeldung/.docker/certs
> export DOCKER_CONFIG=/home/baeldung/.docker/
> export DOCKER_TLS_VERIFY=1
> export DOCKER_HOST=tcp://docker.baeldung.com:2376

## 4.容器管理

这个API给予我们关于容器管理的各种各样的选择。让我们看看每一个。

### 4.1容器列表 List Container

现在，我们已经建立了连接，我们能列出位于Docker主机上的所有正在运行的容器。

> <font color="blue">List<Container> containers = dockerClient.listContainersCmd().exec();</font>

假如显示正在运行的容器不满足要求，我们能使用选项查询容器。

> List<Container> containers = dockerClient.listContainersCmd()
> .withShowSize(true)
>   .withShowAll(true)
>   .withStatusFilter("exited").exec()

它就等价于：

> $ docker ps -a -s -f status=exited
> or 
>  $ docker container ls -a -s -f status=exited

### 4.2创建容器

创建容器由*createContainerCmd*方法提供。我们可以使用以“with”为前缀的方法进行更复杂的声明。

假定我们有一个docker create 命令，定义了一个依赖于主机的MongoDB容器并在容器内部监听27017端口：

> ```java
> $ docker create --name mongo \
>   --hostname=baeldung \
>   -e MONGO_LATEST_VERSION=3.6 \
>   -p 9999:27017 \
>   -v /Users/baeldung/mongo/data/db:/data/db \
>   mongo:3.6 --bind_ip_all
> ```

我们能以变成的方式启动同样的容器，并对其进行配置：

> ```java
> CreateContainerResponse container
>   = dockerClient.createContainerCmd("mongo:3.6")
>     .withCmd("--bind_ip_all")
>     .withName("mongo")
>     .withHostName("baeldung")
>     .withEnv("MONGO_LATEST_VERSION=3.6")
>     .withPortBindings(PortBinding.parse("9999:27017"))
>     .withBinds(Bind.parse("/Users/baeldung/mongo/data/db:/data/db")).exec();
> ```

### 4.3 start、stop和kill container

一旦我们创建了容器，我们就能通过容器名字name或者容器id，启动start、停止stop和杀死kill这个容器：

> ```java
> dockerClient.startContainerCmd(container.getId()).exec();
> 
> dockerClient.stopContainerCmd(container.getId()).exec();
> 
> dockerClient.killContainerCmd(container.getId()).exec();
> ```

### 4.4 查看inspect 容器

*inspectContainerCmd* 方法有一个String类型的参数，这个参数指明容器的name或者id。使用这个方法，我们能直接地观察容器的元数据metadate：

```
InspectContainerResponse container 
  = dockerClient.inspectContainerCmd(container.getId()).exec();
```

4.5 Snapshot 快照 容器

类似与*docker commit* 命令，我们能使用*commitCmd*方法创建一个新的镜像。

在我们的例子中，这个场景是，我们先前运行一个 **alpine:3.6**容器，它的id是“***3464bb547f88\***”，并且在它的顶部安装了git。

现在，我们想从这个容器中创建一个新的镜像快照：

```
String snapshotId = dockerClient.commitCmd("3464bb547f88")
  .withAuthor("Baeldung <info@baeldung.com>")
  .withEnv("SNAPSHOT_YEAR=2018")
  .withMessage("add git support")
  .withCmd("git", "version")
  .withRepository("alpine")
  .withTag("3.6.git").exec();
```

由于与git捆绑的新镜像仍保留在主机上，我们能在Docker主机上搜索到它：

```
$ docker image ls alpine --format "table {{.Repository}} {{.Tag}}"
REPOSITORY TAG
alpine     3.6.git
```

## 5.镜像管理

我们提供了一些适用的命令来管理镜像操作。

### 5.1列出镜像

列出Docker主机上的所有镜像（包括悬挂镜像），我们需要应用的是*listImagesCmd*命令：

```
List<Image> images = dockerClient.listImagesCmd().exec();
```

如果在Docker主机上我们有两个镜像，我们应该在运行时获得它们的***Image\***对象。我们要寻找的镜像是：

```
$ docker image ls --format "table {{.Repository}} {{.Tag}}"
REPOSITORY TAG
alpine     3.6
mongo      3.6
```

接下来，要看中间镜像，我们需要明确地请求它：

```
List<Image> images = dockerClient.listImagesCmd()
  .withShowAll(true).exec();
```

如果在这个情况下，只显示悬挂镜像，必须考虑*withDanglingFilter*方法：

```
List<Image> images = dockerClient.listImagesCmd()
  .withDanglingFilter(true).exec();
```

### 5.2构建镜像

我们专注于使用API方式构建镜像。用**buildImageCmd**方法从Dockerfile中构建镜像。在我们的项目中，我们已经有了一个Dockerfile文件，这个Dockerfile定义了一个 安装了git的Alpine镜像：

```
FROM alpine:3.6

RUN apk --update add git openssh && \
  rm -rf /var/lib/apt/lists/* && \
  rm /var/cache/apk/*

ENTRYPOINT ["git"]
CMD ["--help"]
```



在构建进程开始之前，新镜像无需使用缓存就能被创建，在任何情况下，docker 引擎将尝试拉取较新版本的*alpine:3.6*。如果一切顺利，最终我们应该看到具有给定名称的**alpine:git**镜像：

```
String imageId = dockerClient.buildImageCmd()
  .withDockerfile(new File("path/to/Dockerfile"))
  .withPull(true)
  .withNoCache(true)
  .withTag("alpine:git")
  .exec(new BuildImageResultCallback())
  .awaitImageId();
```

### 5.3查看镜像

使用*inspectImageCmd*方法，我们可以查看镜像的底层信息：

```

```

```java
InspectImageResponse image 
  = dockerClient.inspectImageCmd("161714540c41").exec();
```

### 5.4给图像版本号

使用*docker* *tag* 命令给我们的镜像添加版本号是非常简单的，这个API也不例外。我们可以使用*tagImageCmd* 命令来完成同样的目的。使用git将镜像id为***161714540c41\***的镜像标记到**baeldung/alpine**库中，请执行以下操作：

```
String imageId = "161714540c41";
String repository = "baeldung/alpine";
String tag = "git";

dockerClient.tagImageCmd(imageId, repository, tag).exec();
```

我们可以列出最新创建的镜像，如下所示：

```

```

```bash
$ docker image ls --format "table {{.Repository}} {{.Tag}}"
REPOSITORY      TAG
baeldung/alpine git
```

### 5.5推送镜像

在推送镜像到注册服务之前，docker client必须配置，以便与这个服务协作，因为与注册服务工作需要提前获取授权。

因为假定我们的客户端配置了Docker Hub，所以我们可以将*baeldung/alpine*镜像推送到baeldung DockerHub账户上。

```
dockerClient.pushImageCmd("baeldung/alpine")
  .withTag("git")
  .exec(new PushImageResultCallback())
  .awaitCompletion(90, TimeUnit.SECONDS);
```

我们必须容忍持续的进程。在这个例子中，我们等待了90秒。

### 5.6拉取镜像

从注册服务中拉取镜像，我们利用*pullImageCmd*方法。另外，如果镜像是从私有库中拉取，客户端必须知道我们的证书，否则这个过程就会停止并报错。等同于拉取镜像，我们指定一个回调和一个固定的周期拉取镜像：

```
dockerClient.pullImageCmd("baeldung/alpine")
  .withTag("git")
  .exec(new PullImageResultCallback())
  .awaitCompletion(30, TimeUnit.SECONDS);
```

拉取完成后，检查Docker主机中是否存在我们提及的镜像：

```
$ docker images baeldung/alpine --format "table {{.Repository}} {{.Tag}}"
REPOSITORY      TAG
baeldung/alpine git
```

### 5.7删除镜像

剩下的函数中的另一个简单的函数是*removeImageCmd*方法。我们可以使用它的短或长ID删除镜像：

```
dockerClient.removeImageCmd("beaccc8687ae").exec();
```

### 5.8在注册中搜索

从Docer Hub上搜索镜像，客户端附带一个*searchImagesCmd*方法，这个方法要指定一个字符串类型的值，这个值指示了要搜索的词。在这，我们探索Docker Hub中包含‘java’的镜像。

```
List<SearchItem> items = dockerClient.searchImagesCmd("Java").exec();
```

输出返回的*SearchItem*对象中列出了前25过热相关的镜像。

## 6.卷管理

如果java项目需要使用卷与Docker进行交互，我们也应该重视这一章节。简短地，我们查看由Docker Java API提供的卷的基本技术。

### 6.1列出卷

包括有名的和无名的所有可用的卷都被列出：

```


ListVolumesResponse volumesResponse = dockerClient.listVolumesCmd().exec();
List<InspectVolumeResponse> volumes = volumesResponse.getVolumes();
```



### 6.2查看卷

*inspectVolumeCmd*方法是显示一个卷详细信息的窗体。我们通过指定卷id查看卷：

```
InspectVolumeResponse volume 
  = dockerClient.inspectVolumeCmd("0220b87330af5").exec();
```

### 6.3创建卷

这个API提供了两个不同的选项进行创建卷。无参数的*createVolumeCmd*方法创建一个卷，这个卷的名字由Docker生成：

```
CreateVolumeResponse unnamedVolume = dockerClient.createVolumeCmd().exec();
```

要不使用默认的行为，这个*withName*的方法允许我们给卷命名：

```java
CreateVolumeResponse namedVolume 
  = dockerClient.createVolumeCmd().withName("myNamedVolume").exec();
```

### 6.4删除卷

我们可以使用*removeVolumeCmd*方法从Docker主机中直观地删除一个卷。要注意的是，如果一个卷正在被容器使用，我们不能删除这个卷，这点很重要。我们从卷列表中删除卷，*myNamedVolume*：

```
dockerClient.removeVolumeCmd("myNamedVolume").exec();
```

## 7.网络管理

我们最后一节是关于用API管理网络任务。

### 7.1列出网络

我们可以使用一种传统的API方法，列出网络单元的列表，从列表开始：

```
List<Network> networks = dockerClient.listNetworksCmd().exec();
```

### 7.2创建网络

执行*createNetworkCmd*命令就等价于*docker network create* c命令。如果我们有三部分或者一个客户网络驱动，*withDriver*方法能接受这些除了嵌入驱动。在我们的例子中，让我们创建一个桥接网络，它的名字是*baeldung*：

```java
CreateNetworkResponse networkResponse 
  = dockerClient.createNetworkCmd()
    .withName("baeldung")
    .withDriver("bridge").exec();
```

更进一步，使用默认设置创建一个网络单元不能解决问题，我们可以应用其他辅助方法来构建高级网络。因此，用自定义值覆盖默认子网络：

```
CreateNetworkResponse networkResponse = dockerClient.createNetworkCmd()
  .withName("baeldung")
  .withIpam(new Ipam()
    .withConfig(new Config()
    .withSubnet("172.36.0.0/16")
    .withIpRange("172.36.5.0/24")))
  .withDriver("bridge").exec();


```

我们可以使用docker命令运行的相同命令是：

```
$ docker network create \
  --subnet=172.36.0.0/16 \
  --ip-range=172.36.5.0/24 \
  baeldung
```

### 7.3查看网络

展示网络的底层详细信息也包含在API中：

```
Network network 
  = dockerClient.inspectNetworkCmd().withNetworkId("baeldung").exec();
```

### 7.4删除网络

我们能使用*removeNetworkCmd*方法根据网络名称和id安全地删除网络单元：

```java
dockerClient.removeNetworkCmd("baeldung").exec();
```

## 8总结

在这个广泛的教程中，我们探索*Java Docker API Client*中各种各样的功能，以及用于部署和管理场景的部署方法。

