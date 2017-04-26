---
title: hexo的一些小问题
date: 2017-04-20 15:33:11
tags: hexo
categories: 技术
---

## 常用命令

- hexo clean :清除缓存文件
- hexo generate (hexo g) :生成静态文件
- hexo server :启动服务
- hexo deploy (hexo d) :部署到服务器
- hexo new "文章标题" :新建文章(默认生成在hexo\source\_posts目录下)
<!-- more -->

## 美化页面
- 以下均基于next主题.
- 去除底部xxxx强力驱动等内容:
注释hexo\themes\hexo-theme-next-5.1.0\layout\_partials\footer.swig
的如下内容
```
<div class="powered-by">
  {{ __('footer.powered', '<a class="theme-link" href="https://hexo.io">Hexo</a>') }}
</div>

<div class="theme-info">
  {{ __('footer.theme') }} -
  <a class="theme-link" href="https://github.com/iissnan/hexo-theme-next">
    NexT.{{ theme.scheme }}
  </a>
</div>
```
- 阅读全文美化,鼠标悬浮颜色和默认按钮颜色互换:
hexo\themes\hexo-theme-next-5.1.0\source\css\_variables\base.style
```
$btn-default-hover-bg           = $black-deep
$btn-default-hover-color        = white
$btn-default-hover-border-color = $black-deep
$read-more-color                = $black-deep
$read-more-bg-color             = white
```
注意格式 =后面需要空格
- 文章卡片添加阴影效果:
hexo\themes\hexo-theme-next-5.1.0\source\css\_schemes\Muse 添加
```
.posts-expand { 
	.post{ 
		margin-bottom: 60px;
		padding: 25px;
		-webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
		-moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
		} 
}
```
去除底部多余空白和黑色横线:
hexo\themes\hexo-theme-next-5.1.0\source\css\_common\components\post\post-eof.styl
```
// 注释margin
//margin: $post-eof-margin-top auto $post-eof-margin-bottom;
//width 改为 0%
//width: 8%;
width: 0%;
```


## 官方文档
- hexo:https://hexo.io/zh-cn/docs/
- next:http://theme-next.iissnan.com/