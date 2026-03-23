# Claude Code 开发者工具包

[![安全扫描](https://github.com/giuseppe-trisciuoglio/developer-kit/actions/workflows/security-scan.yml/badge.svg)](https://github.com/giuseppe-trisciuoglio/developer-kit/actions/workflows/security-scan.yml)
[![插件验证](https://github.com/giuseppe-trisciuoglio/developer-kit/actions/workflows/plugin-validation.yml/badge.svg)](https://github.com/giuseppe-trisciuoglio/developer-kit/actions/workflows/plugin-validation.yml)

> 用于在 Claude Code 中自动化开发任务的可重用技能、代理和命令的模块化插件系统

**已上架平台：**
- [context7](https://context7.com/giuseppe-trisciuoglio/developer-kit?tab=skills) — 技能市场
- [skills.sh](https://skills.sh/giuseppe-trisciuoglio/developer-kit) — AI 技能目录

**Claude Code 开发者工具包** 教 Claude 如何以**可重复的方式**在多种语言和框架中执行开发任务。作为模块化市场构建，您只需安装所需的插件。

## 快速开始

```bash
# 从市场安装（推荐）
/plugin marketplace add giuseppe-trisciuoglio/developer-kit

# 或从本地目录安装
/plugin install /path/to/developer-kit
```

**Claude Desktop**: [在设置中启用技能](https://claude.ai/settings/capabilities)

---

## 架构

开发者工具包组织为**模块化市场**，包含 11 个独立插件：

```
plugins/
├── developer-kit-core/            # 核心代理/命令/技能（必需）
├── developer-kit-java/            # Java/Spring Boot/LangChain4J/AWS SDK/GraalVM Native Image
├── developer-kit-typescript/      # NestJS/React/React Native/Next.js/Drizzle/Monorepo
├── developer-kit-python/          # Python 开发/AWS Lambda
├── developer-kit-php/             # PHP/WordPress/AWS Lambda
├── developer-kit-aws/             # AWS CloudFormation/AWS 架构
├── developer-kit-ai/              # 提示工程/RAG/分块
├── developer-kit-devops/          # Docker/GitHub Actions
├── developer-kit-project-management/  # LRA 工作流/会议
├── developer-kit-tools/           # 额外的开发工具
└── github-spec-kit/               # GitHub 规范集成
```

语言插件（Java、TypeScript、Python、PHP）包含**编码规则**（`rules/` 目录），通过 `globs:` 路径范围匹配自动激活，以强制执行命名约定、项目结构、语言最佳实践和错误处理模式。它们还包括**LSP 服务器配置**（`.lsp.json`）用于实时代码智能、诊断和导航功能。

---

## 工作流

开发者工具包遵循系统化的开发工作流，确保从想法到实现的高质量、良好文档化的功能：

### 1. 头脑风暴阶段

**命令：** `/devkit.brainstorm [想法描述]`

当您有新功能想法时从此处开始。此命令指导您创建**功能规范**（系统应该做什么，而不是如何做）：

- **想法细化**：通过结构化对话进行深入探索
- **用例定义**：定义用户行为和业务规则
- **验收标准**：建立可测试的完成条件
- **规范审查**：与用户验证

**输出**：功能规范保存到 `docs/specs/YYYY-MM-DD--feature-name.md`

**示例：**
```bash
/devkit.brainstorm 添加使用 JWT 令牌的用户认证
```

**下一步**：规范完成后，继续使用 `/devkit.spec-to-tasks`

### 2. 规范到任务阶段

**命令：** `/devkit.spec-to-tasks [--lang=java|spring|typescript|nestjs|react|python|general] [spec-file]`

将功能规范转换为原子的、可执行的任务：

- **任务分解**：将规范分解为可操作的任务
- **依赖关系映射**：识别任务依赖关系
- **验收标准**：每个任务都有明确的完成标准
- **实现命令**：预填充执行命令
- **复杂度评分**：自动计算任务复杂度（0-100+ 等级）
- **强约束**：复杂度评分 ≥ 51 的任务必须在实现前拆分

**输出**：任务列表保存到 `docs/specs/[id]/tasks/TASK-XXX.md`，包含复杂度评分

**示例：**
```bash
/devkit.spec-to-tasks docs/specs/001-user-auth/
/devkit.spec-to-tasks --lang=spring docs/specs/001-user-auth/
```

**下一步**：审查任务复杂度并使用 `/devkit.task-manage` 管理任务

### 2.5. 任务管理阶段（新功能）

**命令：** `/devkit.task-manage --action=[list|split|add|mark-optional|update|regenerate-index] [options]`

生成任务后管理任务，确保它们大小合适且优先级正确：

- **列出任务**：查看所有任务及其复杂度评分和建议
- **拆分复杂任务**：自动将复杂度评分 ≥ 51 的任务拆分为更小的任务
- **添加新任务**：动态添加任务到现有规范
- **标记为可选**：通过将非关键任务标记为可选来优先处理 MVP
- **更新任务**：修改需求或验收标准
- **重新生成索引**：修改后更新任务列表

**复杂度评分：**
- 简单（0-30 ✅）：可以作为单个任务
- 中等（31-50 ⚠️）：考虑拆分
- 复杂（51+ ❌）：必须在实现前拆分

**示例：**
```bash
# 列出所有任务及其复杂度评分
/devkit.task-manage --action=list --spec=docs/specs/001-user-auth/

# 拆分复杂任务
/devkit.task-manage --action=split --task=docs/specs/001-user-auth/tasks/TASK-007.md

# 将任务标记为 MVP 可选
/devkit.task-manage --action=mark-optional --task=docs/specs/001-user-auth/tasks/TASK-010.md

# 添加新任务
/devkit.task-manage --action=add --spec=docs/specs/001-user-auth/2026-03-07--user-auth.md --lang=spring
```

**输出**：更新的任务文件和重新生成的任务列表索引

**下一步**：使用 `/devkit.feature-development` 执行任务

### 3. 功能开发阶段

**命令：** `/devkit.feature-development [--lang=spring|typescript|nestjs|react|python|general] [功能描述 | "Task: 任务名称"]`

使用此命令实现功能。支持两种模式：

**模式 1 - 功能开发：** 实现整个功能
- **发现**：了解需要构建的内容
- **代码库探索**：深入理解现有代码模式
- **澄清问题**：填补空白并解决歧义
- **架构设计**：设计实现方案并权衡取舍
- **实现**：按照约定构建功能
- **质量审查**：确保代码质量和正确性
- **总结**：记录完成的内容

**模式 2 - 任务执行：** 执行任务列表中的特定任务（使用 "Task:" 前缀）
- 从 `docs/tasks/YYYY-MM-DD--*--tasks.md` 读取任务详情
- 专注于特定任务实现
- 在任务列表中更新任务进度

**语言/框架支持：**
- `--lang=spring` 或 `--lang=java`：Java/Spring Boot 开发
- `--lang=typescript` 或 `--lang=ts`：TypeScript/Node.js 开发
- `--lang=nestjs`：NestJS 后端开发
- `--lang=react`：React 前端开发
- `--lang=python` 或 `--lang=py`：Python 开发
- `--lang=aws`：AWS 基础设施和 CloudFormation
- `--lang=general` 或无标志：通用开发

**示例：**
```bash
# 功能开发模式
/devkit.feature-development --lang=spring 添加用户管理的 REST API

# 任务执行模式
/devkit.feature-development --lang=spring "Task: 用户登录端点"
```

### 4. 代码审查和调试阶段

实现后，使用这些专门命令进行质量保证：

**任务审查：** `/devkit.task-review [--lang=...] [task-file]`
- 验证任务实现是否符合规范
- 验证验收标准
- 检查规范合规性
- 执行语言特定的代码审查
- 生成综合审查报告

**代码审查：** `/devkit.refactor [--lang=...] [重构描述]`
- 改善代码结构、可维护性和设计
- 减少技术债务
- 应用最佳实践和模式

**修复和调试：** `/devkit.fix-debugging [--lang=...] [问题描述]`
- 系统性根本原因分析
- 最小化、有针对性的修复
- 验证和回归预防

**示例：**
```bash
/devkit.task-review --lang=spring docs/specs/001-user-auth/tasks/TASK-001.md
/devkit.fix-debugging --lang=spring OrderService 中的 Bean 注入失败
/devkit.refactor --lang=typescript 简化认证流程
```

### 典型开发流程

```
1. 想法
   ↓
2. /devkit.brainstorm
   ↓ (创建功能规范: docs/specs/)
3. /devkit.spec-to-tasks
   ↓ (创建带复杂度评分的任务列表: docs/specs/[id]/tasks/)
4. /devkit.task-manage --action=list
   ↓ (审查任务复杂度和建议)
5. /devkit.task-manage --action=split (如果需要)
   ↓ (将复杂度评分 ≥ 51 的复杂任务拆分为更小任务)
6. /devkit.feature-development
   ↓ (按依赖顺序实现任务)
7. /devkit.task-review
   ↓ (验证实现是否符合规范)
8. 代码审查和测试
   ↓
9. /devkit.fix-debugging (如果发现问题)
   ↓
10. /devkit.refactor (用于改进)
   ↓
11. 准备部署
```

**新工作流：** `想法 → 功能规范 → 任务 → 任务管理 → 实现 → 审查 → 质量保证`

### 替代路径

**对于错误修复：**
```
错误报告 → /devkit.fix-debugging → 修复实现 → 验证
```

**对于重构：**
```
代码问题 → /devkit.brainstorm (可选) → /devkit.refactor → 改进的代码
```

**对于快速功能：**
```
简单功能 → /devkit.feature-development → 直接实现
```

---

## 可用插件

### developer-kit-core（必需）

所有其他插件使用的核心代理、命令和技能。

| 组件                    | 描述                            |
|------------------------------|----------------------------------------|
| `general-code-explorer`      | 深度代码库探索和分析 |
| `general-code-reviewer`      | 代码质量和安全审查       |
| `general-refactor-expert`    | 代码重构专家            |
| `general-software-architect` | 功能架构设计            |
| `general-debugger`           | 根本原因分析和调试      |
| `document-generator-expert`  | 专业文档生成       |

**技能**：`claude-md-management`, `drawio-logical-diagrams`, `github-issue-workflow`, `docs-updater`

**钩子**：`prevent-destructive-commands`（用于阻止危险 Bash 命令的 Python 3 PreToolUse 钩子）

**命令**：`/devkit.brainstorm`, `/devkit.spec-to-tasks`, `/devkit.task-manage`, `/devkit.task-review`, `/devkit.refactor`, `/devkit.feature-development`,
`/devkit.fix-debugging`, `/devkit.generate-document`, `/devkit.generate-changelog`, `/devkit.github.create-pr`,
`/devkit.github.review-pr`, `/devkit.lra.*`（7 个 LRA 工作流命令），`/devkit.verify-skill`,
`/devkit.generate-security-assessment`

---

### developer-kit-java

包含 Spring Boot、测试、LangChain4J、AWS SDK 和 GraalVM Native Image 的综合 Java 开发工具包。

**代理**：`spring-boot-backend-development-expert`, `spring-boot-code-review-expert`,
`spring-boot-unit-testing-expert`, `java-refactor-expert`, `java-security-expert`, `java-software-architect-review`,
`java-documentation-specialist`, `java-tutorial-engineer`, `langchain4j-ai-development-expert`

**命令**：`/devkit.java.code-review`, `/devkit.java.generate-crud`, `/devkit.java.refactor-class`,
`/devkit.java.architect-review`, `/devkit.java.dependency-audit`, `/devkit.java.generate-docs`,
`/devkit.java.security-review`, `/devkit.java.upgrade-dependencies`, `/devkit.java.write-unit-tests`,
`/devkit.java.write-integration-tests`, `/devkit.java.generate-refactoring-tasks`

**技能**：

- **Spring Boot**：actuator, cache, crud-patterns, dependency-injection, event-driven-patterns, openapi-documentation,
  rest-api-standards, saga-pattern, security-jwt, test-patterns, resilience4j, project-creator
- **Spring Data**：jpa, neo4j
- **Spring AI**：mcp-server-patterns
- **JUnit 测试**：application-events, bean-validation, boundary-conditions, caching, config-properties,
  controller-layer, exception-handler, json-serialization, mapper-converter, parameterized, scheduled-async,
  security-authorization, service-layer, utility-methods, wiremock-rest-api
- **集成测试**：wiremock-standalone-docker（通过 Docker 进行集成/E2E 测试的 WireMock 独立服务器）
- **LangChain4J**：ai-services-patterns, mcp-server-patterns, rag-implementation-patterns, spring-boot-integration,
  testing-strategies, tool-function-calling-patterns, vector-stores-configuration, qdrant
- **AWS SDK**：rds-spring-boot-integration, bedrock, core, dynamodb, kms, lambda, messaging, rds, s3, secrets-manager
- **整洁架构**：clean-architecture
- **GraalVM**：graalvm-native-image

**规则**：`naming-conventions`, `project-structure`, `language-best-practices`, `error-handling`

**LSP 服务器**：`java` (jdtls), `kotlin` (kotlin-language-server), `scala` (metals)

---

### developer-kit-typescript

使用 NestJS、React、React Native、Next.js、Drizzle ORM、DynamoDB-Toolbox、Zod 验证和 Monorepo 工具进行 TypeScript/JavaScript 全栈开发。

**代理**：`nestjs-backend-development-expert`, `nestjs-code-review-expert`, `nestjs-database-expert`,
`nestjs-security-expert`, `nestjs-testing-expert`, `nestjs-unit-testing-expert`, `react-frontend-development-expert`,
`react-software-architect-review`, `typescript-refactor-expert`, `typescript-security-expert`,
`typescript-software-architect-review`, `typescript-documentation-expert`, `expo-react-native-development-expert`

**命令**：`/devkit.typescript.code-review`, `/devkit.react.code-review`, `/devkit.ts.security-review`

**技能**：
- **后端**：`nestjs`, `nestjs-best-practices`, `clean-architecture`, `nestjs-drizzle-crud-generator`
- **认证**：`better-auth`
- **前端**：`react-patterns`, `shadcn-ui`, `tailwind-css-patterns`, `tailwind-design-system`
- **Next.js**：`nextjs-app-router`, `nextjs-authentication`, `nextjs-data-fetching`, `nextjs-performance`, `nextjs-deployment`
- **数据库和 ORM**：`drizzle-orm-patterns`, `dynamodb-toolbox-patterns`, `zod-validation-utilities`
- **Monorepo**：`nx-monorepo`, `turborepo-monorepo`
- **AWS Lambda**：`aws-lambda-typescript-integration`
- **核心**：`typescript-docs`

**规则**：`naming-conventions`, `project-structure`, `language-best-practices`, `error-handling`,
`nestjs-architecture`, `nestjs-api-design`, `nestjs-security`, `nestjs-testing`,
`react-component-conventions`, `react-data-fetching`, `react-routing-conventions`, `tailwind-styling-conventions`,
`drizzle-orm-conventions`, `shared-dto-conventions`, `nx-monorepo-conventions`, `i18n-conventions`,
`lambda-conventions`, `server-feature-conventions`

**LSP 服务器**：`typescript`/`javascript` (typescript-language-server), `eslint` (eslint-language-server), `vue` (vue-language-server)

---

### developer-kit-python

用于 Django、Flask 和 FastAPI 项目的 Python 开发功能。

**代理**：`python-code-review-expert`, `python-refactor-expert`, `python-security-expert`,
`python-software-architect-expert`

**技能**：`clean-architecture`, `aws-lambda-python-integration`

**规则**：`naming-conventions`, `project-structure`, `language-best-practices`, `error-handling`

**LSP 服务器**：`python` (pyright-langserver)

---

### developer-kit-php

PHP 和 WordPress 开发功能。

**代理**：`php-code-review-expert`, `php-refactor-expert`, `php-security-expert`, `php-software-architect-expert`,
`wordpress-development-expert`

**技能**：`wordpress-sage-theme`（Sage 主题开发）, `clean-architecture`, `aws-lambda-php-integration`

**规则**：`naming-conventions`, `project-structure`, `language-best-practices`, `error-handling`

**LSP 服务器**：`php` (intelephense)

---

### developer-kit-aws

用于基础设施即代码的 AWS 基础设施和 CloudFormation 专业知识。

**代理**：`aws-solution-architect-expert`, `aws-cloudformation-devops-expert`, `aws-architecture-review-expert`

**技能**：
- **CloudFormation** (15)：`vpc`, `ec2`, `lambda`, `iam`, `s3`, `rds`, `dynamodb`, `ecs`, `auto-scaling`, `cloudwatch`,
  `cloudfront`, `security`, `elasticache`, `bedrock`, `task-ecs-deploy-gh`
- **通用 AWS** (4)：`aws-sam-bootstrap`, `aws-drawio-architecture-diagrams`, `aws-cli-beast`, `aws-cost-optimization`

---

### developer-kit-ai

包括提示工程、RAG 和分块策略的 AI/ML 功能。

**代理**：`prompt-engineering-expert`

**命令**：`/devkit.prompt-optimize`

**技能**：`prompt-engineering`, `chunking-strategy`, `rag`

---

### developer-kit-devops

DevOps 和容器化专业知识。

**代理**：`github-actions-pipeline-expert`, `general-docker-expert`

---

### developer-kit-project-management

项目管理和工作流命令。

**命令**：`/devkit.write-a-minute-of-a-meeting`

---

### developer-kit-tools

额外的开发工具和集成。

**技能**：
- `notebooklm`（Google NotebookLM 集成）
- `copilot-cli`（支持多模型的 GitHub Copilot CLI 委托）
- `gemini`（用于大上下文分析的 Gemini CLI 委托）
- `codex`（使用 GPT-5.3-codex 模型进行复杂开发任务的 OpenAI Codex CLI 委托）

---

### github-spec-kit

GitHub 规范集成和验证。

**命令**：`/speckit.check-integration`, `/speckit.optimize`, `/speckit.verify`

---

## 主要特性

- **模块化** — 只安装您技术栈所需的插件
- **专业化** — 用于代码审查、测试、AI 开发和全栈开发的领域特定代理
- **可组合** — 技能根据任务上下文自动堆叠
- **可移植** — 可在 Claude.ai、Claude Code CLI、Claude Desktop 和 Claude API 中使用
- **高效** — 技能按需加载，在主动使用前消耗最少的令牌

---

## 语言支持

| 语言           | 插件                     | 组件                                  |
|--------------------|----------------------------|---------------------------------------------|
| 核心               | `developer-kit-core`       | 4 个技能，6 个代理，17 个命令             |
| Java/Spring Boot   | `developer-kit-java`       | 51 个技能，9 个代理，11 个命令，4 个规则  |
| TypeScript/Node.js | `developer-kit-typescript` | 25 个技能，13 个代理，3 个命令，17 个规则 |
| Python             | `developer-kit-python`     | 2 个技能，4 个代理，4 个规则                 |
| PHP/WordPress      | `developer-kit-php`        | 3 个技能，5 个代理，4 个规则                 |
| AWS CloudFormation | `developer-kit-aws`        | 19 个技能，3 个代理                         |
| AI/ML              | `developer-kit-ai`         | 3 个技能，1 个代理，1 个命令                |

---

## 贡献

有关添加技能、代理和命令的详细说明，请参见 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

## 安全

技能可以执行代码。在部署前审查所有自定义技能。

- 只从可信来源安装
- 启用前审查 SKILL.md
- 首先在非生产环境中测试

---

## 许可证

参见 [LICENSE](LICENSE) 文件。

---

## 支持

- **有问题？** [开启 issue](https://github.com/giuseppe-trisciuoglio/developer-kit/issues)
- **要贡献？** [提交 PR](https://github.com/giuseppe-trisciuoglio/developer-kit/pulls)

## 更新日志

参见 [CHANGELOG.md](CHANGELOG.md) 获取完整历史记录。

---

**为使用 Claude Code 的开发者精心制作**
**也适用于 OpenCode、Github Copilot CLI 和 Codex**