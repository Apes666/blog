---
title: "Hugo部署博客记录"
date: "2025-10-24T15:31:49Z"
image: "images/banner.png"
draft: false
---

## 理解 Hugo 的结构

> hugoplate主题的文件夹结构

|               文件                |                             作用                             |
| :-------------------------------: | :----------------------------------------------------------: |
|   config/_default/language.toml   | 导向不同语言的站点，可以对特定的语言执行params.toml的覆盖代码 |
| config/_default/menus.zh(en).toml |         编辑菜单栏显示名称，导向文件夹，及其附属结构         |
|    config/_default/params.toml    |                 编辑站点图标、底栏等其他属性                 |
|         i18n/zh(en).yaml          |          对各语言实现本地化，对特定词句实现翻译替换          |
|             content/              |         文章、页面的存储场所，对应menus的文件夹结构          |

