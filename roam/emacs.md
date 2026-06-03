---
tags:
  - emacs
  - 工具
---
# emacs

**what**：
高度可定制、可扩展的文本编辑器，可以通过 [[elisp]] 来扩展功能，其中著名的 mode 是 [[org-mode]] ，著名的配置框架是 [[doom emacs]]

**how**：帮助

|命令|作用|
|---|---|
|C-h i|🔥 查看 info 文档|
|C-h ?|所有帮助|
|C-h k|快捷键绑定到了哪个函数|
|C-h c c|示当前缓冲区所有生效的快捷键列表|
|C-h o|符号|
|C-h v|变量|
|C-h f|函数|
|C-h m|模式|
|C-h b k|查看 keymap|

**how**：用户设置
`M-x customize`

**how**：el、elc、eln文件

|文件|说明|生成方式|
|---|---|---|
|.eln|机器码，平台相关，速度快|native-compile|
|.elc|字节码，跨平台，速度中|byte-compile-file|
|.el|源代码，跨平台，速度慢||

## 工具链

| 类别  | 作用                   | 工具               |
| --- | -------------------- | ---------------- |
| 笔记  | 笔记、日程管理              | [[org-mode]]     |
|     | 画图                   | artist           |
| 编程  | 智能感知                 | lsp-mode         |
|     | 项目管理                 | projectile       |
|     | 终端                   | vterm            |
|     | 调试                   | gud dape         |
|     | 工作区                  | persp-mode       |
|     | git（emacs 版）         | magit            |
| 导航  | 字母、行跳转               | avy              |
|     | 搜索导航                 | consult          |
|     | 书签                   | bookmark.el      |
| 编辑  | 模板 管理                | yasnippet        |
|     | 多光标编辑                | multiple-cursors |
| 窗口  | buffer 翻转            | transpose-frame  |
|     | 窗口布局 恢复、重做           | winner           |
|     | 窗口 切换、管理             | ace-window       |
|     | 便签页 管理               | tab-bar          |
|     | minibuffer 增强，弹出补全窗口 | vertico          |
|     | minibuffer 增强，无序搜索   | orderless        |
|     | minibuffer 增强，显示注释   | marginalia       |
| 其他  | 鼠标右键                 | embark           |
|     | 文件管理                 |                  |
|     | 包配置 管理               | use-package      |

## emacs配置

配置文件路径 ：

| 路径                      | 说明           |
| ----------------------- | ------------ |
| ~/.emacs.d              | 符合工程化的配置     |
| ~/.emacs                | 单一文件配置       |
| ~/.config/emacs/init.el | 仅适用于 >=27 版本 |

配置加载顺序 ：
1. .emacs.d/early-init.el
2. site-start.el          系统级配置（对所有用户生效）
3. .emacs.d/init.el    用户级配置
4. default.el
配置最佳实践 ：
5. 系统级配置：将配置放到 site-init.el 中（eg：共享插件路径）
6. 用户级配置：将配置放入 .emacs.d/init.el 中
7. 模块化管理：将配置拆分到 .el文件 中，在 init.el中load/require 加载

### hook

**what**：
某个事件发生时（如：打开文件、切换模式） 自动调用的函数列表 （类似回调函数）
```elisp
(setq default-major-mode 'text-mode)          ; 当打开一个新文件时，如果它不需要进入其他模式，默认进入文本模式；default-major-mode api似乎已失效
(add-hook 'text-mode-hook 'turn-on-auto-fill) ; auto-fill-mode：打开自动换行模式，超出屏幕的部分自动换行(doom无效，原生emacs生效)
```

### aliases

**what**：
为函数起别名，允许不同名字调用同一函数，通过 defalisa 实现
```elisp
(setq mail-aliases t)  ; 使用邮件别名
```

### load、load-path

**what**：load
动态加载 .el 和 .elc文件 的函数

**what**：
load-path
存储搜索 .el、.elc 这些文件的目录路径的变量

```elisp
; load：加载kfill.el文件，当然如果存在kfill.elc速度会更快
(load "~/emacs/kfill")
; load-path
(setq load-path (cons "~/emacs" load-path))
```

### autoload

**what**：
延迟加载机制，声明函数和文件的映射关系，在函数首次调用时，才会加载对应的代码文件
```elisp
(autoload 'html-helper-mode ; 从html-helper-mode.el(或.elc)文件延迟加载html-helper-mode函数。该文件必须在load-path中
  "html-helper-mode" "Edit HtML documents" t) ; 因为函数还没被加载，我们希望在M-x的时候看到相关信息，因此写入注释
```

### keymap

**what**：
(键映射) 快捷键对应函数的键值对列表
1. 优先级：模式相关keymap > 全局keymap
2. define-key：函数绑定与模式相关的键映射
```elisp
(define-key texinfo-mode-map (kbd "C-c C-l") 'texinfo-insert-@group)
```

## emacs加载不同配置启动

```bash
emacs --init-directory ~/.emacs.d.learn   # 加载 ~/.emacs.d.learn 中的配置
```

## emacs设置缩进风格

M-x c-set-style -> gnu

## vterm复制终端内容

打开 vterm-copy mode： C-c C-t
