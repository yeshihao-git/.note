# 1 VTK 渲染管线 流程
本节介绍 VTK 中最核心的渲染管线流程：从数据处理到可视化的完整流程及各个环节涉及的组件。
![[Pasted image 20260409182138.png]]

| 组件                   | 作用                                      |
| -------------------- | --------------------------------------- |
| `Source`（数据源）        | 提供数据（创建或读取）                             |
| `Filters`（过滤器）       | 对数据进行算法处理（剪裁、平移、旋转等）                    |
| `Mapper`（映射器）        | 数据→图元（渲染管线、可视化管线的交汇点）                   |
| `Actor`（实体）          | 场景中的一个物体，绑定`Mapper`和`Property`（形状和外观）   |
| `Renderer`（渲染器）      | 管理`Light`、`Camera`、`Actor`（持有多个`Actor`） |
| `RenderWindow`（渲染窗口） | 连接浏览器`DOM`的容器，如`Canvas`（持有多个`Renderer`） |
| `Interactor` (交互器)   | 监听鼠标、键盘事件，更新相机位置或物体状态                   |

# 2 VTK 重要概念
## 2.1 VTK 内部索引

# 3 了解 VTK 支持的文件格式以及对应 Reader
本节介绍 VTK 中常用的格式，包括：VTK 原生格式、部分外部数据格式；
其次介绍对应的 Reader，在 VTK（C++）中 Reader 类的命名通常遵循 `vtk[格式名]Reader`。

## 3.1 VTK 原生格式
VTK 原生格式是 VTK 自有的专用格式，性能和功能最佳。可以分为：
1. Legacy：老牌，不支持并行IO，对大规模数据集的加载速度较慢
2. XML：主流，支持**随机访问**、**块压缩**、**并行IO**
3. VTKHDF：前沿，VTK 近几年提出的基于 HDF5 标准的新格式，速度更快，性能更好

鉴于 XML 为目前的主流格式，这里只介绍 XML

VTK XML 文件可从两个维度划分：
1. 按**读写方式**划分
    - **串行**：所有数据存于**单个文件**，由**单进程**读写。
	- **并行**：数据拆分为多个 **Piece（块）**，分别存在多个串行文件中；并行文件本身不存数据，只做索引与结构描述，供**多进程**分别读写一个或多个数据块。

2. 按**数据拓扑**划分
	- **结构化数据集**：拓扑规则，通过 **Extent（范围）** 描述矩形子区域。
	- **非结构化数据集**：拓扑不规则，通过 **Piece（块）** 描述子区域。

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
### 3.1.1 VTK 原生格式可视化示例
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

--- 

**参考**：[VTK XML file formats](https://docs.vtk.org/en/latest/vtk_file_formats/vtkxml_file_format.html#)

## 3.2 外部数据格式
外部数据格式是外部软件生成的通用或专用的数据格式，用于数据交换。

| 类型             | 扩展名    | 对应的Reader      |
| -------------- | ------ | -------------- |
| STL（立体光刻）      | `.stl` | `vtkSTLReader` |
| PLY（多边形文件格式）   | `.ply` | `vtkPLYReader` |
| OBJ（Wavefront） | `.obj` | `vtkOBJReader` |

# 4 快速开始
本节通过一个剖切代码示例，来介绍 VTK 渲染管线流程、reader、剖切、切面等内容。

## 4.1 3.1项目结构
```
.
├── CMakeLists.txt
└── CapClip.cxx
```

## 4.2 编写 CMakeLists.txt 
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

## 4.3 编写代码
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

## 4.4 编译运行
```bash
cmake -B build -G Ninja
cmake --build build
./build/vtk_cpp
```

最终效果
![[Pasted image 20260415162645.png|355]]

--- 

**参考**：[vtk-examples CapClip](https://examples.vtk.org/site/Cxx/Meshes/CapClip/#download-and-build-capclip)

# 5 常用 API 接口说明
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

---

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

---

**参考**：[vtk-examples](https://examples.vtk.org/site/Cxx/) [vtk reference](https://vtk.org/doc/nightly/html/classes.html)