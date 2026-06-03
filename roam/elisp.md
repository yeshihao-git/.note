---
tags:
  - emacs
---
# elisp

**what**：
lisp 的方言，是 emacs 内置的编程语言，用于编写 emacs扩展 和 自定义功能

**how**：帮助

|函数|作用|
|---|---|
|🔥 shortdoc|分类常用 elisp 函数，附带使用示例|
|elisp-index-search|emacs lisp manual 中搜索关键字|
|emacs-index-search|emacs manual 中搜索关键字|

## 概念
### cons cell
**what**：点对
(car . cdr) 形式的数据结构，用于构造列表、树、其他数据结构的基础原子单元

**what**：点表示法
当cdr不是列表时，lisp会使用点表示法，否则简化为列表形式
```elisp
; cons函数：创建一个新的cons cell，将两个值组合成(car . cdr)结构的单元
(setq bouquet '(rose violet buttercup))
(cons 'hello 'bouquet) -> (hello . bouquet)             ; cdr不是列表，因此使用点表示法
(cons 'hello bouquet)  -> (hello rose violet buttercup) ; 实际结构：(hello . (rose . (violet . (buttercup . nil))))，cdr是列表，打印时会简化为列表形式
```

### 符号

**what**：
用于表示变量、函数名等（可以理解为C语言中的指针）
```
符号求值规则
1. 变量        ：返回其绑定的值
2. 函数名      ：调用函数
3. 被引用的符号：返回符号本身，不会求值
```

符号底层（可以想象成抽屉）
```elisp
抽屉箱子：[符号名称][符号定义(函数定义)][符号值(变量)][属性列表]
              |           |                   |             |
              V           V                   V             V
抽屉内容： bouquet     [none]     (rose violet buttercup) [not described here]

; cdr这个符号，不过是将符号值变成了地址
(cdr bouquet)   bouquet -> [rose][ptr] -> [violet][ptr] -> [buttercup][ptr] -> nil
                                   ^
                                   |
                                  cdr
```

### S-表达式

**what**：
（sexp）lisp语言家族中用于表示 代码和数据 的统一括号化语法
```elisp
;; S-表达式的任意部分都能求值
(+ 2 (+ 3 3)) ; 光标移动到希望求值的地方，C-x C-e
```

### 原子

**what**：
不可拆分的最小数据单元（eg：数字、字符串、符号等）
```elisp
;; 原子求值规则
1. 数字    ：本身
2. 字符串  ：本身
3. 特殊符号：t和nil
```

```elisp
8                 ; 数字求值 返回本身
"hello"           ; 字符串求值 返回本身
fill-column       ; 符号求值 返回它的值
'(hello world 12) ; 列表（引号保护） 返回列表
```

### 引用

**what**：单引号' 
是(quote 表达式)的简写形式，返回表达式本身，不计算值
1. ' 引用数据
```elisp
'(hello world)
```

**what**：#'
是function宏的简写形式，用于明确引用函数本身
1. #' 引用函数

### 真假
**what**：t 
逻辑真

**what**：nil
逻辑假（等价于空列表）

**what**：non-nil
任何非nil的值（eg：t、数字、字符串）

```elisp
nil ; nil
()  ; nil
```

### 列表

**what**：
用 () 包裹，内部用空格分隔的多个元素（原子或其他列表）组成的复合数据结构；对列表引用： '()
```
列表求值规则
1. 函数调用：列表中第1个元素后的元素视为参数并求值，将参数求值结果作为真正的参数
2. 宏调用  ：列表中的其他元素不立刻求值，而是根据宏定义展开，再求值
3. 特殊表  ：第1个元素若是特殊表，按特殊规则处理参数
```

```elisp
; 有 '() 引用的列表，返回列表本身
'(2 2)

; 无 '() 引用的列表
(+ 2 2)           ; 普通函数，返回函数求值结果
(defun func ()    ; 特殊表，特殊规则
  ; 函数体
)
```

```
;; 列表底层（由 cons cell 链接起来的链表）
(rose violet buttercup)                             [rose][ptr] -> [violet][ptr] -> [buttercup][ptr] -> nil
(setq bouquet '(rose violet buttercup))  bouquet -> [rose][ptr] -> [violet][ptr] -> [buttercup][ptr] -> nil
```

### 变量

**what**：
有值的符号
```elisp
let、let* ; 创建 局部变量
defvar    ; 创建 全局变量
```

```
变量名习惯
- hook：某个事件发生时（如：打开文件、切换模式） *自动调用的函数列表* （类似回调函数）
- function：值为一个函数
- functions：值为一个函数列表
- flag：值为 nil 或 non-nil
- predicate：值是一个作判断的函数，返回 nil 或 non-nil
- program 或 -command：一个程序或 shell 命令名
- form：一个表达式
- forms：一个表达式列表
- map：一个按键映射（keymap）
```

### 函数、参数

**what**：函数
用 defun特殊表 定义

**what**：谓词
返回值为真假的函数（eg：xxx-p 后头有个p）

**what**：特殊表
独特的求值规则，用于控制结构、变量绑定、宏等

**what**：宏
用 defmacro 定义

**what**：附带效果
函数返回时，做了其他事（eg：移动光标、拷贝文件等），这些事就是附带效果

```elisp
;; 函数定义
(defun 函数名 (参数列表)
  ; 可选：函数文档字符串
  函数体...)
```

```elisp
;; 函数求值规则（函数调用规则）
(+ 2 2)
(concat "hello" "world")
(substring "The quick brown fox jumped." 16 19)
(+ 2 fill-column)
(concat "The" (number-to-string (+ 2 fill-column)) "red foxex.")
```

**what**：参数
列表中第一个符号是函数名，后续的符号就是参数

**what**：可变参数
&rest ，表示可以传入任意多个参数

**what**：可选参数
&optional ，告诉lisp解释器某个参数是可选的；函数定义中，若参数在&optional之后，代表参数是可选的

**what**：前缀参数
`C-u [<数字>]` ，可以传入interactive的p或P参数描述符

```elisp
(+)         ; 0
(*)         ; 1
(+ 3)       ; 3
(* 3)       ; 3
(+ 3 4 5)   ; 12
(* 3 4 5)   ; 60
```

### 位点、标记、域

**what**：位点
一个整数，表示 光标所在位置
```elisp
(point)3409       ; 返回光标所在位置：缓冲区首字符到光标所在位置之间的字符数
(point-min)1      ; 返回当前缓冲区位点的最小可能值；除非设置变窄
(point-max)3530   ; 返回当前缓冲区位点的最大可能值
```

**what**：标记
一个整数，表示 缓冲区中的位置
```bash
C-SPC           # 设置标记
C-x C-x         # 光标跳转到标记处
C-u C-SPC (x N) # 基于标记环的光标跳转
```

**what**：域（region）
位点、标记之间的缓冲区

### 缓冲区、变窄

**what**：缓冲区
从文件中拷贝来的信息，缓冲区的变动不会改变文件，除非保存。缓冲区不一定都和文件相联系，比如：scracth help*等

**what**：文件
永久保存在计算机中的信息
```elisp
(buffer-file-name) ; 文件名（绝对路径）
(buffer-name)      ; 缓冲区名
```

**what**：变窄
让 emacs 关注缓冲区的特定部分，默认不开启，开启后， widen 函数使其余部分重新可见

### kill-ring

**what**：kill-ring
一个变量，存的是字符串列表。使用 C-y (M-y)xN 可以将 kill-ring 中的第N个元素插入当前缓冲区，到达最后一个元素就循环到第一个元素，故称kill环

**what**：kill-ring-yank-pointer
一个变量，指向 kill-ring 任意位置

**what**：rotate-yank-pointer
1. 改变 kill-ring-yank-pointer 指向 kill-ring 中的元素，若超过 kill-ring 末尾，则自动指向 kill-ring 第一个元素
2. 是 yank（C-y） 、 yank-pop（M-y）的底层

## debug

1. edebug：新的内置 debug 器，源码级调试器
2. debug ：旧的内置 debug 器

```elisp
;; 错误示例
(defun triangle-bugged (number)
  "Return sum of numbers 1 through NUMBER inclusive."
  (let ((total 0))
    (while (> number 0)
      (setq total (+ total number))
      (setq number (1= number)))      ; Error here.
    total))
(triangle-bugged 4)
```

```elisp
;; 报错信息分析
(+ 2 'hello)

;; 第一行报错信息：(wrong-type-argument number-or-marker-p hello)
; wrong-type-argument：错误的参数类型，参数需要（数字或者标记）
; number-or-marker-p：错误的参数类型，参数需要（数字或者标记）
;; 后续报错信息：从下至上，为lisp解释器求值过程
```

#### edebug

1. 函数定义处 M-x edebug-defun
2. 对使用该函数的表达式进行 C-x C-e
3. 在源码位置有箭头提示，按 SPC 进入下一个表达式，每个表达式的计算结果显示在回显区

#### debug

1. 方式1： 变量debug-on-error -> t，遇到错误自动进入调试器
2. 方式2： M-x debug-on-entry 输入要调试的函数名，在函数调用处 C-x C-e ，在Backtrace缓冲区中每次按 d ，依次对表达式求值
3. 方式3： 变量debug-on-quit -> t，输入 C-g ，就启动debug，适用于调试无限循环
4. 方式4：在需要调试代码的地方写入 (debug)