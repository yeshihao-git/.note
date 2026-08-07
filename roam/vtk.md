---
tags:
  - js
  - 前端
  - 技巧
  - cpp
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

## vtk.js架构

- **Datatypes (数据类型):** 图像数据、几何/非结构化数据
- **Filters (过滤器):** 变换或生成数据
- **Rendering (渲染):** 数据如何被着色、呈现和放置
- **Interaction (交互):** 控制场景视图或与场景中的 Actor（实体）进行交互
- **I/O (输入/输出):** 读取 [[#3. vtk数据文件]] 到 `vtk.js` 
- **Proxy (代理):** 一种抽象机制，用于集中处理典型的管线操作

## vtk数据文件

| 后缀     | 全称                       | 数据类型   | 用途                    |
| ------ | ------------------------ | ------ | --------------------- |
| `.vti` | VTK ImageData XML        | 规则体数据  | 图像、体积、规则网格            |
| `.vtr` | VTK RectilinearGrid XML  | 正交直线网格 | 直角坐标网格                |
| `.vts` | VTK StructuredGrid XML   | 结构化网格  | 结构化网格                 |
| `.vtp` | VTK PolyData XML         | 多边形数据  | 多边形数据（**点、线、面、三角网格**） |
| `.vtu` | VTK UnstructuredGrid XML | 非结构化网格 | 非结构化网格                |

# vtk.js
## pipeline 写法

**why**：
下游依赖上游，当上游变化，下游能感知到，vtk 会自动沿着 pipeline 进行计算

**how**：标准写法
```text
vtkPolyData
      │
setInputData()
      │
TriangleFilter
      │
getOutputPort()
      │
setInputConnection()
      │
ClipFilter
      │
getOutputPort()
      │
setInputConnection()
      │
Normals
      │
getOutputPort()
      │
setInputConnection()
      │
Mapper
      │
Actor
```

对应代码就是：
```js
// polyData 这是数据本身，通过 reader 读取得到
const polyData = reader.getOutputData();

// 数据源
const triangleFilter = vtkTriangleFilter.newInstance();
triangleFilter.setInputData(polyData);

// Filter1 -> Filter2
const clipper = vtkClipPolyData.newInstance();
clipper.setInputConnection(triangleFilter.getOutputPort());

// Filter2 -> Mapper
const mapper = vtkMapper.newInstance();
mapper.setInputConnection(clipper.getOutputPort());

// clipper.getOutputData() 会将 pipeline 截断
// mapper.setInputConnection(clipper.getOutputData()); // 报错！！！！

// Mapper -> Actor
const actor = vtkActor.newInstance();
actor.setMapper(mapper);
```

**一句话总结**：
> 已有数据 → `setInputData()`
> 连接上游算法 → `setInputConnection()`

| 输入类型                     | 使用方法                   |
| ------------------------ | ---------------------- |
| `vtkPolyData`            | `setInputData()`       |
| `vtkImageData`           | `setInputData()`       |
| `vtkUnstructuredGrid`    | `setInputData()`       |
| `filter.getOutputPort()` | `setInputConnection()` |
| `reader.getOutputPort()` | `setInputConnection()` |

## vtkjs 这种 api 文档不全的库，如何查看具体信息

例如：`const config = vtkAxesActor.getConfig();` 使用时发现有智能提示 `config.recenter`，如何知道 config 中还有哪些字段？

### 方法1：debug

终端中启动 vite 服务器，在 launch.json 中将端口改到和 vite 服务器一致，开始 debug
1. 直接鼠标悬浮查看
2. 调试控制台中输入 `axesActor.getConfig()`

### 方法2：看源码

在头文件 `import vtkAxesActor from '@kitware/vtk.js/Rendering/Core/AxesActor';` 中右键选中==转到源定义==（不是转到类型定义）

发现 `defaultValues` 函数定义了 `config` 的默认结构：

```js
config: {
  recenter: true,        // ← 就在这里
  tipResolution: 60,
  tipRadius: 0.1,
  tipLength: 0.2,
  shaftResolution: 60,
  shaftRadius: 0.03,
  invert: false,
  ...initialValues?.config
}
```

所以 `getConfig()` 返回的 `config` 对象包含以下字段：

| 字段 | 默认值 | 含义 |
|------|--------|------|
| `recenter` | `true` | 是否将轴居中（true=原点在中间，false=原点在一端） |
| `tipResolution` | `60` | 箭头尖端分辨率 |
| `tipRadius` | `0.1` | 箭头尖端半径 |
| `tipLength` | `0.2` | 箭头尖端长度 |
| `shaftResolution` | `60` | 箭杆分辨率 |
| `shaftRadius` | `0.03` | 箭杆半径 |
| `invert` | `false` | 是否反转方向 |

同理，`xConfig` / `yConfig` / `zConfig` 各自包含 `color` 和 `invert` 字段。

### 方法 3：打印

```js
const config = axesActor.getConfig();
console.log(config);
```

后续在浏览器的控制台日志中查看，若 trae 开启调试模式，则能在 调试控制台 查看

## 问题解决
### fix：模型旋转中心不在模型中心

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

# vtkcpp
## 重要概念
### 内部索引

**what**：
整数 ID，用于标识 node、elements

**why**：
快速访问 node、elements

## 支持的文件格式以及对应 Reader

本节介绍 VTK 中常用的格式，包括：VTK 原生格式、部分外部数据格式；
其次介绍对应的 Reader，在 VTK（C++）中 Reader 类的命名通常遵循 `vtk[格式名]Reader`。

| 类型                           | 扩展名   | 说明      | 对应的Reader                       |
| ---------------------------- | ----- | ------- | ------------------------------- |
| ImageData（图像数据）              | .vti  | 串行、结构化  | `vtkXMLImageDataReader`         |
| PolyData（多边形数据）              | .vtp  | 串行、非结构化 | `vtkXMLPolyDataReader`          |
| RectilinearGrid（矩形网格）        | .vtr  | 串行、结构化  | `vtkXMLRectilinearGridReader`   |
| StructuredGrid（结构化网格）        | .vts  | 串行、结构化  | `vtkXMLStructuredGridReader`    |
| UnstructuredGrid<br>（非结构化数据） | .vtu  | 串行、非结构化 | `vtkXMLUnstructuredGridReader`  |
|                              |       |         |                                 |
| PImageData                   | .pvti | 并行、结构化  | `vtkXMLPImageDataReader`        |
| PPolyData                    | .pvtp | 并行、非结构化 | `vtkXMLPPolyDataReader`         |
| PRectilinearGrid             | .pvtr | 并行、结构化  | `vtkXMLPRectilinearGridReader`  |
| PStructuredGrid              | .pvts | 并行、结构化  | `vtkXMLPStructuredGridReader`   |
| PUnstructuredGrid            | .pvtu | 并行、非结构化 | `vtkXMLPUnstructuredGridReader` |

### 原生格式可视化示例
#### ImageData

点的排列是 X/Y/Z 轴等间距排列（2D时，X/Y 轴等间距排列）的矩阵栅格；
由 **体素（立方体）** 组成。
![[Pasted image 20260415154608.png|455]]

#### PolyData

点的排列任意散布，主要用于表面，无坐标概念；
由 **点、线、三角形、四边形、多边形** 组成。
![[Pasted image 20260415154632.png|463]]

#### RectilinearGrid

点的排列是 X/Y/Z 轴不等间距的矩阵栅格；
由 **长方体** 组成。
![[Pasted image 20260415154649.png|469]]

#### StructuredGrid

点的排布是矩阵阵列，但点坐标可任意；
由 **曲面六面体** 组成。
![[Pasted image 20260415154705.png|478]]

#### UnstructuredGrid

点的排布是任意散布的，无坐标轴概念；
由 **四面体、金字塔 等** 组成。
![[Pasted image 20260415154713.png|474]]

### 外部数据格式

外部数据格式是外部软件生成的通用或专用的数据格式，用于数据交换。

| 类型             | 扩展名    | 对应的Reader      |
| -------------- | ------ | -------------- |
| STL（立体光刻）      | `.stl` | `vtkSTLReader` |
| PLY（多边形文件格式）   | `.ply` | `vtkPLYReader` |
| OBJ（Wavefront） | `.obj` | `vtkOBJReader` |

## 快速开始

本节通过一个剖切代码示例，来介绍 VTK 渲染管线流程、reader、剖切、切面等内容。

### 项目结构

```
.
├── CMakeLists.txt
└── CapClip.cxx
```

### 编写 CMakeLists.txt 

```cmake
# 1. 指定CMake最低版本要求
cmake_minimum_required(VERSION 3.12 FATAL_ERROR)

# 2. 定义项目名
project(vtk_cpp)

# 3. 查找系统中安装的VTK库，并指定需要依赖的VTK模块
find_package(VTK COMPONENTS 
  CommonColor                # 颜色处理：颜色映射、颜色空间转换等
  CommonCore                 # VTK核心基础：对象系统、智能指针、时间戳、容器等
  CommonDataModel            # 基本数据模型
  FiltersCore                # 核心过滤器：剪切、轮廓、提取子集、变换等
  FiltersSources             # 数据源过滤器：生成球体、立方体、圆柱等基本几何体
  IOGeometry                 # 几何数据读写：OBJ、STL、PLY等常见几何格式
  IOLegacy                   # 传统VTK格式读写：.vtk文件（Legacy格式）
  IOPLY                      # PLY格式读写：多边形文件格式
  IOXML                      # XML格式VTK文件读写：.vtp、.vtu、.vtm等
  InteractionStyle           # 交互样式：鼠标旋转、缩放、平移等标准交互行为
  RenderingContextOpenGL2    # OpenGL2渲染上下文：提供窗口与OpenGL的桥接
  RenderingCore              # 渲染核心：vtkRenderer、vtkRenderWindow、vtkActor等
  RenderingFreeType          # 字体渲染：使用FreeType库渲染文字
  RenderingGL2PSOpenGL2      # 矢量图导出：通过GL2PS生成矢量输出（OpenGL2后端）
  RenderingOpenGL2           # OpenGL2渲染后端：基于现代OpenGL的渲染实现
)

# 4. 如果未找到VTK库，抛出致命错误并终止
if (NOT VTK_FOUND)
  message(FATAL_ERROR "Unable to find the VTK build folder.")
endif()

# 5. 强制 Ninja 构建工具使用响应文件
set(CMAKE_NINJA_FORCE_RESPONSE_FILE "ON" CACHE BOOL "Force Ninja to use response files.")

# 6. 生成可执行文件 vtk_cpp
add_executable(vtk_cpp)

# 7. 指定可执行文件的源文件
target_sources(vtk_cpp PRIVATE CapClip.cxx)

# 8. 将 VTK 库链接到生成的可执行文件
target_link_libraries(vtk_cpp PRIVATE ${VTK_LIBRARIES})

# 9. 自动生成初始化代码
vtk_module_autoinit(
  TARGETS vtk_cpp
  MODULES ${VTK_LIBRARIES}
)
```

### 编写代码

```C++
// CapClip.cxx 文件

#include <vtkActor.h>
#include <vtkCamera.h>
#include <vtkClipPolyData.h>
#include <vtkDataSetMapper.h>
#include <vtkFeatureEdges.h>
#include <vtkNamedColors.h>
#include <vtkNew.h>
#include <vtkPlane.h>
#include <vtkPolyData.h>
#include <vtkPolyDataMapper.h>
#include <vtkProperty.h>
#include <vtkRenderWindow.h>
#include <vtkRenderWindowInteractor.h>
#include <vtkRenderer.h>
#include <vtkSmartPointer.h>
#include <vtkSphereSource.h>
#include <vtkStripper.h>
#include <vtkXMLPolyDataReader.h>

// Readers
#include <vtkBYUReader.h>
#include <vtkOBJReader.h>
#include <vtkPLYReader.h>
#include <vtkPolyDataReader.h>
#include <vtkSTLReader.h>
#include <vtkXMLPolyDataReader.h>

// 辅助函数：用于读取不同格式的多边形数据文件
namespace {
vtkSmartPointer<vtkPolyData> ReadPolyData(std::string const& fileName);
}

int main(int argc, char* argv[])
{
  // 定义颜色
  vtkNew<vtkNamedColors> colors;          
  auto backgroundColor = colors->GetColor3d("steel_blue"); 
  auto boundaryColor = colors->GetColor3d("banana");       
  auto clipColor = colors->GetColor3d("tomato");           
  
  /****** VTK 渲染管线流程 --- start ******/
  
  // 读取多边形数据
  auto polyData = ReadPolyData(argc > 1 ? argv[1] : "");
  
  // 创建裁剪平面
  vtkNew<vtkPlane> plane;                  // 定义一个无限平面
  plane->SetOrigin(polyData->GetCenter()); // 平面通过物体的中心点
  plane->SetNormal(1.0, -1.0, -1.0);       // 设置平面法向量，用于斜向切割
  
  // 进行裁剪
  vtkNew<vtkClipPolyData> clipper;       // 裁剪过滤器
  clipper->SetInputData(polyData);       // 输入原始多边形数据
  clipper->SetClipFunction(plane);       // 设置裁剪函数
  clipper->SetValue(0);                  // 保留平面上值为0的一侧，即法向量指向的一侧
  clipper->Update();                     // 进行裁剪
  polyData = clipper->GetOutput();       // 将裁剪后的结果作为后续的数据源
  
  // 基于裁剪后的数据源创建 Mapper 并与 Actor 关联
  vtkNew<vtkDataSetMapper> clipMapper;   
  clipMapper->SetInputData(polyData);    
  vtkNew<vtkActor> clipActor;            
  clipActor->SetMapper(clipMapper);
  clipActor->GetProperty()->SetDiffuseColor(clipColor.GetData()); // 设置漫反射颜色
  clipActor->GetProperty()->SetInterpolationToFlat();             // 每个面着单一颜色
  clipActor->GetProperty()->EdgeVisibilityOn();                   // 显示边缘线
  
  // 提取裁剪后多边形的边界边缘
  vtkNew<vtkFeatureEdges> boundaryEdges; 
  boundaryEdges->SetInputData(polyData);
  boundaryEdges->BoundaryEdgesOn();      // 开启边界边缘
  boundaryEdges->FeatureEdgesOff();      // 关闭特征边缘
  boundaryEdges->NonManifoldEdgesOff();  // 关闭非流形边缘
  boundaryEdges->ManifoldEdgesOff();     // 关闭流形边缘
  
  // 将提取出的边界边缘合并为多边形
  vtkNew<vtkStripper> boundaryStrips; 
  boundaryStrips->SetInputConnection(boundaryEdges->GetOutputPort());
  boundaryStrips->Update();
  
  // 创建新的多边形数据集，用于存储边界多边形
  vtkNew<vtkPolyData> boundaryPoly;
  // 边界线条的点集
  boundaryPoly->SetPoints(boundaryStrips->GetOutput()->GetPoints());
  // 将线条转换为多边形
  boundaryPoly->SetPolys(boundaryStrips->GetOutput()->GetLines());
  
  // 为边界多边形创建 Mapper 并与 Actor 关联
  vtkNew<vtkPolyDataMapper> boundaryMapper;
  boundaryMapper->SetInputData(boundaryPoly);
  vtkNew<vtkActor> boundaryActor;
  boundaryActor->SetMapper(boundaryMapper);
  boundaryActor->GetProperty()->SetDiffuseColor(boundaryColor.GetData()); 
  
  // 创建 renderer、renderWindow
  vtkNew<vtkRenderer> renderer;          
  renderer->SetBackground(backgroundColor.GetData()); // 设置背景色
  renderer->UseHiddenLineRemovalOn();           // 开启隐藏线消除，使边界线不被实体遮挡
  vtkNew<vtkRenderWindow> renderWindow;         // 渲染窗口
  renderWindow->AddRenderer(renderer);
  renderWindow->SetSize(640, 480);              // 窗口大小
  vtkNew<vtkRenderWindowInteractor> interactor; // 交互器，用于处理鼠标/键盘事件
  interactor->SetRenderWindow(renderWindow);
  // 关联 renderer 和 Actor
  renderer->AddActor(clipActor);          
  renderer->AddActor(boundaryActor); 
       
  // 调整相机视角
  renderer->ResetCamera();                    // 重置相机
  renderer->GetActiveCamera()->Azimuth(30);   // 绕垂直轴旋转30度
  renderer->GetActiveCamera()->Elevation(30); // 绕水平轴俯仰30度
  renderer->GetActiveCamera()->Dolly(1.2);    // 拉近相机（缩放因子1.2）
  renderer->ResetCameraClippingRange();       // 重置裁剪范围，避免物体被近/远平面裁剪
  
  // 开始渲染和交互
  renderWindow->Render();                 // 首次渲染
  renderWindow->SetWindowName("CapClip"); // 设置窗口标题
  renderWindow->Render();                 // 再次渲染，确保标题更新
  interactor->Start();                    // 启动交互循环（等待用户操作）
  
  /****** VTK 渲染管线流程 --- end ******/
  
  return EXIT_SUCCESS;
}

// 辅助函数：读取多边形数据
namespace {
vtkSmartPointer<vtkPolyData> ReadPolyData(std::string const& fileName)
{
  vtkSmartPointer<vtkPolyData> polyData;
  std::string extension = "";
  
  if (fileName.find_last_of(".") != std::string::npos)
  {
    extension = fileName.substr(fileName.find_last_of("."));
  }
  
  std::transform(extension.begin(), extension.end(), extension.begin(),::tolower);
  
  // 根据扩展名选择对应的 reader
  if (extension == ".ply")
  {
    vtkNew<vtkPLYReader> reader;
    reader->SetFileName(fileName.c_str());
    reader->Update();
    polyData = reader->GetOutput();
  }
  else if (extension == ".vtp")
  {
    vtkNew<vtkXMLPolyDataReader> reader;
    reader->SetFileName(fileName.c_str());
    reader->Update();
    polyData = reader->GetOutput();
  }
  else if (extension == ".obj")
  {
    vtkNew<vtkOBJReader> reader;
    reader->SetFileName(fileName.c_str());
    reader->Update();
    polyData = reader->GetOutput();
  }
  else if (extension == ".stl")
  {
    vtkNew<vtkSTLReader> reader;
    reader->SetFileName(fileName.c_str());
    reader->Update();
    polyData = reader->GetOutput();
  }
  else if (extension == ".vtk")
  {
    vtkNew<vtkPolyDataReader> reader;
    reader->SetFileName(fileName.c_str());
    reader->Update();
    polyData = reader->GetOutput();
  }
  else if (extension == ".g")
  {
    vtkNew<vtkBYUReader> reader;
    reader->SetGeometryFileName(fileName.c_str());
    reader->Update();
    polyData = reader->GetOutput();
  }
  else
  {
    // 若扩展名未知，则生成一个球体作为默认数据
    vtkNew<vtkSphereSource> source;
    source->SetThetaResolution(20);   
    source->SetPhiResolution(11);     
    source->Update();
    polyData = source->GetOutput();
  }
  return polyData;
}
} // namespace
```

### 编译运行

```bash
cmake -B build -G Ninja
cmake --build build
./build/vtk_cpp
```

最终效果
![[Pasted image 20260415162645.png|355]]

## 常用 API 接口说明

本节只介绍 VTK 中最核心的 API（其余根据需求查找 [vtk-examples](https://examples.vtk.org/site/Cxx/) 和 [vtk reference](https://vtk.org/doc/nightly/html/classes.html)）

**渲染管线流程相关**

| 常用类                         | 说明                                                                                                                                                | 常用函数                                                                                            |
| :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------- |
| `vtk[形状]Source`             | 生成`[形状]`数据源：<br>1. `Sphere`（球体）<br>2. `Cone`（圆锥体）<br>3. `Cylinder`（圆柱体）<br>4. `Cube`（立方体）<br>5. `Plane`（平面）<br>6. `Line`（直线）                      | 见 [vtk reference](https://vtk.org/doc/nightly/html/classes.html)                                |
| `vtk[格式]Reader`             | 读取`[格式]`多边形文件：<br>1. `PLY`（PLY格式）<br>2. `OBJ`（OBJ格式）<br>3. `STL`（STL格式）<br>4. `XMLPolyData`（VTP格式）<br>5. `PolyData`（旧版VTK）<br>6. `PNG/JPEG`（标准图像） | `SetFileName()`<br>`Update()`                                                                   |
| `vtkPolyDataMapper`         | 将多边形数据映射为图元                                                                                                                                       | `SetInputData()`<br>`SetInputConnection()`                                                      |
| `vtkActor`                  | 渲染场景中的一个对象（包含几何形状、渲染属性）                                                                                                                           | `SetPosition()`<br>`SetMapper()`  <br>`GetProperty()`  `SetRepresentationToWireframe()`         |
| `vtkRenderer`               | 渲染器，管理场景中的 Actor、灯光和相机                                                                                                                            | `AddActor()` /`RemoveActor()`<br>`SetBackground()`  <br>`SetActiveCamera()`/`GetActiveCamera()` |
| `vtkRenderWindow`           | 渲染窗口，承载所有渲染器并输出最终图像                                                                                                                               | `AddRenderer()`    <br>`Render()`<br>`SetSize()`<br>`SetWindowName()`<br>`SetInteractor()`      |
| `vtkRenderWindowInteractor` | 捕获鼠标、键盘事件，驱动交互                                                                                                                                    | `SetRenderWindow()`  <br>`Start()`  <br>`Initialize()`                                          |

**智能指针相关**

| 常用类               | 说明                                                                | 常用函数                  |
| ----------------- | ----------------------------------------------------------------- | --------------------- |
| `vtkSmartPointer` | `VTK` 定制的智能指针，用于自动化管理继承自`vtkObjectBase`的类的资源，类似 `std::shared_ptr` | `New()`<br>`Delete()` |

**代码示例**：显示圆锥体
```C++
#include <vtkConeSource.h>
#include <vtkPolyDataMapper.h>
#include <vtkActor.h>
#include <vtkRenderer.h>
#include <vtkRenderWindow.h>
#include <vtkRenderWindowInteractor.h>
#include <vtkSmartPointer.h>
#include <vtkProperty>

int main() {
    // 数据源
    vtkSmartPointer<vtkConeSource> cone = vtkSmartPointer<vtkConeSource>::New();
    cone->SetHeight(3.0);
    cone->SetRadius(1.0);
    cone->SetResolution(36);

    // 映射器
    vtkSmartPointer<vtkPolyDataMapper> mapper = vtkSmartPointer<vtkPolyDataMapper>::New();
    mapper->SetInputConnection(cone->GetOutputPort());

    // 演员
    vtkSmartPointer<vtkActor> actor = vtkSmartPointer<vtkActor>::New();
    actor->SetMapper(mapper);
    actor->GetProperty()->SetColor(0.2, 0.6, 0.8);  
    actor->GetProperty()->SetOpacity(0.8);          

    // 渲染器
    vtkSmartPointer<vtkRenderer> renderer = vtkSmartPointer<vtkRenderer>::New();
    renderer->AddActor(actor);
    renderer->SetBackground(0.1, 0.2, 0.4);         

    // 渲染窗口
    vtkSmartPointer<vtkRenderWindow> window = vtkSmartPointer<vtkRenderWindow>::New();
    window->AddRenderer(renderer);
    window->SetSize(800, 600);
    window->SetWindowName("VTK Cone Example");

    // 交互器
    vtkSmartPointer<vtkRenderWindowInteractor> interactor = vtkSmartPointer<vtkRenderWindowInteractor>::New();
    interactor->SetRenderWindow(window);

    // 开始渲染
    window->Render();
    interactor->Start();

    return 0;
}
```


