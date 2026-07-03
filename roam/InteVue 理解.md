---
tags:
  - 工作
---
# Render

ScenMeshArray：用于 `Mesh` 排序与渲染量管理，解决 大规模3D模型的渲染性能问题 
```
┌──────────────────────────────────────────────────────────────┐
│                    ScenMeshArray                              │
│                                                              │
│  职责1: Mesh创建与收集                                        │
│    CreateMesh(geometry, material, matrix, rootKey)            │
│    AddInstancedMesh(InstancedMesh)                            │
│                                                              │
│  职责2: 智能排序                                              │
│    SortByRadiusAndCenter()                                    │
│    → 离模型中心越远、尺寸越大的mesh排越前面                      │
│    → 排序公式: scalePos × log(1 + size)                       │
│    → 效果: 大的、外围的零件先渲染，小的、内部的零件后渲染         │
│                                                              │
│  职责3: 渲染量控制                                            │
│    meshNumLimit   = 10000    (最大mesh数)                     │
│    trangNumLimit  = 10000000 (最大三角面数)                    │
│    meshRatio      = 0~100%   (实际渲染比例)                   │
│    trangRatio     = 0~100%   (实际面数比例)                    │
│                                                              │
│  职责4: 挂载到场景树                                          │
│    BuildToThree()                                             │
│    → 线条优先放到children前面                                  │
│    → 排序后的mesh按顺序添加到rootKey.children                  │
│    → 超出限制的mesh仍然添加但标记为不渲染                       │
└──────────────────────────────────────────────────────────────┘
```

CustomRenderList.js 和 TYWebGLRenderer.js 协作关系
```
模型加载完成
       │
       ▼
CustomRenderList.js
  ScenMeshArray.CreateMesh()    → 创建所有mesh
  ScenMeshArray.SortByRadiusAndCenter() → 按大小+位置排序
  ScenMeshArray.BuildToThree()  → 挂载到场景树 rootKey.children
       │
       │  children[0] = LineSegments (边线, 优先)
       │  children[1] = 最大最远的mesh
       │  children[2] = ...
       │  children[N] = 最小最近的mesh
       │
       ▼
TYWebGLRenderer.js
  renderFast_V2()
       │
       ├─ projectObject_V2(rootKey, ...)
       │    遍历 children，按顺序添加到渲染列表
       │    达到 trangNLimit 后停止 → 大零件优先渲染
       │
       ├─ projectObject_noFilter(cutGroup, ...)
       │    剖切面不受限制，全部渲染
       │
       ├─ projectObject_V3(screwThreadGroup, ...)
       │    螺纹单独渲染
       │
       ├─ isSkipObjectByStaticPixSize()
       │    跳过屏幕上小于5像素的物体
       │
       ├─ tyrenderObject1()
       │    ├─ HiddenLineRemoved → 两遍消隐渲染
       │    ├─ FaceAndEdgeLine   → 面片+偏移边线
       │    ├─ WireFrameMesh     → 线框模式
       │    └─ BasicRender       → 标准渲染
       │
       └─ projectHighLightObjects()
            用高亮材质替换渲染选中对象
```

# MarkUp

三个图层负责将批注/测量对象渲染到不同空间
```
┌─────────────────────────────────────────────────────────┐
│                    THREE2DLayer                          │
│  作用: 2D叠加渲染层                                      │
│  特点: 后处理渲染管线(SMAA+高亮+轮廓线)                    │
│  内容: 高亮对象、轮廓线、2D批注                            │
│  相机: 跟随主相机                                         │
├─────────────────────────────────────────────────────────┤
│                THREELockCameraLayer                      │
│  作用: 锁定相机图层                                       │
│  特点: 禁用旋转、平移、缩放                                │
│  内容: 屏幕PMI(产品制造信息)标注                           │
│  相机: 固定正交，不随主相机旋转                             │
├─────────────────────────────────────────────────────────┤
│               ThreeLockRotateLayer                       │
│  作用: 锁定旋转图层                                       │
│  特点: 禁用旋转，支持平移和缩放                             │
│  内容: 需要跟随缩放但不旋转的标注                           │
│  相机: 跟随位置和缩放，不跟随旋转                           │
└─────────────────────────────────────────────────────────┘
```

# Command

实现了一套完整的 命令模式（Command Pattern） ，核心设计为：
CommandObject基类 （run/undo/redo接口） → 50+具体命令 （各自记录状态快照） → CommandManager （工厂创建 + 四栈undo/redo管理）

# 三个层

| 层级  | 对象              | 职责        | 本函数中用到的方法                       |
| --- | --------------- | --------- | ------------------------------- |
| 文档层 | InteVueDocument | 文件管理、命令管理 | `GetInteWebVue()`               |
| 视图层 | InteWebVue      | 模型管理、选择集  | `GetCtrlView()`                 |
| 控制层 | WebCtrlView     | 相机、交互、坐标轴 | `GetAxisHelper()`、`GetCamera()` |
InteVueDocument 中持有 InteWebVue 和 CommandManager，InteWebVue 中持有 WebCtrlView

在 `operateCamera()` 函数中，最终目的是操控相机切换视图，所以需要沿着 `文档 → 视图 → 控制 → 相机` 这条链路逐层拿到相机对象。


# 三维旋转

前视图中的实现方式为使用**四元数**进行三维旋转，CommandChangeView.js 行：153 - 160
```
pos    = 当前相机朝向方向（归一化）
normal = 目标视图朝向方向
quat   = 从 pos 旋转到 normal 的四元数
eye    = 相机相对目标点的偏移向量（方向+距离）
eye.applyQuaternion(quat) = 旋转偏移向量（方向变，长度不变）
新位置 = 目标点 + 旋转后的偏移向量
```

**绕目标点旋转相机**——算出"从当前朝向到目标朝向"的旋转量，把这个旋转应用到"相机到目标的偏移向量"上，偏移向量方向变了但长度不变，加回目标点就是相机新位置。

# H_Send_Message

内部调用全局统一消息入口 

# 剖切代码流程

点击剖切面板的**沿Y轴切割**后，到显示Y轴剖面全流程
- 点击前：
![[Pasted image 20260703090304.png]]

- 点击后：
![[Pasted image 20260703141243.png|338]]

```hl:1-12,14-16,19-21,27
CInteStencilClipControl.setNodesClippingPlane (10.8.254.124꞉5174/src/appCore/Operate/Clip&Explosion/InteStencilClipControl.js:166)
HIOpClippingTwoPlane.setDefaultPlane (10.8.254.124꞉5174/src/appCore/Operate/Clip&Explosion/HIOpClippingTwoPlane.js:547)
HIOpClippingTwoPlane.initOperate (10.8.254.124꞉5174/src/appCore/Operate/Clip&Explosion/HIOpClippingTwoPlane.js:130)
HIOpClippingTwoPlane.constructor (10.8.254.124꞉5174/src/appCore/Operate/Clip&Explosion/HIOpClippingTwoPlane.js:58)
createOpByType_part1 (10.8.254.124꞉5174/src/appCore/Operate/HIOpFactory.js:112)
CreateOpByType (10.8.254.124꞉5174/src/appCore/Operate/HIOpFactory.js:69)
SetOperateByType (10.8.254.124꞉5174/src/appCore/Operate/HIOpFactory.js:407)
registerOperate (10.8.254.124꞉5174/src/appCore/Send_Message.js:1617)
sendMessage3 (10.8.254.124꞉5174/src/appCore/Send_Message.js:168)
H_Send_Message (10.8.254.124꞉5174/src/appCore/Send_Message.js:42)
clipPlane (c:\Users\HP\code\backup\vuejs\src\components\panel\ClippingPanel.vue:282)
clipAxios_Y (c:\Users\HP\code\backup\vuejs\src\components\panel\ClippingPanel.vue:456)
Proxy(Object).checkPane (c:\Users\HP\code\backup\vuejs\src\components\panel\ClippingPanel.vue:470)
CommandCheckClippingPane.run (10.8.254.124꞉5174/src/appCore/Command/CommandCheckClippingPane.js:18)
CommandManager.run (10.8.254.124꞉5174/src/appCore/Command/CommandManager.js:116)
checkClippingPane (10.8.254.124꞉5174/src/appCore/Send_Message.js:951)
<anonymous> (10.8.254.124꞉5174/src/appCore/Send_Message.js:370)
setTimeout (未知源:0)
sendMessage7 (10.8.254.124꞉5174/src/appCore/Send_Message.js:369)
H_Send_Message (10.8.254.124꞉5174/src/appCore/Send_Message.js:46)
clipAxios_UseCommand (c:\Users\HP\code\backup\vuejs\src\components\panel\ClippingPanel.vue:443)
_createVNode.onClick._cache.<computed>._cache.<computed> (10.8.254.124꞉5174/src/components/panel/ClippingPanel.vue:733)
callWithErrorHandling (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-core\dist\runtime-core.esm-bundler.js:158)
callWithAsyncErrorHandling (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-core\dist\runtime-core.esm-bundler.js:166)
emit (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-core\dist\runtime-core.esm-bundler.js:664)
<anonymous> (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-core\dist\runtime-core.esm-bundler.js:7422)
onClick (c:\Users\HP\code\backup\vuejs\node_modules\ant-design-vue\es\vc-checkbox\Checkbox.js:101)
callWithErrorHandling (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-core\dist\runtime-core.esm-bundler.js:158)
callWithAsyncErrorHandling (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-core\dist\runtime-core.esm-bundler.js:166)
invoker (c:\Users\HP\code\backup\vuejs\node_modules\@vue\runtime-dom\dist\runtime-dom.esm-bundler.js:278)
```

这个过程涉及到 **Vue 页面模板事件 → 统一消息分发器 → 命令调度** 的三层架构