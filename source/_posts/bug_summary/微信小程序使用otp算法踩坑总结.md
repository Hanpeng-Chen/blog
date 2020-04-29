---
title: 微信小程序使用otp算法踩坑总结
urlname: wechat-miniprogram-otp-use-summary
tags:
  - otp算法
  - 小程序
categories:
  - 问题总结
abbrlink: 43507
date: 2019-07-09 22:13:05
---

# 背景
前两天项目有个类似动态口令的功能要实现，团队最终决定使用OTP算法来实现：前端先向后端请求获取用户的密钥(secret)，将之保存在缓存中，之后前端根据该secret，使用OTP算法中的TOTP方式生成6位动态密码，将6位动态密码传到后台验证。

# OTP
## 1.1 简介
OTP(One-Time-Password)：一次性密码，也称为动态口令。是使用密码技术实现的在客户端和服务端之间通过共享密钥的一种认证技术，是一种强认证技术，是增强目前静态密码口令认证的一种非常方便的技术手段，是一种重要的双因素认证技术。

## 1.2 OTP认证原理
动态口令的基本认证原理是在认证双方共享密钥，也称种子密钥，并使用同一个种子密钥对某一个事件计数、或时间值、或异步挑战数进行密码算法计算，使用的算法有对称算法、HASH、HMAC，之后比较计算值是否一致进行认证。可以做到一次一个动态口令，使用后作废，口令长度通常为6-8个数字，使用方便，与通常的静态口令认证方式类似。

## 1.3 OTP实现方式
- 时间同步(TOTP)
- 事件同步(HOTP)
- 挑战/应答(OCRA)

本文内容主要是小程序使用OTP算法踩坑总结，不对OTP算法的三种实现方式的工作原理进行详细介绍，有兴趣的朋友自行查找相关资料。

# 踩坑记录
## 2.1 后端使用的otp库说明
opt算法有许多现成的库可以直接调用，后端使用的是aerogear-otp-java这个库，问题不在后端，这里就不对这个库进行讲解，有兴趣的朋友可以自己查看：
github地址： https://github.com/aerogear/aerogear-otp-java

## 2.2 踩坑前提
我们的微信小程序项目目前不支持引用第三方的npm包，所以要使用第三方的js库，需要将其js文件下载放到小程序目录中，通过require去引入。还有一点就是小程序项目还不支持node.js，如果js库有相关的node.js代码，还需要做一些改造。以上两点背景给我挖了一个大坑往里跳。

## 2.3 跳入第一个坑
我们先找可用的otp的js库，上github一搜，还是很多的，先选一个start数量比较的的：https://github.com/yeojz/otplib。

我们现在node.js环境上对 **otplib** 这个进行测试：
```
const authenticator = require('otplib/authenticator');
const crypto = require('crypto');
authenticator.options = { crypto };
const secret = 'BYYHJ5R6C3DNZJX3'
const token = authenticator.generate(secret);
console.log(token);
```
执行上面的代码，可以获得6位动态密码，拿到后端验证，**验证不通过**...各种尝试和排查后，放弃了，这个库和后端的aerogear-otp-java不兼容，尴尬。。。

## 2.4 跳入第二个坑
又到github上一通找，各种尝试，终于找到一个可以与aerogear-otp-java兼容的js库：node-lib-otp
github地址： https://github.com/JCloudYu/node-lib-otp

```
const otp = require( 'lib-otp' );
let otpObj = otp({
	secret:'BYYHJ5R6C3DNZJX3'
});
console.log(otpObj.totp(6));
```
将生成的6位动态密码传到后台验证，通过。

心想终于找到能用的了，有种胜利就在眼前的喜悦，马不停蹄下载相关js，着手修改其中的node.js代码，改到一半发现该库引用了太多node.js的写法和库，短时间内无法完成修改并保证可用，放弃该库了。。。

### 2.5 出坑
在github上搜索许久仍旧未找到合适的js库，突然灵光一闪上码云找一下，搜索结果第一个superzlc/otp有写小程序，尝试一下。
地址：https://gitee.com/superzlc/otp/tree/master
下载js，引入该库生成动态密码
```
const otp = require('./libs/otp')
const TOTP = otp.TOTP
const token = new TOTP('BYYHJ5R6C3DNZJX3', 3, 30).gen()
console.log(token)
```
验证通过，还不用修改代码，终于爬出坑了。

# 总结
OTP算法中的TOTP方式生成6位动态密码可用库如下：
- 后端java ： https://github.com/aerogear/aerogear-otp-java
- node.js:   https://github.com/JCloudYu/node-lib-otp
- 小程序纯js：https://gitee.com/superzlc/otp/tree/master

**注意点：**前后端的计数值应一致，否则无法验证通过