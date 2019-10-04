---
title: Vue-CLI3 环境变量和模式
urlname: Vue-Cli3-mode-and-env
tags:
  - Vue-CLI3
  - 模式
  - 环境变量
categories:
  - 前端
  - Vue
date: 2019-04-29 22:46:30
---

前段时间工作中用Vue-CLI3构建的Vue工程一些静态资源，比如静态H5页面、图片、图标等等，我们一般放在固定的一些服务器上，链接前缀一般相对固定，但我们打包发布一般要区分测试环境和生产环境，此时的静态资源路径也需要区分测试和生产，如果每次打包都要根据部署的环境去修改路径十分麻烦，这时候vue-cli的模式和环境变量则能够很好地解决这个麻烦。

## 模式
**模式**是Vue CLI项目中一个重要的概念。默认情况下，一个Vue CLI项目有三个模式：

- development 模式用于vue-cli-service serve

- production 模式用于vue-cli-service build和vue-cli-service test:e2e

- test 模式用于vue-cli-service test:unit

模式不同于NODE_ENV，一个模式可以包含多个环境变量。每个模式都会将NODE_ENV的值设置为模式的名称，比如：development模式下NODE_ENV的值会被设置为development

当然，我们也可以通过为.env文件增加后缀来设置某个模式下特有的环境变量。

#### 示例：test模式

我们在项目根目录下创建一个名为.env.test和.env的文件

```
// .env文件：
VUE_APP_TITLE=VUE-CLI3-DEMO
```

```
// .env.test文件
NODE_ENV=production
VUE_APP_TITLE=VUE-CLI3-DEMO(test)
```

- vue-cli-service build 会加载可能存在的 .env、.env.production 和 .env.production.local 文件然后构建出生产环境应用；

- vue-cli-service build --mode test 会在 staging 模式下加载可能存在的 .env、.env.test 和 .env.test.local 文件然后构建出生产环境应用。

这两种情况下，根据 NODE_ENV，构建出的应用都是生产环境应用，但是在 test 版本中，process.env.VUE_APP_TITLE 被覆写成了另一个值。

我们在vue.config.js文件中添加console.log(process.env.VUE_APP_TITLE)

在package.json文件中添加"build-test": "vue-cli-service build --mode test"

执行npm run build和npm run build-test

通过查看控制台打印出的分别是VUE-CLI3-DEMO和VUE-CLI3-DEMO(test)，由此可见不同模式下的环境变量不同。

--------------------
## 环境变量
只有以**VUE_APP_**开头的变量才会被webpack.DefinePlugin静态嵌入到客户端侧的包中，我们可以在代码中以下面的方式访问：process.env.VUE_APP_TITLE

除了VUE_APP_*变量外，还有两个始终可用的特殊变量：
- **NODE_ENV**  值为development、productin、test中的一个。
- **BASE_URL**  与vue.config.js中的publicPath相符，即应用部署的基础路径

-------------------
回到我们最开始的关于静态资源在不同环境下的路径问题，我们可以分别创建.env.test和.env.production两个文件，在文件中添加变量VUE_APP_STATIC_BASE_URL，根据不同环境赋予不同的值，在代码中用到静态资源的时候通过process.env.VUE_APP_STATIC_BASE_URL + 静态资源后续具体路径，再package.json中添加相应的模式打包命令，这样就可以比较好解决我们最开始提出的问题了。