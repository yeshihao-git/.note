---
tags:
  - cpp
---
# inline

**what**：
内联关键字
1. 建议编译器将 函数 嵌入到调用处（没有函数调用开销）
2. 允许 函数 / 变量 在多个翻译单元重复定义而不违反 单一定义规则（最终由 链接器 去重）；（使用条件：定义必须完全相同、编译器在使用处必须看到完整定义）
```cpp
// math.cpp
inline double pi() { return 3.14159; } // pi() 第一定义

double circumference(double radius) { // circumference(double) 定义
    return 2.0 * pi() * radius;
}
```
```cpp
// main.cpp
double circumference(double radius);   // circumference(double) 前向声明

inline double pi() { return 3.14159; } // pi() 第二定义，因为是 inline函数，因此链接器会帮我们去重，不会违反 单一定义规则

int main() {
    std::cout << pi() << '\n';
    std::cout << circumference(2.0) << '\n';
}
```
3. （cpp17）inline变量具有 外部链接性

**why**：
我们有较短的几行代码需要经常使用，考虑用函数封装（好处在于模块化，更容易使用和管理），但函数调用有开销且需要频繁调用，因此使用内联函数