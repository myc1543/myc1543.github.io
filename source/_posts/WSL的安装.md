---
title: WSL的安装
typora-root-url: WSL的安装
date: 2025-11-21 23:08:37
categories: [技术,Linux]
tags: [Linux]
description: windows11环境下安装WSL2
---



本篇文章参考微软的文档

https://learn.microsoft.com/zh-cn/windows/wsl/install



## 1. 安装WSL命令

### 1.1 管理员身份打开PowerShell

输入

```powershell
wsl --install
```

![](image-20251121223823461.png)

该命令会自动启用WSL相关的组件。接着重启电脑。

或者使用windows功能(启用或关闭Windows功能)，启用**适用于Linux的Windows子系统**

![image-20251121222954406](image-20251121222954406.png)



## 2. 设置WSL版本为WSL2

```
wsl --set-default-version 2
```



## 3.安装Linux发行版(注意安装位置)

建议从**Microsoft Store**安装稳定的Linux发行版，这里以Ubuntu 18.04为例

### 3.1 安装发行版

![](image-20251121223552582.png)



### 3.2 初始化信息

![](image-20251121223934668.png)

输入自己的用户名以及密码



## 4. 检查安装

### 4.1 查看已安装的发行版及其版本

```text
wsl -l -v
```



### 4.2 进入WSL

````
bash
````

命令行输入bash进入WSL，建议更换终端Hyper



## 5. WSL的迁移

如果Linux发行版安装到了C盘，并且C盘空间不足时，可以参考一下操作将其迁移到其他盘

### 5.1 停止WSL的运行

```
wsl -l -v
```

首先查看WSL的运行状态

![](image-20251121225104829.png)

如果WSL正在运行，首先终止其运行

```
wsl --shutdown
```

确保处于stopped的状态



### 5.2 导出/恢复备份

在新的盘中创建一个新的目录来存放迁移后的WSL，这里以F盘为例，新的目录为WSL

```
1.导出备份，命名为Ubuntu.tar
wsl --export Ubuntu-{版本} F:\WSL\Ubuntu.tar
2.确定新的目录有备份文件后，注销原有文件
wsl --unregister Ubuntu-{版本}
3.恢复备份文件
wsl --import Ubuntu-{版本} F:\WSL F:\WSL\Ubuntu.tar
```



![](image-20251121230524992.png)

