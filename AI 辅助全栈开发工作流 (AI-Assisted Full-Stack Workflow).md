# AI 辅助全栈开发工作流 (AI-Assisted Full-Stack Workflow)

## 核心理念

- **角色转变**：从“代码撰写者 (Coder)”转变为“架构师 (Architect) + 产品经理 (PM) + 代码审查员 (Reviewer)”。
- **核心策略**：**Schema-Driven (数据库驱动)**。以数据库设计为源头，通过 AI 自动推导后端代码，再通过 Swagger 契约驱动前端开发。
- **效率关键**：利用“上下文注入”和“标准提示词模板”，让 AI 一次性生成可运行、规范的代码块。

------

## 阶段一：需求拆解与数据建模 (The Blueprint)

**目标**：将模糊的想法转化为精确的 SQL (DDL) 语句。这是整个项目的基石。

### 1. 需求描述 (User Stories)

不要直接设计表结构，而是向 AI 描述**业务场景**。

- **操作**：告诉 AI 你的产品功能。

- **Prompt 示例**：

  > “我要做一个博客系统。用户可以发布文章，文章有多级标签。用户可以评论文章，评论支持嵌套回复。请帮我分析这就需要哪些实体，以及它们之间的关系。”

### 2. 可视化验证 (Mermaid ER 图)

在生成代码前，先确认逻辑关系。

- **操作**：要求 AI 生成 Mermaid 格式的 ER 图。

- **Prompt 示例**：

  > “请基于上述分析，用 Mermaid 语法画出实体关系图 (ER Diagram)。请特别注意多对多关系和自引用的处理。”

- **验证**：将代码复制到 [Mermaid Live Editor](https://mermaid.live/) 或支持 Mermaid 的编辑器中查看。检查：

  - 关联关系是否正确？（如：文章删除后，评论还在吗？）
  - 是否缺少关键字段？（如：`created_at`, `updated_at`）

### 3. 生成“宪法” (SQL DDL)

确认无误后，生成数据库建表语句。

- **操作**：要求 AI 生成 PostgreSQL 适配的 DDL。

- **Prompt 示例**：

  > “作为 PostgreSQL 专家，请生成详细的 DDL 建表语句。 **要求**：

  > 1. 包含详细的字段注释。
  > 2. 设置正确的外键约束 (Foreign Keys)。
  > 3. 针对高频查询字段（如 `slug`, `status`）自动添加索引。
  > 4. 使用 `bigint` 作为主键。”

- **产出**：一份完整的 `.sql` 文件。**保存好它，这是后续所有步骤的上下文核心。**

------

## 阶段二：后端核心与接口实现 (The Backbone)

**目标**：基于 DDL 和项目结构，一次性生成带 Swagger 文档的 Go 代码。

### 1. 准备“超级提示词” (The Super Prompt)

每次生成新模块（如 User, Article, Comment）时，都使用此模板。

#### **上下文注入卡片 (Context Card)**

> **【角色设定】** 你是一个高级 Go 全栈工程师，精通 Gin, GORM v2, 和 Clean Architecture。
>
> **【项目上下文】**
>
> 1. **目录结构 (不可修改)**：
>
>    Plaintext
>
>    ```
>    ├── internal/
>    │   ├── model/       <-- GORM 结构体
>    │   ├── dao/         <-- 数据库操作 (Repository)
>    │   ├── service/     <-- 业务逻辑
>    │   └── api/v1/      <-- Gin 控制器
>    ├── router/          <-- 路由注册
>    ```
>
> 2. **技术栈**：Go 1.23, Gin, GORM, PostgreSQL, Swagger (swaggo)。
>
> 3. **核心输入**：以下是经过确认的 PostgreSQL DDL：
>
>    SQL
>
>    ```
>    [在此粘贴你的 DDL 代码]
>    ```
>
> **【任务要求】** 请基于上述 DDL，**一次性生成** [模块名称，如：文章模块] 的完整 CRUD 代码。
>
> **【关键约束】**
>
> 1. **Swagger 文档化 (必须)**：Controller 层的所有方法必须包含标准的 `swaggo` 注释 (`@Summary`, `@Tags`, `@Param`, `@Success` 等)。
> 2. **依赖注入**：Service 层和 DAO 层应使用接口 (Interface) 定义，支持依赖注入。
> 3. **错误处理**：统一使用项目定义的错误码（假设为 `pkg/e`）。
> 4. **代码输出**：请按文件路径（如 `internal/model/article.go`）分块输出代码。

### 2. 代码审查与修正 (Human Review)

AI 生成代码后，你需要进行 **Code Review**：

- **检查逻辑**：特别是复杂的查询（如分页、多条件搜索）。
- **检查注解**：Swagger 注释中的 `@Success` 是否对应了正确的 Struct？

### 3. 生成文档

- **操作**：运行 `swag init`。
- **产出**：`docs/swagger.json` 和 Swagger UI 界面。

------

## 阶段三：前端构建与对接 (The Face)

**目标**：利用 Swagger 契约，快速生成 TypeScript 类型定义和 Vue 组件。

### 1. 生成 API Client (TypeScript)

不要手写 `axios` 请求。

- **输入**：复制 `swagger.json` 中相关模块的 JSON 片段。

- **Prompt 示例**：

  > “这是后端生成的 Swagger JSON 定义（片段）：[粘贴 JSON]。 请基于此，为我生成一个 Vue 3 Composable 函数 `useArticleApi.ts`。 **要求**：
  >
  > 1. 包含完整的 TypeScript Interface 定义（Request/Response）。
  > 2. 使用 `axios` 实例进行请求。
  > 3. 包含 `loading` 和 `error` 状态管理。”

### 2. 生成 UI 组件 (Vue 3 + Tailwind)

结合你的设计审美（Josh W. Comeau 风格）。

- **输入**：刚刚生成的 `useArticleApi.ts` 代码 + 设计要求。

- **Prompt 示例**：

  > “我已经有了 `useArticleApi`（代码如下）。请帮我编写 `ArticleList.vue` 组件。 **设计风格**：
  >
  > 1. 参考 Josh W. Comeau 的个人博客风格：卡片式布局，轻微的阴影，hover 时有上浮动效 (`-translate-y-1`)，色彩使用柔和的渐变。
  > 2. 使用 Tailwind CSS v4 实现样式。
  > 3. 必须处理 Loading 骨架屏 (Skeleton) 和 Error 提示。”

------

## 阶段四：部署与运维 (The Delivery)

**目标**：解决环境配置和报错，确保应用在腾讯云 Ubuntu 上稳定运行。

### 1. 环境配置生成

- **Prompt 示例**：

  > “我的项目由 Go (Gin) 后端和 Vue 3 前端组成。我需要在 Ubuntu 22.04 上部署。 请为我生成：
  >
  > 1. **Dockerfile**：用于构建 Go 后端，采用多阶段构建减小体积。
  > 2. **Nginx 配置**：用于反向代理后端 API (`/api`) 和托管前端静态文件 (`dist/`)，并配置 HTTPS（假设已有证书）。
  > 3. **Docker Compose**：编排 App, PostgreSQL 和 Redis。”

### 2. 报错排查 (Debug)

- **操作**：直接复制错误日志。

- **Prompt 示例**：

  > “我在启动 Docker 容器时遇到了这个报错：[粘贴日志]。 我的 `docker-compose.yml` 内容是：[粘贴配置]。 请分析原因并给出修复方案。”

------

## 迭代闭环 (The Loop)

当需求发生变更时（例如：给文章增加“点赞数”字段），**严禁直接修改前端或后端代码**，必须遵循以下流程以保证一致性：

1. **修改 DDL**：让 AI 修改 SQL，增加字段。
2. **更新后端**：将新 SQL 喂给 AI，让它更新 Go Model 和 Swagger 注释。
3. **重新生成 Swagger**：运行 `swag init`。
4. **更新前端**：将新 Swagger 喂给 AI，更新 TypeScript 类型定义。

------

## 核心提示 (Pro Tips)

- **保持上下文清洁**：如果对话过长，AI 可能会遗忘之前的设定。建议每个模块（User, Article, Auth）开启一个新的 Chat 会话，并重新发送“超级提示词”。
- **善用伪代码**：对于复杂的业务逻辑（如权限校验中间件），先写一段伪代码或注释告诉 AI 你的思路，它生成的代码会更准确。
- **版本控制**：生成的代码在合入主分支前，务必先在 Git 分支上测试。