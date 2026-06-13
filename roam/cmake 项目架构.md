---
tags:
  - cpp
  - cmake
sources: "[bilibili - C哥智驾说 - vscode+cmake+git实战开发系列演示](https://www.bilibili.com/video/BV18W421c7Cy?vd_source=4441bc96046659b39d059d583f36ff52&spm_id_from=333.788.videopod.sections)"
---
# cmake 项目架构
## 简单项目1

![[Pasted image 20260613115406.png|193]]
  
## 简单项目2

![[Pasted image 20260613115416.png|361]]

## 复杂项目

![[Pasted image 20260613122729.png|599]]

```zsh
.
├── CMakeLists.txt # 一级 CMakeLists.txt
├── CMakePresets.txt
├── docs
├── src
│   ├── CMakeLists.txt # 二级 CMakeLists.txt
│   ├── main.cpp
│   ├── module1
│   │   ├── CMakeLists.txt # 三级 CMakeLists.txt
│   │   ├── module1.cpp
│   │   └── module1.h
│   └── module2
│       ├── CMakeLists.txt # 三级 CMakeLists.txt
│       ├── module2.cpp
│       └── module2.h
├── test
└── third_party
```

一级 `CMakeLists.txt`
```cmake
# cmake 最低版本
cmake_minimum_required(VERSION 3.22.0)

# 项目名称（cpack 打包时自动录入相关信息）
project(demo 
    VERSION 0.1.0
    DESCRIPTION "a demo of cmake"
    HOMEPAGE_URL "https://github.com/yeshihao-git/cmake_demo.git"
    LANGUAGES CXX
)

# 变量和目录设置
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

set(MODULE1_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/src/module1)
set(MODULE2_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/src/module2)

# 三方库设置
find_package(Eigen3 3.4 REQUIRED) # find_path 会自动检索 /usr/include、/usr/local/include

# 添加二级子目录
add_subdirectory(src)
```

二级 `CMakeLists.txt`
```cmake
# 项目名称
project(demo_main LANGUAGES CXX)

# 添加可执行文件
add_executable(${PROJECT_NAME} main.cpp)

# 包含头文件
target_include_directories(${PROJECT_NAME}
    PUBLIC
    ${MODULE1_INCLUDE_DIR}
    ${MODULE2_INCLUDE_DIR}
)

# 链接库文件
target_link_libraries(${PROJECT_NAME}
    PUBLIC
    module1
    module2
)

# 添加三级子目录
add_subdirectory(module1)
add_subdirectory(module2)
```

三级 `CMakeLists.txt`（module1）
```cmake
# 项目名称
project(module1)

# 添加库
add_library(${PROJECT_NAME}
    SHARED
    module1.cpp
)

# 包含头文件
target_include_directories(${PROJECT_NAME}
    PUBLIC
    ${MODULE1_INCLUDE_DIR}
    ${MODULE2_INCLUDE_DIR}
)

# 链接库文件
target_link_libraries(${PROJECT_NAME}
    PUBLIC
    Eigen3::Eigen
)
```

三级 `CMakeLists.txt`（module2）
```cmake
# 项目名称
project(module2)

# 添加库
add_library(${PROJECT_NAME}
    SHARED
    module2.cpp
)

# 包含头文件
target_include_directories(${PROJECT_NAME}
    PUBLIC
    ${MODULE1_INCLUDE_DIR}
)
```

**附录**：关键变量
```cmake
CMAKE_CXX_STANDARD          # 指定项目全局的 C++ 标准
CMAKE_CXX_STANDARD_REQUIRED # 是否必须使用上面指定的 C++ 标准
CMAKE_CXX_EXTENSIONS        # 是否启用编译器扩展语法

CMAKE_RUNTIME_OUTPUT_DIRECTORY # 可执行文件 输出路径
CMAKE_LIBRARY_OUTPUT_DIRECTORY # 动态库 输出路径
CMAKE_ARCHIVE_OUTPUT_DIRECTORY # 静态库 输出路径

PROJECT_NAME     # project 命令设置的项目名称
CMAKE_SOURCE_DIR # 顶层源码目录绝对路径（最外层 CMakeLists.txt 所在目录）
CMAKE_CURRENT_SOURCE_DIR # 当前正在解析的 CMakeLists.txt 目录（子目录专用）
CMAKE_BINARY_DIR # 顶层构建输出目录（通常为 build 文件夹）
```

**参考**：
- [github：cmake_demo](https://github.com/yeshihao-git/cmake_demo)（完整示例）
- [cmake_planning_demo](https://github.com/CODspielen/cmake_planning_demo)