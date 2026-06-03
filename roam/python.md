---
tags:
  - python
---
# python

**what**：
一门编程语言
1. 默认包管理器：pip
2. 深度学习库：pytorch

## 概念
### 基础

**what**：变量
本质是名称

**what**：名称
对对象的引用，可以重新绑定到不同对象（不同于 C++）

### 类型

**what**：mutable（可变的）
值可以被改变的对象，且改变前后对象地址一致（通过id()取值）
```python
lst = [1,2,3]
print(id(lst))
lst.append([2])
print(id(lst))
```

**what**：immutable（不可变的）
值不能被改变的对象
```python
int、float、complex # 数字
str
tuple
```

**what**：iterable（可迭代的）
能逐个返回其内部元素的对象（实现了__iter__()或sequence语义的__getitem__()的对象），有哪些：
```
sequence类型
set类型
mapping类型等
```

**what**：hashable（可哈希的）
生命周期内有immutable哈希值、且能与其他对象比较的对象（实现了__hash__()和__eq__()的对象），有哪些：
```
mapping类型
```

**what**：sequence（序列类型）
顺序、可重复，是 iterable的，支持 索引、切片、拼接 等操作，相同sequence之间可以进行比较，有哪些：
```
基本序列类型：list、tuple、range对象
附加序列类型：str、bytes等
```

**what**：set（集合类型）
无序、不可重复，是iterable的
```
set等
```

**what**：mapping（映射类型）
将 hashable 的值映射到任意对象
```
dict
```

### 数据结构
**what**：list（列表）
mutable的sequence类型，因此支持索引、切片、拼接，可以存储不同类型的数据
```python
# 实现栈
append()、pop()

# 实现队列
collections.deque包装list # 原因：列表作为队列效率低：列表尾插入删除快，但列表头插入删除很慢
append()
popleft()
```

**what**：列表推导式
将一些操作应用到 iterable 对象中的每一个元素，生成新的 list
```python
[表达式 for子句... if子句...]     # 先执行子句，后执行表达式
# for子句、if子句 -> 可以有多个，输出结果传入表达式计算后得到list
# 表达式          -> 可以是 复杂表达式、嵌套函数、嵌套列表推导式
```

**what**：tuple（元组）
immutable的sequence类型，因此支持索引、切片、拼接，可以存储不同类型的数据

**what**：set（集合） 
mutable的set类型，因此支持索引、切片、拼接，可以存储不同类型的数据，支持集合推导式，外层为花括号{}

**what**：dict（字典）
key是hashable（意味着immutable、唯一）的，是存储key-value对的集合，支持字典推导式，外层为花括号{}

#### 打包和解包

iterable的对象就能解包，打包看情况
```
sequence类型        # 支持 打包、解包
```

### 控制语句
#### if语句
if、elif、else

#### for语句
for xxx in iterable对象（可以是以下）：
1. sequence、set、mapping
2. range()      -> 生成等差数列，可以与len()搭配使用
3. items()      -> 提取dict的key、value
4. enumerate()  -> 获取index和对应值
5. zip()        -> 将元素匹配
6. set()        -> 去重(集合构造函数)

break、continue、else子句：循环结束时进入else语句，若执行for的break则不进入else语句

##### for语句的底层实现

对iterable对象调用iter()方法，返回一个定义了`__next__`方法的iterator，`__next__`会逐一访问容器中的元素；也能通过next调用`__next__`方法

#### pass语句

空操作，常用于占位

#### match语句

match后跟目标值，case后跟模式 用于匹配目标值中的内容
case中的模式有哪些？
1. |(或)
2. `_`(通配符默认匹配，没有匹配的内容就会匹配它)
3. if(提供条件判断)
```python
match point:
    case Point(x, y) if x == y:
        print(f"Y=X at {x}")
    case Point(x, y):
        print(f"Not on the diagonal")
```
4. as(捕获子模式)
```python
case (Point(x1, y1), Point(x2, y2) as p2): ... # 将第二个元素捕获为p2
```

#### del语句

用于删除条目；可以按索引(sequence)、切片(sequence)、key(mapping)、对象删除

### 函数
**what**：函数名
只是一个符号，指向函数对象（因此我们可以创建其他名称指向同一函数对象）

函数中的局部符号表存储：函数参数的名称和值、函数中局部变量的名称和值、嵌套函数名 -> 存放在局部符号表中

查找函数中变量的顺序  ：函数局部符号表 => 外层函数局部符号表 => 全局符号表 => 内置名称符号表

**what**：位置参数
可以传递值（位置参数必须在关键字参数之前）；可以使用特殊参数/限制在/之前只能传递位置参数；任意数量参数`*arg`（可以和`**kwds`组合使用，但是必须在`**kwds`之前），表示接收一个tuple

**what**：关键字参数
可以传递键值对；可以设置默认值参数(此时也是可选参数)；可以使用特殊参数限制在之后只能传递关键字参数；任意参数`**kwds`，表示接收一个dict
```python
def foo(name): ... -> 可以传递'ysh'、name='ysh'
```

#### 函数参数列表如何解包？

`*`解包list、tuple；`**`解包dict，作为关键字参数传递

### 运算符

**what**：in / not in
成员检测运算符，用于确定值是否存在于某个容器中；sequence、set、mapping类型都支持此运算
```python
x in s -> 若x是s的成员，则返回True
```

### 脚本、模块、包

**what**：脚本
作为 运行入口 的.py文件，执行`if __name__ == "__main__":`里的代码(若作为模块导入到其他地方，则不执行此处代码)
1. `__name__的值为__main__`
2. 全局命名空间（脚本中定义了函数func()，则func()在全局命名空间，直接func()调用）

**what**：模块
作为 复用单元 的.py文件，模块中定义了可复用的变量、函数、类等
1. `__name__`的值为模块名(模块名就是函数名)
2. 模块的命名空间（模块mod中定义了函数func()，则func()在模块mod的命名空间，mod.func()调用）

**what**：import
将 模块的名称 导入到当前命名空间，搭配 as 使用时会将导入的 模块的名称 与as后的名称关联

**what**：from import
将 模块中某个内容的名称 (不会导入模块名称)导入到当前命名空间，若是from xxx import ，则导入模块中所有不以下划线(`_`)开头的名称，搭配 as 使用时会将导入的 模块中的某个内容的名称 和as后的名称关联
示例：
```
情景1:import mod到全局命名空间，调用mod中的func()函数
mod.func()
情景2:from mod import func到全局命名空间，调用mod中的func()函数
func()
```

**what**：`__pycache__`目录
里面缓存着编译好的python文件(文件名：模块.版本.pyc)，用于 加速加载模块 (没有加速执行速度)
1. 工作原理：python会 对比编译版和源码文件的修改日期 ，编译版过期，自动重新编译；命令行直接加载的模块，每次重新编译，且不会生成编译版

**what**：包
包含__init__.py文件的目录；作为模块的容器，用于组织多个模块或子包

**what**：子包
包中的存在__init__.py文件的子目录

**what**：`__init__.py`文件
让python将一个目录识别为包，可用于包的初始化代码，设置__all__变量等
1. 从包导入`*`时，实际上导入的是`__init__.py`文件中`__all__`变量中的模块名列表，若没有定义`__all__`变量，则不会导入包中的所有模块名称，而只是导入包名；注意（`__all__`中有模块名称`func`，`__init__.py`中定义了函数`func`，则从包导入`*`不会导入`func`模块，因为函数 `func` 的名称覆盖了模块 `func` 的名称）

当包由多个子包构成，则模块的导入方式为：相对导入、绝对导入
**what**：相对导入
基于当前模块名(从当前模块所在的目录开始)，来导入

**what**：绝对导入
从最高层级包名开始，来导入

最佳实践：若包中的一个模块需要作为python应用程序的主模块，则该主模块的导入语句必须是绝对导入；原因：相对导入，基于当前模块名；若当前模块是主模块，则主模块名永远是__main__，那又如何进行相对导入呢？

#### 当导入一个模块时，是如何搜索的？

内置模块(sys.builtin_module_names) => 模块搜索路径(sys.path)(里面包含了标准模块的搜索路径)

#### sys.path(模块搜索路径)是如何初始化的？

初始化顺序：脚本运行的当前目录 -> PYTHONPATH环境变量 -> site-packages目录(pip安装的第三方库) -> python标准库目录
可能出现的问题：脚本所在目录有与标准库同名的文件，则加载的是脚本所在目录的文件，而不是标准库的

### 命名空间、作用域

**what**：命名空间
名称到对象的映射，通过字典对象(`__dict__`)实现：存放了key-value对，key是名称(变量、函数、类等)，value是对象。有哪些？
1. 局部
2. 闭包
3. 全局
4. 内置

- `dir()`使用方式：
	1. 没有实参：返回当前作用域的名称list
	2. 有实参 ：返回该对象的有效属性list(调用obj的__dir__；若无定义，则从__dict__收集信息，从__mro__收集类及父类的属性和方法)
- `__dir__`的作用：调用dir(obj)时，会调用obj.__dir__()，返回一个iterable对象
- `__dict__`的作用：返回对象中可写名称的dict
- `__mro__`的作用：(Method Resolution Order)方法解析顺序，用于查看类的继承链

**what**：作用域
命名空间的可见性范围 => 用于控制命名空间的可见性

名称查找顺序（LEGB规则）
1. Local：局部命名空间
2. Enclosing：闭包命名空间(非局部非全局的空间)
3. Global：全局命名空间
4. Built-int：内置命名空间

**what**：nonlocal
将名称绑定到 E 中的名称，使得我们能在 L 中读写它；E 中的 名称不存在，报错

**what**：global
将名称绑定到 G 中的名称，使得我们能在 L、E 中读写它；若 G 中的 名称不存在，创建

### 类

**what**：类的命名空间
独立于前面的LEGB

- 类内部名称查找顺序：
	1. 类命名空间
	2. 父类命名空间
	3. 类定义所在的作用域(通常是G，除非类嵌套在函数中)

- 成员变量：public
- 成员函数：virtual

**what**：类变量
是类中定义的变量，代表类共有的属性

**what**：实例变量
是类的__init__中定义的变量，代表实例所特有的属性（相同的名称同时出现在实例和类中，则优先选择实例）

- `__init__`的作用：执行类初始化相关操作(为成员属性赋值)
- `type()`：底层调用obj.__class__
- `__class__`的作用：获取类型(eg：int、str、class、class方法...)
- `__doc__`的作用：返回类的文档字符串

- isinstance()：检查一个实例是否为某个类
- issubclass()：检查某个类是否是另一个类的子类

- 定义私有变量：规范是使用_(一个下划线开头)作为私有变量名的前缀，这只是一个约定，python中没有真正的私有变量，推荐替代方式是使用名字改写

**what**：名称改写
使用__(两个下划线)作为私有变量名的前缀，python会将其改写为`_classname__attribute`
```python
class test:
    __name # python将其改写为_test__name；只能通过_test__name来访问它，实现伪私有变量
```

**what**：数据类
类似 C专门存放数据的struct；使用装饰器@dataclass自动生成`__init__`、`__repr__`等方法

### 迭代器、生成器

**what**：迭代器（iterator）
实现了迭代器协议的类，用于逐个返回数据

**what**：迭代器协议
实现`__iter__`(返回自身)、`__next__`(返回下一个元素或抛出StopIteration)

**what**：生成器（generator）
是iterator的子集，内部有yield语句(自动实现迭代器协议)的函数；用于生成一系列值供for语句、next()使用；通过yield实现 惰性生成值（值按需生成，而不是一次性全部加载 => 节省内存）

**what**：生成器表达式 
类似列表推导式，但是外层为 圆括号()而不是方括号[]，会立即被外层函数所使用

### 官方推荐的编码风格

1. 类命名：全大写驼峰
2. 函数命名：全小写带下划线
3. 私有变量：使用双下划线(`__`)生成单下划线的变量
