---
tags:
  - 前端
---
# CSS
## 选择器

**what**：
选择页面元素，并用于设置样式

```css
/* 1. 元素选择器（标签） */
div { color: red; }

/* 2. 类选择器 */
.box { background: #fff; }

/* 3. ID 选择器 */
#header { font-size: 20px; }

/* 4. 通配符（全部元素） */
* { margin: 0; }

/* 5. 后代选择器（所有子子孙孙） */
div p { line-height: 1.5; }

/* 6. 子选择器（仅直接子元素） */
ul > li { list-style: none; }

/* 7. 相邻兄弟（下一个同级） */
h1 + p { margin-top: 10px; }

/* 8. 群组选择器（批量） */
h1,h2,h3 { color: #333; }

/* 9. 属性选择器 */
/* 以 [attr] 开头，根据属性匹配元素 */
input[type="text"] { border: 1px solid #ccc; }

/* 10. 伪类 */
/* 以 : 开头，描述元素状态（hover/active 等） */
a:hover { text-decoration: none; }

/* 11. 伪元素 */ 
/* 以 :: 开头，创建虚拟元素（装饰元素：线条、图标、前缀） */
p::before { content: ""; color: blue; }
```

## 样式优先级

**what**：
决定哪个样式生效的计算规则

**how**：
给不同选择器打分
1. 分高者生效
2. 分相同时，后写的生效

| 等级    | 选择器类型            | 权重   | 示例代码                                                            |
| :---- | :--------------- | :--- | :-------------------------------------------------------------- |
| 1（最高） | `!important`     | ∞    | `div { color: red !important; }`                                |
| 2     | 内联样式（`style` 属性） | 1000 | `<div style="color: red"></div>`                                |
| 3     | ID 选择器           | 100  | `#box { color: red; }`                                          |
| 4     | 类 / 伪类 / 属性选择器   | 10   | `.btn { color: red; }`<br>`button:active { background: #ddd; }` |
| 5     | 元素 / 伪元素选择器      | 1    | `p { color: red; }`                                             |
| 6（最低） | 通配符 `*` / 继承样式   | 0    | `* { color: red; }`                                             |

## CSS变量

**what**：
`--` 开头的变量

**why**：
用于存样式（颜色、尺寸、间距等），一处定义全局复用，方便统一改样式

**how**：调用
`var(变量名, 备用值)` 读取已声明的变量，无变量时使用备用值
```css
/* 声明变量 */
:root {
  --main-color: #2563eb;
  --bg-opacity: 0.9;
}

/* 调用变量 */
body {
  color: var(--main-color);
  /* 变量不存在时，使用 #333 作为备用色 */
  background: var(--random-color, #333);
}
```

### 作用域

作用域分为 全局和局部，遵循 局部优先原则
```css
/* 全局变量，全页面可用 */
:root {
  --color: red;         /* 全局 */
}

/* 局部变量 */
.demo {
  --color: blue;       /* 局部，覆盖全局 */
  color: var(--color); /* 最终显示蓝色 */
}
```

## CSS属性

**what**：
[[CSS#选择器|选择器]]大括号内==描述元素样式的键==，按功能可以分类为：
1. 布局类
2. 字体类
3. 背景视觉类
4. [[CSS#变换动画类|变换动画类]]
5. 交互显示类
6. 其他

### 变换动画类
#### transform

**what**：
实现元素的几何变换，不影响其他盒子布局

**how**：语法
```css
transform: 变换函数1() 变换函数2();
```

变换函数如下：

| 值 | 效果 |
|---|---|
| `translate(x, y)` | 平移 |
| `rotate(45deg)` | 旋转 |
| `scale(1.5)` | 缩放 |
| `skew(10deg)` | 倾斜 |

```css
/* 示例 */
transform: translate(10px, 20px) rotate(45deg)
```


