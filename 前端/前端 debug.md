# 1. 使用 vscode debug vue.js 代码
1. 下载插件
![[Pasted image 20260423134301.png]]

2. 在 vite.config.js 开启 SourceMap
```js
export default defineConfig( ( { mode } ) => {
  return {
    // ...
    server: {
      sourcemap: true // 开启 sourcemap
    }
  };
} );
```

3. 编写 launch.json
```js
{
  // 调试配置列表（当前生效的配置）
  "configurations": [
    {
      "name": "Launch Chrome",              // 调试配置显示名称（F5时可见）
      "request": "launch",                  // 启动模式：新建Chrome窗口
      "type": "chrome",                     // 调试类型：调试Chrome浏览器
      "url": "http://10.8.254.124:5173",     // 调试时自动打开的项目地址
      "webRoot": "${workspaceFolder}",      // 项目源码根目录（VSCode打开的文件夹）
      
      // 源码路径映射：让VS Code正确找到本地src文件
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/src/*"
      }
    }
  ]
}
```

4. 通过 npm run dev 启动后，即可使用 vscode 进行 debug

> **SourceMap**：打包压缩后的代码 <---映射---> 原始源码，用于调试