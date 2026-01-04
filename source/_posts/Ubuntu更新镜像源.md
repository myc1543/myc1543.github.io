---
title: Ubuntu更新镜像源
typora-root-url: Ubuntu更新镜像源
date: 2025-11-23 23:03:48
tags: [Linux]
categories: [技术,Linux]
description: 在Ubuntu环境中替换Ubuntu的软件源为镜像源，提高下载速度。
---

### 1. Ubuntu软件源配置文件

```sudo cd /etc/apt
sudo cd /etc/apt
```

Ubuntu的软件源配置文件在`/etc/apt`的`sources.list`中



### 2.备份配置文件

```
sudo cp sources.list sources.list.bak
```

备份一份配置文件，以便以后需要时找回



### 3.修改配置文件

```
sudo vi sources.list
```

依次按下`g` `d` `shift` `g`删除所有内容

````
# aliyun
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
````

复制对应版本的镜像源，我这里是Ubuntu22.04 阿里镜像源，其余版本可自行Google

按下`:` `wq` 保存并退出



### 4.更新软件源

```
sudo apt-get update
sudo apt-get upgrade
```

更新软件源

