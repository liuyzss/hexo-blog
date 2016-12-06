---
title: hexo wiki
date: 2016-12-06 11:49:52
updated: 2016-12-06 11:49:52
tags: 
- wiki 
- hexo
categories:
- wiki 
- hexo
---

### 常用命令

#### 启动hexo服务
```
hexo s -w
# -w/--watch 动态部署，文件改变实时更新
# s/start 启动
```

#### 发布新文章
```
hexo new post 'title' --path path/filename
# post 指定模板
# title 文章标题
# --path 指定文章目录和文件名; path/filename 目录/文件名.md
```

#### 发布新文章
```
hexo g # 生成静态文件
hexo deploy # 发布到git
```
### 常用配置

> _config.yml

#### 站点信息配置
```
# Site
title: 蓝色骑士
subtitle: I'CODING MY LIFE.
description:
author: Liu Yang
language: zh-Hans
timezone: 
```

#### 文章配置
```
# Writing
new_post_name: :title/:title-:year:month:day.md # File name of new posts # 指定新建文章的文件名
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: true # 与文件同目录建立同名的文件夹
relative_link: false
future: true
highlight:
    enable: true
    line_number: true
    auto_detect: false
    tab_replace:
```

#### git配置
```
# Writing
deploy:
    type: git
    repo: https://github.com/liuyzss/liuyzss.github.io.git
    branch: master
```

#### 安装next  主题
```
theme: hexo-theme-next
```

> post 模板 scaffolds/post.md

#### post 模板
```
---
title: {{ title }}
date: {{ date }}
updated: {{ date }}
tags:
categories:
---
```
