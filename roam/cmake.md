---
tags:
  - 工具
  - cpp
---
# cmake

**what**：
构建系统生成器

```zsh
cd <项目目录>

# 创建 build 目录，使用 Ninja 构建系统，设置构建类型，设置安装前缀（linux 默认为 /usr/local）
cmake -B build [-G Ninja] [-DCMAKE_BUILD_TYPE=XXX] [-DCMAKE_INSTALL_PREFIX=XXX]

# 编译 build 文件夹中的项目，安装
cmake --build build [--target install]

# 运行项目
./build/<可执行文件> 
```

## 四个阶段

`cmake -B build [-G Ninja] [-DCMAKE_BUILD_TYPE=XXX] [-DCMAKE_INSTALL_PREFIX=XXX]`
- **配置阶段**：获取项目、环境、用户提供的配置信息，生成 `CMakeCache.txt`，此阶段生成器表达式只会被当成字符串
- **生成阶段**：根据选择的生成器生成实际的构建文件；多阶段配置在这一步区分

`cmake --build build -j8`
- **构建阶段**：调用编译器编译链接生成可执行文件

`cmake --build build --target install`
- **安装阶段**：将构建完的产物复制到指定位置

## CMakeLists.txt
### 概念
#### 变量

| 类型   | 说明                           | 定义                        | 引用       |
| ---- | ---------------------------- | ------------------------- | -------- |
| 普通变量 | 作用域是当前及子目录的 CMakeLists.txt   | set (var 值)               | ${}      |
| 缓存变量 | 写入 CMakeCache.txt；作用域是整个构建目录 | set (var 值 CACHE 类型 "描述") | $CACHE{} |
| 环境变量 |                              |                           | $ENV{}   |

父模块定义的变量 会 传递给子模块，子模块定义的变量 不会 传递给父模块；
设置子传父：通过在子模块中 `set(<变量> <值> PARENT_SCOPE)`

#### 伪目标和真实目标

**what**：伪目标
不生成文件，用于清理、测试、安装等辅助操作（类似make clean）
add_custom_target

**what**：真实目标
生成可执行文件、库或输出文件，用于编译代码、链接库
add_executable / add_library

#### 缓存生成机制

1. 检测环境：cmake -B build
2. 将结果写入缓存文件 CMakeCache.txt，可以通过ccmake可视化查看 CMakeCache.txt
3. 清理缓存：外部环境发生变化 => 删除CMakeCache.txt即可

#### 属性传播规则

**why**：
控制对目标的可见性
- PUBLIC    ：对当前目标可见，传递给链接到此目标的其他目标
- PRIVATE   ：对当前目标可见，不传递给链接到此目标的其他目标
- INTERFACE ：对当前目标不可见，传递给链接到此目标的其他目标

### 模板：模块化项目

```bash
# 目录组织
├── biology                     # 每新增一个功能模块，都和biology一样的目录结构
│   ├── CMakeLists.txt          # 创建库
│   ├── include
│   │   └── biology
│   │       ├── Animal.h
│   │       └── Carer.h
│   └── src
│       ├── Animal.cpp
│       └── Carer.cpp
├── cmake
│   └── MyUsefulFuncs.cmake
├── CMakeLists.txt              # 设置全局设定；通过add_subdirectory将两个子项目biology和pybmain添加进来
└── pybmain
    ├── CMakeLists.txt          # 可执行文件
    ├── include
    │   └── pybmain
    │       └── myutils.h
    └── src
        └── main.cpp
```

```cpp
// 代码层面
// 头文件（项目名/include/项目名/模块名.h）
#pragma once
namespace 项目名 {      // 避免符号冲突
    void 函数名();
}

// 实现文件（项目名/src/模块名.cpp）
#include <项目名/模块名.h>
namespace 项目名 {
    void 函数名() {
      // 函数实现
    }
}
```

### 调试/打印信息

```cmake
message([<mode>] "message" ...) # <mode> 用于指定消息类型
```

|消息类型|输出格式|
|---|---|
|空|message|
|STATUS|--message|
|WARNING|(黄色字体带警告) message|
|FATAL_ERROR|(红色字体终止运行) message|

### 三方库配置
#### find_package

**what**：
cmake 原生查找库的方式

##### 查找包的模式

默认先 MODULE模式，后 CONFIG模式

1. CONFIG模式
包配置文件   ： `<Package>Config.cmake= / =<Package>-config.cmake`
包版本文件   ： `<Package>ConfigVersion.cmake= / =<Package>-config-version.cmake`
变量/缓存变量：

|变量 / 缓存变量|作用|
|---|---|
|<包名>_FOUND|包是否找到|
|<包名>_DIR|包所在目录|
|<包名>_VERSION|包版本|

2. MODULE模式
包配置文件： `Find<Package>.cmake`

#### pkg_check_modules

cmake 调用 pkg-config 查找库的方式
```cmake
find_package(PkgConfig REQUIRED)
# pkg_check_modules(<prefix> [REQUIRED] <package> [<version>])
# 1. <prefix> 指定相关变量前缀，这里prefix设置为HIREDIS，因此hiredis头文件目录为HIREDIS_INCLUDE_DIRS、库文件路径为HIREDIS_LIBRARIES
# 2. <package> 库的名称，和 pkg-config 的 .pc 文件名相同
pkg_check_modules(HIREDIS REQUIRED hiredis)
...
target_include_directories(test_redis PRIVATE ${HIREDIS_INCLUDE_DIRS})
target_link_libraries(test_redis PRIVATE ${HIREDIS_LIBRARIES})
```

#### 查找没有.cmake或.pc配置文件的库

```cmake
include_directories(/usr/lib)                # 包含库文件目录
file(GLOB absl_libs "/usr/lib/libabsl_*.so") # 将目录中所需的库文件存入变量
target_link_libraries(main ${absl_libs})     # 目标链接该变量
```

### 文件收集

将通配符匹配的文件存入变量
```cmake
file(GLOB <variable> [option] <glob_pattern>)
file(GLOB_RECURSE <variable> [option] <glob_pattern>)
```
- `GLOB`           ：通配符匹配文件，不递归子目录
- `GLOB_RECURSE`   ：通配符匹配文件，递归子目录
- `<variable>`     ：存储匹配文件路径的变量
- `<glob_pattern>` ：通配符模式，通常是文件扩展名，如 `*.cpp`
- `option`         ：设置 CONFIGURE_DEPENDS 选项 ，添加新文件会自动更新变量

### 安装

用于复制文件到指定目录
配置阶段设置安装路径： `cmake -B build -DCMAKE_INSTALL_PREFIX=xxx`（CMAKE_INSTALL_PREFIX只控制相对路径；不控制绝对路径）

相对路径和绝对路径
```cmake
# NOTE 相对路径
# 安装到 ${CMAKE_INSTALL_PREFIX}/bin/myapp
install(TARGETS myapp
  DESTINATION bin
)

# NOTE 绝对路径
# 安装到 /bin/myapp
install(TARGETS myapp
  DESTINATION /bin
)
```

## CMakeCache.txt

cmake的缓存文件，包含以下内容：
1. set设置的缓存变量，可以使用 ccmake命令行工具 可视化查看/编辑
2. find_package找到的库文件信息
3. 环境信息（编译器、操作系统等）

```zsh
# 目标系统默认安装前缀。Linux 默认值：`/usr/local`
CMAKE_INSTALL_PREFIX

# 可执行程序目录
CMAKE_INSTALL_BINDIR      # 默认 bin
# 库文件目录
CMAKE_INSTALL_LIBDIR      # 默认 lib
# 头文件目录
CMAKE_INSTALL_INCLUDEDIR  # 默认 include



#-------------------------------------------------------#
# /usr/local/include
${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR}

# /usr/local/lib
${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}

# /usr/local/bin
${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}
```


```zsh
# 可执行程序、Windows动态库dll
CMAKE_RUNTIME_OUTPUT_DIRECTORY

# 静态库 .a / .lib
CMAKE_ARCHIVE_OUTPUT_DIRECTORY

# Linux/macOS 动态库 .so / .dylib
CMAKE_LIBRARY_OUTPUT_DIRECTORY
```

## cmake 项目架构

![[cmake 项目架构]]