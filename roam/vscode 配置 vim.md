---
tags:
  - 工具
  - vscode
  - vim
sources: https://github.com/jlvihv/vscode-vim-keybindings
---
# vscode 配置 vim

1. 安装 `vim` 插件
![[Pasted image 20260618155940.png|315]]
2. `Ctrl+Shift+p` 打开命令面板，选择
![[Pasted image 20260618164929.png|451]]
3. 通过上一步打开 `settings.json` 后编辑以下内容
```json
{
  "vim.useSystemClipboard": true, // 系统剪切板
  "vim.smartRelativeLine": false, // 关闭 智能行号（用于跳转）
  "vim.useCtrlKeys": false, // vim 不捕获 Ctrl 组合键
  "vim.leader": "<space>", // leader key 设置为 空格
  "vim.handleKeys": { // 单独设置 <C-[> 交给 vim 捕获，其余交给 vscode 捕获
    "<C-[>": true
  },
  "vim.easymotion": true, // 开启全局字母跳转
  "vim.hlsearch": true, // 高亮搜索内容
  // 普通模式
  "vim.normalModeKeyBindingsNonRecursive": [
    // 块可视化模式
    {
      "before": ["<leader>","v"],
      "after": ["<C-v>"]
    },
    // 定义速览
    {
      "before": ["K"],
      "commands": ["editor.action.showHover"]
    },
    // 转到定义
    {
      "before": ["g","d"],
      "commands": ["editor.action.revealDefinition"]
    },
    // 转到引用
    {
      "before": ["g","r"],
      "commands": ["editor.action.goToReferences"]
    },
    // 转到实现
    {
      "before": ["g","i"],
      "commands": ["editor.action.goToImplementation"]
    },
    // 回退到上一个位置
    {
      "before": ["g","b"],
      "commands": ["workbench.action.navigateBack"]
    },
    // 空格+s 一键启动全局字符搜索
    {
      "before": ["<leader>","s"],
      "after": ["<leader>","<leader>","s"]
    }
  ]
}
```
4. 编辑结束后，`vscode` 的使用快捷键如下

| 作用                    | 快捷键                   |
| --------------------- | --------------------- |
| 命令面板                  | `Ctrl+Shift+p`        |
| 打开 `settings.json`    | `Ctrl+,`              |
| 打开 `keybindings.json` | `Ctrl+k Ctrl+s`       |
| 资源管理器                 | `Ctrl+0`              |
| 切换 编辑窗口               | `Ctrl+1` `Ctrl+2` ... |
| 切换 标签页                | `Alt+1` `Alt+2` ...   |
| 显示/隐藏 侧边栏             | `Ctrl+b`              |
| 显示/隐藏 终端              | `Ctrl+j`              |
| 搜索项目文件                | `Ctrl+p`              |
| vim返回普通模式             | `Ctrl+[`              |
| vim字母跳转               | `<leader> s <字母>`     |
| vim普通模式：块可视化          | `<leader> v`          |
| vim普通模式：定义速览          | `K`                   |
| vim普通模式：转到定义          | `g d`                 |
| vim普通模式：转到引用          | `g r`                 |
| vim普通模式：转到实现          | `g i`                 |
| vim普通模式：回退到上一个位置      | `g b`                 |
