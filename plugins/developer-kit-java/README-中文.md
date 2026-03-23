# Developer Kit Java

全面的Java开发工具包，包含Spring Boot、测试、LangChain4J和AWS集成。

## 概述

`developer-kit-java` 插件为Java开发提供广泛支持，包括Spring Boot应用程序、使用JUnit的单元/集成测试、用于AI应用程序的LangChain4J以及AWS SDK集成。

## 智能体

- **spring-boot-backend-development-expert** - Spring Boot开发
- **spring-boot-code-review-expert** - Spring Boot代码审查
- **spring-boot-unit-testing-expert** - Spring Boot测试
- **java-refactor-expert** - Java特定重构
- **java-security-expert** - Java安全最佳实践
- **java-software-architect-review** - Java架构审查
- **java-documentation-specialist** - Java文档
- **java-tutorial-engineer** - Java学习资源
- **langchain4j-ai-development-expert** - LangChain4J集成

## 命令

- `devkit.java.code-review` - 全面的Java代码审查
- `devkit.java.generate-crud` - 生成CRUD操作
- `devkit.java.refactor-class` - 重构Java类
- `devkit.java.architect-review` - 架构审查
- `devkit.java.dependency-audit` - 依赖分析
- `devkit.java.generate-docs` - 生成文档
- `devkit.java.security-review` - 安全审查
- `devkit.java.upgrade-dependencies` - 升级依赖
- `devkit.java.write-unit-tests` - 生成单元测试
- `devkit.java.write-integration-tests` - 生成集成测试
- `devkit.java.generate-refactoring-tasks.md` - 从限界上下文生成重构任务

## 技能

### Spring Boot
- Spring Boot Actuator、缓存、CRUD模式、依赖注入
- 事件驱动模式、OpenAPI文档、REST API标准
- Saga模式、安全JWT、测试模式、Resilience4j
- **spring-boot-project-creator** - 从规范生成项目

### 架构
- **clean-architecture** - 适用于Java 21+ Spring Boot 3.5+的整洁架构、六边形架构和DDD模式

### Spring Data
- Spring Data JPA、Spring Data Neo4j

### Spring AI
- **spring-ai-mcp-server-patterns** - 模型上下文协议服务器实现

### JUnit测试
- 所有层和场景的单元测试模式

### LangChain4J
- AI服务、MCP服务器、RAG实现、Spring Boot集成
- 测试策略、工具函数调用、向量存储配置
- **qdrant** - Qdrant向量数据库集成

### AWS Java SDK
- RDS、Bedrock、DynamoDB、KMS、Lambda、消息传递、S3、密钥管理器

### AWS Lambda集成
- **aws-lambda-java-integration** - 使用Micronaut框架和原生Java处理程序的Java AWS Lambda集成模式

### GraalVM原生镜像
- **graalvm-native-image** - 从Java应用程序构建原生可执行文件，优化冷启动时间，配置Maven/Gradle原生构建工具，解决反射和资源问题

## 依赖项

- `developer-kit` (必需)
- `developer-kit-ai` (用于LangChain4J技能)