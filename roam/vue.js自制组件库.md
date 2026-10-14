---
tags:
  - vue
  - js
aliases:
  - vue.js自制组件库作为插件使用
---
# vue.js自制组件库

以 Vue 3 + TypeScript 为例把一个 Vue 组件打包成 npm 包，让其他 Vue 项目可以像使用 Element Plus 一样使用它
整个过程分成 5 步：

![[Pasted image 20261014100548.png]]

## 第一步：创建组件库项目

建议先把组件库独立成一个项目，而不是直接在现有业务项目里打包。

```
my-vue-ui/
├── src/
│   ├── components/
│   │   └── MyButton/
│   │       ├── MyButton.vue
│   │       └── index.ts
│   └── index.ts
├── package.json
├── vite.config.ts
├── tsconfig.json
└── tsconfig.build.json
```

这里有几个关键文件：

| 文件                  | 职责            |
| ------------------- | ------------- |
| `MyButton.vue`      | 组件本身          |
| `MyButton/index.ts` | 导出单个组件        |
| `src/index.ts`      | 组件库统一出口、插件入口  |
| `vite.config.ts`    | 配置如何打包        |
| `package.json`      | 包名、入口文件、依赖等信息 |

## 第二步：编写一个 Vue 组件

在 `src/components/MyButton/MyButton.vue` 中：

```

<script setup lang="ts">
withDefaults(
  defineProps<{
    label?: string
    width?: string
  }>(),
  {
    label: '点击我',
    width: '120px',
  },
)

const emit = defineEmits<{
  click: [event: MouseEvent]
}>()
</script>

<template>
  <button
    class="my-button"
    :style="{ width }"
    @click="emit('click', $event)"
  >
    {{ label }}
  </button>
</template>

<style scoped>
.my-button {
  padding: 10px 16px;
  border: none;
  border-radius: 6px;
  background: #2563eb;
  color: white;
  cursor: pointer;
}
</style>

```

这个组件有两个可配置属性：

- `label`：按钮文字。
- `width`：按钮宽度。

还有一个 `click` 事件，可以通知使用者按钮被点击了。

## 第三步：导出组件

创建 `src/components/MyButton/index.ts`：

```
import MyButton from './MyButton.vue'

export { MyButton }
export default MyButton
```

这个文件的作用是提供统一的导出入口，方便其他代码导入组件。

接下来创建 `src/index.ts`，作为整个组件库的入口：

```

import type { App, Plugin } from 'vue'
import MyButton from './components/MyButton'

export { MyButton }

const MyVueUI: Plugin = {
  install(app: App) {
    app.component('MyButton', MyButton)
  },
}

export default MyVueUI

```

这里最关键的是：

```
app.component('MyButton', MyButton)
```

它会将 `MyButton` 注册为全局组件。

而 `install` 是 Vue 插件规范约定的安装入口，不是 JavaScript 语言内置的特殊语法。Vue 的 `app.use()` 会调用插件的 `install` 方法。

## 第四步：配置打包工具 Vite

在组件库项目中安装依赖：

```
npm install vue
npm install -D vite typescript vue-tsc
npm install -D @vitejs/plugin-vue vite-plugin-dts
```

修改 `vite.config.ts`：

```

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import dts from 'vite-plugin-dts'

export default defineConfig({
  plugins: [
    vue(),
    dts({ tsconfigPath: './tsconfig.build.json' }),
  ],

  build: {
    lib: {
      entry: 'src/index.ts',
      formats: ['es', 'umd'],
      name: 'MyVueUI',
      fileName: (format) => `my-vue-ui.${format}.js`,
    },

    rollupOptions: {
      external: ['vue'],
      output: {
        globals: {
          vue: 'Vue',
        },
      },
    },
  },
})

```

这段配置主要做三件事：

1. 把 Vue 单文件组件编译成 JavaScript。
2. 将 Vue 本身排除在打包文件之外，避免把 Vue 重复打进组件库。
3. 生成 TypeScript 类型声明文件，方便使用者获得代码提示。

这里的 `es` 是 ES Module 格式；`umd` 主要用于传统脚本环境。一般给现代 Vue 项目使用时，ES Module 就够了，UMD 可以按需保留。

## 第五步：配置 package.json

这是非常关键的一步。打包工具负责生成文件，而 `package.json` 告诉其他项目应该从哪里加载你的组件库。

```
{
  "name": "my-vue-ui",
  "version": "1.0.0",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-vue-ui.umd.js",
  "module": "./dist/my-vue-ui.es.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/my-vue-ui.es.js",
      "require": "./dist/my-vue-ui.umd.js"
    }
  },
  "scripts": {
    "build": "vue-tsc -p tsconfig.build.json --noEmit && vite build"
  },
  "peerDependencies": {
    "vue": "^3.3.0"
  }
}
```

注意：`name` 是未来安装组件库时使用的包名；`files` 限制发布时包含的文件。`peerDependencies` 表示组件库需要宿主项目提供 Vue，而不是自己再捆绑一份。

创建 `tsconfig.build.json`：

```
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "declaration": true,
    "skipLibCheck": true,
    "lib": ["ES2020", "DOM"],
    "types": [],
    "noEmit": true
  },
  "include": ["src/**/*.ts", "src/**/*.vue"]
}
```

## 第六步：执行打包

在组件库项目根目录运行：

```
npm run build
```

正常情况下会生成 `dist/` 目录，里面包含编译后的 JavaScript 和类型声明文件。

再执行：

```
npm pack
```

它会生成一个类似 `my-vue-ui-1.0.0.tgz` 的压缩包。这就是可以拿去其他项目安装的组件库包。

## 第七步：在另一个 Vue 项目中使用

假设你已经有另一个 Vue 3 项目。

先安装本地生成的包：

```
npm install ../my-vue-ui/my-vue-ui-1.0.0.tgz
```

路径要根据你实际生成的 `.tgz` 文件位置调整。

### 方式 A：作为插件全局注册

在使用方项目的 `src/main.ts` 中：

```
import { createApp } from 'vue'
import App from './App.vue'

import MyVueUI from 'my-vue-ui'

const app = createApp(App)

app.use(MyVueUI)

app.mount('#app')
```

然后在任何组件中都可以直接使用：

```
<template>
  <MyButton
    label="提交"
    width="180px"
    @click="console.log('按钮被点击')"
  />
</template>
```

因为执行了 `app.use(MyVueUI)`，所以这里不需要再单独导入 `MyButton`。

### 方式 B：只使用单个组件

如果不想全局注册，也可以按需导入：

```
<script setup lang="ts">
import { MyButton } from 'my-vue-ui'
</script>

<template>
  <MyButton label="确认" width="100px" />
</template>
```

这种方式更明确，也方便按需使用组件。

## 最后：你需要记住的三个概念

| 概念                         | 作用                     |
| -------------------------- | ---------------------- |
| `install()`                | 定义 Vue 插件被安装时要执行什么操作   |
| `vite build`               | 把 Vue 组件编译、打包成可分发的文件   |
| `npm pack` / `npm publish` | 分别用于生成本地安装包、发布到 npm 仓库 |
