# C++程序内存布局
![[Pasted image 20260331153054.png|535]]
1. **栈**（stack） ：函数形参、局部变量...
2. **内存映射段**：共享内存（内核中的）、动态库（在磁盘中被加载到内存映射段里）、文件映射（将文件映射到进程空间 -> 实现高效的文件IO操作）
3. **堆**（heap）：动态分配的内存
4. **BSS段**（.bss）：（未初始化）全局变量、static变量，`main()`之前初始化为 0
5. **数据段**（.data）：（初始化）  全局变量、static变量
6. **代码段**（.text）：
   - 文本区（编译后的 *机器指令* ）
   - 只读存储区（常量）


## 代码验证
```cpp
// 全局变量（数据段）
int g_a = 10;
int g_b = 10;

// 全局常量（代码段-常量区）
const int c_g_a = 10;
const int c_g_b = 10;

// 全局静态变量（数据段）
static int s_g_a = 11;
static int s_g_b = 11;

int main() {
  // 局部变量（栈）
  int a = 10;
  int b = 10;

  // 静态局部变量（数据段）
  static int s_l_a = 10;
  static int s_l_b = 10;

  // 局部常量（栈）
  const int c_l_a = 10;
  const int c_l_b = 10;

  // 堆区内存（堆）
  int* p_l_a = new int(100);
  int* p_l_b = new int(100);

  // 地址打印
  cout << "局部变量a         ：" << reinterpret_cast<uintptr_t>(&a) << endl;
  cout << "局部变量b         ：" << reinterpret_cast<uintptr_t>(&b) << endl;

  cout << "const局部变量c_l_a：" << reinterpret_cast<uintptr_t>(&c_l_a) << endl;
  cout << "const局部变量c_l_b：" << reinterpret_cast<uintptr_t>(&c_l_b) << endl;
  cout << '\n';

  cout << "全局变量g_a        ：" << reinterpret_cast<uintptr_t>(&g_a) << endl;
  cout << "全局变量g_b        ：" << reinterpret_cast<uintptr_t>(&g_b) << endl;

  cout << "static全局变量s_g_a：" << reinterpret_cast<uintptr_t>(&s_g_a) << endl;
  cout << "static全局变量s_g_b：" << reinterpret_cast<uintptr_t>(&s_g_b) << endl;
  cout << "static局部变量s_l_a：" << reinterpret_cast<uintptr_t>(&s_l_a) << endl;
  cout << "static局部变量s_l_b：" << reinterpret_cast<uintptr_t>(&s_l_b) << endl;
  cout << '\n';

  cout << "const全局变量c_g_a ：" << reinterpret_cast<uintptr_t>(&c_g_a) << endl;
  cout << "const全局变量c_g_b ：" << reinterpret_cast<uintptr_t>(&c_g_b) << endl;
}
```