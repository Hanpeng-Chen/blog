---
title: JavaScript-函数防抖
abbrlink: 32139
date: 2020-04-29 09:27:20
urlname: javascript-debounce
tags:
 - JavaScript
 - 防抖
categories:
 - [前端, JavaScript]
---

# 前言

在前端开发过程中，我们会遇到一些频繁触发的事件，但我们需要控制回调的频率，比如下面几种场景：
- 游戏中的按键响应，比如格斗，比如射击，需要控制出拳和射击的速率。
- 自动完成，按照一定频率分析输入，提示自动完成。
- 鼠标移动和窗口滚动，鼠标稍微移动一下，窗口稍微滚动一下会带来大量的事件，因而需要控制回调的发生频率。

下面我们通过代码来看看mousemove事件是如何频繁触发的：

index.html文件代码如下：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>debounce防抖</title>
		<style>
		  #container{
		    width: 100%; height: 200px; line-height: 200px; text-align: center; color: #fff; background-color: #444; font-size: 30px;
		  }
		</style>
	</head>
	<body>
		<div id="container"></div>
		
    <script src="debounce.js"></script>
		<script>
			var count = 1;
			var container = document.getElementById("container");
			
			function mouseMove() {
				console.log(this);
				container.innerHTML = count++;
			};
			
			container.onmousemove = mouseMove
		</script>
	</body>
</html>
```

运行该html文件，我们将鼠标在我们定义的矩形区域移动，只是简单的从下往上滑动，mouseMove函数就被触发了99次。

{% qnimg javascript/java-debounce-1.gif %}

假设mouseMove函数时复杂的回调函数或者是ajax请求，如果我们没有对事件处理函数调用的频率进行限制，会加重浏览器的负担，导致用户体验极差。这时候我们可以采用debounce（防抖）或throttle（节流）的方式来减少调用频率，同时又不影响实际效果。

今天我们主要讲讲防抖。

> 欢迎关注我的微信公众号：前端极客技术(FrontGeek)

# 防抖
## 原理
函数防抖(debounce)：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

## 初步实现
根据上面防抖的描述，我们可以用setTimeout写第一版防抖函数的实现代码：

```javascript
function debounce(func, wait) {
  var timeout
  return function () {
    clearTimeout(timeout)
    timeout = setTimeout(func, wait)
  }
}
```

在最开始的例子中使用debounce：
```javascript
container.onmousemove = debounce(mouseMove, 1000)
```

这样子修改后，只有在我们移动完1s内不再触发，才会执行回调事件mouseMove。

## this指向问题
我们在mouseMove函数中执行 console.log(this)，会发现不使用debounce和使用debounce情况下，this的值是不一样的。

不使用debounce时this的值为
```
<div id="container"></div>
```

而使用debounce函数时，this则会指向window对象。

所以我们需要将this指向正确的函数，这时候我们可以利用apply()方法实现。代码修改如下：

```javascript
function debounce(func, wait) {
  var timeout
  return function () {
    var context = this
    clearTimeout(timeout)
    timeout = setTimeout(function() {
      func.apply(context)
    }, wait)
  }
}
```
修改完成后，我们再触发事件，可以看到此时this的指向正确了。

## event对象
JavaScript在事件处理函数中会提供事件对象event，我们将mouseMove函数修改如下：

```javascript
function mouseMove(e) {
	console.log(this);
	console.log(e);
	container.innerHTML = count++;
};
```

如果不使用debounce函数，控制台打印的e是MouseEvent对象，但当我们调用debounce函数，打印出来的却是`undefined`。

所以我们再次修改代码如下：
```javascript
function debounce(func, wait) {
  var timeout
  return function () {
    var context = this
    var args = arguments

    clearTimeout(timeout)
    timeout = setTimeout(function() {
      func.apply(context, args)
    }, wait)
  }
}
```

至此，我们修复了this指向和event对象问题，整个防抖函数已经算是比较完善了。

## 立即执行
接下来我们再考虑一个新的需求：如果我们希望是在事件一触发就立刻执行函数，而不是等到事件停止触发后再执行；并且等到停止触发n秒后，才可以重新触发执行。

我们可以通过immediate参数来判断是否立刻执行，代码修改如下：
```javascript
function debounce(func, wait, immediate) {
  var timeout
  return function () {
    var context = this
    var args = arguments

    if (timeout) {
			clearTimeout(timeout)
		}
		if (immediate) {
			// 已经执行过不再执行
			var callNow = !timeout
			timeout = setTimeout(function() {
				timeout = null
			}, wait)
			if (callNow) {
				func.apply(context, args)
			}
		} else {
			timeout = setTimeout(function() {
			  func.apply(context, args)
			}, wait)
		}
  }
}
```
```javascript
container.onmousemove = debounce(mouseMove, 1000, true)
```

## 返回值
我们需要注意的一点是：mouseMove函数可能是有返回值的，所以我们也要返回函数的执行结果，但是immediate为false的时候，因为使用setTimeout，我们将func.apply(context, args)的返回值赋给变量，最后再return的时候，值会一直是undefined，所以我们只在immediate为true的时候返回函数的执行结果。

```javascript
function debounce(func, wait, immediate) {
  var timeout, result
  return function () {
    var context = this
    var args = arguments

    if (timeout) {
			clearTimeout(timeout)
		}
		if (immediate) {
			// 已经执行过不再执行
			var callNow = !timeout
			timeout = setTimeout(function() {
				timeout = null
			}, wait)
			if (callNow) {
				result = func.apply(context, args)
			}
		} else {
			timeout = setTimeout(function() {
			  func.apply(context, args)
			}, wait)
		}
		return result
  }
}
```

## 取消
假设防抖的时间间隔为10秒，immediate为true的情况下，只有等10秒后才能重新触发事件，这时候我希望有个按钮可以取消防抖，这样我再去触发时，又可以立即执行了。

下面我们来实现这个取消功能：
```javascript
function debounce(func, wait, immediate) {
  var timeout, result
  var debounced = function () {
    var context = this
    var args = arguments

    // 每次新的尝试调用func，会使抛弃之前等待的func
    if (timeout) clearTimeout(timeout)

    // 如果允许新的调用尝试立即执行
		if (immediate) {
			// 如果之前尚没有调用尝试，那么此次调用可以立马执行，否则就需要等待
			var callNow = !timeout
      // 刷新timeout
			timeout = setTimeout(function() {
				timeout = null
			}, wait)
      // 如果能被立即执行，立即执行
			if (callNow) result = func.apply(context, args)
		} else {
			timeout = setTimeout(function() {
			  func.apply(context, args)
			}, wait)
		}
		return result
  }
	
	debounced.cancel = function () {
		clearTimeout(timeout)
		timeout = null
	}
	return debounced
}
```

如何调用这个cancel函数？
```javascript
var setMouseMove = debounce(mouseMove, 10000, true)
container.onmousemove = setMouseMove

// buttonClick为button的click事件
function buttonClick() {
  setMouseMove.cancel()
}
```

效果如下：
{% qnimg javascript/java-debounce-2.gif %}

到这里，一个完整的debounce函数已经实现了。

# 总结
debounce防抖函数，满足的是：高频下只响应一次。

在实际开发过程中，常见的应用场景有：
- 在输入框快速输入文字（高频），我们只想在其完全停止输入时再对输入文字做处理（一次）
- ajax，大多数场景下，每个异步请求在短时间内只能响应一次，比如下拉刷新、不停地上拉加载，但只发送一次ajax请求。
