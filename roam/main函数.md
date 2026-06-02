---
tags:
  - cpp
---
# main函数

**what**：
程序运行的 入口 ，由操作系统调用，返回值传递状态码（程序执行状态）给操作系统
1. main函数 的可选参数（ argc / argv ）是接收 命令行参数 的唯一入口
2. （cpp11）隐式 return 0 （return后的数字就是 退出状态码 ）
3. 可以 “正常退出”，也能 “强制退出”（终止函数）
4. main函数 由操作系统直接调用，但 不一定是第一个被调用的 ，全局变量、static变量 的初始化发生在 main函数 前，若通过自定义函数初始化，则自定义函数为第一个调用的函数

正常退出流程
1. 销毁 main函数 和 调用栈 中的局部变量（🔥 与 std::exit() 的核心区别）
2. 逆序销毁 静态存储期 的变量（static变量、全局变量）
3. 清理操作（关闭标准流、刷新缓冲区确保内容写入文件）
4. 返回状态码给操作系统

## 终止函数

**what**：
主动或被动控制程序的终止流程


### std::exit()

**what**：
触发 正常终止程序 ，调用后程序进行一系列清理操作，返回状态码给操作系统（转交控制权）

执行流程
1. 逆序销毁 静态存储期 的变量（🔥不会 栈展开 ：不会清理当前函数或调用栈中的局部变量）
2. 调用 std::atexit() 注册的函数
3. 清理操作（关闭标准流、刷新缓冲区确保内容写入文件）
4. 返回状态码给操作系统
```cpp
void cleanup() {
    std::cout << "cleanup!\n";
}

int main() {
    std::atexit(cleanup);   // 注册 cleanup()，调用 std::exit() 时自动调用 cleanup()
    std::cout << 1 << '\n';
    std::exit(0);           // 终止程序并且返回 状态码0 给 操作系统
    std::cout << 2 << '\n'; // 不会执行
    return 0;
}
```

### std::abort()

**what**：
触发 强制终止程序 ，调用后直接终止程序且 不执行任何清理操作（不销毁静态、全局、 局部对象、不刷新流缓冲），返回异常终止的状态码给操作系统

### std::terminate()

**what**：
异常处理失败时自动调用，默认会调用 std::abort() 强制终止程序

## 命令行参数

**what**：
用于在程序外部给 main函数 传参（通过 main函数 的 argc/argv 传递）
1. argc 代表参数个数，从 0 开始，依次是： 程序路径 参数1 参数2...
2. argv 代表参数，传入的都是 C风格字符串
```cpp
// 命令行中程序路径：/tmp/babel-WDQeyq/C-bin-tKs22K
// 命令行中参数1：myfile
// 命令行中参数2：100

// int main(int argc, char** argv)  NOTE 等价形式
int main(int argc, char* argv[]) {
    std::cout << "There are " << argc << " arguments:\n";

    for (int count{ 0 }; count < argc; ++count) {        // argv是 已退化的C风格数组，不能用 基于范围的for循环
        std::cout << count << ' ' << argv[count] << '\n';
    }
}
```

**why**：
程序外给程序传数据的 两种方式 ：
1. 程序外输入（命令行参数）
2. 程序内输入（std::cin）
使用 std::cin 就得每次在程序运行的时候输入，假设一个程序每次处理一张图片，我们需要反反复复启动程序，然后输入...
因此考虑用 命令行参数
