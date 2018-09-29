---
layout: '关于css样式-webkit-box-orient:'
title: vertical不生效问题
date: 2018-09-29 18:42:35
categories:
- CSS
tags:
- CSS
---

这几天做项目的时候遇到了有多行文本省略号这个需求

<!-- more -->

### 遇到问题

加了如下几个属性后在本地是没问题的，但是到测试环境后，发现一个重要的属性-webkit-box-orient: vertical消失了


```

.app {
  font-size: 20px;
  font-weight: 700;
  color: #333;
  width: 60%;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

```

### 解决方法

在-webkit-box-orient: vertical这行代码上面添加/*! autoprefixer: off */

```

.app {
  font-size: 20px;
  font-weight: 700;
  color: #333;
  width: 60%;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  /*! autoprefixer: off */
  -webkit-box-orient: vertical;
}

```

### 原因 autoprefixer

因为在webpack配置中配置了autoprefixer

autoprefixer不仅会帮我们加-webkit-之类的prefixer,还会删除我们写在 css/sass/less里的样式

所以我们关闭它的自动删除功能

```
  /* autoprefixer: off */

  -webkit-box-orient: vertical;

  /* autoprefixer: on */

  //或者不用写下面的，直接这样写

  /*! autoprefixer: off */

  -webkit-box-orient: vertical;

```

我们也可以在webpack中自己配置，关掉自动移除功能

```

postcss([ autoprefixer({ remove: false }) ]);

```


参考链接[https://github.com/postcss/autoprefixer/issues/776](https://github.com/postcss/autoprefixer/issues/776)

