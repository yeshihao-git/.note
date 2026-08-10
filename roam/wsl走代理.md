---
tags:
  - wsl
  - windows
---
# wsl走代理

```powershell
vim C:\Users\<你的用户名>\.wslconfig

# .wslconfig 内容
[wsl2]
networkingMode=mirrored
autoProxy=true
# end

wsl --shutdown
```