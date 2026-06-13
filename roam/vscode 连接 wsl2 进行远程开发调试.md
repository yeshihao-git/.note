---
tags:
  - vscode
---
# 环境搭建
## 配置 windows 功能

1. 配置 `windows` 功能
![[Pasted image 20260410103334.png|697]]

2. 重启生效

## 安装 wsl2 和 ubuntu22.04

1. 命令行中输入以下命令
```powershell
# 1. 更新 WSL2（自动下载并安装最新的 WSL2）
wsl --update

# 2. 验证安装
wsl --version 

# 3. 安装 ubuntu
wsl --install -d Ubuntu-22.04
```

2. [[zsh#zsh 配置|配置 zsh]]

## vscode 远程连接 wsl

1. 安装 `WSL` 插件  
![[Pasted image 20260410103847.png]]

2. 远程连接 `wsl`  
**方式1**：vscode 中点击 远程资源管理器
![[Pasted image 20260410104019.png]]  
**方式2**：登录 `wsl2` 后 `powershell` 中输入命令
```bash
code
```

3. 连接成功  
![[Pasted image 20260410104240.png]]

# 编译调试 C++ 代码（cmake 项目）

1. 安装依赖 
```zsh
sudo apt install -y build-essential gdb cmake
```

2.  `vscode` 中安装 `C++` 相关插件  
![[Pasted image 20260613131213.png|322]]
![[Pasted image 20260613131136.png|325]]

3. `Ctrl+Shift+p` 选择 `CMake: Quick Start` 进行一系列配置后生成 `CMakeLists.txt` 、`CMakePresets.json`

4. 在左侧 cmake 工具栏中点击相应按钮即可编译、调试
![[Pasted image 20260613172515.png|280]]

5. 项目结构与各级 `CMakeLists.txt` 编写，参考 [[cmake 项目架构#复杂项目|cmake 架构]]

> **普通 C++ 项目编译调试**
> - vscode：[[vscode#vscode 编译调试 C++ 项目|vscode 编译调试 C++ 项目]]
> - 命令行：[[编译流程#使用 gcc 进行编译|使用 gcc 进行编译]] [[vscode#gdb 调试 C++ 项目|gdb 调试 C++ 项目]]
> 
> **cmake 项目**
> - 命令行：[[cmake]]

## 番外篇：使用 Trae 调试 C++

1. 命令行中安装 `lldb`
```bash
sudo apt update 
sudo apt install lldb
```

2. `Trae` 中安装插件
![[Pasted image 20260506172653.png|345]]

3. 编写 `launch.json`（注意删除 gdb 相关的配置）
```js
{
    "version": "0.2.0",
    "configurations": [
        // 调试 Vue
        {
            "name": "调试Vue前端",
            "type": "chrome",
            "request": "launch",
            "url": "http://localhost:5173",
            "webRoot": "${workspaceFolder}",
            "sourceMaps": true,
            "sourceMapPathOverrides": {
                "webpack:///src/*": "${webRoot}/src/*"
            }
        },

        // 调试 Java (wsl)
        {
            "name": "调试Java后端",
            "type": "java",
            "request": "launch",  
            "jarPath": "${workspaceFolder}/target/inteCAEServer-1.0.0.jar",
            "vmArgs": "-Djava.library.path=/mnt/c/Users/HP/code/test/IntevueServer/build/bin"
        },
        
        // 调试 C++ (wsl)
        {
            "name": "Attach to Java Process",
            "type": "cppdbg",
            "request": "attach",
            "program": "/usr/bin/java",
            "processId": "${command:pickProcess}",
            "additionalSOLibSearchPath": "${workspaceFolder}/IntevueServer/build/bin",
            "sourceFileMap": {
                "/build/": "${workspaceFolder}/IntevueServer/build"
            }
        }
    ]
}
```

# 编译调试 Java 代码

1. `vscode` 中安装 `Java` 相关插件  
![[Pasted image 20260413111521.png]]  

2. 编写 `launch.json`
```js
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "调试后端(Java Jar)", // 配置名称
            "type": "java",              // 配置类型 
            "request": "launch",         // 请求配置类型（launch启动或attach附加进程）
            "jarPath": "${workspaceFolder}/target/inteCAEServer-1.0.0.jar", // 要运行的 jar 包
            "vmArgs": "-Djava.library.path=/mnt/c/Users/HP/code/test/IntevueServer/build/bin" // JVM 参数
        }
    ]
}
```

3. 在右上角选择 `Run Java` 进行编译，选择 `Debug Java` 进行 `debug`  
![[Pasted image 20260413112108.png]]  

## Java 调用 C++ 动态库
### 方法1：通过 JNI 调用 C++ 代码

1. 创建带有 `native` 方法的 `Java` 类
```java
// HelloWorldJNI.java 
package com.baeldung.jni;

public class HelloWorldJNI {

    static {
        System.loadLibrary("native");
    }
    
    public static void main(String[] args) {
        new HelloWorldJNI().sayHello();
    }

    private native void sayHello();
}
```

2. 基于 `Java` 类 生成 `C++` 头文件 
```bash
javac -h . -d . HelloWorldJNI.java
# -h 选项：生成 com_baeldung_jni_HelloWorldJNI.h 
# -d 选项：生成 .class 文件到包目录结构中
```

3. 编写 `.h` 文件对应的 `C++` 实现文件
```C++
// com_baeldung_jni_HelloWorldJNI.cpp 
JNIEXPORT void JNICALL Java_com_baeldung_jni_HelloWorldJNI_sayHello
  (JNIEnv* env, jobject thisObject) {
    std::cout << "Hello from C++ !!" << std::endl;
}
```

4. 编译、链接生成 共享库
```bash
# 将 C++ 源码文件编译成 中间目标文件（.o）
g++ -c -fPIC -I${JAVA_HOME}/include -I${JAVA_HOME}/include/linux com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o 

# 将 .o 文件 打包成 动态库（.so）
g++ -shared -fPIC -o libnative.so com_baeldung_jni_HelloWorldJNI.o -lc
```

**补充说明**：代码中的 `JNI` 元素

`Java` 侧元素：
- **`native` 关键字**：任何标记为 `native` 的方法，都必须在**本地共享库**中实现。
- **`System.loadLibrary(String libname)`**：一个静态方法，用于将文件系统中的共享库加载到内存中，并使其中导出的函数可供 Java 代码调用。

`C/C++` 侧元素
- **`JNIEXPORT`**：将共享库中的函数标记为**可导出**，使其被纳入函数表，使 `JNI` 能够找到该函数。
- **`JNICALL`**：与 `JNIEXPORT` 配合使用，确保我们的方法对 `JNI` 框架可用。
- **`JNIEnv`**：一个结构体，包含了一系列方法，可以在本地代码中通过它来访问 `Java` 元素（如对象、类、方法等）。
- **`JavaVM`**：一个结构体，用于操作正在运行的 `JVM`。

---

**参考**：
- [JNI官方文档](https://www.baeldung.com/jni)

### 方法2：通过 SWIG 调用 C++ 代码
#### 前置条件

0. 项目结构
```
.
├── CMakeLists.txt
├── example.cpp
├── example.i
└── main.java
```

1. `Java` 测试类：用于调用 `C++` 代码
```java
// main.java
public class main {
  public static void main(String argv[]) {
    System.loadLibrary("example");
    
    System.out.println(example.getMy_variable());
    System.out.println(example.fact(5));
    System.out.println(example.get_time());
  }
}
```

2. `C++` 类：类中方法被 `Java` 调用
```C++
// example.cpp 

#include <time.h>
double My_variable = 3.0;

int fact(int n) {
    if (n <= 1) return 1;
    else return n*fact(n-1);
}

int my_mod(int x, int y) {
    return (x%y);
}
	
char *get_time()
{
    time_t ltime;
    time(&ltime);
    return ctime(&ltime);
}
```

3. `SWIG` 接口文件：用于生成胶水代码（即 `wrap` 文件）
```C++
// example.i
%module example
%{
/* Put header files here or function declarations like below */
extern double My_variable;
extern int fact(int n);
extern int my_mod(int x, int y);
extern char *get_time();
%}

extern double My_variable;
extern int fact(int n);
extern int my_mod(int x, int y);
extern char *get_time();
```

#### 使用 命令行 编译
##### 方法1：gcc

```bash
# 1. 生成 JNI 包装代码
swig -java -c++ example.i

# 2. 编译成 .o
g++ -c -fPIC example.cpp example_wrap.cxx \
  -I$JAVA_HOME/include \
  -I$JAVA_HOME/include/linux

# 3. 链接成 .so（把 example.o 打包进去）
g++ -shared -o libexample.so example.o example_wrap.o

# 4. 编译 Java
javac *.java

# 5. 指定动态库路径，运行
java -Djava.library.path=. main
```

---

**参考**：
- [SWIG Basics](https://www.swig.org/Doc4.4/SWIGDocumentation.html#SWIG) 

##### 方法2：cmake

1. 编写 `CMakeLists.txt` 
```cmake
# 1. 指定 cmake 版本
cmake_minimum_required(VERSION 3.22)

# 2. 指定项目名、使用的语言
project(swig_example LANGUAGES CXX)

# 3. 找包
find_package(SWIG REQUIRED COMPONENTS java)
find_package(Java REQUIRED)
find_package(JNI REQUIRED)

# 4. SWIG 相关配置，具体见参考
include(UseSWIG)

include_directories(${JNI_INCLUDE_DIRS})

set_property(SOURCE example.i PROPERTY CPLUSPLUS ON)
set_property(SOURCE example.i PROPERTY SWIG_FLAGS "-java")

swig_add_library(example
  TYPE SHARED
  LANGUAGE java
  SOURCES example.i example.cpp)

# 5. 链接库
target_link_libraries(example ${JNI_LIBRARIES})

# 6. 构建后处理：将 build 中的 .java 文件移动到源目录
add_custom_command(
  TARGET example
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
  ${CMAKE_CURRENT_BINARY_DIR}/*.java
  ${CMAKE_CURRENT_SOURCE_DIR}
  COMMENT "已复制 Java 文件到源目录"
)

# 7. 编译 Java 代码
add_custom_target(
  compileJava ALL
  DEPENDS example
  COMMAND ${Java_JAVAC_EXECUTABLE} -d ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/*.java
  COMMENT "正在编译 Java 代码..."
)

# 8. 运行 Java 程序
add_custom_target(
  run ALL
  DEPENDS compileJava
  COMMAND ${Java_JAVA_EXECUTABLE}
  -Djava.library.path=${CMAKE_CURRENT_BINARY_DIR}
  main
  COMMENT "正在运行程序..."
)
```

2. 命令行输入以下命令
```bash
# 1. 配置与生成
cmake -B build -G Ninja

# 2. 构建
cmake --build build
```

---

**参考**：
- [cmake命令行](https://cmake.org/cmake/help/v3.22/manual/cmake.1.html#run-a-command-line-tool)
- [FindSWIG](https://cmake.org/cmake/help/latest/module/FindSWIG.html)
- [UseSWIG](https://cmake.org/cmake/help/latest/module/UseSWIG.html#module:UseSWIG)
- [FindJava](https://cmake.org/cmake/help/latest/module/FindJava.html)
- [FindJNI](https://cmake.org/cmake/help/latest/module/FindJNI.html) 
- [add_custom_command](https://cmake.org/cmake/help/latest/command/add_custom_command.html#command:add_custom_command)
- [add_custom_target](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target)

#### 使用 vscode 编译
##### 方法1：gcc

不推荐，略  

##### 方法2：cmake

基于 [[#4.2.1 环境配置和示例代码]] 中的 `CMakeLists.txt`，在 `vscode` 中点击左侧 `CMake` 后，选择 生成 即可  
![[Pasted image 20260414115351.png]]

# 编译调试 Vue 代码

1. `chrome` 下载 `Vue DevTools` 插件

2. 下载 `vscode` 插件
![[Pasted image 20260423134301.png]]

3. 在 `vite.config.js` 开启 `SourceMap`
```js
export default defineConfig( ( { mode } ) => {
  return {
    // ...
    server: {
      sourcemap: true // 开启 sourcemap
    }
  };
} );
```

4. 编写 `launch.json`
```js
{
  // 调试配置列表（当前生效的配置）
  "configurations": [
    {
      "name": "Launch Chrome",              // 调试配置显示名称（F5时可见）
      "request": "launch",                  // 启动模式：新建Chrome窗口
      "type": "chrome",                     // 调试类型：调试Chrome浏览器
      "url": "http://10.8.254.124:5173",     // 调试时自动打开的项目地址
      "webRoot": "${workspaceFolder}",      // 项目源码根目录（VSCode打开的文件夹）
      "sourceMaps": true,
      
      // 源码路径映射：让VS Code正确找到本地src文件
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/src/*"
      }
    }
  ]
}
```

5. 通过 `npm run dev` 启动后，即可使用 `vscode` 进行 `debug`

> **SourceMap**：
> 打包压缩后的代码 <---映射---> 原始源码，用于调试

# 同时调试 Vue、Java、C++ 代码

**项目流程**：
用户上传 `.fem` 模型文件到 `Vue` 前端 ，`Vue` 前端传给 `Java` 后端，`Java` 后端调用 `C++` 数据处理模块生成 `.vtp` 文件返回给 `Java` 后端，`Java` 后端返回给 `Vue` 前端并显示

**项目结构**：
```bash
.
├── IntevueServer
│   └── IntevueCAE
│       ├── build
│       ├── backend          # Java 后端
│       ├── foundation       # C++ 数据处理模块（生成动态库后被 Java 调用）
│       └── vtk-install
└── fronted                  # Vue 前端
    └── src
        ├── Elements
        ├── InterFace
        ├── Operate
        ├── Reader
        ├── appcore
        └── assets
```

1. 在根目录编写 `launch.json`（**注意**：规范用法是在前后端分别写 `launch.json` 进行配置）
```js
{
    "version": "0.2.0",
    "configurations": [
        // 调试 Vue
        {
            "name": "调试Vue前端",
            "type": "chrome",
            "request": "launch",
            "url": "http://localhost:5173",
            "webRoot": "${workspaceFolder}",
            "sourceMaps": true,
            "sourceMapPathOverrides": {
                "webpack:///src/*": "${webRoot}/src/*"
            }
        },

        // 调试 Java (wsl)
        {
            "name": "调试Java后端",
            "type": "java",
            "request": "launch",  
            "jarPath": "${workspaceFolder}/target/inteCAEServer-1.0.0.jar",
            "vmArgs": "-Djava.library.path=/mnt/c/Users/HP/code/test/IntevueServer/build/bin"
        },
        
        // 调试 C++ (wsl)
        {
            "name": "Attach to Java Process",
            "type": "cppdbg",
            "request": "attach",
            "program": "/usr/bin/java",
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",
            "additionalSOLibSearchPath": "${workspaceFolder}/IntevueServer/build/bin",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "sourceFileMap": {
                "/build/": "${workspaceFolder}/IntevueServer/build"
            }
        }
    ]
}
```

2. `CMakeLists.txt` 文件中加入调试信息
```cmake
set(CMAKE_BUILD_TYPE Debug)
```

3. 在终端中输出以下命令
```bash
# attach 到 C++ 代码，需要权限，使用以下方式临时开启
sudo sysctl -w kernel.yama.ptrace_scope=0
# 作用：临时关闭 Linux 的 ptrace 安全限制，允许普通进程随意附加、调试、读取其他进程内存
```

4. 在 `Java` 和 `C++` 相关位置打断点，先启动 `Vue` 前端
   再启动 `Java` 调试器
   ![[Pasted image 20260424103648.png|144]]
   最后启动 `C++` 调试器 附加到 `Java` 进程
   ![[Pasted image 20260424103849.png|143]]
   ![[Pasted image 20260424103933.png|454]]

5. 最终效果：实现一个 vscode 窗口同时调试 Java、C++ 代码
   ![[Pasted image 20260424104217.png|329]] 


