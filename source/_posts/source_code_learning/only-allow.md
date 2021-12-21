---
title: '阅读一行代码统一规范团队包管理器神器[only-allow] 源码'
urlname: read-source-code-of-only-allow
tags:
  - 阅读源码
categories:
  - 阅读源码
abbrlink: 29441
date: 2021-11-30 09:47:14
---
当前市场上包管理器常用的有npm、yarn、pnpm。我们在实际项目开发中一般通过是在文档中说明约定使用的包管理器，但偶尔会遇到不遵守约定的人，他使用了其他的包管理器安装了其他的依赖，上传了代码。这种情况有可能导致严重的线上问题。如果我们可以通过代码进行强制约束使用同一包管理器，那么问题也就解决了。only-allow就是一款解决该问题的工具。

## 克隆代码
```js
// only-allow 官方仓库地址
git clone https://github.com/pnpm/only-allow.git
cd only-allow

pnpm i
```

从项目的`README.md`文件我们可以了解到`only-allow`的作用：在项目上强制使用特定的包管理器。

## only-allow的用法

only-allow具体的用法如下：

Add a `preinstall` script to your project's `package.json`

如果你想强制使用`npm`，添加：

```json
{
  "scripts": {
    "preinstall": "npx only-allow npm"
  }
}
```

如果你想强制使用`pnpm`，添加：

```json
{
  "scripts": {
    "preinstall": "npx only-allow pnpm"
  }
}
```

如果你想强制使用`yarn`，添加：

```json
{
  "scripts": {
    "preinstall": "npx only-allow yarn"
  }
}
```

## 调试源码

从`package.json`文件我们可以得知项目的主入口文件为：`bin.js`

在`package.json`文件中添加如下命令：

```json
{
    "scripts" {
        "preinstall": "node bin.js pnpm"
    }
}
```


```js
#!/usr/bin/env node
const whichPMRuns = require('which-pm-runs')
const boxen = require('boxen') // 在终端中创建盒子，使打印信息更加显眼

const argv = process.argv.slice(2)
if (argv.length === 0) {
  console.log('Please specify the wanted package manager: only-allow <npm|pnpm|yarn>')
  process.exit(1)
}

// 获取到用户传入的希望使用的包管理器
// npm  yarn pnpm都不是，报错
const wantedPM = argv[0]
if (wantedPM !== 'npm' && wantedPM !== 'pnpm' && wantedPM !== 'yarn') {
  console.log(`"${wantedPM}" is not a valid package manager. Available package managers are: npm, pnpm, or yarn.`)
  process.exit(1)
}
// 使用的包管理器
const usedPM = whichPMRuns()
// 如果使用的包管理和希望的不一致，根据对应的情况进行报错提示，并退出进程
if (usedPM && usedPM.name !== wantedPM) {
  const boxenOpts = { borderColor: 'red', borderStyle: 'double', padding: 1 }
  switch (wantedPM) {
    case 'npm':
      console.log(boxen('Use "npm install" for installation in this project', boxenOpts))
      break
    case 'pnpm':
      console.log(boxen(`Use "pnpm install" for installation in this project.

If you don't have pnpm, install it via "npm i -g pnpm".
For more details, go to https://pnpm.js.org/`, boxenOpts))
      break
    case 'yarn':
      console.log(boxen(`Use "yarn" for installation in this project.

If you don't have Yarn, install it via "npm i -g yarn".
For more details, go to https://yarnpkg.com/`, boxenOpts))
      break
  }
  process.exit(1)
}
```

## which-pm-runs源码

```js
'use strict'

module.exports = function () {
    // process.env.npm_config_user_agent拿到的是如下字符串
    // 'pnpm/6.23.2 npm/? node/v16.13.0 darwin arm64'
  if (!process.env.npm_config_user_agent) {
    return undefined
  }
  return pmFromUserAgent(process.env.npm_config_user_agent)
}

function pmFromUserAgent (userAgent) {
  const pmSpec = userAgent.split(' ')[0]
  const separatorPos = pmSpec.lastIndexOf('/')
//   返回包管理器及其版本，例如
// {
//     name: 'pnpm',
//     version: '6.23.2'
// }
  return {
    name: pmSpec.substr(0, separatorPos),
    version: pmSpec.substr(separatorPos + 1)
  }
}
```

从which-pm-runs的源码，学到了获取当前运行脚本的包管理器和版本的方法：通过获取`process.env.npm_config_user_agent`变量，进行相应的截取。

## npm命令钩子

```
preinstall：在install之前执行

install

postinstall：在install之后执行

```

## 总结

`only-allow`这个包对多人协作开发的项目还是很有用的，找个时间用到公司的项目上，进一步规范团队开发。