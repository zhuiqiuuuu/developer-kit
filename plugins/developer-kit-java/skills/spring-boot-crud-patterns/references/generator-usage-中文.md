# CRUD生成器使用指南

快速开始：

```
python skills/spring-boot-crud-patterns/scripts/generate_crud_boilerplate.py \
  --spec skills/spring-boot-crud-patterns/assets/specs/product.json \
  --package com.example.product \
  --output ./generated \
  --templates-dir skills/spring-boot-crud-patterns/templates [--lombok]
```

规范（JSON/YAML）：
- entity：帕斯卡命名法名称（例如，Product）
- id：{ name, type (Long|UUID|...), generated: true|false }
- fields：字段数组，包含 { name, type }
- relationships：可选（目前在字段中建模为外键id）

生成的内容：
- REST控制器位于 /v1/{resources}，POST 201 + Location头
- 可分页列表端点返回 PageResponse<T>
- 应用程序映射器（application/mapper/${Entity}Mapper）用于 DTO↔领域 转换
- 异常类型：${Entity}NotFoundException、${Entity}ExistException + ${Entity}ExceptionHandler
- GlobalExceptionHandler 处理验证 + DataIntegrityViolationException→409

DTO：
- 当 id.generated=true 时，请求不包含id
- 响应始终包含id

JPA实体：
- `@`Id 带 `@`GeneratedValue(IDENTITY) 用于数字生成的id

注意事项：
- 在 templates/ 中提供所有模板（参见 templates/README.md）
- 使用 --lombok 添加Lombok注解，不会在注解之间引入空行