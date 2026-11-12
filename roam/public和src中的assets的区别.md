---
tags:
  - 前端
  - 八股
---
# 区别
## 1. public 和 src/assets 的区别

假设有一张图片 `logo.png`。

方式一：放在 public

```
public/
└── logo.png
```

使用：

```
<template>
  <img src="/logo.png" />
</template>
```

构建后：

```
dist/
├── index.html
└── logo.png
```

特点：

- Vite 不处理图片内容。
- 构建时原样复制。
- 浏览器运行时通过 URL 请求图片。
- 不参与 Vite 的资源依赖分析。

方式二：放在 src/assets

```
src/
└── assets/
    └── logo.png
```

使用：

```
<script setup>
import logo from '@/assets/logo.png'
</script>

<template>
  <img :src="logo" />
</template>
```

构建后：

```
dist/
├── index.html
└── assets/
    └── logo.a82f31.png
```

特点：

- Vite 在构建阶段识别图片资源。
- 参与构建依赖关系分析。
- 可以进行文件名哈希、资源优化。
- 浏览器运行时仍然通过 URL 请求图片。

[image](https://www.google.com/s2/favicons?domain=https://vite.dev\&sz=32)

Vite+1

## 2. 两者运行速度有区别吗？

这里是关键：

两者通常都需要浏览器向服务器请求图片，并不是 assets 就不需要动态加载。

例如：

```
import logo from '@/assets/logo.png'
```

这段代码在构建阶段会被 Vite 转换成类似：

```
const logo = '/assets/logo.a82f31.png'
```

浏览器最终还是需要请求这个 URL。

因此：

| 对比        | public        | src/assets |
| --------- | ------------- | ---------- |
| 构建阶段处理    | 原样复制          | 参与资源处理     |
| 浏览器是否请求资源 | 是             | 通常是        |
| 是否支持文件名哈希 | 否             | 是          |
| 是否支持资源优化  | 不经过 Vite 资源处理 | 是          |
| 是否自动纳入依赖图 | 否             | 是          |

## 3. 那么应该如何选择？

我建议记住一个原则：



优先推荐

`src/assets`



| 使用场景               | 推荐位置       |
| ------------------ | ---------- |
| 组件使用的图片            | src/assets |
| 页面背景图片             | src/assets |
| 字体文件               | src/assets |
| CSS 图标资源           | src/assets |
| favicon.ico        | public     |
| robots.txt         | public     |
| 必须保持原始文件名的资源       | public     |
| 不需要 import 的固定路径资源 | public     |

Vite 官方也明确建议：一般优先使用 import 管理资源，只有需要 public 特殊保证时才使用 public。

[image](https://www.google.com/s2/favicons?domain=https://vite.dev\&sz=32)

Vite



最后纠正你最初的理解：

不是：

- public：运行时动态加载，所以慢。
- assets：构建时打包进去，所以快。

而是：

- public：构建时原样复制，运行时通过固定 URL 获取。
- assets：构建时纳入资源依赖管理，运行时通过构建生成的 URL 获取。

两者的核心差异在于构建阶段的资源管理方式，而不是浏览器运行阶段的加载方式。

# 为什么组件使用的资源放 src/assets 更好

当然可以！组件使用的图片、字体文件完全可以放在 `public` 中。并不是说组件资源必须放在 `src/assets`，而是两种方式各有优势。

我之前推荐 `src/assets`，主要是考虑组件的可维护性和资源管理。

## 1. 放在 public 中

例如你有一个通用按钮组件：

```
public/
└── static/
    └── icons/
        └── search.svg

src/
└── components/
    └── SearchButton.vue
```

组件中这样使用：

```
<template>
  <button>
    <img src="/static/icons/search.svg" />
    搜索
  </button>
</template>
```

完全没问题。

但有一个问题：组件依赖了一个外部固定路径。

如果你把这个组件单独抽出来，放到另一个项目中使用，那么另一个项目也必须提供：

```
/static/icons/search.svg
```

否则图片就会加载失败。

## 2. 放在 src/assets 中

目录：

```
src/
├── assets/
│   └── icons/
│       └── search.svg
└── components/
    └── SearchButton.vue
```

组件中：

```
<script setup lang="ts">
import searchIcon from '@/assets/icons/search.svg'
</script>

<template>
  <button>
    <img :src="searchIcon" />
    搜索
  </button>
</template>
```

优势：

- 资源成为项目构建依赖的一部分。
- Vite 能够分析资源依赖。
- 构建时自动处理资源路径。
- 组件迁移时，可以一起迁移资源。
- 不容易出现组件依赖的图片找不到的问题。

## 3. 两种方式如何选择？

| 场景             | 推荐位置          |
| -------------- | ------------- |
| 组件内部使用的图标      | src/assets    |
| 组件内部使用的字体      | src/assets    |
| 页面背景图片         | src/assets    |
| 网站 Logo        | 两者均可          |
| favicon        | public        |
| robots.txt     | public        |
| 外部动态配置的图片 URL  | public 或外部服务器 |
| 需要保持固定 URL 的文件 | public        |

## 4. 对于通用组件库，我更推荐这样设计

假设你正在开发自己的 Vue 通用组件库：

```
src/
├── assets/
│   ├── icons/
│   ├── fonts/
│   └── images/
│
├── components/
│   ├── Button/
│   ├── Dialog/
│   ├── Tree/
│   └── Menu/
│
└── index.ts

public/
└── static/
    ├── models/
    └── config/
```

这里有一个重要原则：

组件自身运行所必需的资源，尽量跟随组件一起管理；项目级别、需要固定 URL 访问的资源，放在 public。

这样以后你将组件库打包成 npm 包，其他项目安装后也能正确使用组件及其内部资源。