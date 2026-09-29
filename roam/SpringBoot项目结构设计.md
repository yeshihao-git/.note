---
tags:
  - java
  - springboot
---
# SpringBoot项目结构设计

Spring Boot 项目的结构设计，没有唯一标准，但有几套业界比较公认的**分层/分模块模式**。选哪种，主要看项目规模、团队人数和业务复杂度。下面从简单到复杂梳理几种主流方案。

## 一、传统三层架构

适合小型项目、单体应用、Demo。

```
src/main/java/com/example/demo/
├── controller/      // 控制层：接收请求，参数校验，返回响应
├── service/         // 业务层：业务逻辑
│   └── impl/
├── dao/ 或 mapper/  // 数据访问层（MyBatis 用 mapper / dao，JPA 用 repository）
├──────────────────────────────────────────────────────────────────────────────
├── entity/ 或 model/ // 数据库实体
├── dto/             // 数据传输对象（入参/出参）
├── vo/              // 视图对象（返回给前端）
├── config/          // 配置类
├── common/          // 通用工具、常量、统一响应
└── exception/       // 全局异常处理
```

**特点：** 按“技术角色”分包，简单直观。
**缺点：** 业务一多，`controller`、`service` 会堆满各种不相关的类，改一个功能要跳好几个包。

## 二、按业务功能分包（Package by Feature）架构

适合中大型项目，是目前比较推荐的做法。

```
com.example.demo/
├── user/                    // 用户模块
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── entity/
│   └── dto/
├── order/                   // 订单模块
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── entity/
│   └── dto/
├── product/                 // 商品模块
└── common/                  // 跨模块通用
    ├── config/
    ├── exception/
    ├── response/
    └── util/
```

**特点：** 每个业务模块自包含，高内聚，改一个功能基本只动一个包。
**优点：** 便于拆分微服务，团队可按模块分工。

## 三、领域驱动设计（DDD）四层结构

适合业务复杂、领域模型重的项目。

```
com.example.demo/
├── interfaces/ 或 adapter/    // 用户接口层：Controller、DTO、Assembler
├── application/               // 应用层：应用服务、编排、事务
├── domain/                    // 领域层：实体、值对象、领域服务、仓储接口
│   ├── model/
│   ├── service/
│   └── repository/
└── infrastructure/            // 基础设施层：仓储实现、外部调用、配置
    ├── persistence/
    ├── client/
    └── config/
```

**特点：** 领域层不依赖框架，业务逻辑最纯粹。
**缺点：** 学习成本高，简单 CRUD 项目用它会过度设计。

## 四、常见目录补充说明

| 目录                          | 作用                               |
| --------------------------- | -------------------------------- |
| `controller`                | 只做参数校验和转发，不写业务                   |
| `service`                   | 业务逻辑，事务边界一般在这层                   |
| `dao`/`mapper`/`repository` | 只做数据库操作                          |
|                             |                                  |
| `entity`                    | 与数据库表一一对应                        |
| `dto`                       | 接收前端参数                           |
| `vo`                        | 返回给前端的对象                         |
| `convert`/`assembler`       | entity、dto、vo 之间转换（MapStruct 常用） |
| `config`                    | 各种 Bean 配置                       |
| `common`                    | 统一响应、常量、枚举、工具                    |
| `exception`                 | 全局异常 + 自定义异常                     |
