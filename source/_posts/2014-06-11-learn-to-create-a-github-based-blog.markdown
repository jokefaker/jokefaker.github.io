---
layout: post
title: "learn-to-create-a-github-based-blog"
date: 2014-06-11 18:05:35 +0800
comments: true
categories: blog

---

###教程
破船之家：[利用Octopress搭建一个Github博客](http://beyondvincent.com/blog/2013/08/03/108-creating-a-github-blog-using-octopress/)  
唐巧：[象写程序一样写博客：搭建基于github的博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)  

###遇到的问题
* 安装不了ruby 1.9.3  
	* 其实只要安装ruby 1.9.3 或者以上版本就行了，不一定必须 1.9.3    
* 执行rake命令的时候提示：

```
You have already activated rake 10.1.0, but your Gemfile requires rake 0.9.2.2. Prepending `bundle exec` to your command may solve this.
// 意思就是在命令前面加上bundle exec就可以了
```