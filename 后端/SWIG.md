**what**：
简化包装器与接口生成器，是一个开源、跨语言的自动代码生成工具

**why**：
自动写跨语言调用的 "胶水代码"

---

**示例代码**：
```swig
// 1. 模块名 = 最终生成的 Java 主类名
%module IntevueLibModule

// 2. 纯 C++ 代码块，给 C++ 编译器看的
%{
#include "../include/IntevueLib.h"   // 你的 C++ 头文件路径
%}

// 3. 导入常用类型（自动支持 string / vector）
%include <std_string.i>   // 支持 string <-> Java String
%include <std_vector.i>   // 支持 vector <-> Java List

// 4. 让 SWIG 自动解析并包装你的 C++ 函数/类，给 SWIG 看的
%include "../include/IntevueLib.h"

// 生成：
// IntevueLibModule.java（主接口类）
// IntevueLibModuleJNI.java（包装代码）
// IntevueLibModule_wrap.cxx（JNI 胶水代码）
```

# 1. 跨语言类型转换
`typemap` 是 `SWIG` 定义**跨语言类型转换规则**的关键字，用于实现跨语言类型转换。
`SWIG` 内置 `.i` 文件实现了常用的**跨语言类型转换**，直接 `%include` 就能用，无需手写自定义 `typemap`；常见的内置 `.i` 文件如下：

| 内置 .i 文件              | C++ 类型                      | Java 对应类型       |
| --------------------- | --------------------------- | --------------- |
| `std_string.i`        | `std::string`               | `String`        |
| `std_vector.i`        | `std::vector<T>`            | `List<T>`       |
| `std_map.i`           | `std::map<K,V>`             | `Map<K,V>`      |
| `std_unordered_map.i` | `std::unordered_map<K,V>`   | `HashMap<K,V>`  |
| `std_set.i`           | `std::set<T>`               | `Set<T>`        |
| `std_unordered_set.i` | `std::unordered_set<T>`     | `HashSet<T>`    |
| `std_pair.i`          | `std::pair<T1,T2>`          | 自定义 Pair 类      |
| `std_list.i`          | `std::list<T>`              | `LinkedList<T>` |
| `stdint.i`            | `int8_t/uint32_t/int64_t` 等 | Java 基本整型       |
| `carrays.i`           | C 风格数组 `type[]`             | 数组类型            |
| `cpointer.i`          | 普通原生指针                      | 句柄 / 基本类型       |
