---
tags:
  - windows
aliases:
  - windows 服务器部署网站
---
# windows server部署网站 
## 部署网站

1. 在 IIS（微软在 windows 上提供的 web 服务器软件） 中，右键 `网站`，选择`添加网站`，完成相关配置
![[Pasted image 20260907164646.png|1144]]

2. 在 `高级安全 Windows 防火墙` 中添加`入站规则`，并完成相应配置
![[Pasted image 20260907164824.png|1003]]

## 更新网站

1. 点击图中 “浏览” 后，将新的打包后的产物覆盖更新，按下 `Ctrl + F5` 强制刷新网站
![[Pasted image 20261012120415.png]]

2. 如果希望不强制刷新网页的方式更新网站内容，需要在 `dist/public/web.config` 中编写以下内容，并 `F5`
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="index.html">
    <system.webServer>
      <httpProtocol>
        <customHeaders>
          <add name="Cache-Control"
               value="no-cache, no-store, must-revalidate" />
          <add name="Pragma" value="no-cache" />
          <add name="Expires" value="0" />
        </customHeaders>
      </httpProtocol>
    </system.webServer>
  </location>
</configuration>
```
以上是针对 `index.html` 禁用缓存（即每次访问 index.html 都会发送网络请求）