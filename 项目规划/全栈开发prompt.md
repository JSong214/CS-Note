## 全栈开发prompt

### 第 0 步：全局上下文设定 (Context Anchor)

在开始任何一个新“切片”或开启新对话窗口时，必须先让 AI 理解项目的技术栈和协作原则。你需要维护一个包含核心设计（ER 图、API 规范）的本地文件（Context Anchor）。

**Prompt (初始化/恢复记忆):**

> “你现在是我的全栈开发助手。我们正在采用‘垂直切片 (Vertical Slices)’和‘API 先行 (API-First)’的模式开发一个应用。
>
> **技术栈**：[例如：Go (Gin) + React (Vite) + PostgreSQL] **当前上下文**：
>
> 1. 这是目前的数据库 ER 图（Mermaid格式）：[粘贴核心 ER 图]
> 2. 这是核心 API 规范（OpenAPI YAML）：[粘贴核心 YAML]
>
> **协作规则**：
>
> - **设计阶段**：你主导。负责将我的模糊需求转化为具体的数据库变更和 OpenAPI 契约。
> - **开发阶段**：我主导核心逻辑。你负责生成 Model、类型定义、Mock 数据和简单的 CRUD 骨架。
> - **约束**：不要过度设计，每次只专注当前的‘用户故事’。”

------

### 第 1 阶段：切片规划与设计 (AI 主导，你 Review)

**目标**：从一个具体的 User Story 出发，产出数据库变更和 API 契约。

#### 1.1 需求拆解与数据库设计

**场景**：你想做一个新功能（例如“用户头像上传”）。 **你的动作**：提出需求，要求 AI 进行最小化的 DB 设计。

**Prompt:**

> “我们要开始一个新的垂直切片：**[功能名称，如：用户头像上传]**。 请扮演**数据库架构师**。基于当前上下文，分析这个功能需要对数据库做什么变更？
>
> 1. 请列出新增的表或字段（保持最小可行性）。
> 2. 请输出更新后的 Mermaid ER 图代码。
> 3. 解释设计理由。”

#### 1.2 制定 API 契约 (The Contract)

**场景**：确认 DB 设计无误后，锁定前后端交互标准。 **你的动作**：要求 AI 生成 OpenAPI 文档。

**Prompt:**

> “DB 设计已确认。现在请扮演**系统架构师**。遵循‘API 先行’原则，为这个切片设计 RESTful API。
>
> 1. **格式**：直接输出 OpenAPI 3.0 (Swagger) 的 YAML 片段。
> 2. **内容**：定义好 Path, Method, Request Body, Response Schema 和 错误码。
> 3. **要求**：确保字段名称与数据库模型一致。”

------

### 第 2 阶段：后端开发 (你主导核心，AI 辅助)

**目标**：实现后端逻辑。你写复杂的 Service/Business Logic，AI 写繁琐的转换代码。

#### 2.1 生成模型与骨架

**你的动作**：把设计好的契约给 AI，让它生成基础代码。

**Prompt:**

> 我们进入后端开发。
>
> 1. **Model 层 (Entity)**：
>    - 根据数据库设计生成 Struct。
>    - 包含必要的 ORM tags (如 `gorm:"..."`) 和 JSON tags。
>    - *引用来源： (基于数据库设计生成模型)*
> 2. **Repository 层 (Data Access)**：
>
>    - 定义 `Repository` **接口 (Interface)**。
>    - 实现基本的 CRUD 操作（如 `Create`, `FindByID`）。
>    - 这部分代码通常是标准的，你可以直接写出实现。
> 3. **Service 层 (Business Logic) ——  重点**：
>    - 定义 `Service` **接口 (Interface)**。该接口的方法签名必须严格对应 OpenAPI 定义的输入输出。
>    - 生成 `Service` 的**实现结构体 (Struct)**，但 **方法内部必须留空**。
>    - **约束**：不要尝试编写复杂的业务逻辑（如权限判断、复杂计算）。
>    - **动作**：在方法体内只写一行 `panic("TODO: Implement business logic here")` 或返回零值。
> 4. **Handler 层 (Controller)**：
>    - 定义 HTTP Handler 函数。
>    - **完整实现**：处理 Request Body 解析 (Binding)、参数校验 (Validation)。
>    - **完整实现**：调用 `Service` 接口的方法。
>    - **完整实现**：根据 Service 返回的结果，按照 OpenAPI 定义的格式组装 Response (JSON) 并处理 HTTP 状态码。

#### 2.2 单元测试生成

**你的动作**：当你写完核心业务逻辑后，让 AI 补全测试。

**Prompt:**

> “这是我实现的业务逻辑代码 [粘贴代码]。请作为**测试工程师**，为这个函数编写单元测试。 请覆盖：正常路径、参数校验失败、以及数据库错误这三种场景。”

------

### 第 3 阶段：前端开发 (你主导交互，AI 辅助对接)

**目标**：实现前端 UI。利用 API 契约自动生成代码，你专注状态管理和交互体验。

#### 3.1 类型定义与 API Client

**你的动作**：利用 Swagger YAML 生成前端代码，避免手写 Fetch。

**Prompt:**

> “现在切换到前端开发。
>
> 1. 请解析之前的 OpenAPI YAML，生成对应的 TypeScript Interface 定义。
> 2. 使用 [Axios/Fetch] 封装一个 Service 函数来调用这个接口。确保包含类型注解。”

#### 3.2 UI 组件模版

**你的动作**：描述 UI 结构，让 AI 写 HTML/CSS，你来填充 JS 逻辑。

**Prompt:**

> “我需要开发这个功能的 UI 组件。 **技术栈**：[Vue3/React] + [Tailwind/ElementUI]。 **需求**：[描述布局，如：左侧图片，右侧上传按钮]。 **任务**：请编写 Template 和 Style 部分的代码。**Script 部分只写出 Props 定义（引用刚才生成的 TS 类型），具体的交互逻辑不要写，留给我来实现**。”

------

### 第 4 阶段：联调与 Debug (AI 辅助分析)

**场景**：通过“垂直切片”快速跑通后，如果遇到报错。

**Prompt:**

> “在联调 `[接口路径]` 时出现错误。
>
> 1. **报错日志**：[粘贴后端或浏览器 Console 日志]
> 2. **相关代码**：[粘贴你的核心代码] 请结合之前的 API 定义，分析可能的原因并给出修复建议。”

------

### 总结：你的“核心工作流”

1. **Start** -> 发送 **Context Anchor** (ER图 + Swagger)。
2. **Design** -> 告诉 AI 需求 -> AI 产出 **DB 变更** + **OpenAPI YAML** -> **你 Review**。
3. **Backend** -> AI 产出 **Model/Controller 骨架** -> **你写 Service 核心逻辑** -> AI 产出 **Tests**。
4. **Frontend** -> AI 产出 **TS 类型/API Client** -> AI 产出 **UI 模版** -> **你写 State/Event 逻辑**。
5. **Finish** -> 提交代码，更新 Context Anchor 文件。