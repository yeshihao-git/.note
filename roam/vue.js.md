---
tags:
  - js
  - 前端
---
# 核心特性
## SFC（单文件组件）

- `<template>`：编写 HTML 结构
- `<script>`：编写 [[javascript]] 逻辑
- `<style>`：编写 CSS 样式

### 区别：`<script>` 和 `<script setup>` 

- `<script>`：原生 选项式/组合式 API 写法，支持 vue2/3，写法繁琐
- `<script setup>`：vue3 组合式 API 语法糖，支持 vue3，写法简单

|                   | `<script>`                        | `<script setup>`                      | 说明                 |
| ----------------- | --------------------------------- | ------------------------------------- | ------------------ |
| **导出组件**          | 手动 `export default` 导出            | 自动导出                                  | 导出组件：告诉 vue 这是一个组件 |
| **暴露变量方法**        | `return` 暴露给模板                    | 自动暴露                                  | 组件导出后才能暴露给模板       |
| **组件 / 指令引入**     | 导入后手动注册 `components`/`directives` | 导入后自动注册                               |                    |
| **props / emits** | 手动声明后从 `setup()` 形参获取使用           | 使用编译器宏 `defineProps`/`defineEmits` 定义 |                    |
| **使用场景**          | vue2、vue3 老项目                     | vue3 新项目                              |                    |

```js
/* <script> */

<template>
  <div>{{ msg }}</div>
  <button @click="handleClick">点击</button>
  <Child v-color />
</template>

<script>
// 1. 手动导入组件、指令
import Child from './Child.vue'
import vColor from './vColor'

export default {
  // 2. 手动注册组件、指令
  components: { Child },
  directives: { color: vColor },

  // 3. 手动声明 props / emits
  props: ['msg'],
  emits: ['ok'],

  setup(props, { emit }) {
    // 4. 使用 props / emits
    const handleClick = () => emit('ok')

    // 5. return 暴露方法给模板
    return { handleClick }
  }
}
</script>
```

```js
/* <script setup> */

<template>
  <div>{{ msg }}</div>
  <button @click="handleClick">点击</button>
  <Child v-color />
</template>

<script setup>
// 1. 组件、指令导入即自动注册
import Child from './Child.vue'
import vColor from './vColor'

// 2. 直接用编译器宏定义 props / emits
const props = defineProps(['msg'])
const emit = defineEmits(['ok'])

// 3. 自动暴露方法给模板
const handleClick = () => emit('ok')
</script>
```

### 区别：`<style>` 和 `<style scoped>`
`
- `<style>`：**全局样式**，作用于整个项目所有组件
- `<style scoped>`：**局部样式**，**只作用于当前组件**，不会污染其他组件

## 响应式

**what**：
监听数据变化，视图自动更新。定义方式：
- `ref`：适用于**基本类型**
- `reactive`：适用于**引用类型**（对象、数组、函数 等）

## props、emit（组件通信）

**what**：
- `props`：父 → 子
- `emit`：子 → 父
`props` 和 `emit` 都是在**子组件**里声明、定义的，父组件只是 “传值 / 监听”

## 生命周期

**what**：
vue组件创建、挂载、更新、销毁的完整运行过程，vue 提供了多个生命周期钩子，可以在这几个过程中插入自定义逻辑

**How**：四大阶段 + 核心钩子用法
1. 创建阶段（组件实例刚生成，DOM 未生成）
2. 挂载阶段（渲染 DOM，页面出现组件）
3. 更新阶段（响应式数据改变，视图重新渲染）
4. 销毁阶段（组件卸载）

## computed（计算属性 ）

**what**：
基于响应式依赖自动缓存的派生值
- 依赖变了 → 自动重算
- 依赖不变 → 读缓存，不重算

**how**：使用场景
1. 模版里的复杂表达式（动态绑定的类名等也可以）
```vue
<!-- 不推荐 -->
<div>{{ list.filter(i=>i.age>18).length }}</div>

<!-- 推荐 -->
<div>{{ adultCount }}</div>
computed:{ adultCount(){ return this.list.filter(...).length } }
```

## watch（侦听器）

**what**：
监听响应式数据值变化

| watch       | 事件监听        |
| ----------- | ----------- |
| 监听 响应式数据值变化 | 监听 DOM 交互行为 |
