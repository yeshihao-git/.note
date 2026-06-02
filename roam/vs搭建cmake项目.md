---
tags:
  - vs
  - cpp
  - cmake
---
# vs搭建cmake项目
## cmake项目创建 

**why**：
在现代 C++ 开发中，优先选择 cmake 项目 而非传统的 .vcxproj
- 优点：跨平台、易于版本管理、配置灵活。

**how**：创建步骤
VS 启动页 -> 创建新项目 -> 搜索 "CMake" -> 命名项目（如 MathMaster）。

## 制作和使用静态库

**what**：静态库
静态库在链接阶段会被直接“拷贝”并嵌入到最终的 .exe 中。

**how**：制作静态库
1. 编写 MathUtils.h 和 MathUtils.cpp
2. 在 CMakeLists.txt 中添加：
```cmake
add_library(MathLib STATIC MathUtils.cpp)
```

**how**：使用静态库
1. 包含 ：在主程序中使用 `#include "MathUtils.h"`
2. 链接 ：
```cmake
target_link_libraries(MathMaster PRIVATE MathLib)
```

## 制作和使用动态库

**why**：动态库（DLL）
- 共享性 ：内存中只需一份拷贝，多个程序可共享
- 模块化 ：更新功能只需替换 `.dll` ，无需重新编译主程序

**how**：制作动态库
1. 编写 cmake 文件
```cmake
add_library(MathLib SHARED MathUtils.cpp)
# 激活导出宏
target_compile_definitions(MathLib PRIVATE EXPORT_MATH_LIB)
```
2. 源文件中导出（Windows 默认隐藏 DLL 内部函数，必须显式声明出口）
```cpp
// MathUtils.h 中定义的宏
#ifdef EXPORT_MATH_LIB
    #define MATH_API __declspec(dllexport)
#else
    #define MATH_API __declspec(dllimport)
#endif

extern "C" MATH_API int add(int a, int b);
```

**how**：使用动态库
1. 程序运行时进行 DLL 寻址
   程序运行阶段， .exe 必须在同级目录或系统路径下找到 .dll 文件
   自动搬运技巧：使用 CMake 的 POST_BUILD 命令自动将 DLL 复制到 EXE 目录
```cpp
// 一个自动搬运的例子
add_custom_command(TARGET MathMaster POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    $<TARGET_FILE:MathLib>
    $<TARGET_FILE_DIR:MathMaster>
)
```

