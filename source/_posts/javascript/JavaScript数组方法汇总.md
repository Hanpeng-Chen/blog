---
title: JavaScript-数组方法汇总
urlname: javascript-array-methods
tags:
  - JavaScript
  - 数组方法
categories:
  - 前端
  - JavaScript
abbrlink: 5225
date: 2020-06-15 14:52:59
---

## valueOf()、toString()
valueOf()和toString()是JavaScript对象的通用方法。

valueOf()表示对该对象求值。不同的对象的valueOf方法不尽一致，数组的valueOf方法返回数组的本身。

```js
let array = [1, 2, 3]
array.valueOf()    // [1, 2, 3]

let array1 = [1, 3, 3, 'a', [1, 2, 3]]
array1.valueOf()   // [1, 3, 3, 'a', [1, 2, 3]]
```

toString()返回的是数组的字符串形式。
```js
let array = [1, 2, 3]
array.toString()    // 1,2,3

let array1 = [1, 3, 3, 'a', [1, 2, 3], {obj: 1}]
array1.toString()   // 1,3,3,a,1,2,3,[object Object]
```

> 欢迎关注我的微信公众号：前端极客技术(FrontGeek)

## Array.isArray()
该方法返回一个布尔值，表示参数是否为数组。

因为数组本质上是一种特殊的对象，所以typeof运算符会返回数组的类型为`object`。使用Array.isArray方法可以识别数组，可以弥补typeof运算符的不足。

```js
let array = [1, 2, 3]
typeof array   // object

Array.isArray(array)   // true
```

## 添加：push()、unshift()
push()方法用于在数组的末端添加一个或多个元素，并返回添加新元素后的数组长度，**该方法会改变原数组**。

```js
let array = [1, 3]

array.push(2)
array.push('a', 'b')
array.push(true, {})

console.log(array)   // [1, 3, 2, "a", "b", true, {}]
```

unshift()方法用于在数组的第一个位置添加元素，并返回添加新元素后的数组长度，该方法会改变原数组。

unshift()可以接受多个参数，这些参数都会添加到目标数组头部。
```js
let array = [1, 2, 3]

array.unshift('a')

array.unshift('b', 'c')

console.log(array)  // ["b", "c", "a", 1, 2, 3]
```


## 删除：pop()、shift()
pop()方法用于删除数组的最后一个元素，并返回该元素，**该方法会改变原数组**。

如果对空数组使用pop方法，不会报错，而是返回undefined

```js
let array = [1, 3, 3, 4]

console.log(array.pop())  // 4
console.log(array)  // [1, 3, 3]

[].pop()   // undefined
```

shift()方法用于删除数组的第一个元素，并返回该元素，该方法会改变原数组。

```js
let array = [1, 3, 3, 4]

array.shift()
console.log(array)  // [3, 3, 4]
```

- push和pop两个方法结合使用，就构成了“后进后出”的栈结构（stack）。

- push和shift结合使用，构成了“先进先出”的队列结构（queue）。

## join()
join方法以指定参数作为分隔符，将所有数组成员连接成一个字符串返回。如果没有提供参数，默认逗号分隔。

如果数组成员是undefined或null或空位，会被转成空字符串。

```js
let array = [1, 2, 3, 4, ,, null, undefined]

console.log(array.join())  // 1,2,3,4,,,,
console.log(array.join(' '))  // 1 2 3 4    
console.log(array.join('-'))   // 1-2-3-4----
```

通过call方法，join也可以用于字符串或者类似数组的对象。
```js
Array.prototype.join.call('hello', '-')  // h-e-l-l-o

let obj = {0: 'a', 1: 'b', 2: 'c', length: 3}
Array.prototype.join.call(obj, '-')  // a-b-c
```

## 合并：concat()
concat方法用于多个数组的合并，它将新数组的成员，添加到原数组成员的后面，然后返回一个新数组，原数组不变。

```js
['hello'].concat(['world'])
// ["hello", "world"]

['hello'].concat(['world'], ['!'])
// ["hello", "world", "!"]

[].concat({a: 1}, {b: 2})
// [{ a: 1 }, { b: 2 }]

[2].concat({a: 1})
// [2, {a: 1}]
```

concat还可以接受其他类型的值作为参数，添加到目标数组尾部。
```js
[1, 2, 3].concat(4, 5, 6)
// [1, 2, 3, 4, 5, 6]
```

如果数组成员包括对象，concat方法返回当前数组的一个浅拷贝。

```js
var obj = { a: 1 };
var oldArray = [obj];

var newArray = oldArray.concat();

obj.a = 2;
newArray[0].a // 2
```

## reverse()
reverse()方法用于颠倒排列数组元素，返回改变后的数组。

```js
let array = [1, 2, 3, 4]

array.reverse()
array   // [4, 3, 2, 1]
```

## splice()
splice()方法用于删除原数组的一部分成员，并可以在删除的位置添加新的数组成员，返回的是被删除的元素。

```js
arr.splice(start, count, addElement1, addElement2, ...);
```

splice的第一个参数是删除的起始位置，第二个参数是被删除的元素个数。如果后面有更多的参数，表示这些是要被插入数组的元素。

如果起始位置是负数，表示从倒数位置开始删除。

```js
let arr = ['a', 'b', 'c', 'd', 'e', 'f', 'g']

arr.splice(1, 2) // ["b", "c"]
arr // ['a', 'd', 'e', 'f', 'g']

arr.splice(0, 1, 1, 2, 3, 4)
arr   // [1, 2, 3, 4, "d", "e", "f", "g"]

arr.splice(-6, 4)
arr   // [1, 2, "f", "g"]
```

## sort()
对数组成员进行排序，**默认按照字典顺序排序**。排序后，原数组将被改变。

如果要自定义排序方式，可以传入一个函数作为参数。

```js
let array = [1, 4, 3, 'a', 6, 2, 1, 10, 98, 7]

array.sort()
array   // [1, 1, 10, 2, 3, 4, 6, 7, 98, "a"]

// 降序
let array1 = [1, 4, 3, 6, 2, 1, 10, 98, 7]
array1.sort((a, b) => b - a)
array1  // [98, 10, 7, 6, 4, 3, 2, 1, 1]
```

## map()
将数组的所有成员依次传入参数函数，然后把每一次的执行结果组成一个新数组返回。

```js
let array = [1, 3, 5]
let new_array = array.map(function(n) {
  return n + 2
})
console.log(new_array)  // [3, 5, 7]
console.log(array)  // [1, 3, 5]
```

调用参数函数时，map向函数传入三个参数：当前成员、当前位置、数组本身。
```js
let array = [1, 3, 5]
let new_array = array.map(function(n, index, arr) {
  console.log(n, index, arr)
  return n * index
})
console.log(new_array)
```

此外，map方法还可以接受第二个参数，用来绑定回调函数内部的this变量
```js
var arr = ['a', 'b', 'c'];

[1, 2].map(function (e) {
  return this[e];
}, arr)
// ['b', 'c']
```

## filter()
用于过滤数组成员，满足条件的成员组成一个新数组返回。

```js
[1, 2, 3, 4, 5].filter(function (elem) {
  return (elem > 3);
})
// [4, 5]
```

filter方法和map一样，可以接受第二个参数，用来绑定参数函数内部的this，参数函数可接收三个参数。
```js
var obj = { MAX: 3 };
var myFilter = function (item) {
  if (item > this.MAX) return true;
};

var arr = [2, 8, 3, 4, 1, 3, 2, 9];
arr.filter(myFilter, obj) // [8, 4, 9]
```

## some()、every()
这两个方法接受一个函数作为参数，所有数组成员依次执行该函数。该函数接受三个参数：当前成员、当前位置、整个数组，然后返回一个布尔值，表示判断数组成员是否符合某个条件。

some()方法只要一个成员的返回值为true，则整个some方法的返回值就是true，否则返回false。

```js
let array = [1, 3, 5, 7]
array.some(function(el, index, arr) {
  return el > 5
})   // true
```

every方法是所有成员的返回值都是true，整个every方法才返回true，否则返回false。
```js
let array = [1, 3, 5, 7]
array.some(function(el, index, arr) {
  return el > 1
})   // false
```

对于空数组，some方法返回false，every方法返回true，回调函数都不会执行。

## reduce()、reduceRight()
reduce方法和reduceRight方法依次处理数组的每个成员，最终累计为一个值。

reduce是从左到右处理，reduceRight是从右到左处理。

```js
array.reduce(func, defaultValue)
```
reduce和reduceRight接收两个参数，第一个是函数，第二个是累积变量初始值。

第一个参数函数接受下面四个参数：
- 累积变量，默认为数组的第一个成员（必须）
- 当前变量，默认为数组的第二个成员（必须）
- 当前位置，从0开始（可选）
- 原数组（可选）

累积变量初始值是个可选参数。

```js
[1, 2, 3, 4, 5].reduce(function (a, b) {
  return a + b;
}, 10);
// 25
```

## indexOf()、lastIndexOf()
indexOf方法返回给定元素在数组中第一次出现的位置，如果没有出现则返回-1。该方法可接受第二个参数，用来指定搜索开始的位置。

```js
['a', 'b', 'd'].indexOf('b')  // 1
['a', 'b', 'd'].indexOf('b', 2)  // -1
```

lastIndexOf方法返回给定元素的最后一次出现的位置，如果没有出现返回-1。

```js
var a = [2, 5, 9, 2];
a.lastIndexOf(2) // 3
a.lastIndexOf(7) // -1
```

> 这两个方法不能用来搜索NaN的位置，因为方法内部使用严格相等运算符`===`进行比较，而NaN是唯一一个不等于自身的值。

**下面我们来看下ES6新增的数组方法：**

## Array.from()
用来将下面两类对象转为真正的数组。
- 类似数组的对象（array-like object）
- 可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）

Array.from()会将数组的空位转为undefined

```js
// 类似数组的对象
let arrayLike = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
}
// ES5 写法
let newArr1 = [].slice.call(arrayLike)  // ["a", "b", "c"]
// ES6 写法
let newArr2 = Array.from(arrayLike)  // ["a", "b", "c"]

// 可遍历的对象
Array.from('hello')  // ["h", "e", "l", "l", "o"]
let set = new Set(['a', 'b', 'c'])
Array.from(set)   // ['a', 'b', 'c']
```

## Array.of()
用于将一组值转换为数组。如果没有参数则返回一个空数组。

```js
Array.of(1, 2, 3, 4)  // [1, 2, 3, 4]
```

## 数组实例的 copyWithin()
在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。

copyWithin()会连空位一起拷贝。

```js
Array.prototype.copyWithin(target, start = 0, end = this.length)
```

- target：必需，从该位置开始替换数据，如果为负值，表示倒数。
- start：可选，从该位置开始读取数据，默认为0。如果为负值，表示从尾部开始计算。
- end：可选，到该位置停止读取数据，默认为数组长度，如果为负值，表示从末尾开始计算。

```js
[1, 2, 3, 4, 5].copyWithin(0, 3)  // [4, 5, 3, 4, 5]

[1, 2, 3, 4, 5].copyWithin(0, 3, 4) // [4, 2, 3, 4, 5]
```

## 数组实例的 find()和findeIndex()
find：找出第一个符合条件的数组成员。参数是一个回调函数。如果没有符合条件的成员，返回undefined

```js
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```

findIndex的用法和find方法类似，返回第一个符合条件的数组成员的位置，如果没有符合条件的成员，返回-1。
```js
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

## 数组实例的 fill()
fill方法使用给定值，填充一个数组。

如果数组中已有元素，会被全部抹去。

fill方法还可以接受第二个和第三个参数，用来指定填充的起始和结束位置。

fill()会将空位视为正常的数组位置。

```js
[1, 2, 3].fill(4)  // [4, 4, 4]

new Array(3).fill(4) // [4, 4, 4]

[1, 2, 3, 4, 5].fill(10, 1, 3) //  [1, 10, 10, 4, 5]
```

> 如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。

```js
let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```

## 数组实例的 entries()、keys()、values()
这三个是ES6新增的三个用于遍历数组的方法。它们都返回一个遍历器对象，可用for...of循环进行遍历。

keys()是对键名的遍历，values()是对键值的遍历，entries()是对键值对的遍历。
```js
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

## 数组实例的 includes()
Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。ES2016 引入了该方法。
```js
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
```

该方法的第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。
```js
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

可以用该方法判断数组中是否有NaN
```js
[NaN].includes(NaN)
// true
```

## 数组实例的 flat()、flatMap()
### flat()
Array.prototype.flat()用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。

flat()默认只会“拉平”一层，如果要“拉平”多层嵌套数组，可以将flat()方法的参数写成一个整数，表示要拉平的层数。

如果不管多少层嵌套，都要转成一维数组，可以用`Infinity`关键字作为参数。

如果原数组存在空位，flat方法会跳过空位。

```js
[1, 2, [3, [4, 5]]].flat()
// [1, 2, 3, [4, 5]]

[1, 2, [3, [4, 5]]].flat(2)
// [1, 2, 3, 4, 5]

[1, [2, [3]]].flat(Infinity)
// [1, 2, 3]
```

### flatMap()
flatMap方法对原数组的每个成员执行一个函数，然后对返回值组成的数组执行flat方法。该方法返回一个新的数组。

**flatMap方法只能“拉平”一层数组。**
```js
[2, 3, 4].flatMap((x) => [x, x * 2])
// [2, 4, 3, 6, 4, 8]

[1, 2, 3, 4].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

flatMap()方法的参数是一个遍历函数，该函数可以接受三个参数，分别是当前数组成员、当前数组成员的位置（从零开始）、原数组。
```js
arr.flatMap(function callback(currentValue[, index[, array]]) {
  // ...
}[, thisArg])
```

flatMap方法可以有第二个参数，用来绑定遍历函数中的this



参考资料：
- https://wangdoc.com/javascript/stdlib/array.html
- https://es6.ruanyifeng.com/#docs/array