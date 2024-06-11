---
title: vue项目配合nginx开启gzip提升页面打开速度
urlname: use-gzip-in-vue-project
tags:
  - 解决方案
  - gzip压缩
  - vue
categories:
  - 解决方案
abbrlink: 16872
date: 2021-05-20 15:01:05
---
用Vue开发移动端H5，可能很多同学会遇到最后打包上线，用户第一次打开可能有3s，甚至更久的白屏时间，白屏时间过长是很容易导致用户流失的，所以是必须要解决的问题。

经排查后发现是因为打包后的app.js太大，以及引用的一些插件安装包加载比较慢。

针对这种情况有很多解决方案：
- 路由懒加载
- 打包文件中去掉map文件
- CDN引入第三方库
- 代码压缩
- 分包
- 预渲染
- 开启Gzip

gzip压缩一种非常方便，且效果明显的解决方法，使用gzip压缩可以减小60%以上的体积。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/summer/20210520152024.png)

从上图我们可以看到gzip压缩后的文件体积基本减小了60%左右，效果喜人。

下面我们主要看下如何开启Gzip打包，已经nginx如何配置开启Gzip。

## gzip介绍
gzip属于在线压缩，在资源通过http发送报文给客户端的过程中，进行压缩，可以减少客户端带宽占用，减少文件传输大小。

html、js、css文件，甚至Json数据都可以用其来进行压缩。

但是不是每个浏览器都支持gzip的，如果知道客户端是否支持gzip呢，请求头中有个Accept-Encoding来标识对压缩的支持。客户端http请求头声明浏览器支持的压缩方式，服务端配置启用压缩，压缩的文件类型，压缩方式。当客户端请求到服务端的时候，服务器解析请求头，如果客户端支持gzip压缩，响应时对请求的资源进行压缩并返回给客户端，浏览器按照自己的方式解析，在http响应头，我们可以看到content-encoding:gzip，这是指服务端使用了gzip的压缩方式。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/summer/20210520152903.png)

那么怎么看有没有用gzip压缩的文件呢，打开f12，查看network，按照下面的方式过滤，如果content-encoding是gzip，说明返回的是gzip。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/summer/20210520153445.png)

nginx的gzip分为两种：
- nginx动态压缩，对每个请求先压缩再输出
- nginx静态压缩，使用现成的扩展名为.gz的预压缩文件


## nginx动态压缩配置 gzip
gzip使用环境:http、server、location、if(x)，一般把它定义在nginx.conf的http{…..}之间

### 配置
```js
http {
    gzip  on;
    gzip_min_length    1k;
    gzip_buffers        4 8k;
    gzip_http_version  1.0;
    gzip_comp_level    4;
    gzip_proxied        any;
    gzip_types          text/css
                        text/javascript
                        text/xml
                        text/plain
                        image/x-icon
                        application/javascript
                        application/x-javascript
                        application/json;
    gzip_vary          on;
    gzip_disable        "MSIE [1-6]\.";
    #以下的配置省略...
}
```

nginx -s reload ：修改配置后重新加载生效

### 配置项说明
- gzip：on为启用，off为关闭
- gzip_min_length：设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于1k的字节数，小于1k可能会越压越大。
- gzip_buffers：获取多少内存用于缓存压缩结果，‘4 16k’表示以16k*4为单位获得
- gzip_comp_level：gzip压缩比（1~9），越小压缩效果越差，但是越大处理越慢，所以一般取中间值;
- gzip_types：对特定的MIME类型生效,其中'text/html’被系统强制启用
- gzip_http_version：识别http协议的版本,早起浏览器可能不支持gzip自解压,用户会看到乱码
- gzip_vary：启用应答头"Vary: Accept-Encoding"
- gzip_proxied：nginx做为反向代理时启用,off(关闭所有代理结果的数据的压缩),expired(启用压缩,如果header头中包括"Expires"头信息),no-cache(启用压缩,header头中包含"Cache-Control:no-cache"),no-store(启用压缩,header头中包含"Cache-Control:no-store"),private(启用压缩,header头中包含"Cache-Control:private"),no_last_modefied(启用压缩,header头中不包含"Last-Modified"),no_etag(启用压缩,如果header头中不包含"Etag"头信息),auth(启用压缩,如果header头中包含"Authorization"头信息)
- gzip_disable：(IE5.5和IE6 SP1使用msie6参数来禁止gzip压缩 )指定哪些不需要gzip压缩的浏览器(将和User-Agents进行匹配),依赖于PCRE库


### 问题

开启gzip，资源下载体积减小了，下载效率提升了，但是在线gzip比较占用CPU，和 `gzip_static` 相比还是不太友好。


## nginx静态压缩配置gzip_static
在前端代码打包构建bundle的时候，一般都有根据一定的算法自动压缩代码成gz文件的webpack插件。

当我们不在 nginx 开启 gzip_static的时候，发现生产的gz文件并没有被运行。

gzip_static是会自动执行gz文件的，这样的就避免了通过gzip自动压缩。

```js
http {
  gzip_static  on;
}
```

### 如何判断gzip_static是否生效
在请求的response headers里面的Etag里面，没有`W/`就表明使用的是我们自己的 .gz 文件。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/summer/20210521113526.png)

### Vue项目打包.gz文件

#### 1. 安装插件

```js
npm install compression-webpack-plugin -D
```

#### 2. 在vue.config.js中配置

```js
const CompressionPlugin = require('compression-webpack-plugin')

module.exports = {
  ...
  configureWebpack: {
    plugins: [
      new CompressionPlugin({
        test: /\.js$|\.html$|\.css/,
        threshold: 10240,
        deleteOriginalAssets: false,
        algorithm: 'gzip'
      })
    ]
  }
}
```

## 动态压缩和静态压缩结合使用

```js
http {
    gzip_static  on;
    gzip_proxied expired no-cache no-store private auth;
    gzip  on;
    gzip_min_length    1k;
    gzip_buffers        4 8k;
    gzip_http_version  1.0;
    gzip_comp_level    8;
    gzip_types          application/javascript text/css image/gif;
    gzip_vary          on;
    gzip_disable        "MSIE [1-6]\.";
}
```
首先尝试使用静态压缩，如果有则返回 .gz 的预压缩文件，否则尝试动态压缩。


## 总结
vue重新打包部署后，nginx配置后重新加载，再打开页面速度是不是快了很多。