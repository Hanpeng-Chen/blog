---
title: PDF.js实现在线展示pdf文件
urlname: PDF.js-use
date: 2019-06-28 00:03:24
tags:
  - pdf.js
  - 在线预览PDF文件
categories:
  - 前端
---

# 背景
现在很多项目开发过程中都会碰到PDF在线预览的需求，对于PC端浏览器，一般直接提供PDF文件，iframe一下就可以直接预览。但在移动端要预览PDF则较为麻烦，有些浏览器检测到文件流，就会直接下载，无法实现预览功能。

PDFJS就是解决这一问题比较好用的一款插件。

# PDF.js

PDF.js是一个使用HTML5构建的可移植文档格式库。

PDF.js官网：http://mozilla.github.io/pdf.js/

官网给出的使用方法是将PDF.js下载到项目静态资源目录中，在html文件中引入pdf.js去使用，但这样会导致最后项目编译打包后的资源包比较大。

pdfjs-dist这一node库这好可以解决我们的问题。pdfjs-dist是pdf.js源代码的预构建版本，我们直接在项目中引入该库即可使用，接下来我们介绍如何使用。

需要预览的PDF一般是后台接口返回的base64编码数据或者是本地上传获取到文件，接下来我们在vue工程上实现本地上传PDF文件，获取文件的base64编码，利用pdf.js实现预览效果。


# 基于Vue工程实现PDF在线预览

## 1、安装pdfjs-dist依赖
```
npm install pdfjs-dist
```

## 2、初始化PDF，核心代码
```
      previewPDF() {
        // 引入pdf.js的字体
        let CMAP_URL = 'https://unpkg.com/pdfjs-dist@2.0.943/cmaps/'
        //读取base64的pdf流文件
        let loadingTask = pdfJS.getDocument({
          data: this.pdfData, // PDF base64编码
          cMapUrl: CMAP_URL,
          cMapPacked: true
        })
        loadingTask.promise.then((pdf) => {
          this.loadFinished = true
          let numPages = pdf.numPages
          let pageNumber = 1
          this.getPage(pdf, pageNumber, numPages)
        })
      },
      getPage(pdf, pageNumber, numPages) {
        let _this = this
        pdf.getPage(pageNumber)
          .then((page) => {
            // 获取DOM中为预览PDF准备好的canvasDOM对象
            let canvas = this.$refs.myCanvas
            let viewport = page.getViewport(_this.scale)
            canvas.height = viewport.height
            canvas.width = viewport.width

            let ctx = canvas.getContext('2d')
            let renderContext = {
              canvasContext: ctx,
              viewport: viewport
            }
            page.render(renderContext).then(() => {
              pageNumber += 1
              if (pageNumber <= numPages) {
                _this.getPage(pdf, pageNumber, numPages)
              }
            })
          })
      }
```

## 3、效果
<div style="display: flex; justify-content: center;">
![Alt](/images/articles/2019/vue-pdfjs-demo.png)
</div>


## 4、注意点

当PDF文件中有中文的情况下，在引用pdfjs过程中可能会出现中文不显示问题，在console中会报下面的错误，
```
Warning: The CMap "baseUrl" parameter must be specified, ensure that the "cMapUrl" and "cMapPacked" API parameters are provided.
```

主要原因是有pdf不支持的字体格式，可以通过引入pdf.js的字体来解决该问题。
```
// 引入pdf.js的字体
let CMAP_URL = 'https://unpkg.com/pdfjs-dist@2.0.943/cmaps/'
//读取base64的pdf流文件
let loadingTask = pdfJS.getDocument({
  data: this.pdfData, // PDF base64编码
  cMapUrl: CMAP_URL,
  cMapPacked: true
})
```

**完整代码获取：**
https://github.com/HamptonChen/hampton-demo-repo/tree/master/example-project
