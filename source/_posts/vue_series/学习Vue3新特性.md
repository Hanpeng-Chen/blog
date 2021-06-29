---
title: 一文学会Vue3新特性
urlname: learn-vue3
tags:
  - Vue3
categories:
  - [前端, Vue]
abbrlink: 35488
date: 2021-06-29 17:26:36
---

Vue3.0 周边生态现在已经完善得差不多了，新项目可以开始使用Vue3来开发了，今天我们先来学习Vue3的一些新特性。

## Composition API
### 为什么选择Composition API
Composition API翻译成中文就是组合式API。有的人可能会疑惑为什么要用Composition API？原来Vue2中的options API不是也能实现吗？

我们先来Vue3官方文档中的例子：

假设我们的应用中有一个显示某个用户的仓库列表的视图。此外，我们还希望有搜索和筛选功能。实现此视图组件的代码可能如下所示：

```js
// src/components/UserRepositories.vue

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { 
      type: String,
      required: true
    }
  },
  data () {
    return {
      repositories: [], // 1
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    getUserRepositories () {
      // 使用 `this.user` 获取用户仓库
    }, // 1
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

该组件有以下几个职责：

- 从假定的外部 API 获取该用户的仓库，并在用户有任何更改时进行刷新
- 使用 searchQuery 字符串搜索仓库
- 使用 filters 对象筛选仓库

使用 (data、computed、methods、watch) 组件选项来组织逻辑通常都很有效。然而，当我们的组件开始变得更大时，逻辑关注点的列表也会增长。尤其对于那些一开始没有编写这些组件的人来说，这会导致组件难以阅读和理解。

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/20210627221817.png)

这是一个大型组件的示例，其中逻辑关注点按颜色进行分组。

这种碎片化使得理解和维护复杂组件变得困难。选项的分离掩盖了潜在的逻辑问题。此外，在处理单个逻辑关注点时，我们必须不断地“跳转”相关代码的选项块。

如果能够将同一个逻辑关注点相关代码收集在一起会更好。而这正是组合式 API 使我们能够做到的。

> `Vue3`兼容大部分`Vue2`语法，所以在`Vue3`中写Vue2语法是没问题的（废除的除外）。

### setup

setup是Vue3新增的一个选项，是使用Composition API的入口。

#### setup的执行时机
`setup`只在初始化时执行一次，所有的 Composition API 函数都在这里使用。

`setup`是在生命周期`beforeCreate`之前执行。我们通过下面的代码来验证一下：

```js
beforeCreate() {
  console.log('---------beforeCreate--------');
},
setup() {
  console.log('---------setup--------');
  return {};
},
// ---------setup--------
// ---------beforeCreate--------
```
由上面的执行结果我们可以推断出，setup执行时，组件对象还没有创建，此时`this`是undefined，因此在setup函数中不能通过`this`对象访问`data/computed/methods/props`。

#### setup参数
setup有两个可选参数：
- props：组件传入的属性（响应式对象，且可以监听）
- context：上下文对象。setup中不能访问Vue2中最常用的`this`对象，所以context中就提供了this中最常用的三个属性：`attrs`、`slot` 和`emit`。

```js
import { toRef } from 'vue'
setup(props) {
  const title = toRef(props, 'title')
  console.log(title.value)
}
```

> 由于props是响应式的，所以**不能使用 ES6 解构**，解构会消除prop的响应性。如果需要解构prop，可以在setup函数中使用`toRefs`函数来完成此操作。

```js
export default {
  setup(props, context) {
    // Attribute (非响应式对象)
    console.log(context.attrs)
    // 插槽 (非响应式对象)
    console.log(context.slots)
    // 触发事件 (方法)
    console.log(context.emit)
  }
}
```

### reactive
`reactive`函数接受一个普通对象，返回一个响应式数据对象。

```js
const data = reactive({
  count: 0,
  result: computed(() => data.count + 1)
})
```

### ref 和 isRef
- ref：接受一个内部值并返回一个响应式且可变的 ref 对象。ref 对象具有指向内部值的单个 property .value。

- isRef：检查值是否为一个 ref 对象。

```js
export default {
  setup() {
    const count = ref(0)
    console.log('count is ref：', isRef(count))
    console.log('count: ', count)
    console.log('count.value：', count.value)
    return {
      count
    }
  }
}
```

### toRefs
将响应式对象转换为普通对象，其中结果对象的每个 property 都是指向原始对象相应 property 的 `ref`。

我们通过`reactive`创建的对象，如果在模板中使用，就必须以`xxxx.xxx`的形式，但如果用到的地方比较多就比较麻烦。如果用ES6解构，就会失去响应式。

如果我们利用toRefs可以将一个响应式 reactive 对象的所有原始属性转换为响应式的ref属性。

前面提到的`setup`函数的`props`要解构可以使用`toRefs`。

示例代码如下：
```js
<template>
  <div class="hello">
    <p>number: {{number}}</p>
  </div>
</template>

<script>
import { toRefs, reactive } from 'vue'
export default {
  setup() {
    const state = reactive({
      number: 0,
      date: '20210628'
    })
    const state2 = toRefs(state)
    setInterval(() => {
      state.number++ 
    }, 1000)
    return {
      ...state2
    }
  }
}
</script>
```

### computed函数
与Vue2中的 computed 配置功能一致，返回的是一个 ref 类型的对象。

```js
setup() {
  const state = reactive({
    count: 0,
    plusOne: computed(() => state.count + 1)
  })
  return {
    ...toRefs(state)
  }
}
```

### watch函数
监视指定的一个或多个响应式数据，一旦数据变化，就自动执行监视回调。

```js
import { watch, ref, reactive } from 'vue'
export default {
  setup() {
    const name = ref('前端极客技术')
    const otherName = reactive({
      firstName: '前端',
      lastName: '技术'
    })
    watch(name, (newValue, oldValue) => {
      console.log(newValue, oldValue)
    })
    watch(
      () => {
        return otherName.firstName + otherName.lastName
      },
      value => {
        console.log(value)
      }
    )

    setTimeout(() => {
      name.value = '前端极客'
      otherName.firstName = '前端极客'
    }, 3000)
  }
}
```

### watchEffect
为了根据响应式状态自动应用和重新应用副作用，我们可以使用 watchEffect 方法。它立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> logs 0

setTimeout(() => {
  count.value++
  // -> logs 1
}, 100)
```

### 生命周期
通过下面的图来看下Vue2和Vue3生命周期钩子的对比：

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/20210628224421.png)

- Vue3新增了setup
- Vue2中的beforeDestroy名称变更为beforeUnmount
- Vue2中的destroyed变更为 unmounted
- Vue3.x 还新增用于调试的钩子函数onRenderTriggered和onRenderTricked

```js
setup() {
  onBeforeMount(() => {
    console.log('--onBeforeMount')
  })
  onMounted(() => {
    console.log('--onMounted')
  })
  onBeforeUpdate(() => {
    console.log('--onBeforeUpdate')
  })
  onUpdated(() => {
    console.log('--onUpdated')
  })
  onBeforeUnmount(() => {
    console.log('--onBeforeUnmount')
  })
  onUnmounted(() => {
    console.log('--onUnmounted')
  })
}
```

### getCurrentInstance
getCurrentInstance 支持访问内部组件实例，用于高阶用法或库的开发。

```js
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties // 访问 globalProperties
  }
}
```

> getCurrentInstance 只能在 setup 或生命周期钩子中调用。
>
> 如需在 setup 或生命周期钩子外使用，请先在 setup 中调用 getCurrentInstance() 获取该实例然后再使用。

## Teleport
Teleport是Vue3中新增的特性，翻译成中文就是传送的意思。`Teleport`提供了一种简洁的方式，让组件的`html`在父组件界面外的特定标签下插入显示。也就是我们可以把 子组件 或者 dom节点 插入到任何你想插入到的地方去。

语法：
> 使用`to`属性，指定要插入的节点。其值为选择器

```vue
<teleport to="body"></teleport>
```

我们使用Teleport实现一个模态窗口：
```vue
<template>
  <button @click="modalOpen = true">
    弹出一个全屏模态窗口</button>

  <teleport to="body">
    <div v-if="modalOpen" class="modal">
      <div>
        这是一个模态窗口!
        我的父元素是"body"！
        <button @click="modalOpen = false">Close</button>
      </div>
    </div>
  </teleport>
</template>

<script>
import { ref } from 'vue';
export default {
  setup() {
    const modalOpen = ref(true)
    return {
      modalOpen
    }
  }
};
</script>

<style scoped>
.modal {
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  background-color: rgba(0,0,0,.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.modal div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: white;
  width: 300px;
  height: 300px;
  padding: 5px;
}
</style>
```

## Suspense
> Suspense 是一个试验性的新特性并且其 API 可能随时更改。它不应该被用在生产环境。

在正确渲染组件之前进行一些异步请求是很常见的事。组件通常会在本地处理这种逻辑，绝大多数情况下这是非常完美的做法。

该 `<suspense>` 组件提供了另一个方案，允许等待整个组件树处理完毕而不是单个组件。

这个 `<suspense>` 组件有两个插槽。它们都只接收一个子节点。default 插槽里的节点会尽可能展示出来。如果不能，则展示 fallback 插槽里的节点。

常见的一个用例：用到了异步组件

```vue
// 父组件
<template>
  <suspense>
    <template #default>
      <async-demo />
    </template>
    <template #fallback>
      <div>
        Loading...
      </div>
    </template>
  </suspense>
</template>

<script>
import {defineAsyncComponent} from 'vue'
export default {
  components: {
    AsyncDemo: defineAsyncComponent(() => import('./AsyncDemo.vue'))
  }
}
</script>


// 子组件
<template>
  <h2>AsyncDemo</h2>
  <p>{{msg}}</p>
</template>

<script lang="ts">
export default {
  name: 'AsyncDemo',
  setup () {
     return new Promise((resolve) => {
       setTimeout(() => {
         resolve({
           msg: 'async import'
         })
       }, 2000)
     })
  }
}
</script>
```

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/vue3-suspense-demo.gif)


## Fragment
在Vue2中，`template`中只允许有一个根节点，但是在Vue3中，可以直接写多个根节点。

```html
<template>
  <div></div>
  <div></div>
  <div></div>
</template>
```

## 破坏性变化

- Global API 改为应用程序实例调用
- Global and internal APIs重构为可做摇树优化
- model选项和v-bind的sync 修饰符被移除，统一为v-model参数形式
- 渲染函数API修改
- 函数式组件仅能通过简单函数方式创建
- 废弃在SFC的template上使用functional或者添加functional选项的方式声明函数式组件
- 异步组件要求使用defineAsyncComponent 方法创建
- 组件data选项应该总是声明为函数
- 自定义组件白名单执行于编译时
- is属性仅限于用在component标签上
- `$scopedSlots` 属性被移除，都用$slots代替
- 特性强制策略变更
- 自定义指令API和组件一致
- 一些transition类名修改:
  - v-enter -> v-enter-from
  - v-leave -> v-leave-from
- watch 选项 和$watch 不再支持点分隔符字符串路径, 使用计算函数作为其参数
- Vue 2.x中应用程序根容器的 outerHTML 会被根组件的模板替换 (或被编译为template)。Vue 3.x现在使用应用根容器的innerHTML取代.

## 移除

- 移除keyCode 作为 v-on 修饰符
- on,on, on,off and $once 移除
- Filters移除
- Inline templates attributes移除

## 最后
本文只是介绍了Vue3中的一些常用的新特性，更多具体内容需要大家详细阅读Vue3的官方文档：

https://v3.cn.vuejs.org/


> 如果你觉得这篇内容对你有帮助的话：
>
> 1. **点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2. 关注公众号：**前端极客技术**，我们一起学习一起进步。


![](https://gitee.com/HanpengChen/blog-images/raw/master/%E5%89%8D%E7%AB%AF%E6%9E%81%E5%AE%A2%E6%8A%80%E6%9C%AF%E4%BA%8C%E7%BB%B4%E7%A0%81.png)