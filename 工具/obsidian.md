# 1 基础
## 1.1 常用快捷键

| 作用    | 快捷键      |
| ----- | -------- |
| 搜索文件  | `C-o`    |
| 执行命令  | `C-p`    |
| 切换标签页 | `C-<数字>` |

# 2 数学（latex）

| 类型   | 语法        | 示例                     | 渲染效果               |
| ---- | --------- | ---------------------- | ------------------ |
| 行内公式 | `$...$`   | `$E=mc^2$`             | $E=mc^2$           |
| 块级公式 | `$$...$$` | `$$\sum_{i=1}^n x_i$$` | $\sum_{i=1}^n x_i$ |

| 符号      | LaTeX 代码                         | 示例                               |
| ------- | -------------------------------- | -------------------------------- |
| 分数      | `\frac{a}{b}`                    | $\frac{a}{b}$                    |
| 平方 / 上标 | `x^2`                            | $x^2$                            |
| 下标      | `x_i`                            | $x_i$​                           |
| 根号      | `\sqrt{x}`                       | $\sqrt{x}$​                      |
| 求和      | `\sum_{i=1}^n`                   | $\sum_{i=1}^n$​                  |
| 积分      | `\int_a^b`                       | $\int_a^b$                       |
| 极限      | `\lim_{x\to\infty}`              | $\lim_{x\to\infty}​$             |
| 加减乘除    | `+ - \times \div`                | $+ - \times \div$                |
| 不等号     | `\le \ge \ne`                    | $\le \ge \ne$                    |
| 希腊字母    | `\alpha \beta \gamma \theta \pi` | $\alpha \beta \gamma \theta \pi$ |

| 功能   | 代码模板                                               | 渲染效果                                            |
| ---- | -------------------------------------------------- | ----------------------------------------------- |
| 多行对齐 | `$$\begin{aligned}a&=b\\c&=d\end{aligned}$$`       | $\begin{aligned}a&=b\\c&=d\end{aligned}$        |
| 矩阵   | `$$\begin{pmatrix}1&2\\3&4\end{pmatrix}$$`         | $\begin{pmatrix}1&2\\3&4\end{pmatrix}$          |
| 行列式  | `$$\begin{vmatrix}a&b\\c&d\end{vmatrix}$$`         | $\begin{vmatrix}a&b\\c&d\end{vmatrix}$          |
| 分段函数 | `$$f(x)=\begin{cases}x&x\ge0\\-x&x<0\end{cases}$$` | $f(x)=\begin{cases}x&x\ge0\\-x&x<0\end{cases}$​ |

# 3 技巧
## 3.1 关系图谱与笔记联动
实现左侧点击关系图谱中节点时，右侧自动显示对应笔记
1. 打开关系图谱
2. 右上角选择 左右分屏
3. 右键 关系图谱标签页 选择 锁定

## 3.2 自动清理未使用图片
设置图片存放文件夹
1. 设置 - 文件与链接 - 附件默认存放路径 - 当前文件所在文件夹下指定的子文件夹
2. 子文件夹名称 设置为 `assets`

自动清理未使用图片
3. 设置 - 第三方插件 - 安全模式 - 关闭
4. 社区插件市场 - `Clear Unused Images`
5. 在命令面板使用

## 3.3 给表格加上横向滑动条
1. 设置 - 外观 - CSS代码片段
![[Pasted image 20260416105633.png]]

2. 选择文件夹，新建 css 文件，写入以下内容
```css
.markdown-source-view div.cm-embed-block.cm-table-widget {
    overflow-x: auto;
}
.markdown-source-view div.cm-embed-block.cm-table-widget > div > table {
    width: max-content !important;
}
.markdown-reading-view .markdown-preview-view.markdown-rendered table {
    width: max-content !important;
}
```

3. 开启 table-scroll 后刷新

## 3.4 标题自动编号
1. 下载插件 `Number Headings`
   
2. 设置如下
![[Pasted image 20260509114350.png|575]]