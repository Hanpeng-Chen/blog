---
title: 在cloudflare上创建一个免费的在线代理来解决gitalk授权登录报403问题
urlname: create-own-cors-anywhere-to-resolve-the-request-with-403
tags:
  - gitalk
  - 在线代理
categories:
  - 技术杂谈
abbrlink: 57490
date: 2021-04-21 11:30:11
---

## 2021.04.28 更新
针对403的问题，gitalk开发团队已经对其做了修复，解决方法如下：

更新版本到 1.7.2 或者修改配置增加 proxy: 'https://cors-anywhere.azm.workers.dev/https://github.com/login/oauth/access_token'

当然我们也可以自己动手搭建代理，如果有兴趣的话可以直接下拉看下面的 [利用cloudflare worker搭建在线代理](#create-cors-by-self)

---

## 问题说明
前两天突然发现个人博客的gitalk评论功能出了问题，点击使用Github登录一直失败，打开控制台一查，发现下面这个请求报403。

> https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token

cors-anywhere是一个用来解决跨域问题而生的反向代理，是一个开源框架。

> [PSA: Public demo server (cors-anywhere.herokuapp.com) will be very limited by January 2021, 31st #301](https://github.com/Rob--W/cors-anywhere/issues/301)

从Github上这个issue上我们可以知道，gitalk中用到的 `cors-anywhere.herokuapp.com` 这个网站原本是用来演示用的，从2021.1.31开始不再作为开放的代理服务，作者建议开发者自己维护一个代理网站。

## 初步解决方案（不稳定不建议使用）

在gitalk的issue中看到别人分享的一个在线代理，先拿来用下：

> https://netnr-proxy.cloudno.de/https://github.com/login/oauth/access_token

我们现在配置文件_config.yml的gitalk配置项中加一个proxy：
```js
gitalk:
   enable: true
   clientID: 
   clientSecret: 
   accessToken: 
   repo: blog-comments
   owner: Hanpeng-Chen
   admin: ['Hanpeng-Chen']
   perPage: 10
   distractionFreeMode: true
   language: zh-CN
   proxy: https://netnr-proxy.cloudno.de/https://github.com/login/oauth/access_token
```

在gitalk.ejs中加上proxy配置：
```js
var gitalk = new Gitalk({
    // Gitalk配置
    language: "<%= config.language%>",
    clientID: "<%= theme.gitalk.clientID%>",
    clientSecret: "<%= theme.gitalk.clientSecret%>",
    repo: "<%= theme.gitalk.repo%>",
    owner: "<%= theme.gitalk.owner%>",
    admin: ["<%= theme.gitalk.admin%>"],
    id: md5(location.pathname),
    distractionFreeMode: <%= theme.gitalk.distractionFreeMode%>,
    proxy: "<%= theme.gitalk.proxy %>"
});
```

重新部署后，点击Github登录，这次没有没有报403的问题，但是出现了下面“Error：Network Error”。

![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/summer/20210421101906.png)
![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/summer/20210421102116.png)

查看了对应请求的应答码 `429`，表示请求太多，我个人估计是白嫖这个在线代理的人太多导致的。

既然白嫖的代理不能用，那我们就自己搭一个在线代理吧。

<div id="create-cors-by-self"></div>

## 利用cloudflare worker搭建在线代理

利用CloudFlare Worker创建在线代理，不需要我们有服务器，也不需要搭建Node.js服务，只需要注册一个CloudFlare账号，创建一个Worker，部署一个JS脚本就可以了，简单方便，下面我们就来看看如何创建吧。


首先你需要一个 CloudFlare 的账号，如果还没有的话就先注册一个吧：[点我注册](https://dash.cloudflare.com/)

选择Workers，创建一个免费的Worker。

![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/summer/20210421095610.png)

免费版本每天10万次请求也足以应对个人使用或者是小范围分享了。

填写自己喜欢的二级域名，然后创建worker。

进入github项目的 [index.js](https://github.com/Hanpeng-Chen/cloudflare-cors-anywhere/blob/master/index.js)，复制代码。

清除脚本编辑器中的示例代码，将复制的代码粘贴进去。

> 这里有个点需要注意：我们可以设置请求的黑白名单，这里的白名单我只设置了自己博客域名，大家可以根据自己的情况修改，当然也可以设置为`whitelist = [ ".*" ]`，这样的话知道你代理地址的人都可以用了，然而免费版本的每天只有10万次请求，如果用的人多了很容易就用完了，所以还是建议大家设置 whitelist。

```js
blacklist = [ ];           // regexp for blacklisted urls
// whitelist = [ ".*" ];     // regexp for whitelisted origins
whitelist = [ "^http.?://www.chenhanpeng.com$", "chenhanpeng.com$" ]
```

修改好之后，点击 保存并部署，如果部署正常的话，我们就可以使用我们创建的在线代理了。

从右侧获取到你的worker域名并记下来，在上面提到的proxy配置项修改为如下代码：

```js
proxy: https://cloudflare-cors-anywhere.hanpengchen.workers.dev/?https://github.com/login/oauth/access_token
```

重新部署我们的博客，再次点击 使用Github登录，这次登录成功，没有报错，至此，个人在线代理就搭建成功了，博客的评论功能也能正常使用了，撒花！！！


## 在线代理的原理了解
传统在线代理都是在服务端替换 HTML/JS/CSS 等资源中的 URL。这不仅需要对内容做大量的分析和处理，还需对流量进行解压和再压缩，消耗大量 CPU 资源。并且由于逻辑较复杂，通常使用 Python/PHP 等编程语言自己实现。

为降低服务端开销，本项目使用浏览器的一个黑科技 —— Service Worker。它能让 JS 拦截网页产生的请求，并能自定义返回内容，相当于在浏览器内部实现一个反向代理。这使得绝大部分的内容处理都可以在浏览器上完成，服务器只需纯粹的转发流量。


要是还有其它问题的话，欢迎留言讨论。

> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术**，我们一起学习一起进步。


![](https://image.chenhanpeng.com/static/blog-images/%E5%89%8D%E7%AB%AF%E6%9E%81%E5%AE%A2%E6%8A%80%E6%9C%AF%E4%BA%8C%E7%BB%B4%E7%A0%81.png)