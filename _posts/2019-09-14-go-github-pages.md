---
layout: post
title: 开始 Blog
categories: tools
description: 使用Github Pages，开始 Blog
keywords: 使用Github Pages, Blog
---


## 为啥用 Github Pages 来写 Blog
主要是用来做学习笔记，积累经验。使用 Github Pages 有以下几个原因
- 平常记日记用 Markdown 较多，Github Pages 可以方便把 Markdown、Textile、Html 转换成 Blog。
- 提交至 Github 的代码自动生成 Blog 服务，省去自己搞定服务托管。
- 使用比较简单，使用 Jekyll 框架。

## 有哪些例子可以参考
[jekyll/wiki/Sites](https://github.com/jekyll/jekyll/wiki/Sites)

## 使用图片
可以在根目录下建个 images 目录，把图片放到 images 内，比如2019-10-23这天 post 的，可以放到/images/posts/2019-10-23下。引用方式如下：  

```html
<a href="http://www.wakaleo.io/">
  <img src="/images/posts/2019-10-23/breaches.png">
</a>
```

## 借用 Theme
写 Blog 比较怕的是一直在研究，没有动手，所以先找个 Theme 拿来修改一下，关键是写起来。
借用theme： [mzlogin.github.io](https://github.com/mzlogin/mzlogin.github.io)

## 其他选择
除了用 Github Pages 自带的 Jekyll，也可以用[hexo](https://hexo.io/zh-cn/docs/)，使用 Nodejs，可以支持部署到git