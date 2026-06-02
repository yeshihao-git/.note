---
tags:
  - cpp
---
# lambda表达式

**what**：
本质上是 匿名仿函数

**what**：仿函数
重载了 `operator()` 的类对象，使得它们可以像函数一样被调用，仿函数是 带状态的（不同于函数，类中的成员变量可以保存状态）
```cpp
[ captureClause ] ( parameters ) -> returnType
{
    statements;
}
// [] 捕获外部块的标识符
// returnType 可省略。若没指定 returnType，则 () 也能省略
[]{}; // 省略的终极形态
```

## 性质
1. 闭包 类型 （每个 lambda表达式 的类型都是独一无二的，因为本质是匿名类对象，而类名就是类型名），用 auto 推导类型，总结 存储 lambda表达式 的方式：
	- auto（推荐）
	- std::function
	- 函数指针
2. 返回类型推导 ：若 lambda表达式内部多个 return 的类型不同，指定 后置返回类型
```cpp
int main()
{
  // FIXME 发生错误的情况：两个 return 返回类型不同
  // auto divide{ [](int x, int y, bool intDivision) {
  //   if ...
  //     return x / y;                      // int
  //   else
  //     return static_cast<double>(x) / y; // double
  // }

  // NOTE 显式指定 后置返回类型
  auto divide{ [](int x, int y, bool intDivision) -> double {
    if (intDivision)
      return x / y;                         // 编译器根据 后置返回类型 进行隐式转换
    else
      return static_cast<double>(x) / y;
  } };

  std::cout << divide(3, 2, true) << '\n';
  std::cout << divide(3, 2, false) << '\n';
}
```
3. lambda捕获 ：值捕获、引用捕获
	- 值捕获 原理 ：值捕获的变量，在lambda内部创建一个外部变量的拷贝（名称相同且用外部变量初始化）作为 成员变量
	- 值捕获 的变量是 常量 ，使用 mutable 才能修改（修改的是内部的副本）
	    引用捕获 的变量 不是常量 ，可以修改捕获的变量
	- 默认捕获 ： = 值捕获所有变量， & 引用捕获所有变量
	- 捕获的同时定义新变量
```cpp
// 捕获外部的 width、height 定义新变量 userArea
[userArea{ width * height }](int knownArea) {
    return userArea == knownArea
}
```
