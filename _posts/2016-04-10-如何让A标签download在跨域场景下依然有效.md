---
layout: post
title: "如何让A标签download在跨域场景下依然有效？"
tags: [一课吧, 实战技巧, Web开发]
---

在HTML5中，A标签支持download属性，该属性用来表明用户在点击超链接时，目标资源将被下载。download属性只有在href被设置时有效。 语法如下：

```html
  <a download="filename">
```
其中,filename是可选的，可以为下载的文件指定新文件名。


在我现在的项目中碰到这样的场景：

  1. 用户需要从网站上下载图片；
  2. 网站的图片存储在图片云服务上，并通过CDN压缩，而现在的CDN图片压缩方案是在URL加东西，比如: xxxxx.jpg!thumb。

在这种情况下，如果采用HTML4的方法：

```html
  <a href="http://prod.upaiyun.com/images/xxx.jpg!thumb">download</a>
```
在不同的浏览器中会有不同的效果，有直接打开的，有直接下载的。而且保存的文件名为xxx.jpg!thumb，不利于使用，理想的情况是使用xxx.jpg。

如果采用HTML5，我们就可以使用download属性：

```html
  <a href="https://prod.upaiyun.com/images/xxx.jpg!thumb" download="xxx.jpg">download</a>
```
这样就会直接下载，并且使用xxx.jpg文件名。


这是非常不错的选择，但别忙着开心，上面的HTML5代码是不能工作的，除非你的图片源和你的网站同源。

是的，同源策略导致download失效！但图片云不受我们控制，我们怎么办呢？

反向代理强制同源 —— 可用但不完美的解决方案。

步骤如下：

  1. 将原先访问图片云服务的资源改为访问网站图片资源，例如：

```html
  <a href="http://prod.upaiyun.com/images/xxx.jpg!thumb" download="xxx.jpg">download</a>
  改为:
  <a href="http://www.mysite.com/images/xxx.jpg!thumb" download="xxx.jpg">download</a>
```
  2. 配置nginx反向代理

```html
  location /images {
    proxy_redirect    off;
    proxy_pass https://prod.upaiyun.com;
  }
```

参考资源:

  * http://www.w3schools.com/tags/att_a_download.asp
  * http://w3c.github.io/html/links.html#downloading-resources
