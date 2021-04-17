---
title: JavaScript-函数节流
abbrlink: 29044
date: 2020-05-06 09:00:54
urlname: javascript-throttle
tags:
 - JavaScript
 - 节流
categories:
 - [前端, JavaScript]
---

在上一篇文章 [JavaScript-函数防抖](http://www.chenhanpeng.com/javascript-debounce/) 中我们学习了什么是防抖，并且一步步实现了防抖函数，今天我们一起来学习节流(throttle)。

# 什么是节流
函数节流(throttle)：当持续触发事件时，保证一定时间段内只调用一次事件处理函数。简单的说，就是让一个函数无法在很短时间间隔内被连续调用，只有当上一次函数执行后过了规定的时间间隔，才能进行下一次函数的执行。

函数节流主要有两种实现方法：时间戳和定时器。

> 欢迎关注我的微信公众号：前端极客技术(FrontGeek)

# 节流的实现

## 时间戳

### 思路
只要触发，就用Date方法获取当前时间 now，与上一次调用的时间 previous 作比较

  - 如果时间差大于等于规定的时间间隔，就执行一次目标函数，执行以后，将存储上一次调用时间previous的值更新为当前时间now
 
  - 如果时间差小于规定的时间间隔，则等待下一次触发重新进行第一步操作。

### 代码实现
```javascript
// delay：规定的时间间隔
function throttle(func, delay) {
  var context, args
  var previous = 0
  return function() {
    context = this
    args = arguments
    var now = +new Date()
    if (now - previous >= delay) {
      func.apply(context, args)
      previous = now
    }
  }
}
```

我们依旧采用防抖那篇文章用到的鼠标移动的例子来验证节流，调用节流函数方式如下：

```javascript
container.onmousemove = throttle(mouseMove, 2000)
```
效果如下：

{% qnimg javascript/java-throttle-1.gif %}


从上图中可以看到：当鼠标移入时，事件立即执行，每过2秒会执行一次，假设在第7秒时移出，停止触发，以后不会再执行事件。

## 定时器
### 思路
用定时器实现时间间隔。
  - 当定时器不存在，说明可以执行函数，定义一个定时器来向任务队列注册目标函数。目标函数执行后设置保存定时器ID变量为空
  - 当定时器已经被定义，说明已经在等待过程中，则等待下次触发事件时再进行查看。

### 代码实现
```javascript
function throttle(func, delay) {
  var timeout = null
  var context, args
  return function() {
    context = this
    args = arguments
    if (!timeout) {
      timeout = setTimeout(function() {
        timeout = null
        func.apply(context, args)
      }, delay)
    }
  }
}
```
代码执行效果如下：
{% qnimg javascript/java-throttle-2.gif %}

从上面的动图我们可以看到：鼠标移入时，事件不会立即执行，之后每隔2秒执行一次，假设在第5秒时移出，事件停止触发，但在第6秒时依旧会执行一次事件。

两者区别：
- 时间戳实现：触发事件一发生先执行目标函数，然后再等待规定的时间间隔再次执行目标函数。如果在等待过程中停止触发，后续不会再执行目标函数。
- 定时器实现：触发事件一发生，先等待够规定的时间间隔再执行目标函数。即使在等待过程中停止触发，若定时器已经在任务队列里注册了定时器，也会执行最后一次。

## 强强联合：时间戳+定时器
如果我们想要能够控制鼠标移入能够立即执行，停止触发的时候能够再执行一次，我们可以综合时间戳和定时器两种方法来实现“有头有尾”的效果。

在这里我们需要注意：控制好在上一周期的“尾”和下一周期的“头”之间时间间隔，我们引入变量remaining表示还需要等待的时间，来让尾部那一次的执行也符合时间间隔。

### 代码实现
```javascript
function throttle(func, delay) {
	var timeout, context, args, result
	var previous = 0
	
	var throttled = function() {
		context = this
		args = arguments
		var now = +new Date()
		// 下次触发func剩余时间
		var remaining = delay - (now - previous)
    // 如果没有剩余的时间了或者你改了系统时间
		if (remaining <= 0 || remaining > delay) {
			if (timeout) {
				clearTimeout(timeout)
				timeout = null
			}
			func.apply(context, args)
			previous = now
		} else if (!timeout) {
			timeout = setTimeout(function(){
				previous = +new Date()
				timeout = null
				func.apply(context, args)
			}, remaining)
		}
	}
  return throttled
}
```
代码执行效果如下：
{% qnimg javascript/java-throttle-3.gif %}

## 优化
在上面结合时间戳和定时器的解法的基础上，如果我们想实现是否启用第一次 / 尾部最后一次计时回调的执行，如何实现？

我们可以设置个options作为第三个参数，然后根据传的值判断到底哪种效果，我们约定：

- leading：false表示禁用第一次执行
- trailing：false表示禁用停止触发的回调

代码实现如下：
```javascript
function throttle(func, delay, options) {
  var timeout, context, args, result
  var previous = 0
  if (!options) options = {}

  var later = function() {
    previous = options.leading === false ? 0 : new Date().getTime()
    timeout = null
    func.apply(context, args)
    if (!timeout) context = args = null
  }

  var throttled = function() {
    var now = new Date().getTime()
    if (!previous && options.leading === false) previous = now
    var remaining = delay - (now - previous)
    context = this
    args = arguments
    if (remaining <= 0 || remaining > delay) {
      if (timeout) {
        clearTimeout(timeout)
        timeout = null
      }
      previous = now
      func.apply(context, args)
      if (!timeout) context = args = null
    } else if (!timeout && options.trailing !== false) {
      timeout = setTimeout(later, remaining)
    }
  }
  return throttled
}
```
我们想要第一次立即执行，并且禁用停止触发的回调，调用throttle方法如下：
```javascript
container.onmousemove = throttle(mouseMove, 2000, {leading: true, trailing: false})
```
效果如下图：
{% qnimg javascript/java-throttle-4.gif %}

## 取消
在防抖函数中，我们实现了cancel方法，在节流函数中，同理：

```javascript
function throttle(func, delay, options) {
  .....

  var throttled = function() {
    ....
  }

  throttled.cancel = function() {
    clearTimeout(timeout)
    previous = 0
    timeout = null
  }

  return throttled
}
```

## 存在的问题
至此，一个完整的节流函数已经实现好了，但是仍然存在一个问题：就是 leading：false 和 trailing: false 不能同时设置。

如果同时设置的话，比如当你将鼠标移出的时候，因为 trailing 设置为 false，停止触发的时候不会设置定时器，所以只要再过了设置的时间，再移入的话，就会立刻执行，就违反了 leading: false，bug 就出来了。如下图所示：
{% qnimg javascript/java-throttle-5.gif %}

# 总结

防抖和节流的作用都是防止函数多次调用。区别在于：假设一个用户一直触发这个函数，且每次触发函数的间隔小于wait，防抖的情况下只会调用一次，而节流的情况会每隔一段时间wait调用函数。

相比 debounce，throttle 要更加宽松一些，其目的在于：按频率执行调用。


> 参考来源：https://github.com/mqyqingfeng/Blog/issues/26