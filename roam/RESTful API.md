---
tags:
  - 计算机网络
---
# REST 思想

1. URI 表示 **资源**
```
/users/123/orders：表示 “ID 123 的用户名下的所有订单”（订单是用户的子资源）
/orders/456      ：表示 “ID 456 的单个订单”（独立资源）
```

2. HTTP 请求方法 表示 **对资源的操作**
3. HTTP状态码 表示 **操作结果**
4. **无状态**（服务端不保存客户端信息）

## URI

**what：**
 统一资源标识符，用于 标识资源 ；所有的 URL 都是 URI，反之不成立
```sh
ISBN:978-3-16-148410-0             # 这是一本书的URN，也是URI
tel:+1-800-555-1234                # 这是一个电话号码的URI
urn:isbn:0451450523                # 这是一个URN，也是URI
https://www.example.com/index.html # 这是一个URL，同时也是URI
```

## URL

**what：**
统一资源定位符，用于 定位、访问资源
```sh
# URL 形式：访问协议（http、ftp等）+ 位置
https://www.example.com/index.html # 协议 + 域名 + 路径
ftp://example.com/files/file.zip   # 协议 + 地址 + 路径
```

# RESTful API

**what：**
REST 是思想，RESTful 是符合这个思想的接口；具体来说使用 [[HTTP]] 协议 管理资源

## URL 设计规范

1. URL 只放**名词**（资源），且用**复数**
2. **小写 + 横短线分隔**（驼峰式不推荐）

```sh
/vtp-models
```

## HTTP 请求方法规范

用请求方式区分操作：
- `GET`：**查询**资源
- `POST`：**新增**资源
- `PUT`：**全量修改**资源
- `DELETE`：**删除**资源

```sh
POST   /vtpModels       # 上传VTP模型
GET    /vtpModels       # 查询模型列表
GET    /vtpModels/{id}  # 查询单个模型详情
DELETE /vtpModels/{id}  # 删除模型
```

## HTTP 状态码规范

使用原生状态码标识请求结果：
- `2xx`：成功
- `4xx`：客户端错误
- `5xx`：服务器错误

## 无状态

服务器**不保存客户端状态**，身份信息（Token）放在请求头传输，不依赖会话

# JSON 设计
## 请求 JSON

无统一标准

## 响应 JSON

统一设计
```json
/* 字段规范 */
// 1. 小驼峰命名，禁止下划线
// 2. 布尔类型字段，is 开头（isValid）

/* 空值规范 */
// 1. 空对象：{}
// 2. 空数组：[]
// 3. 空数据：null

{
  "code": 200,       // HTTP 状态码，用于表示 操作结果
  "msg": "操作成果",  // 提示信息，给开发人员看
  "data": {}         // 业务数据，无数据返回 null
}
```


