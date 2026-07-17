---
tags:
  - 图形学
  - opengl
  - 前端
---
# WebGL2

## 右手坐标系

**what**：
是 OpenGL、WebGL 的默认坐标系
* X轴 ：朝屏幕右边
* Y轴 ：朝屏幕上方
* Z轴 ：朝屏幕外面（指向观察者）
* -Z轴：朝屏幕里面（远离观察者）

```
                 +Y
                  ↑
                  |
                  |
                  |
                  O────────→ +X
                 /
                /
               /
             +Z

        观察者在这里 👁
```

右手手指的对应关系：
```
拇指  → +Z
食指  → +X
中指  → +Y
```

## WebGL2 底层数据格式

下面总结 WebGL 底层数据格式。这部分建议你放到 **WebGL / GPU数据模型** 章节，因为它属于 API 层和 GPU 内存交互，不是纯图形学理论。

## 1. JavaScript 数据不能直接给 GPU

JavaScript 中：

```javascript
const vertices = [
  -1, -1, -1,
  1, -1, -1
];
```

这是普通 `Array`：

```
JavaScript Array
      |
      |
      ↓
   Number
```

但是 GPU 不认识 JS 的 Number。

GPU 需要：

```
TypedArray
      |
      |
      ↓
GPU Buffer
```

因此 WebGL 使用：

* Float32Array
* Uint16Array
* Uint32Array
* Uint8Array
* Int32Array 等

作为 CPU 到 GPU 的数据格式。

---

## 2. WebGL 数据流

完整流程：

```
JavaScript数据

(ArrayBuffer / TypedArray)

        |
        |
        ↓

WebGL Buffer

(VBO / EBO)

        |
        |
        ↓

Shader读取

        |
        |
        ↓

GPU执行
```

---

## 3. TypedArray类型

## 3.1 浮点数据

#### Float32Array

最常用。

对应 GPU：

```javascript
gl.FLOAT
```

例如顶点位置：

```javascript
const vertices = new Float32Array([
    -1,-1,-1,
     1,-1,-1,
     1, 1,-1
]);
```

用于：

* position
* normal
* color
* uv
* tangent

原因：

GPU 图形计算大量使用 32 位浮点数。

---

## 3.2 整数数据

### Uint16Array

16位无符号整数：

范围：

[
0 \sim 65535
]

常用于：

```javascript
indices
```

例如：

```javascript
const indices = new Uint16Array([
    0,1,2,
    0,2,3
]);
```

对应：

```javascript
gl.UNSIGNED_SHORT
```

---

### Uint32Array

32位索引。

范围：

[
0 \sim 4294967295
]

用于大型模型：

```javascript
const indices =
new Uint32Array([
    0,
    70000,
    100000
]);
```

对应：

```javascript
gl.UNSIGNED_INT
```

---

### Uint8Array

8位整数：

范围：

[
0\sim255
]

常用于：

颜色压缩：

```javascript
const color = new Uint8Array([
    255,
    0,
    0
]);
```

GPU中：

```glsl
vec3 color
```

可以转换：

```
255 -> 1.0
0   -> 0.0
```

通过：

```javascript
normalized=true
```

---

# 4. VBO数据格式

VBO：

Vertex Buffer Object

保存：

```
顶点属性数据
```

例如：

```javascript
const vertices =
new Float32Array([
    x,y,z,
    x,y,z,
    x,y,z
]);
```

上传：

```javascript
gl.bufferData(
    gl.ARRAY_BUFFER,
    vertices,
    gl.STATIC_DRAW
);
```

GPU：

```
VBO

+----------------+
| x y z          |
| x y z          |
| x y z          |
+----------------+

```

---

# 5. Attribute如何读取VBO

VBO只是：

```
一堆二进制数据
```

GPU不知道：

```
哪里是x？
哪里是y？
哪里是z？
```

需要：

```javascript
gl.vertexAttribPointer()
```

例如：

```javascript
gl.vertexAttribPointer(
    location,
    3,
    gl.FLOAT,
    false,
    0,
    0
);
```

告诉 GPU：

```
每3个float

组成一个vertex
```

数据：

```
VBO:

-1
-1
-1

1
-1
-1

1
1
-1
```

读取：

```
vertex0:

x=-1
y=-1
z=-1


vertex1:

x=1
y=-1
z=-1

```

---

# 6. EBO数据格式

EBO：

Element Buffer Object

保存：

```
顶点索引
```

例如：

```javascript
const indices =
new Uint16Array([
    0,1,2,
    0,2,3
]);
```

GPU：

```
EBO

0
1
2

0
2
3

```

绘制：

```javascript
gl.drawElements(
    gl.TRIANGLES,
    6,
    gl.UNSIGNED_SHORT,
    0
);
```

GPU：

```
读取EBO

0
 |
 ↓
VBO[0]


1
 |
 ↓
VBO[1]


2
 |
 ↓
VBO[2]

```

---

# 7. VBO和EBO的数据类型区别

| 对象  | 存储内容 | TypedArray   | draw类型            |
| --- | ---- | ------------ | ----------------- |
| VBO | 顶点数据 | Float32Array | gl.FLOAT          |
| EBO | 索引数据 | Uint16Array  | gl.UNSIGNED_SHORT |
| EBO | 索引数据 | Uint32Array  | gl.UNSIGNED_INT   |
| 颜色  | 颜色数据 | Uint8Array   | gl.UNSIGNED_BYTE  |

---

# 8. 一个完整顶点结构

实际模型通常不是只有 position：

例如：

```
Vertex

+-------------+
| position    |
| normal      |
| uv          |
| color       |
+-------------+
```

内存：

```
VBO


vertex0:

x y z
nx ny nz
u v


vertex1:

x y z
nx ny nz
u v

```

通常：

```javascript
Float32Array
```

存储：

```
position
normal
uv
```

---

# 9. WebGL底层数据本质

WebGL中：

```
所有数据
    |
    ↓

TypedArray

    |
    ↓

Buffer

    |
    ↓

GPU Memory

    |
    ↓

Shader

```

其中：

| 层级               | 作用       |
| ---------------- | -------- |
| JavaScript Array | 方便编程     |
| TypedArray       | 确定二进制格式  |
| Buffer           | GPU存储    |
| Attribute        | 解释Buffer |
| Shader           | 使用数据     |


```
VAO
 |
 +---- VBO绑定
 |
 +---- attribute配置
 |
 +---- EBO绑定
```

背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面






VAO
 |
 +---- VBO绑定
 |
 +---- attribute配置
 |
 +---- EBO绑定VAO
 |
 +---- VBO绑定
 |
 +---- attribute配置
 |
 +---- EBO绑定VAO
 |
 +---- VBO绑定
 |
 +---- attribute配置
 |
 +---- EBO绑定背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面背面剔除：摄像机方向看去，顶点按逆时针顺序排列的三角形是正面
