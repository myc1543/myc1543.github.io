---
title: 使用hexo和github创建自己的博客

date: 2025-11-22 11:42:53
categories: [技术,工具]
tags: [Hexo]
typora-root-url: ./使用hexo和github创建自己的博客
description: 在windows11环境下，完成hexo以及git的配置，搭建自己的博客。
---





本篇文章参考[**Hexo官方文档**](https://hexo.io/zh-cn/docs/)、[**Hexo博客如何插入图片**](https://zhuanlan.zhihu.com/p/265077468)、 [**Github Pages**](https://docs.github.com/en/pages/quickstart)，一步一步搭建自己的博客。

## 1.前置准备

安装Hexo需要先安装Node.js和Git

### 1.1 安装Node.js

[**node官网**](https://nodejs.org/zh-cn)，推荐下载12.0及以上的LTS版本(默认包含npm工具包)，一路安装即可。

输入``node -v`` `npm -v`验证是否安装成功

![](image-20251122143108724.png)



### 1.2 安装git(已经配置的可跳过，后续命令都在git中完成)

[**git官网**](https://git-scm.com/)，一路安装即可。

```
git --version  # 检查是否已经安装git
```



**配置用户名、密码以及密钥**

```java
 git config --global user.name "用户名"
 git config --global user.email "github绑定的邮箱"
```

```text
 # 生成 ssh 密钥
 ssh-keygen -t rsa -C "github 注册邮箱"
```

使用生成ssh密钥的命令后，会生成`id_rsa` (私有)`id_rsa.pub`（公有）两个文件，打开`id_rsa.pub`中的密钥，复制到`GitHub-Settings-SSH and GPG Keys`页面，创建一个新的SSH key

![](image-20251122141133630.png)

```
ssh -T git@github.com  # 测试SSH连接
```

如果成功，会显示Hi，xxx! You've successfully authenticated...

## 2.安装Hexo并初始化

使用以下命令安装hexo相关工具

```
npm install -g hexo-cli
```

**初始化项目**

1. 创建一个新文件夹用来存放博客，并执行hexo init命令
2. 执行npm install安装必备组件

```
hexo init <folder>
cd <folder>
npm install
```

初始化后，目录结构如图所示

```
.
 ├── _config.yml # 网站配置信息
 ├── package.json # 应用程序信息
 ├── scaffolds # 模板文件夹
 ├── source # 存放用户资源
 |   ├── _drafts
 |   └── _posts
 └── themes # 主题文件夹
```



**本地部署博客**



1. 新建博客

```
hexo new "博客名"
```

执行此命令会在/source/_posts目录下创建一个md文件，用markdown编辑器即可对博客进行编辑

2. 生成静态网页

```
hexo g
```

3. 打开本地服务器

```
hexo s
```

在浏览器打开``http://localhost:4000``，就可以看到部署的网页

![](image-20251122151002542.png)

## 3. 部署到Github Pages

如果博客需要其他人也访问，则需要部署到服务器上，在这里选择使用Github Pages，是免费的。访问的地址是`https://用户名.github.io`

### 3.1 新建一个仓库

**注意点**

- 仓库一定是`public`的，否则别人访问不到

- 仓库名一定是`用户名.github.io`

  ![](image-20251122145400207.png)





### 3.2 更改hexo配置

打开目录下_config.Yaml文件

![](image-20251122145713620.png)

找到Deployment选项

```
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: 'git'
  repository: git@github.com:myc1543/myc1543.github.io.git
  branch: main

```

- 修改type项为git
- repository项为仓库地址
- branch为main

注意每项配置`:`后有空格



### 3.3 远程部署

```
hexo g ## 生成静态网页
hexo d ## 部署到Github Pages
```



访问`https://用户名.github.io` 即可



## 4.hexo中插入图片

用md有个蛋疼的问题，就是对图片的展示不是很友好。

### 4.1 插入图片

在md中插入图片的语法为`![图片描述](图片路径)`

这里图片路径有三种方式

- 绝对路径
- 相对路径
- 网络路径

对应的，在hexo中使用图片资源，除了网络路径无需更改，相对路径和绝对路径都是需要先在资源文件下存放， 才能正确显示。

**(1)绝对路径引用**

绝对路径使用需要在`/source` 文件夹下创建一个资源文件夹如`/source/pictures/`,作为全局资源文件夹

引用方式为`![](/pictures/picture1.jpg)`

**(2)相对路径引用**

hexo提供了一个资源文件管理功能，通过config.yml文件中的`post_assert_folder`设置为true来打开。此时当创建一个新的md博客，会同步创建一个相同名字的文件夹，在博客中之需要使用相对路径引用图片即可。

举个例子：创建了一个新的博客为blog.md，目录结构为

```
- source
	- _posts
		- blog.md
		- blog(文件夹)
```

blog文件夹中存放了1.png，引用方式为`![](1.png)`

**预览不正确的问题**

但这时，直接使用相对路径引入的图片，只能在**博文详情页**展示，在首页或者存档页的预览会不正确，此时应该改为`![](% assert_img 1.png %)`

这时候可以引入`hexo-renderer-marked`插件来解决这个问题

使用`npm install hexo-renderer-marked`命令直接安装，之后在`config.yaml`中更改配置如下：

```yaml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

### 4.2 配合Typora来简化操作

这时候会发现一个问题，就是每次我想使用图片的时候，都需要先在资源文件下存放图片，才能使用，这对复制黏贴的截图来说很不友好。

Typora的设置可以自动将插入的图片复制一份到文章资源文件夹

``文件-偏好设置-图像``，设置为复制到指定路径`/.${filename}`，并且设置优先显示相对路径

![](image-20251122173743632.png)

这时候图片的路径就为`文章名/picture.jpg`。但是hexo的路径应该是没有前面的前缀的，用全局搜索替换`文章名/`为空即可。

此时本地typora就显示不出图片，可以设置`格式-图像-设置图像根目录`来解决

![image-20251122174149386](image-20251122174149386.png)





### 4.3 插入图片的总结

1. 下载`hexo-renderer-marked`插件并修改`config.yaml`配置
2. 设置Typora插入图片设置(`插入到指定路径` `优先使用相对路径`)

![](image-20251122174615272.png)

3. 使用`<C-f>`快捷键，将所有的`文章名/`替换为空

![](image-20251122174748275.png)

4. 设置图片根目录，在本地编辑器显示
