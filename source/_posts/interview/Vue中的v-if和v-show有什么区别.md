---
title: 【面试真题系列】Vue中的v-if和v-show有什么区别？
urlname: interview-vue-v-if-v-show
tags:
  - vue
  - 面试
categories:
  - 面试
abbrlink: 63270
date: 2021-01-21 22:07:53
---

在回答这个问题前，我们先来看下Vue文档中对这两个指令的说明：

- v-if：用于条件性地渲染一块内容。这块内容只会在指令的表达式返回truthy值的时候被渲染。
- v-show：用于根据条件展示元素的指令。

## v-if和v-show的共同点
两者的作用效果是相同的，都能控制元素在页面是否显示。

在用法上也相同：
```js
<div v-if="show"></div>
<div v-show="show"></div>
```

## v-if和v-show的区别
两者的区别主要表现在下面四个方面：
- 编译过程
- 编译条件
- 性能消耗
- 应用场景

**编译过程**
v-if 是真正的条件渲染，因为它会确保切换过程中条件块内的`事件监听器`和 `子组件`适当地被销毁和重建。

v-if由false变为true的时候，触发组件的beforeCreate、create、beforeMount、mounted钩子，由 true 变为false的时候触发组件的beforeDestroy、destroyed钩子。

v-show的元素始终会被渲染并保留在DOM中，只是简单切换元素的CSS属性`display`进行隐藏或显示。

**编译条件**
v-if 是惰性的：如果在初始渲染时条件为假，则什么也不做。直到条件第一次变为真时，才会开始渲染条件块。

v-show不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

**性能消耗**
v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。

**应用场景**
如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。


> 欢迎关注我的微信公众号：**前端极客技术(FrontGeek)**

## v-if和v-show原理分析
下面我们通过Vue 2.x的源码，来看看v-if和v-show的原理。

在开始之前我们要知道vue2中字符串模板解析编译成真实DOM的过程，大致流程如下：

- 将模板template转为ast结构的JS对象
- 用ast得到的JS对象拼装render和staticRenderFns函数
- render和staticRenderFns函数被调用后生成虚拟VNODE节点，该节点包含创建DOM节点所需信息
- vm.patch函数通过虚拟DOM算法利用VNODE节点创建真实DOM节点

### v-if原理
在模板编译的parse阶段，会使用 `processIfConditions` 函数处理条件渲染指令的内容：
```js
function processIfConditions (el, parent) {
  const prev = findPrevElement(parent.children)
  if (prev && prev.if) {
    addIfCondition(prev, {
      exp: el.elseif,
      block: el
    })
  } else if (process.env.NODE_ENV !== 'production') {
    // 警告信息
  }
}
```

在模板编译的 `codegen` 阶段，会调用 `genIf` 函数处理 v-if 所在的标签：
```js
export function genIf (
  el: any,
  state: CodegenState,
  altGen?: Function,
  altEmpty?: string
): string {
  el.ifProcessed = true // avoid recursion
  return genIfConditions(el.ifConditions.slice(), state, altGen, altEmpty)
}

function genIfConditions (
  conditions: ASTIfConditions,
  state: CodegenState,
  altGen?: Function,
  altEmpty?: string
): string {
  if (!conditions.length) {
    return altEmpty || '_e()'
  }

  const condition = conditions.shift()
  if (condition.exp) {
    return `(${condition.exp})?${
      genTernaryExp(condition.block)
    }:${
      genIfConditions(conditions, state, altGen, altEmpty)
    }`
  } else {
    return `${genTernaryExp(condition.block)}`
  }

  // v-if with v-once should generate code like (a)?_m(0):_m(1)
  function genTernaryExp (el) {
    return altGen
      ? altGen(el, state)
      : el.once
        ? genOnce(el, state)
        : genElement(el, state)
  }
}
```
从代码中可以看出，v-if 指令会转化成三目运算符的形式。

带有 v-if 指令的模板会编译成根据数据源真假值来调用具体辅助方法的渲染函数，v-if 会根据数据源真假值来决定是否渲染该节点，这一点与 v-show 不同。

### v-show原理

v-show 指令根据表达式之真假值，切换元素的 display CSS 属性。当条件变化时该指令触发过渡效果。

在模板编译和生成VNode的过程中，v-show指令与自定义指令的过程一样

在调用处理指令的钩子函数 updateDirectives 时，v-show 指令有所不同，相当于 v-show 内部实现了自定义指令的 bind、update、unbind 三个阶段的钩子函数。
```js
export default {
  bind (el, { value }, vnode) {
    vnode = locateNode(vnode)
    const transition = vnode.data && vnode.data.transition
    const originalDisplay = el.__vOriginalDisplay =
      el.style.display === 'none' ? '' : el.style.display
    if (value && transition) {
      vnode.data.show = true
      enter(vnode, () => {
        el.style.display = originalDisplay
      })
    } else {
      el.style.display = value ? originalDisplay : 'none'
    }
  },

  update (el, { value, oldValue }, vnode) {
    if (!value === !oldValue) return
    vnode = locateNode(vnode)
    const transition = vnode.data && vnode.data.transition
    if (transition) {
      vnode.data.show = true
      if (value) {
        enter(vnode, () => {
          el.style.display = el.__vOriginalDisplay
        })
      } else {
        leave(vnode, () => {
          el.style.display = 'none'
        })
      }
    } else {
      el.style.display = value ? el.__vOriginalDisplay : 'none'
    }
  },

  unbind (el,binding,vnode,oldVnode,isDestroy){
    if (!isDestroy) {
      el.style.display = el.__vOriginalDisplay
    }
  }
}
```

从上述代码可以看到，v-show 指令仅仅是通过调用 DOM.style.display 的值来显示和隐藏DOM元素。

## 参考文章
- https://cn.vuejs.org/v2/guide/conditional.html#v-if
- https://mp.weixin.qq.com/s/9CtghxcWPDZYIOiCjaYDcQ

> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。