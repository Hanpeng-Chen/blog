---
title: vue-router使用history模式配置说明
date: 2021-05-10 17:29:02
urlname:
tags:
categories:
---
vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。

如果不想要很丑的 hash，我们可以用路由的 history 模式，这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
当你使用 history 模式时，URL 就像正常的 url，例如 http://yoursite.com/user/id，也好看！

不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 http://oursite.com/user/id 就会返回 404，这就不好看了。

所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。

#后端配置例子
注意：下列示例假设你在根目录服务这个应用。如果想部署到一个子目录，你需要使用 Vue CLI 的 publicPath 选项 (opens new window)和相关的 router base property (opens new window)。你还需要把下列示例中的根目录调整成为子目录 (例如用 RewriteBase /name-of-your-subfolder/ 替换掉 RewriteBase /)。

## nginx
location / {
  try_files $uri $uri/ /index.html;
}

可能有些同学配置了nginx后，但是刷新页面后控制台报下面的错误：

**Uncaught SyntaxError: Unexpected token <**

如果是通过vue-cli脚手架创建的项目，请检查`vue.config.js`文件中的publicPath是否配置正确

如果是自己手动搭建的Webpack打包，确认assetsPublic的配置，将 `./` 修改为 `/`

https://forum.vuejs.org/t/uncaught-syntaxerror-unexpected-token/32862/18