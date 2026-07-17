---
tags:
  - 图形学
---
# 管线流程

$$
\boldsymbol{模型坐标}
\xrightarrow{M}
\boldsymbol{世界坐标}
\xrightarrow{V}
\boldsymbol{相机坐标}
\xrightarrow{P}
\boldsymbol{裁剪坐标}
\xrightarrow{齐次(透视)除法}
\boldsymbol{标准化设备坐标}
\xrightarrow{视口变换}
\boldsymbol{屏幕像素坐标}
$$
整个管线流程涉及 MVP 变换和视口变换，其目标是将模型坐标（3D）转换为屏幕像素坐标（2D）。其中坐标与空间对应关系：

| 坐标           | 空间          | 说明                                         |
| ------------ | ----------- | ------------------------------------------ |
| 模型坐标         | 模型空间 / 局部空间 |                                            |
| 世界坐标         | 世界空间        | 3D                                         |
| 相机坐标 / 视图坐标  | 相机空间 / 视图空间 |                                            |
| 裁剪坐标         | 裁剪空间        | 齐次坐标 (x, y, z, w) w != 1；；x、y、z坐标范围[-w, w] |
| 标准化设备坐标（NDC） | 标准化设备空间     | 齐次坐标 (x/w, y/w, z/w, 1)；x、y、z坐标范围[-1, 1]   |
| 屏幕像素坐标       | 屏幕窗口空间      | 2D；[0, 窗口视口宽] x [0, 窗口视口高]                 |

# MVP 变换和视口变换

MVP 变换和视口变换为管线流程中涉及到的变换

## 模型变换公式

模型坐标系(Model Space) 转换到 世界坐标系(World Space)：

$$
P_{world}=M_{model}P_{model}
$$

其中：
- $P_{model}$：模型空间坐标
- $P_{world}$：世界空间坐标
- $M_{model}$：模型矩阵

## 齐次坐标

三维坐标转换为齐次坐标：
$$
P=
\begin{bmatrix}
x\\
y\\
z\\
1
\end{bmatrix}
$$

## M = Model 模型变换

**what**：
对模型坐标（模型空间自身局部原点）进行仿射变换后转换为世界坐标（世界空间中的某个位置）

>**线性变换**：旋转 + 缩放（旋转、缩放是保持坐标原点不动的变换，平移是动原点的，因此旋转、缩放、平移操作无法统一，解决方式为通过 4 维齐次坐标将这些变换统一为矩阵乘法操作）
>
>**仿射变换**：线性变换（旋转 + 缩放）+ 平移

**how**：
模型矩阵由：
$$
M_{model}=TRS
$$

组成：
- $S$：缩放矩阵 Scale
- $R$：旋转矩阵 Rotation
- $T$：平移矩阵 Translation

实际执行顺序：
$$
P_{world}=T \times R \times S \times P_{model}
$$

即：
$$
P_{model}
\rightarrow
Scale
\rightarrow
Rotate
\rightarrow
Translate
\rightarrow
P_{world}
$$

### 缩放矩阵 Scale

$$
S=
\begin{bmatrix}
S_x&0&0&0\\
0&S_y&0&0\\
0&0&S_z&0\\
0&0&0&1
\end{bmatrix}
$$

作用：
$$
\begin{bmatrix}
x'\\
y'\\
z'\\
1
\end{bmatrix}
=
S
\begin{bmatrix}
x\\
y\\
z\\
1
\end{bmatrix}
$$

结果：
$$
\begin{cases}
x'=S_xx\\
y'=S_yy\\
z'=S_zz
\end{cases}
$$

### 平移矩阵 Translation

$$
T=
\begin{bmatrix}
1&0&0&T_x\\
0&1&0&T_y\\
0&0&1&T_z\\
0&0&0&1
\end{bmatrix}
$$

作用：
$$
P'=TP
$$

结果：
$$
\begin{cases}
x'=x+T_x\\
y'=y+T_y\\
z'=z+T_z
\end{cases}
$$

### 旋转矩阵 Rotation
#### 绕X轴旋转

$$
R_x=
\begin{bmatrix}
1&0&0&0\\
0&\cos\theta&-\sin\theta&0\\
0&\sin\theta&\cos\theta&0\\
0&0&0&1
\end{bmatrix}
$$

#### 绕Y轴旋转

$$
R_y=
\begin{bmatrix}
\cos\theta&0&\sin\theta&0\\
0&1&0&0\\
-\sin\theta&0&\cos\theta&0\\
0&0&0&1
\end{bmatrix}
$$

#### 绕Z轴旋转

$$
R_z=
\begin{bmatrix}
\cos\theta&-\sin\theta&0&0\\
\sin\theta&\cos\theta&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

## V = View 视图变换

**what**：
将相机移到世界空间的原点（世界坐标 -> 相机坐标），看向 -z 方向

**why**：
将世界坐标系（World Space）转换为相机坐标系（Camera Space）

**how**：
$$
P_{view}=M_{view}P_{world}
$$

其中：
- $P_{world}$：世界坐标
- $P_{view}$：相机坐标
- $M_{view}$：视图矩阵

### 视图矩阵
#### LookAt 相机参数
视图矩阵通常由：
$$
M_{view}=LookAt(Eye,Target,Up)
$$

三个参数决定：
- $Eye$：相机位置（Camera Position）
- $Target$：相机观察目标点（Look At Point）
- $Up$：相机上方向

#### 计算观察方向 Forward

观察方向：
$$
F=\frac{Target-Eye}{||Target-Eye||}
$$

其中：
$$
||F||=\sqrt{x^2+y^2+z^2}
$$
表示向量长度

#### 计算相机坐标轴
##### Right轴（相机X轴）

$$
R=
\frac{F\times Up}
{||F\times Up||}
$$

其中：
$\times$ 表示叉积

##### Up轴（相机Y轴）
$$
U=R\times F
$$

##### Forward轴（相机Z轴）
OpenGL中相机朝向负Z方向：
$$
D=-F
$$

因此相机坐标系：
$$
\begin{cases}
X_{camera}=R\\
Y_{camera}=U\\
Z_{camera}=D
\end{cases}
$$

#### View Matrix构造

视图矩阵：
$$
M_{view}
=
\begin{bmatrix}
R_x&R_y&R_z&-R\cdot Eye\\
U_x&U_y&U_z&-U\cdot Eye\\
D_x&D_y&D_z&-D\cdot Eye\\
0&0&0&1
\end{bmatrix}
$$

其中：
- $R\cdot Eye$：Right方向与相机位置的点积
- $U\cdot Eye$：Up方向与相机位置的点积
- $D\cdot Eye$：Forward方向与相机位置的点积

##### View Matrix的组成

View Matrix实际上由两部分组成：
$$
M_{view}=R_{camera}T_{camera}
$$

其中：

###### 平移矩阵

将世界移动到相机位置：
$$
T_{camera}
=
\begin{bmatrix}
1&0&0&-E_x\\
0&1&0&-E_y\\
0&0&1&-E_z\\
0&0&0&1
\end{bmatrix}
$$

###### 旋转矩阵

将世界坐标轴旋转到相机坐标轴：
$$
R_{camera}
=
\begin{bmatrix}
R_x&R_y&R_z&0\\
U_x&U_y&U_z&0\\
D_x&D_y&D_z&0\\
0&0&0&1
\end{bmatrix}
$$

## P = Projection 投影变换

**what**：
将相机视野范围的视锥体转换为坐标范围[-w, w]的立方体。后续通过**齐次（透视）除法**转换为标准设备坐标，一个坐标范围[-1, 1]的立方体

> **视锥体**：
> ![[Pasted image 20260422103251.png]]

投影变换有两种方案：
1. 透视投影
2. 正交投影

### 透视投影

**what**：
近大远小，人眼效果，视锥体是四棱锥台
![[Pasted image 20260422114839.png|212]]

### 正交投影

**what**：
无视远近，物体大小不变，视锥体是长方体
![[Pasted image 20260422114852.png|217]]

## 视口变换 = Viewport

**what**：
将范围为 [-1, 1] 的标准设备坐标通过仿射变换映射到窗口视口内的坐标范围为 [0, 视口宽] x [0, 视口高] 屏幕像素坐标

> **视口**：屏幕上观察画面的窗口区域（例如：非全屏打开游戏窗口，显示 3D 游戏画面的那块矩形区域）

# Framebuffer（帧缓冲）

**what**：
Framebuffer（帧缓冲）是 GPU 中用于存储渲染结果的对象；
GPU 不会直接把图像绘制到屏幕，而是先将结果写入 Framebuffer；
Framebuffer 由多个缓冲区（Buffer Attachment）组成：
$$
Framebuffer =
Color\ Buffer +
Depth\ Buffer +
Stencil\ Buffer
$$

```
Framebuffer

+-------------------+
| Color Buffer      |
|                   |
| 保存像素颜色       |
+-------------------+

+-------------------+
| Depth Buffer      |
|                   |
| 保存深度信息       |
+-------------------+

+-------------------+
| Stencil Buffer    |
|                   |
| 保存模板信息       |
+-------------------+
```

## Color Buffer（颜色缓冲区）

**what**：
Color Buffer 保存最终显示的像素颜色；
每个像素通常保存 RGBA：
$$
Pixel=(R,G,B,A)
$$

例如：
```
Pixel(100,200)

R = 255
G = 0
B = 0
A = 255
```
表示该像素为红色；
最终显示到屏幕上的图像来自 Color Buffer

## Depth Buffer（深度缓冲区）

**what**：
Depth Buffer 保存每个像素距离摄像机的深度，也称 $Z-Buffer$

例如：
```
Camera

近物体:
Depth = 0.2

远物体:
Depth = 0.8
```
GPU 根据深度判断哪个物体应该显示
深度测试：
$$
NewDepth < StoredDepth
$$
如果通过：
- 更新 Color Buffer
- 更新 Depth Buffer

如果失败：
- 丢弃 Fragment

## Stencil Buffer（模板缓冲区）

**what**：
Stencil Buffer 用于保存一个整数标记，它可以控制哪些区域允许渲染

例如：模板
```
Stencil Buffer


0 0 0 0

0 1 1 0

0 1 1 0

0 0 0 0
```
只允许：
$$
Stencil=1
$$
的区域通过

常见用途：
- 镜面效果
- 阴影
- 局部渲染
- 剪裁区域