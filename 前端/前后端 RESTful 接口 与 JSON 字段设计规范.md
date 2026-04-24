# 1. REST 思想
1. **URL 只用来定位资源**，不写动作
2. **HTTP 请求方法代表操作**
3. **无状态**

REST 是思想，RESTful 是符合这个思想的接口

# 2. RESTful 接口规范
## 2.1 URL 设计规范
1. URL 只放名词（资源），且用**复数**
2. 小写 + 横短线分隔（驼峰式不推荐）

---

**示例**：`/vtp-models`

## 2.2 HTTP 请求方法规范
用请求方式区分操作：
- `GET`：**查询**资源
- `POST`：**新增**资源
- `PUT`：全量修改资源
- `PATCH`：局部修改资源
- `DELETE`：删除资源

---

**示例**：
```
POST   /vtpModels       上传VTP模型
GET    /vtpModels       查询模型列表
GET    /vtpModels/{id}  查询单个模型详情
DELETE /vtpModels/{id}  删除模型
```

## 2.3 HTTP 状态码规范
使用原生状态码标识请求结果：
- `2xx`：成功
- `4xx`：客户端错误
- `5xx`：服务器错误

## 2.4 JSON 后端返回统一格式
固定三层结构
```json
{
  "code": 200,
  "msg": "接口信息",
  "data": {}
}
```

## 2.5 无状态约束
服务器**不保存客户端状态**，身份信息（Token）放在请求头传输，不依赖会话。

# 3. JSON 字段设计规范
1. 字段：小驼峰，布尔类型统一 is 开头（filePath、isValid）