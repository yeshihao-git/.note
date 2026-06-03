---
tags:
  - emacs
  - org
---
# org-babel

**what**：
用于在 org文件 中 嵌入代码块，实时计算

**how**：org-babel代码块格式
```org
#+BEGIN_SRC <language> <switches> <header arguments> // <language>：指定代码语言
    <body>                                           // <header arguments>：控制代码块行为
#+END_SRC
```
在 `<header arguments>` 中设置代码运行结果 `:results <值>`

|<值> 分类|<值>|作用|
|---|---|---|
|结果输出|value（默认）|最后一个表达式|
||output|标准输出|
|结果格式|raw|原始结果（不进行额外处理）|
||org|org 格式|
||html|html 格式|
|结果存储|replace（默认）|替换 #+RESULTS: 中内容|
||append|追加 #+RESULTS: 内容后|
||silent|不显示结果|

## org-babel配置其他语言

1. M-x customize -> Org Babel Load Languages

## org-babel多代码块联动运行

1. headline 中使用 M-x org-babel-tangle （此时 org-babel-tangle 作用范围为子树而不是整个文件）
2. 在最后一个代码块（bash）： C-c C-c
```org
#+begin_src C++ :tangle ../example/two_Sum.h :includes '(<iostream> <vector>) :main no
std::vector<int> twoSum(std::vector<int>& nums, int target);
#+end_src

#+BEGIN_SRC C++ :tangle ../example/two_sum.cpp :main no
#include <iostream>
#include <vector>

std::vector<int> twoSum(std::vector<int>& nums, int target) {
    int n = nums.size();
    for (int i = 0; i < n; ++i) {
        for (int j = i + 1; j < n; ++j) {
            if (nums[i] + nums[j] == target) {
                return {i, j};
            }
        }
    }
    return {};
}
#+END_SRC

#+BEGIN_SRC C++ :tangle ../example/main.cpp :includes "two_Sum.h"
int main() {
    std::vector<int> nums = {2, 7, 11, 15};
    int target = 9;
    std::vector<int> result = twoSum(nums, target);

    if (!result.empty()) {
        std::cout << "Index: " << result[0] << ", " << result[1] << std::endl;
    } else {
        std::cout << "No solution found!" << std::endl;
    }

    return 0;
}
#+end_src

#+begin_src bash :results output
g++ -o ../example/main ../example/main.cpp ../example/two_sum.cpp && ../example/main
#+end_src
```

## org-babel运行cpp代码块

| `<header arguments>` | 说明               |
| -------------------- | ---------------- |
| :main                | 是否用 main 函数包裹代码块 |
| :includes            | 包含头文件            |
| :libs                | 链接库文件            |
| :namespaces          | 命名空间             |
| :flags               | 编译器参数            |
| :cmdline             | 设置命令行参数          |
