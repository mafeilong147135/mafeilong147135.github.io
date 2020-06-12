---
title: docker容器时区问题
date: 2020-06-12 15:53:11
tags: docker
typora-root-url: 2019-06-12-docker-zone-primer
categories: docker
toc: true
summary: 容器时间与Linux主机时间不一致的解决方案
top: true
---

## Docker 解决容器时间与Linux主机时间不一致的问题三种解决方案

Docker容器时间与主机时间不一致
通过date命令分别查看容器和linux的时间

```shell
# 进入docker 容器  
docker exec -it tjfx8006 /bin/sh
```

CST应该是指（China Shanghai Time，东八区时间） 
UTC应该是指（Coordinated Universal Time，标准时间） 
所以，这2个时间实际上应该相差8个小时。(ps: 所以没有设置过的容器, 一般跟宿主机时间相差8h)
所以，必须统一两者的时区。

### Dockerfile 时区设置

```dockerfile
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' > /etc/timezone
```

### Java 时区设置

**注意**, 如果能确保所在主机/etc/timezone内容正确,则不需要再对Java时区进行设置

```java
TimeZone tz = TimeZone.getTimeZone("Asia/Shanghai");
TimeZone.setDefault(tz);
```

### Linux 时间知识

windows的系统时间就是硬件时间,除了windows的系统(如Linux,Mac)的系统时间是硬件时间结合时区.
也就是说,对于Linux系统来说,硬件时间是 UTC 标准时间,本地时间需要结合时区计算出来,在linux中与时间相关的文件有 /etc/localtime和 /etc/timezone。其中，/etc/localtime是用来描述本机时间，而 /etc/timezone是用来描述本机所属的时区。

### Linux 修改本机时间

旧版本(如CentOS6)

```shell
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

新版本(如CentOS7)

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 原因

​       原本的时区文件变成了链接文件,直接使用cp就相当于把原时区文件内容给覆盖掉,而且文件名不变! 例如,原本的 /etc/localtime 是链接到 /usr/share/zoneinfo/America/New_York 的,使用

```shell
cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```

之后原本New_York 的内容就变成了 Shanghai 的内容,文件名不变.

### 注意

​      调整了时间格式，本机所属的时区仍保持不变.
​      Linux 修改本机时区在linux中，有一些程序会自己计算时间，不会直接采用带有时区的本机时间格式，会根据UTC时间和本机所属的时区等计算出当前的时间。 比如jdk应用，时区为“Etc/UTC”，本机时间改为北京时间，通过java代码中new 出来的时间还是utc时间，所以必须得修正本机的时区。

```dockerfile
echo 'Asia/Shanghai' > /etc/timezone
```

