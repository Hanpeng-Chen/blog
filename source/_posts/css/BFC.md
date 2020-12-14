---
title: CSS面试题：什么是BFC？BFC有什么用？
urlname: css-bfc
tags:
  - 前端
  - CSS
  - BFC
categories:
  - 前端
  - CSS
  - 面试
abbrlink: 26971
date: 2020-12-09 09:30:48
---

BFC是之前前端面试中经常问到一个问题，这篇文章我们一起来学习BFC。

## 什么是BFC

BFC(Block Formatting Context)：快格式化上下文，是web页面的可视化CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。它有一套渲染规则，决定了其子元素将如何定位，以及和其他元素的关系和相互作用。

快格式化上下文包含创建它的元素内部的所有内容。

具有BFC特性的元素我们可以将其看作隔离的独立容器，容器内的元素不会在布局上影响到容器外的元素，而且BFC具有普通容器所没有的一些特性。

## 如何创建BFC
在了解BFC特性之前，我们先来看下都有哪些方式会创建BFC(摘抄自MDN)：

- 根元素
- 浮动元素（元素的 float 不是 none）
- 绝对定位元素（元素的 position 为 absolute 或 fixed）
- 行内块元素（元素的 display 为 inline-block）
- 表格单元格（元素的 display 为 table-cell，HTML表格单元格默认为该值）
- 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
- 匿名表格单元格元素（元素的 display 为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot 的默认属性）或 inline-table）
- overflow 值不为 visible 的块元素
- display 值为 flow-root 的元素
- contain 值为 layout、content 或 paint 的元素
- 弹性元素（display 为 flex 或 inline-flex 元素的直接子元素）
- 网格元素（display 为 grid 或 inline-grid 元素的直接子元素）
- 多列容器（元素的 column-count 或 column-width 不为 auto，包括 column-count 为 1）
- column-span 为 all 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（标准变更，Chrome bug）。

> 欢迎关注我的微信公众号：前端极客技术(FrontGeek)

## BFC的布局规则

- 内部的Box会在垂直方向上一个接一个的放置
- 内部的Box垂直方向上的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生折叠，不同BFC不会发生折叠。
- 每个元素的左外边距与包含块的左边界相接触（从左向右），即使浮动元素也是如此。（这说明BFC中子元素不会超出他的包含块，而position为absolute的元素可以超出他的包含块边界）
- BFC的区域不会与float的元素区域重叠
- 计算BFC的高度时，浮动子元素也参与计算

## BFC特性及应用

### 同一BFC下发生外边距塌陷

我们先来看下面的代码

```html
<style>
  div {
    width: 100px;
    height: 100px;
    margin: 50px;
    background-color: #108EE9;
  }
</style>
<body>
  <div></div>
  <div></div>
</body>
```

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209084006.png)

从代码和实际渲染效果看出：两个div处于同一个BFC容器下（body元素）,第一个div元素的下边距和第二个div元素的上边距发生了重叠，两个盒子之间的距离只有50px，这就是外边距折叠，也就叫外边距塌陷。

这个不是CSS的bug，我们可以理解为这是一种规范。如果想要避免外边距发生重叠，可以将其放在不同的BFC容器中。

我们将上面代码做以下调整，代码及效果如下：

```html
<style>
  .container {
    overflow: hidden;
  }
  p {
    width: 100px;
    height: 100px;
    margin: 50px;
    background-color: #108EE9;
  }
</style>
<body>
  <div class="container">
    <p></p>
  </div>
  <div class="container">
    <p></p>
  </div>
</body>
```
![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209084457.png)

### BFC可以包含浮动的元素（清除浮动）
我们先来看下面的代码及其执行结果：

```html
<style>
  .father {
    border: 1px solid royalblue;
    overflow: auto;
  }
  .float-child {
    width: 100px;
    height: 100px;
    background-color: #108EE9;
    float: right;
  }
</style>

<div class="father">
  <div class="float-child"></div>
</div>
```

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209085305.png)

容器内的子元素都设置为浮动，容器的高度只剩下2px的边框高度，出现这个结果是由于浮动元素会脱离文档流，所以导致不占据页面空间，所以会对父元素高度带来一定影响。如果一个元素中包含的元素全部是浮动元素，那么该元素高度将变成0，也就是我们常说的高度塌陷。

如果使触发容器的 BFC，那么容器将会包裹着浮动元素。

```html
<style>
  .father {
    border: 1px solid royalblue;
  }
  .float-child {
    width: 100px;
    height: 100px;
    background-color: #108EE9;
    float: right;
  }
</style>

<div class="father">
  <div class="float-child"></div>
</div>
```

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209085831.png)

### BFC可以阻止元素被浮动元素覆盖

BFC可以阻止元素被浮动元素覆盖，在了解这个特性之前，先看下面的代码：

```html
<div style="float: left;width: 200px;background-color: #108EE9;">
  浮动区域
</div>

<div style="height: 100px;background-color: #808080;">
  非浮动区域，height: 100px;background-color: #808080;
</div>
```

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209091046.png)

这时候其实第二个元素有部分被浮动元素所覆盖，(但是文本信息不会被浮动元素所覆盖) 如果想避免元素被覆盖，可触第二个元素的 BFC 特性，在第二个元素中加入 overflow: hidden，就会变成：

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209091140.png)

利用BFC可以阻止元素被浮动元素覆盖的特性，我们解决CSS的两列自适应布局问题。

### 阻止因为浏览器因为四舍五入造成的多列布局换行的情况
有时候因为多列布局采用小数点位的width导致因为浏览器因为四舍五入造成的换行的情况，可以在最后一列触发BFC的形式来阻止换行的发生。比如下面的例子：

```html
<style>
.container {
  min-height: 200px;
}
.column1,.column2 {
  width: 31.3%;
  background-color: green;
  float: left;
  min-height: 100px;
  margin: 0 1%;
}
.column3 {
  width: 31.3%;
  background-color: green;
  min-height: 100px;
  margin: 0 1%;
}
</style>

<div class="container">
  <div class="column1">column 1</div>
  <div class="column2">column 2</div>
  <div class="column3">column 3</div>
</div>
```
![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209114510.png)

从上图看出，我们最后一列出现换行情况，我们在对最后一列触发BFC特性，加入`overflow: hidden`，页面重新渲染结果如下：
![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2020/winter/20201209114736.png)

#### 参考资料：

https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context

https://mp.weixin.qq.com/s/K7Ph4yuG0UMZYGKuDyMHjg

https://zhuanlan.zhihu.com/p/25321647
