---
tags:
  - js
  - 前端
---
# 核心特性
## 函数是一等公民

**what**：
函数是==对象==（普通对象能做的，函数全部能做）

```js
// 赋值给变量、对象、数组
const fn = function() {}
const obj = { hello: () => {} }
const list = [() => 1, () => 2]

// 作为参数（回调）
arr.forEach(fn)

// 作为返回值（闭包核心）
function outer() { return function inner() {} }
```

**why**：
javascript 中的高级能力都是基于此（高阶函数、闭包、回调、异步、函数式编程等）

### this 的绑定（为什么方法引用会丢失 this）

**what**： 
因为函数是一等公民，方法可以被赋值 / 传递； 一旦脱离 `对象.` 调用形式，this 就会丢失

```javascript
window.fn = obj.method;           // ❌ fn() 时 this 不是 obj
window.fn = () => obj.method();   // ✅ 箭头函数 + 点号调用
window.fn = obj.method.bind(obj); // ✅ bind 显式绑定
```

**原因**：
- this 在调用时确定，取决于调用形式
- "对象.方法()" 时 this 是对象；单独调用时 this 不是对象
- 赋值方法引用会断开方法与对象的关联

**修复方案**
1. 箭头函数包装：`window.fn = () => obj.method()`
2. bind 绑定：`window.fn = obj.method.bind(obj)`
3. 直接点号调用：`obj.method()`

## javascript 是主线程单线程的

**what**：
javaScript 是单线程，且内部使用的是==事件循环机制==；但运行环境（浏览器 / Node）是多线程的
```
运行环境中的 定时器、网络请求、IO、渲染、Worker 都是别的线程
它们干完活，把结果丢到任务队列，等主线程空了，再回来执行回调
```

**why**：
为了简化 DOM 同步操作（浏览器环境首要任务是操作 DOM，如果 js 是多线程，多个线程同时修改 DOM 会产生竞态冲突）

# 订阅与取消订阅

用于防止内存泄露

# JSX 和 TSX 之间的区别

- 写 `JavaScript` 且用 `HTML` 语法 → `.jsx`
```jsx
function App() {
  return <div>Hello JSX</div>; // 直接写 HTML
}
```

- 写 `TypeScript` 且用 `HTML` 语法 → `.tsx`
```tsx
interface Props {
  name: string;
}

function App(props: Props) {
  return <div>Hello {props.name}</div>;
}
```

- `vue3` 完全支持 `JSX / TSX`

# 开发、打包、部署、发布

1. **开发**：编写源代码

2. **打包（build / package）**： 源代码 → 浏览器能直接运行的代码，放入`dist` 文件夹
> **示例**：`.vue`、`.tsx`、`scss` 打包后变成 `js`/`css`/`html`   
> **打包工具**：vite、webpack、parcel 
> **树摇**：打包时自动把没用到的代码 “摇” 掉，只保留实际运行需要的代码

3. **部署（deploy）**： `dist` 文件夹 → 放到服务器上
> **示例**： 将打包产物放到阿里云/腾讯云服务器、GitHub Pages、Vercel 等平台

4. **发布（publish / release）**： 对外发布新版本

