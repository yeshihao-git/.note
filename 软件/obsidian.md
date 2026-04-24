# 基础
- 二级列表：在一级列表行按 **回车 (Enter)** 产生新行后，按下 **`Tab`** 键
- 标签：`#标签内容`


## 链接
嵌入：将笔记内容嵌入到当前笔记`![[]]`
创建笔记链接 `[[]]`
打开链接

| 操作        | MacOS                | Windows/Linux           |
| --------- | -------------------- | ----------------------- |
| **新建标签页** | `⌘`（在源码模式下加上`Shift`） | `Ctrl`（在源码模式下加上`Shift`） |
| **新建标签组** | `⌘` `⌥`              | `Ctrl` `Alt`            |
| **新建窗口**  | `⌘` `⌥` `Shift`      | `Ctrl` `Alt` `Shift`    |


# 设置大纲
- 打开大纲
	1. 方式1：命令面板 - 显示大纲页面
	2. 方式2：右侧展开 - 点击大纲缩略图 - 将大纲缩略图拖到左侧，更方便使用


# 关系图谱与笔记联动
实现左侧点击关系图谱中节点时，右侧自动显示对应笔记
1. 打开关系图谱
2. 右上角选择 左右分屏
3. 右键 关系图谱标签页 选择 锁定


# 图片文件夹与自动清理未使用图片
设置图片存放文件夹
1. 设置 - 文件与链接 - 附件默认存放路径 - 当前文件所在文件夹下指定的子文件夹
2. 子文件夹名称 设置为 `assets`

自动清理未使用图片
3. 设置 - 第三方插件 - 安全模式 - 关闭
4. 社区插件市场 - `Clear Unused Images`
5. 在命令面板使用

# 给表格加上横向滑动条
设置 - 外观 - CSS代码片段
![[Pasted image 20260416105633.png]]
选择文件夹，新建 css 文件，写入以下内容
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

开启 table-scroll 后刷新