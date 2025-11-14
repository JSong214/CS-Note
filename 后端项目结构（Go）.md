## 后端项目结构

```
/your-project
├── /cmd
│   └── /api
│       └── main.go           	# 🏁 程序入口：负责配置加载、依赖注入和启动服务器
├── /configs
│   └── config.yaml         	# 📄 静态配置文件 (DB, Port, JWT Secret等)
├── /internal
│   ├── /config             	# ⚙️ 配置加载：读取 config.yaml 并映射到 struct
│   │   └── config.go
│   ├── /dto                	# 📦 API数据传输对象 (我们讨论的拆分)
│   │   ├── auth_dto.go     	# (例如: LoginRequest, RegisterRequest)
│   │   └── user_dto.go     
│   ├── /handler            	# 📦 HTTP处理层 (Controller)
│   │   ├── auth_handler.go 	# (Gin) 解析请求到 DTO, 调用 Service
│   │   └── user_handler.go
│   ├── /middleware         	# 🔐 Gin 中间件
│   │   └── auth_middleware.go  # (golang-jwt) 验证JWT Token
│   ├── /model              	# 📝 数据库模型 (GORM)
│   │   └── user_model.go   	# (GORM) 定义 User struct (带 gorm tags)
│   ├── /repository         	# 💾 数据访问层 (GORM)
│   │   ├── user_repository.go  # 封装 GORM 操作 (如 GetUserByEmail)
│   │   └── db.go           	# GORM 初始化 (InitDB)
│   ├── /router             	# 🛣️ 路由定义
│   │   └── router.go       	# (Gin) 注册所有路由和中间件
│   ├── /service            	# 🧠 业务逻辑层
│   │   ├── auth_service.go 	# 登录(bcrypt)和Token(JWT)逻辑
│   │   └── user_service.go 	# 注册时加密密码(bcrypt)
│   ├── /server             	# 🌐 HTTP 服务器
│   │   └── server.go       	# 封装 http.Server 启动与优雅关闭(可选)
│   └── /util               	# 🛠️ 通用工具
│       ├── response.go     	# 封装统一的 JSON 响应格式
│       └── validation.go   	# (可选) 自定义验证器
├── /api                    	# 📜 API 文档 (例如 Swagger/OpenAPI)
│   └── openapi.yml
├── go.mod                  	# Go 模块依赖
├── go.sum
└── README.md
```



### 项目结构作用分析



#### 一级目录（项目根目录）

- **`/cmd`**
  - **作用**：项目的**程序入口 (Main Applications)**。
  - **详解**：这是Go项目的标准实践。如果你的项目未来可能包含多个可执行文件（例如一个 `api` 服务、一个 `worker` 进程），你可以在 `cmd` 下创建多个子目录。
  - `main.go` 的核心职责是“组装”应用：加载配置、初始化数据库连接（来自 `repository`）、设置路由（来自 `router`）、注入依赖，最后启动HTTP服务器（来自 `server`）。
- **`/configs`**
  - **作用**：存放**静态配置文件**。
  - **详解**：`config.yaml` 在这里定义了数据库连接信息、服务器端口、JWT密钥等。这些配置在程序启动时会被 `/internal/config` 模块读取。
- **`/internal`**
  - **作用**：项目的**核心内部代码**。
  - **详解**：这是Go语言的一个特性。放在 `internal` 目录下的所有包（package）**只能被其父目录（即 `your-project`）下的代码引用**。外部项目无法 `import` 你的 `internal` 代码，这为你提供了强有力的封装和保护。
- **`/api`**
  - **作用**：存放 **API 文档**。
  - **详解**：这里使用 `openapi.yml` (Swagger) 来定义API的规格。这对于前端开发、API测试和团队协作至关重要。
- **`go.mod` / `go.sum`**
  - **作用**：**Go 模块依赖管理**。
  - **详解**：`go.mod` 定义了项目模块名称和所有依赖的库（如 Gin, GORM）及其版本。`go.sum` 是校验和文件，确保依赖的完整性。

------



#### 二级目录（`/internal` 内部结构）

这是你应用的核心，采用了经典的三层架构（Handler -> Service -> Repository）。

- **`/internal/config`**
  - **作用**：配置加载模块。
  - **详解**：`config.go` 会读取 `/configs/config.yaml` 文件，并将其内容解析（Unmarshal）到一个Go结构体 (struct) 中，供应用全局使用。
- **`/internal/dto` (Data Transfer Object)**
  - **作用**：**数据传输对象**，用于API层。
  - **详解**：这是你特别提到的拆分点。它定义了API请求（Request）和响应（Response）的数据结构。例如 `LoginRequest`。这非常重要，因为它**解耦了API接口和数据库模型**：
    - `handler` 层使用 DTO 来解析 Gin 的请求 JSON。
    - `service` 层接收 DTO，将其转换为 `model` 再去调用 `repository`。
- **`/internal/handler` (Controller / C)**
  - **作用**：**HTTP 处理层**，类似其他框架中的 Controller。
  - **详解**：这是 **Gin** 框架直接交互的地方。`auth_handler.go` 这类文件中的函数：
    1. 使用 Gin 的 `c.BindJSON()` 将请求体绑定到 `dto` 结构体。
    2. 调用 `/internal/service` 中的业务逻辑。
    3. 获取 `service` 的返回结果，并使用 `/internal/util/response.go` 封装成统一的JSON格式返回给客户端。
- **`/internal/middleware`**
  - **作用**：**Gin 中间件**。
  - **详解**：`auth_middleware.go` 是一个完美的例子。它会拦截需要认证的API请求，使用 **golang-jwt** 包 来解析和验证 "Authorization" Header 中的 JWT Token。如果验证通过，则放行；否则，提前终止请求并返回错误。
- **`/internal/model` (Model / M)**
  - **作用**：**数据库模型**。
  - **详解**：这是 **GORM** 框架直接交互的地方。`user_model.go` 定义了 `User` 结构体，并使用 GORM 的 `gorm` tags (如 `gorm:"column:user_name"`) 来映射数据库中的表和字段。
- **`/internal/repository` (Data Access Object / DAO)**
  - **作用**：**数据访问层**。
  - **详解**：这一层**封装了所有 GORM 操作**。`user_repository.go` 会提供诸如 `GetUserByEmail(email string)` 这样的方法。
    - **好处**：`service` 层不需要知道任何 GORM 的语法（如 `db.Where(...)`）。它只调用 `repository` 的方法。如果未来你想从 GORM 迁移到别的ORM，你只需要修改 `repository` 层的实现，而 `service` 层完全不用动。
    - `db.go` 负责 GORM 的初始化（`gorm.Open`）。
- **`/internal/router`**
  - **作用**：**路由定义**。
  - **详解**：`router.go` 负责创建 **Gin** 引擎实例 (`gin.Default()`)，并注册所有的API路由规则。它会将特定的URL（如 `/login`）绑定到 `handler` 层的特定函数（如 `authHandler.Login`），并在这里应用 `middleware`。
- **`/internal/service` (Business Logic Layer / BLL)**
  - **作用**：**业务逻辑层**。
  - **详解**：这是应用的核心。它处理所有不属于 "HTTP请求" 或 "数据库查询" 的逻辑。
    - `auth_service.go`：负责登录逻辑。它会调用 `user_repository` 查用户，然后使用 **bcrypt** 包来比较哈希密码。成功后，使用 **golang-jwt** 生成 Token。
    - `user_service.go`：负责注册逻辑。它会调用 **bcrypt** 来加密明文密码，然后再调用 `user_repository` 创建用户。
- **`/internal/server`**
  - **作用**：**HTTP 服务器**封装。
  - **详解**：`server.go` 封装了Go原生的 `http.Server`。这样做的好处是你可以精细控制服务器的行为，例如配置超时（Timeouts），以及实现**优雅关闭 (Graceful Shutdown)**，确保在服务器停止时能处理完所有进行中的请求。
- **`/internal/util`**
  - **作用**：**通用工具**包。
  - **详解**：`response.go` 提供了统一的API响应结构（例如 `{"code": 0, "msg": "success", "data": ...}`），这让前端处理起来更方便。`validation.go` 可以放一些自定义的验证逻辑。

------



### 总结

这个结构非常清晰，数据的流向（请求生命周期）是这样的：

1. **Gin 路由** (`router`) 接收请求。
2. **中间件** (`middleware`) (例如 JWT 认证)。
3. **处理层** (`handler`) 解析请求到 **DTO** (`dto`)。
4. `handler` 调用**业务逻辑层** (`service`)。
5. `service` (可能使用 `bcrypt`) 调用**数据访问层** (`repository`)。
6. `repository` 使用 **GORM** 和 **Model** (`model`) 操作数据库。
7. 数据逐层返回，最后由 `handler` 使用 `util/response` 封装成 JSON 返回给客户端。
