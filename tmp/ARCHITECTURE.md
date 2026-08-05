# MiniEngine 架构说明

这个项目实现了一个最小 WebGL2 渲染引擎，用 TypeScript 拆分为 Core、Renderer、Scene、Geometry、Math 和 Asset 六层。当前示例会在全屏 canvas 中显示一个持续旋转的六面体。

## 数据流图

```mermaid
flowchart TD
  A["main.ts 创建 Engine/Scene/Camera/Mesh"] --> B["Engine.start()"]
  B --> C["requestAnimationFrame"]
  C --> D["Time.update()"]
  D --> E["示例逻辑更新 cube.rotation"]
  E --> F["WebGLRenderer.render(scene, camera)"]
  F --> G["Scene.updateMatrixWorld()"]
  F --> H["Camera.updateViewMatrix()"]
  G --> I["遍历 Scene 查找 Mesh"]
  I --> J["Geometry 顶点/索引数据"]
  J --> K["WebGLRenderer 上传 Buffer + VertexArray"]
  I --> L["Material.apply()"]
  L --> M["Program.use() + uniform 上传"]
  K --> N["gl.drawElements()"]
  M --> N
  N --> O["Canvas 显示旋转六面体"]
```

## 类图

```mermaid
classDiagram
  class Engine {
    +Time time
    +start() void
    +stop() void
  }

  class Time {
    +number elapsed
    +number delta
    +update(now) void
  }

  class Input {
    +isKeyDown(code) boolean
    +dispose(target) void
  }

  class WebGLRenderer {
    +WebGL2RenderingContext gl
    +setPixelRatio(pixelRatio) void
    +setSize(width, height) void
    +render(scene, camera) void
  }

  class Shader {
    +WebGLShader handle
    +dispose() void
  }

  class Program {
    +WebGLProgram handle
    +use() void
    +setMatrix4(name, matrix) void
    +setVector3(name, value) void
    +dispose() void
  }

  class Buffer {
    +WebGLBuffer handle
    +bind() void
    +dispose() void
  }

  class VertexArray {
    +WebGLVertexArrayObject handle
    +bind() void
    +setAttributes(attributes) void
    +unbind() void
    +dispose() void
  }

  class Texture {
    +WebGLTexture handle
    +uploadImage(image) void
    +bind(unit) void
    +setDefaultParameters() void
    +dispose() void
  }

  class Material {
    +Program program
    +apply(modelMatrix, viewMatrix, projectionMatrix) void
  }

  class Object3D {
    +Vector3 position
    +Vector3 rotation
    +Quaternion quaternion
    +Vector3 scale
    +Matrix4 matrix
    +Matrix4 worldMatrix
    +add(child) Object3D
    +remove(child) Object3D
    +traverse(callback) void
    +updateMatrix() void
    +updateMatrixWorld(force) void
  }

  class Scene
  class Camera {
    +Matrix4 projectionMatrix
    +Matrix4 viewMatrix
    +updateProjectionMatrix() void
    +updateViewMatrix() void
  }

  class Mesh {
    +Geometry geometry
    +Material material
  }

  class Geometry {
    +Float32Array vertices
    +Uint16Array indices
  }

  class BoxGeometry
  class Vector3
  class Matrix4
  class Quaternion
  class TextureLoader {
    +load(url) Promise~Texture~
  }

  Engine --> Time
  WebGLRenderer --> Scene
  WebGLRenderer --> Camera
  WebGLRenderer --> Mesh
  WebGLRenderer --> Buffer
  WebGLRenderer --> VertexArray
  Program --> Shader
  Material --> Program
  Object3D --> Vector3
  Object3D --> Quaternion
  Object3D --> Matrix4
  Scene --|> Object3D
  Camera --|> Object3D
  Mesh --|> Object3D
  Mesh --> Geometry
  Mesh --> Material
  BoxGeometry --|> Geometry
  TextureLoader --> Texture
```

## 渲染链路

1. `Engine` 驱动主循环，每帧更新 `Time`。
2. 示例逻辑根据 `time.delta` 更新 `cube.rotation`。
3. `WebGLRenderer.render()` 更新场景和相机矩阵。
4. `WebGLRenderer` 遍历 `Scene`，找到所有 `Mesh`。
5. `Geometry` 的顶点和索引数据被上传到 GPU，并缓存为 `Buffer` 和 `VertexArray`。
6. `Material` 启用 `Program`，上传模型、视图、投影矩阵和颜色。
7. `gl.drawElements()` 输出最终画面。

## 目录职责

- `Core`：主循环、时间和输入。
- `Renderer`：WebGL2 API 封装、GPU 资源和材质。
- `Scene`：场景树、对象变换、相机和 Mesh。
- `Geometry`：CPU 侧几何数据。
- `Math`：向量、矩阵和四元数。
- `Asset`：资源加载入口，目前包含纹理加载器。
