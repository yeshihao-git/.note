---
tags:
  - cpp
---
# extern

**what**：
声明 变量/函数 存在外部定义，extern修饰的 全局变量 具有 外部链接性

```cpp
// a.cpp
int g_x { 2 };              // 非const全局变量 默认具有 外部链接性
extern const int g_y { 3 }; // extern 给予 g_y 外部链接性
```

```cpp
// main.cpp
extern int g_x;       // extern 前向声明 变量 g_x 定义在其他地方
extern const int g_y; // extern 前向声明 const变量 g_y 定义在其他地方

int main() {
    std::cout << g_x << ' ' << g_y << '\n'; // prints 2 3
}
```
