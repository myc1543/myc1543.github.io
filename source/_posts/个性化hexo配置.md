---
title: 个性化Hexo配置 
date: 2025-11-23 15:40:54
tags: [Hexo]
categories: [技术,工具]
description: 在Hexo中配置Next主题，并且设置一些个性化元素。
typora-root-url: ./个性化hexo配置
---

## 1.Hexo配置Next主题

### 1.1 检查Hexo配置

输入`hexo v`命令，出现如下信息，表示Hexo已安装，可以进行后续操作

```
$ hexo v
INFO  Validating config
hexo: 8.1.1
hexo-cli: 4.3.2
os: win32 10.0.26200 undefined
node: 24.11.1

```



### 1.2 安装Next主题

- 进入站点文件夹，也就是包含`themes` `source` `_config.yml`的文件夹

```
npm install hexo-theme-next
```

下载next主题

- **创建配置文件**

站点文件夹下，复制hexo-theme-next下的`_config.yml`，命名为_config.next.yml

```
cp node_modules/hexo-theme-next/_config.yml _config.next.yml
```

- **为什么不直接使用next主题下的配置文件？**
  - 在升级主题时，对这些默认配置文件所做的自定义修改可能会导致冲突或被覆盖，从而造成数据丢失。



### 1.3 配置Next主题

Hexo配置文件`_config.yml`找到`theme`选项，改为`next`(主题名称)

```
theme: next
```



### 1.4 清理缓存

```
hexo clean
```

清理Hexo的缓存，生成本地文件，运行。运行的效果看起来和图片相似，说明安装成功

![](next-default-scheme.png)

## 2.个性化设置

可以对Hexo以及Next作一些设置，满足自己的需求

### 2.1 Hexo配置

```
title: Hexo
subtitle: ''
description: ''
keywords:
author: John Doe
language: en 
timezone: '' 
```

- title：网站的名字
- subtitle：网站的副标题
- description：描述自己的一些关键字
- keywords：网站的关键字
- author：网站的作者
- language：网站的语言
- timezone：网站对应的时区，默认使用电脑的时区，如果发布到服务器上，最好手动设置时区，国内的时区一般设置为`Asia/Shanghai`



### 2.2 添加分类与标签

Hexo支持为文章添加分类`categories`与标签`tags`

- 首先，配置文章模板，进入`/scaffolds`文件夹，打开post文件，添加以下配置，可以在创建文章时自动添加模板

```
categories: 
tags: 
```

- `[可选]`打开`_config.next.yml`，找到`menu`，删除tags和categories的注释，主要应用于菜单栏

```
menu:
  #home: / || fa fa-home
  #about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  #archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```



- **创建分类并添加type属性**

在站点所在文件夹

```
hexo new page categories
```

成功后提示

```
INFO  Created: ~/Documents/blog/source/categories/index.md
```

找到上面`index.md`文件

默认内容为

```
---
title: categories
date: 2025-11-23 13:47:40
---
```

添加`type`内容

```
---
title: categories # 可以修改title标题
date: 2025-11-23 13:47:40
type: “categories”
---
```



- **添加标签并添加type属性**

和添加分类类似

```
hexo new page tags
```

成功后提示

```
INFO  Created: ~/Documents/blog/source/tags/index.md
```

打开index.md，添加type属性

```
title: 标签
date: 2025-11-23 15:38:49
type : "tags"
```

- **使用分类和标签**

```
title: 个性化Hexo配置 
typora-root-url: test
date: 2025-11-23 15:40:54
tags: [工具,hexo]
categories: [技术,工具]
description: 在Hexo中配置Next主题，并且设置一些个性化元素。
```

分类是一个层级目录，到代表`技术` -> `工具`，一级分类为`技术`，二级分类为`工具`，[c1,c2,c3]，从左到右，层级为`/c1/c2/c3`

标签是一个同级的目录，没有上下级的区分。 [t1,t2,t3]

在创建分类和标签时，如果不存在，会自动创建；如果已存在，会自动合并



### 2.3 设置next主题

next的主题默认为`Muse`，可以编辑`_config.next.yml`来更改

```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
# scheme: Muse
# scheme: Mist
#scheme: Pisces
scheme: Gemini
```



### 2.4 设置头像

- 在站点文件夹中，进入`/source`目录，创建文件夹`images`，在`images`中将要保存的头像命名为`avatar.jpg`

- 修改`_config.next.yml`的`Avatar`选项

````
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.jpg
  # If true, the avatar will be displayed in circle.
  rounded: false
  # If true, the avatar will be rotated with the cursor.
  rotated: false

````



### 2.5 搜索功能

- 安装插件

```
npm install hexo-generator-searchdb --save
```

- 修改`_config.yml`，增加以下内容

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000

```

- 编辑`_config.next.yml`

```
# Local search
local_search:
  enable: true

```



### 2.6 增加字数统计以及阅读时间

- 安装插件

```
npm install hexo-symbols-count-time --save
```

- 修改`_config.next.yml`配置

```
symbols_count_time:
  separated_meta: true # 是否换行显示 字数统计 及 阅读时长
  item_text_post: true # 文章 字数统计 阅读时长 使用图标 还是 文本表示
  item_text_total: false # 博客底部统计 字数统计 阅读时长 使用图标 还是 文本表示
  awl: 4 # awl（Average Word Length）的数值是设定多少字符统计为一个字（word），中文博客建议设置为 2
  wpm: 275 # （Words Per Minute）是假设的读者阅读速度，多少字（word）统计为阅读时长一分钟。
```

官方文档的参考值：

- 慢速：200
- 中速：275
- 快速：350





## 3.其他

Hexo以及Next还支持其他的有意思的配置，读者可根据自身需要选择，比如[SEO引擎优化](https://pengtech.net/hexo/blog_seo_optimize.html?t=1763890310451)、[图标功能](https://pengtech.net/hexo/hexo_mermaid_plugin.html?t=1763890368545)、[评论](https://pengtech.net/hexo/hexo_blog_comments.html?t=1763890399783)、[官方文档](https://theme-next.js.org/)中也有一些进阶用法。
