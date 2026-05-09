# 1. 环境搭建
## 1.1 配置 windows 功能
1. 配置 `windows` 功能
![[Pasted image 20260410103334.png|697]]

2. 重启生效

## 1.2 安装 wsl2 和 ubuntu22.04
1. 命令行中输入以下命令
```powershell
# 1. 更新 WSL2（自动下载并安装最新的 WSL2）
wsl --update

# 2. 验证安装
wsl --version 

# 3. 安装 ubuntu
wsl --install -d Ubuntu-22.04
```

## 1.3 vscode 远程连接 wsl
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

---

**参考**：  
[wsl2 非系统盘下安装ubuntu22.04的极简方法](https://www.cnblogs.com/go17/p/19008703)  
[Windows 10 离线安装 WSL2 Ubuntu 22.04.5 LTS 简明教程 - 踩坑及解决方案](https://zhuanlan.zhihu.com/p/23836826798)

# 2. 编译调试 C++ 代码
## 2.1 前置条件
0. 项目结构
```
.
├── CMakeLists.txt
└── main.cpp
```

1. 安装相关依赖
```bash
sudo apt install -y build-essential cmake qtbase5-dev qtchooser qt5-qmake qttools5-dev
```

2. 示例代码
```C++
// main.cpp 文件

#include <QApplication>
#include <QPushButton>
#include <iostream>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    QPushButton button("Hello Qt WSL!");
    button.resize(200, 100);
    button.show();
    
    int a = 1, b = 2;
    
    std::cout << "hello world" << std::endl;
    std::cout << a + b << std::endl;

    return app.exec();
}
```

3. 在项目根目录编写 `CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.22) # 确定 cmake 版本

# 新建一个 cmake 项目
# LANGUAGES 指定启用的语言，默认启用 C 和 CXX（C++）
# 这里指定启用 CXX，不启用 C，减少项目生成时间
project(hello_qt LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17) # 设置 C++ 标准为 C++17
set(CMAKE_CXX_STANDARD_REQUIRED ON) # 强制使用 C++17 标准

set(CMAKE_AUTOMOC ON) # 启用自动 MOC 功能
set(CMAKE_AUTORCC ON) # 启用自动 RCC 功能
set(CMAKE_AUTOUIC ON) # 启用自动 UIC 功能

find_package(Qt5 COMPONENTS Core Widgets REQUIRED) # 查找 Qt5 的 Core 和 Widgets 模块

add_executable(hello_qt) # 添加可执行文件

target_sources(hello_qt PRIVATE main.cpp) # 将 main.cpp 添加到 hello_qt 可执行文件的源文件列表中

target_link_libraries(hello_qt Qt5::Core Qt5::Widgets) # 将 Qt5 的 Core 和 Widgets 模块链接到 hello_qt 可执行文件中
```

4. `vscode` 中安装 `C++` 相关插件  
![[Pasted image 20260413111317.png]]  
![[Pasted image 20260413111338.png|307]]

## 2.2 编译 C++ 代码

### 2.2.1 使用 命令行 编译
#### 方式1：gcc
1. 使用 `gcc` 进行编译
``` bash
g++ -fPIC main.cpp -o qt-test \ 
-I/usr/include/x86_64-linux-gnu/qt5 \ 
-I/usr/include/x86_64-linux-gnu/qt5/QtCore \ 
-I/usr/include/x86_64-linux-gnu/qt5/QtGui \ 
-I/usr/include/x86_64-linux-gnu/qt5/QtWidgets \ 
-lQt5Core -lQt5Gui -lQt5Widgets
# 可以发现这样执行头文件路径、链接库文件的方式较为繁琐，可考虑使用 pkg-config 简化
```

2. 运行程序
```C++
./qt-test
```

3. 最终效果  
![[Pasted image 20260410135812.png]]

---

**补充说明**：
```bash
g++ -std=<C++标准> -g <源文件路径> -o <可执行文件路径> \
    -I<头文件路径> \
    -L<库文件路径> \
    -l<库名>
```
- `-std`：指定 C++ 标准
- `-g`：生成调试信息
- `-o`：指定输出的可执行文件名
- `-I`：指定头文件路径
- `-L`：指定库文件路径
- `-l`：指定库名（eg：若库文件名是 `libvtk.so`，则写成 `-lvtk`，省去前面的 `lib` 和后缀）
- `-fPIC`：位置无关编码。在制作动态库时使用，其次在编译 `PIE` 可执行文件时必须使用 `PIC` 代码
- `-O<级别>`：优化级别

> **PIE（Position-Independent Executable）**：位置无关可执行文件，一种安全机制，让程序每次加载到内存的位置都不一样。若要生成 `PIE` 程序，则所有代码必须是位置无关（`PIC`）。现代64位 `linux` 全部默认开启 `PIE`。

#### 方法2：cmake
1. 使用 `cmake` 命令行工具进行构建、运行
```bash
cd <项目目录>

# 创建目录 build 并在其中生成 Ninja 构建系统文件
cmake -B build -G Ninja 

# 使用构建工具 Ninja 编译 build 文件夹中的项目
cmake --build build 

# 运行项目
./build/<可执行文件> 
```

### 2.2.2 使用 vscode 编译
#### 方式1：gcc
1. 配置 `IntelliSence`：`Ctrl +Shift + p` 打开命令面板  
![[Pasted image 20260410142041.png|449]]  
在包含路径中输入头文件路径  
![[Pasted image 20260410142427.png]]

2. 配置 `tasks.json`：`vscode` 右上角选择 “运行 `C/C++` 文件”  
![[Pasted image 20260410143125.png]]  
![[Pasted image 20260410143130.png]]  
此时出现错误，因为我们还没进行头文件路径执行、链接库等操作  
![[Pasted image 20260410143135.png]]  
打开 `tasks.json`  
![[Pasted image 20260410143411.png]]  
指定头文件路径，链接库  
![[Pasted image 20260410143543.png]]  
重新点击`vscode` 右上角 “运行 `C/C++` 文件”  

> **tasks.json**：编译任务文件，`vscode` 编译代码用 配置文件
> 
> **c_cpp_properties.json**：`C++` 编辑配置文件（配置 `IntelliSence`时的相关信息就在这里）

#### 方式2：cmake
![[Pasted image 20260413102707.png]]  
点击左侧边栏 `CMake` 图标后，即可进行配置、生成、测试、调试、启动等操作。

也可以通过 `Ctrl + Shift + p` 打开命令面板后选择 `CMake` 相关命令：`cmake build`、`cmake run`、`cmake debug`、`cmake clean` 等。

## 2.3 调试 C++ 代码
### 2.3.1 使用 命令行 调试
1. 配置调试信息、优化级别： `gcc` 中增加 `-g` 、`-O0` 选项

2. 使用 `gdb` 进行调试
```bash
gdb <可执行文件/core dump文件> # 启动 gdb 并加载程序 / core dump
gdb -p <PID> # 启动 gdb 并附加到正在运行的进程
```

3. `gdb` 常用操作
```bash
l # 查看源码
bt # 查看调用栈
f # 切换栈帧
b # 打断点
r # 运行/重新运行
n # 单步执行（不进入函数）
s # 单步执行（进入函数）
f # 从函数返回
c # 继续
q # 退出 gdb
p # 打印/修改变量
```

---

**具体调试示例**：调试 `core dump`
```bash
# 1. 启动 gdb 并打开 core dump文件
gdb <core dump文件>
# 2. 查看调用栈信息
bt 
# 3. 查看崩溃点
# 4. 打印变量值
p <变量>
# 5. 修改后重新测试
r
# 6. 没问题后退出 gdb
q
```

> **core dump**：`core dump` 文件为程序崩溃时的内存快照，用于分析程序崩溃原因

### 2.3.2 使用 vscode 调试
`vscode` 新版 `C/C++` 插件无需手写 `launch.json` 就能一键调试了
![[Pasted image 20260410155840.png]]
1. 打断点

2. 点击右上角 “调试 C/C++ 文件”

3. 上方出现调试面板：继续、逐过程、单步调试、单步跳出、重启、停止

4. 左侧用于查看：局部变量、监视变量变化、调用栈信息、断点信息等

5. 下方为终端输出，用于查看：日志信息

> **launch.json**：调试任务文件，`vscode` 调试代码用 配置文件

---

**参考**：[vscode官方文档](https://code.visualstudio.com/docs/cpp/launch-json-reference)

# 3. 编译调试 Java 代码
## 3.1 使用 vscode 编译调试
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

---

**参考**：[vscode官方文档](https://code.visualstudio.com/docs/java/java-debugging)

# 4. Java 调用 C++ 动态库
## 4.1 方法1：通过 JNI 调用 C++ 代码
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

---

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

**参考**：[JNI官方文档](https://www.baeldung.com/jni)

## 4.2 方法2：通过 SWIG 调用 C++ 代码
### 4.2.1 前置条件
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

### 4.2.2 使用 命令行 编译
#### 方法1：gcc
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

**参考**：[SWIG Basics](https://www.swig.org/Doc4.4/SWIGDocumentation.html#SWIG) 

#### 方法2：cmake
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

**参考**：[cmake命令行](https://cmake.org/cmake/help/v3.22/manual/cmake.1.html#run-a-command-line-tool) [FindSWIG](https://cmake.org/cmake/help/latest/module/FindSWIG.html) [UseSWIG](https://cmake.org/cmake/help/latest/module/UseSWIG.html#module:UseSWIG) [FindJava](https://cmake.org/cmake/help/latest/module/FindJava.html) [FindJNI](https://cmake.org/cmake/help/latest/module/FindJNI.html)  [add_custom_command](https://cmake.org/cmake/help/latest/command/add_custom_command.html#command:add_custom_command) [add_custom_target](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target)

### 4.2.3 使用 vscode 编译
#### 方法1：gcc
不推荐，略  

#### 方法2：cmake
基于 [[#4.2.1 环境配置和示例代码]] 中的 `CMakeLists.txt`，在 `vscode` 中点击左侧 `CMake` 后，选择 生成 即可  
![[Pasted image 20260414115351.png]]

# 5. 调试 C++ 动态库（一个综合联调示例）
本节介绍的项目流程为：用户上传 `.fem` 模型文件到 `Vue` 前端 ，`Vue` 前端传给 `Java` 后端，`Java` 后端调用 `C++` 数据处理模块生成 `.vtp` 文件返回给 `Java` 后端，`Java` 后端返回给 `Vue` 前端并显示。项目结构如下：
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


# 6. 番外篇：使用 Trae 调试 C++
1. 命令行中安装 lldb
```bash
sudo apt update 
sudo apt install lldb
```

2. Trae 中安装插件
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