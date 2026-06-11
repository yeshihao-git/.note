---
tags:
  - java
---
# SpringBoot 核心概念

| 概念             | 类比 C++        | 说明                                 |
| -------------- | ------------- | ---------------------------------- |
| IoC 容器         | 全局对象管理器       | Spring 自动创建和管理对象（叫 Bean），不需要手动 new |
| 注解（Annotation） | 类似 C++ 属性 / 宏 | 以 @ 开头，给类 / 方法附加元数据，Spring 读取后自动配置 |
| 依赖注入（DI）       | 构造函数传参        | Spring 自动把需要的对象传给你，不用自己创建          |


# SpringBoot 常用注解
## SpringBoot 启动总注解

| 注解                       | 作用             | 内部三合一本质                                                                                 |
| ------------------------ | -------------- | --------------------------------------------------------------------------------------- |
| `@SpringBootApplication` | springboot启动入口 | 1.`@Configuration`配置<br>2.`@ComponentScan`自动扫包造Bean<br>3.`@EnableAutoConfiguration`自动配置 |

## 分层标记注解

| 注解            | 归属层         | 核心含义            | 底层本质             |
| ------------- | ----------- | --------------- | ---------------- |
| `@Component`  | 通用类         | 交给Spring IOC 管理 |                  |
| `@Service`    | 业务层 Service | 业务逻辑类           | **就是@Component** |
| `@Repository` | 数据层 DAO     | 数据库操作类          | **就是@Component** |
| `@Controller` | 控制层 Web     | 接收前端请求的类        | **就是@Component** |

## IOC & 配置相关注解

| 注解               | 作用    | 核心一句话                  |
| ---------------- | ----- | ---------------------- |
| `@Bean`          | 方法上注解 | 返回对象交给 Spring IOC 管理   |
| `@Autowired`     | 自动注入  | 自动从容器拿 Bean 赋值，不用自己new |
| `@Configuration` | 配置类注解 | 标记当前类是配置类，配合`@Bean`使用  |

## Web接口专用注解

| 注解                | 作用                                             |
| ----------------- | ---------------------------------------------- |
| `@RestController` | 接口控制器（返回JSON）= `@Controller` + `@ResponseBody` |
| `@ResponseBody`   | return 的内容直接作为 HTTP 响应体返回（JSON）                |
| `@RequestMapping` | 映射接口路径                                         |
| `@GetMapping`     | GET请求接口                                        |
| `@PostMapping`    | POST请求接口                                       |

## 如何使用？

```
启动用 @SpringBootApplication
配置用 @Configuration + @Bean
组件用 @Component / @Service / @Repository / @Controller
注入用 @Autowired
Web 用 @RestController + @GetMapping/@PostMapping
WebSocket 用 @ServerEndpoint + @OnMessage
事务用 @Transactional
```


# SpringBoot 标准后端项目目录结构

| 文件夹                           | 中文名称   | 核心作用                        |
| ----------------------------- | ------ | --------------------------- |
| `controller`                  | 控制器    | 接收前端请求、返回JSON               |
| `dto`                         | 数据传输对象 | 接口间数据传输的Java类（例如：前后端JSON交互） |
| `config`                      | 配置类    | 项目所有配置                      |
| `service`                     | 业务层    | 业务逻辑处理                      |
| `service/impl`                | 业务实现层  | service接口的具体实现代码            |
| `entity` / `domain` / `model` | 数据库实体  | 数据库实体映射                     |
| `mapper`                      | 数据访问层  | MyBatis 风格的数据访问层            |
| `vo`                          | 返回视图对象 | 专门封装返回给前端的视图对象              |
| `util`                        | 工具类    | 通用工具方法                      |
| `exception`                   | 全局异常处理 | 统一捕获报错                      |
| `common`                      | 公共模块   | 公共常量、枚举、通用类                 |

## SpringBoot 后端分层规范

| 对象        | 名称     | 作用           |
| --------- | ------ | ------------ |
| PO/Entity | 持久化对象  | 一对一映射数据库表    |
| DTO       | 数据传输对象 | 跨层 / 跨系统传输数据 |
| VO        | 视图对象   | 封装页面展示数据     |

# target 文件夹中有什么？

**target 文件夹**：Spring Boot / Maven 项目里自动生成的 编译产物输出目录。包含以下内容：
1. 最终可运行的 jar 包
```
target/demo-0.0.1-SNAPSHOT.jar
```

2. 编译后的 `.class` 
```
target/classes/
```

3. 配置文件
```bash
target/classes/application.yml # 配置文件
```

4. 静态资源
```bash
target/classes/static/         # 静态文件：html、js、图片等
target/classes/templates/
```

**与 target 相关的命令**：
```bash
mvn clean   # 删除 target 文件夹
mvn compile # 生成 target/classes
mvn package # 生成 可运行的 jar 包
```

