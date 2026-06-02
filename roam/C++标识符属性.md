---
tags:
  - cpp
---
# C++标识符属性
## 存储期

**what**：
（ 运行时 ） 对象何时创建和销毁 ，分为：
+ 自动存储期：定义时 创建 ，代码块结束时 销毁 。 存储 在 栈内存
+ 静态存储期：程序开始时 创建（ main() 执行前），程序结束时 销毁。 存储 在 .data（已初始化）或 .bss（未初始化）段
+ 动态存储期：程序员手动创建和销毁（由 new 或 malloc 手动创建 ，delete 或 free 手动销毁 ）。 存储 在 堆内存
+ 线程存储期：对象存储期和线程绑定

## 链接性

**what**：
（链接时 ）标识符在多个文件之间的可见范围（局部变量无链接性），分为：
  1. 内部链接性
  2. 外部链接性

### 内部链接性

**what**：
标识符只在当前文件可见，跨文件不可见（两个翻译单元中有同名标识符也不会冲突），有哪些？
  1. const/constexpr 全局变量
  2. static 全局变量
```cpp
// a.cpp
[[maybe_unused]] constexpr int g_x { 2 }; // g_x 具有内部链接性，只能在 a.cpp 中访问
```
```cpp
// main.cpp
static int g_x { 3 };  // g_x 具有内部链接性，只能在 main.cpp 中访问
const int g_y { 2 };   // const全局变量，只能在 main.cpp 中访问
constexpr int g_z {4}; // constexpr全局变量，只能在 main.cpp 中访问

int main() {
    std::cout << g_x << '\n';
}
```
  3. static 函数 
```cpp
// add.cpp
[[maybe_unused]] static int add(int x, int y) // add() 是 static的，具有内部链接性
{
    return x + y;
}
```
```cpp
// main.cpp
int add(int x, int y); // 前向声明 add()
int main() {
    std::cout << add(3, 4) << '\n';
}
```
  4. 未命名命名空间内的内容

### 外部链接性

**what**：
标识符跨文件可见，有哪些？
1. 函数
2. 全局变量
3. inline 全局变量（cpp17）
4. extern const/constexpr 全局变量
5. inline const/constexpr 全局变量

## 作用域
**what**：
（ 编译时 ）标识符在单个文件内的可见范围，分为：
  1. 全局作用域
  2. 块作用域
  3. 类作用域
  4. 命名空间 作用域
  5. 枚举作用域