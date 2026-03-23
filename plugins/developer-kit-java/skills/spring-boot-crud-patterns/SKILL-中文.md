---
name: spring-boot-crud-patterns
description: 为Spring Boot 3服务提供可复用的CRUD工作流，使用Spring Data JPA和面向功能的架构。在建模聚合、仓库、控制器和REST API的DTO时使用。
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Spring Boot CRUD模式

## 概述

提供面向功能的CRUD服务，分离领域、应用、表示和基础设施层，同时保持Spring Boot 3.5+约定。此技能提炼了基本工作流，并将详细代码清单推迟到参考文件以实现渐进式披露。

## 何时使用

- 为使用Spring Data JPA支持的REST端点实现创建/读取/更新/删除工作流。
- 使用聚合、仓库和应用程序服务，遵循DDD启发的架构来优化功能包。
- 为外部客户端引入DTO记录、请求验证和控制器映射。
- 诊断现有Spring Boot服务中的CRUD回归、仓库契约或事务边界。
- 触发短语：**"实现Spring CRUD控制器"**、**"优化基于功能的仓库"**、**"为JPA聚合映射DTO"**、**"向REST列表端点添加分页"**。

## 操作说明

按照这些步骤实现面向功能的CRUD服务：

### 1. 建立功能包结构

创建 feature/<name>/ 目录，包含 domain、application、presentation 和 infrastructure 子包，以保持架构边界。

### 2. 定义领域模型

创建实体类，通过工厂方法（create、update）强制执行不变量。保持领域逻辑无框架，不使用Spring注解。

### 3. 声明仓库接口

在 domain/repository 中定义描述持久化契约的领域仓库接口，不包含实现细节。

### 4. 实现基础设施适配器

在 infrastructure/persistence 中创建映射到领域模型的JPA实体。实现委托给领域接口的Spring Data仓库。

### 5. 构建应用程序服务

创建带有 `@`Transactional 方法的 `@`Service 类，协调领域操作、仓库访问和DTO映射。

### 6. 定义请求/响应DTO

对API契约使用Java记录或不可变类。应用jakarta.validation注解进行输入验证。

### 7. 暴露REST控制器

创建映射到 /api 端点的 `@`RestController 类。返回带有适当状态码的ResponseEntity（POST为201，DELETE为204）。

### 8. 编写测试

对领域逻辑进行单元测试。使用 `@`DataJpaTest 和 Testcontainers 对持久化层进行集成测试。

## 前置条件

- 使用Spring Boot 3.5.x（或更高版本）和 `spring-boot-starter-web` 和 `spring-boot-starter-data-jpa` 的Java 17+项目。
- 启用构造函数注入（Lombok `@RequiredArgsConstructor` 或显式构造函数）。
- 访问关系数据库（建议集成测试使用Testcontainers）。
- 熟悉验证（`jakarta.validation`）和错误处理（`ResponseStatusException`）。

## 快速入门工作流

1. **建立功能结构**  
   为 `domain`、`application`、`presentation` 和 `infrastructure` 创建 `feature/<name>/` 目录。
2. **建模聚合**  
   定义无Spring依赖的领域实体和值对象；在 `create` 和 `update` 等方法中捕获不变量。
3. **暴露领域端口**  
   在 `domain/repository` 中声明描述持久化契约的仓库接口。
4. **提供基础设施适配器**  
   在 `infrastructure/persistence` 中实现Spring Data适配器，将领域模型映射到JPA实体并委托给 `JpaRepository`。
5. **实现应用程序服务**  
   在 `application/service` 下创建事务性用例，协调聚合、仓库和映射逻辑。
6. **发布REST控制器**  
   在 `presentation/rest` 下映射DTO记录，用适当的状态码暴露端点，并连接验证注解。
7. **用测试验证**  
   运行领域逻辑的单元测试，使用Testcontainers对仓库/服务进行集成测试以验证持久化。

参考 `references/examples-product-feature.md` 获取与每个步骤对齐的完整代码清单。

## 实现模式

### 领域层

- 使用工厂方法（`Product.create`）定义不可变聚合，以集中不变量。
- 使用值对象（`Money`、`Stock`）强制执行类型安全并封装验证。
- 保持领域对象无框架；使用适配器时避免在领域包中使用 `@Entity` 注解。

### 应用层

- 在 `@Service` 类中使用构造函数注入和 `@Transactional` 包装用例。
- 将请求映射到领域操作，并通过领域仓库持久化。
- 返回由专用映射器生成的响应DTO或记录，以将领域与传输解耦。

### 基础设施层

- 实现聚合和JPA实体之间的适配器；为清晰起见，优先选择MapStruct或手动映射器。
- 使用Spring Data接口配置仓库（例如，`JpaRepository<ProductEntity, String>`），并为分页或批量更新定制查询。
- 通过 `application.yml` 外部化持久化属性（命名策略、DDL模式）；参见 `references/spring-official-docs.md`。

### 表示层

- 按功能结构化控制器（`ProductController`）并暴露REST路径（`/api/products`）。
- 返回带有适当代码的 `ResponseEntity`：POST返回 `201 Created`，GET/PUT/PATCH返回 `200 OK`，DELETE返回 `204 No Content`。
- 在请求DTO上应用 `@Valid` 并用 `@ControllerAdvice` 或 `ResponseStatusException` 处理错误。

## 验证和可观察性

- 编写单元测试以断言领域不变量和仓库契约；参考 `references/examples-product-feature.md` 集成测试片段。
- 使用 `@DataJpaTest` 和Testcontainers验证持久化映射、分页和批量操作。
- 通过Spring Boot Actuator暴露健康和指标；监控CRUD吞吐量和错误率。
- 在 `info` 级别记录关键操作的生命周期事件（创建、更新、删除），并对审计跟踪使用结构化日志记录。

## 示例

### 输入：产品创建请求

```json
{
  "name": "无线键盘",
  "description": "带背光的人体工程学无线键盘",
  "price": 79.99,
  "stock": 50
}
```

### 输出：创建的产品响应

```json
{
  "id": "prod-123",
  "name": "无线键盘",
  "description": "带背光的人体工程学无线键盘",
  "price": 79.99,
  "stock": 50,
  "createdAt": "2024-01-15T10:30:00Z",
  "_links": {
    "self": "/api/products/prod-123",
    "update": "/api/products/prod-123",
    "delete": "/api/products/prod-123"
  }
}
```

### 输入：产品更新请求

```json
{
  "price": 69.99,
  "stock": 45
}
```

### 输出：更新的产品响应

```json
{
  "id": "prod-123",
  "name": "无线键盘",
  "description": "带背光的人体工程学无线键盘",
  "price": 69.99,
  "stock": 45,
  "updatedAt": "2024-01-15T11:45:00Z",
  "_links": {
    "self": "/api/products/prod-123"
  }
}
```

### 输入：删除产品

```bash
curl -X DELETE http://localhost:8080/api/products/prod-123
```

### 输出：204无内容

```
HTTP/1.1 204 No Content
```

### 输入：带分页的产品列表

```bash
curl "http://localhost:8080/api/products?page=0&size=10&sort=name,asc"
```

### 输出：分页产品列表

```json
{
  "content": [
    {
      "id": "prod-123",
      "name": "无线键盘",
      "price": 69.99,
      "stock": 45
    },
    {
      "id": "prod-456",
      "name": "无线鼠标",
      "price": 29.99,
      "stock": 100
    }
  ],
  "pageable": {
    "page": 0,
    "size": 10,
    "total": 2,
    "totalPages": 1
  },
  "_links": {
    "self": "/api/products?page=0&size=10",
    "next": null,
    "last": "/api/products?page=0&size=10"
  }
}
```

## 最佳实践

- 优先使用边界清晰的功能模块；按聚合将领域、应用和表示代码放在一起。
- 通过Java记录保持DTO不可变；在服务边界转换领域类型。
- 在并发重要的地方用事务和乐观锁保护写操作。
- 规范化分页默认值（页面、大小、排序）并记录查询参数。
- 在需要与消息传递或审计集成的地方捕获命令和事件之间的链接。

## 约束和警告

- 避免直接在控制器中暴露JPA实体，以防止延迟加载泄漏和序列化问题。
- 不要将字段注入与构造函数注入混用；保持不可变性以便于测试。
- 避免在控制器或仓库适配器中嵌入业务逻辑；将其保留在领域/应用层中。
- 积极验证输入，以防止约束违规并产生一致的错误有效负载。
- 在部署模式更改之前，确保迁移（Liquibase/Flyway）镜像聚合演进。

## 参考资料

- [HTTP方法矩阵、注解目录、DTO模式。](references/crud-reference.md)
- [从入门到高级功能实现的渐进式示例。](references/examples-product-feature.md)
- [官方Spring指南和Spring Boot参考文档的摘录。](references/spring-official-docs.md)
- [Python生成器，用于从实体规范搭建CRUD样板。](scripts/generate_crud_boilerplate.py) 用法：`python skills/spring-boot-crud-patterns/scripts/generate_crud_boilerplate.py --spec entity.json --package com.example.product --output ./generated`
- 所需模板：将 .tpl 文件放在 `skills/spring-boot-crud-patterns/references/` 中或传递 `--templates-dir <path>`；没有内置回退。参见 `references/README.md`。
- 使用指南：[references/generator-usage.md](references/generator-usage.md)
- 示例规范：`skills/spring-boot-crud-patterns/assets/specs/product.json`
- 关系示例：`skills/spring-boot-crud-patterns/assets/specs/product_with_rel.json`