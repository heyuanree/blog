---
title: yii踩坑记录
tags:
  - yii
  - web
categories: []
date: 2017-06-01 09:34:01
---


为了这个网站可是踩了我太多太多坑了TAT。。
最最重要的一点经验就是，_**好好读文档比什么都重要**_

<!--more-->

1. 首先是yii的rewrite，由于yii会重写url，所以需要对apache或yii配置
apache的配置:

```
# 设置文档根目录为 "basic/web"
DocumentRoot "path/to/basic/web"

<Directory "path/to/basic/web">
    # 开启 mod_rewrite 用于美化 URL 功能的支持（译注：对应 pretty URL 选项）
    RewriteEngine on
    # 如果请求的是真实存在的文件或目录，直接访问
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    # 如果请求的不是真实文件或目录，分发请求至 index.php
    RewriteRule . index.php

    # ...其它设置...
</Directory>
```

2. 各种php拓展

其实这个在文档中也说明了。。`php requestments.php`即可，把fail的安装上去就好了。。