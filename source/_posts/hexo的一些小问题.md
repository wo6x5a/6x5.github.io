---
title: hexo的一些小问题
date: 2017-04-20 15:33:11
tags: hexo
categories: 备忘
---

## 常用命令

- hexo clean :清除缓存文件
- hexo generate (hexo g) :生成静态文件
- hexo server :启动服务
- hexo deploy (hexo d) :部署到服务器
- hexo new "文章标题" :新建文章(默认生成在hexo\source\_posts目录下)
<!-- more -->

## 美化页面
- next 主题挺好看.
- 去除底部xxxx强力驱动等内容:注释(hexo\themes\hexo-theme-next-5.1.0\layout\_partials\footer.swig)的' if theme.copyright ..... endif '中的内容

## 官方文档
- hexo:https://hexo.io/zh-cn/docs/
- next:http://theme-next.iissnan.com/