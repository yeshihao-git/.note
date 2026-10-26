---
tags:
  - 字体
---
# Web 字体格式：WOFF2 / WOFF / TTF / SVG

|格式|全称|主要特点|兼容性|当前建议|
|---|---|---|---|---|
|**WOFF2**|Web Open Font Format 2.0|WOFF 的升级版，压缩率更高，文件体积更小|现代浏览器普遍支持|网页字体首选格式|
|**WOFF**|Web Open Font Format|TTF/OTF 的 Web 压缩封装版，支持元数据|现代浏览器普遍支持|推荐使用，常作为 WOFF2 的兼容回退|
|**TTF**|TrueType Font|通用桌面字体，兼容性强，但未压缩，文件较大|系统和浏览器兼容性最好|适合作为源文件转换，网页不建议首选|
|**SVG**|SVG Fonts|基于矢量图形，早期 iOS Safari 支持较好|现代浏览器基本不再支持|已淘汰，新项目不建议使用|
