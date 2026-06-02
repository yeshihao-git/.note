---
tags:
  - js
  - 前端
---
# 渲染管线原理

![[Pasted image 20260409182138.png]]

| 术语                   | 作用                                      |
| -------------------- | --------------------------------------- |
| `Source`（数据源）        | 提供数据（创建或读取）                             |
| `Filters`（过滤器）       | 对数据进行算法处理（剪裁、平移、旋转等）                    |
| `Mapper`（映射器）        | 数据→图元（渲染管线、可视化管线的交汇点）                   |
| `Actor`（实体）          | 场景中的一个物体，绑定`Mapper`和`Property`（形状和外观）   |
| `Renderer`（渲染器）      | 管理`Light`、`Camera`、`Actor`（持有多个`Actor`） |
| `RenderWindow`（渲染窗口） | 连接浏览器`DOM`的容器，如`Canvas`（持有多个`Renderer`） |
| `Interactor` (交互器)   | 监听鼠标、键盘事件，更新相机位置或物体状态                   |

# vtk.js架构

- **Datatypes (数据类型):** 图像数据、几何/非结构化数据
- **Filters (过滤器):** 变换或生成数据
- **Rendering (渲染):** 数据如何被着色、呈现和放置
- **Interaction (交互):** 控制场景视图或与场景中的 Actor（实体）进行交互
- **I/O (输入/输出):** 读取 [[#3. vtk数据文件]] 到 `vtk.js` 
- **Proxy (代理):** 一种抽象机制，用于集中处理典型的管线操作


# vtk数据文件

| 后缀     | 全称                       | 数据类型   | 用途                    |
| ------ | ------------------------ | ------ | --------------------- |
| `.vti` | VTK ImageData XML        | 规则体数据  | 图像、体积、规则网格            |
| `.vtr` | VTK RectilinearGrid XML  | 正交直线网格 | 直角坐标网格                |
| `.vts` | VTK StructuredGrid XML   | 结构化网格  | 结构化网格                 |
| `.vtp` | VTK PolyData XML         | 多边形数据  | 多边形数据（**点、线、面、三角网格**） |
| `.vtu` | VTK UnstructuredGrid XML | 非结构化网格 | 非结构化网格                |

# 问题解决
## fix：模型旋转中心不在模型中心

> 问题背景：模型围绕着模型底部旋转

**vtk.js 中的三个中心**：
1. camera.focalPoint 是相机焦点，说明相机看向的位置
2. actor.setOrigin 是模型自身的旋转中心
3. interactorStyle.setCenterOfRotation 是交互旋转中心，控制世界围哪个点转

**解决**：计算模型的包围盒中心后，设置为交互器的旋转中心
```js
// 获取交互器
const interactor = renderWindow.getInteractor();

if (interactor) {
  interactor.setEnabled(true);
}

// 计算包围盒中心
const bounds = surfaceData.getBounds();

const center = [
  (bounds[0] + bounds[1]) / 2,
  (bounds[2] + bounds[3]) / 2,
  (bounds[4] + bounds[5]) / 2
];

// 重置相机
// 1. 自动计算模型包围盒：找到场景里所有模型的最小 / 最坐标
// 2. 自动设置相机焦点：focalPoint：让相机看向模型中心。
// 3. 自动设置相机位置：position：让相机退到合适距离，保证整个模型完整显示在画面里
renderer.resetCamera();

// 设置交互器旋转中心
const interactorStyle = interactor.getInteractorStyle();
interactorStyle.setCenterOfRotation(...center);

renderWindow.render();
```