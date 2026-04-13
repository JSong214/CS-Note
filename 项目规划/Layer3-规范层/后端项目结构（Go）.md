# Go Project Layout (Standard)

本文档定义了本项目的目录结构规范。
核心设计理念：**Router 集中管理，业务逻辑分层清晰，依赖单向流动。**

## 1. 目录结构全貌

```text
.
├── cmd/
│   └── server/                     # 服务名称 (如 server, api-gateway)
│       └── main.go                 # 【入口】负责依赖注入(DI)、初始化 Router、启动 Server
│
├── internal/                       # 【核心】私有业务代码
│   ├── config/                     # 配置对象定义 (Structs) 与 加载逻辑 (Viper/Godotenv)
│   │
│   ├── handler/                    # 【接口层】(HTTP Handler / gRPC Server)
│   │   ├── user_handler.go         # 解析请求参数，调用 Service，返回响应
│   │   └── order_handler.go
│   │
│   ├── router/                     # 【路由层】(集中式管理)
│   │   ├── middleware/             # 存放项目级中间件
│   │   │   ├── auth.go             # JWT/Session 鉴权
│   │   │   ├── cors.go             # 跨域配置
│   │   │   └── logger.go           # 请求日志/TraceID 注入
│   │   └── router.go               # 【路由主入口】NewRouter()，注册中间件，绑定 Handler
│   │
│   ├── service/                    # 【业务层】(Service)
│   │   ├── service.go              # Service 接口定义 (为了 Mock 测试)
│   │   └── user_service.go         # 具体的业务逻辑实现
│   │
│   ├── repository/                 # 【数据层】(DAO/Repository)
│   │   ├── sql/                    # 数据库 实现
│   │   └── redis/                  # Redis 实现
│   │
│   ├── model/                      # 【模型层】
│   │   ├── entity/                 # 数据库实体 (GORM Model)
│   │   └── dto/                    # 数据传输对象 (Request/Response Structs)
│   │
│   └── pkg/                        # 【内部工具】仅限本项目使用的工具库 (如特定算法、业务常量)
│
├── pkg/                            # 【公共库】明确可被外部项目引用的通用库 (如 util, kit)
│
├── api/                            # API 定义
│   ├── openapi/                    # Swagger/OpenAPI yaml
│   └── proto/                      # Protobuf files
│
├── configs/                        # 配置文件模板
│   ├── config.local.yaml
│   └── config.prod.yaml
│
├── scripts/                        # 构建与运维脚本
│   ├── build.sh
│   └── db_migrate.sh
│
├── Makefile                        # 统一命令管理 (run, build, test, lint)
├── go.mod
└── go.sum
```



## 2.目录职责详解

### 1. 核心应用代码 (`internal/`)

`internal` 是 Go 语言特有的目录，**Go 编译器强制规定**：该目录下的代码只能被当前模块内部引用，外部项目无法 `import`，确保了业务逻辑的私有性和安全性。

按**调用链路**从上到下划分职责：

- **`router/` (路由层)**
  - **职责**：HTTP 请求的分发中心。
  - **详解**：负责定义 URL 路径与 Handler 的映射关系；注册全局或路由级中间件（如：CORS 跨域、Auth 鉴权、Request ID 注入）。
- **`handler/` (传输/接口层)**
  - **职责**：处理网络协议（HTTP/gRPC），不包含复杂业务逻辑。
  - **详解**：
    - **解析**：将 HTTP 请求（Body, Query, Header）解析为 Go 结构体。
    - **校验**：进行基础参数校验（如非空检查）。
    - **调用**：调用 `service` 层执行业务。
    - **响应**：将业务结果封装为标准 HTTP 响应格式（JSON/Protobuf）返回。
- **`service/` (业务逻辑层)**
  - **职责**：项目的核心，封装具体的业务规则。
  - **详解**：实现产品功能（如订单计算、用户状态流转）。它是**协议无关**的（不关心是 HTTP 还是 CLI 调用的它），也是**存储无关**的（不关心底层是 MySQL 还是文件）。
- **`repository/` (数据访问层 / DAO)**
  - **职责**：直接与数据库或外部存储交互。
  - **详解**：封装 SQL 语句或 ORM 操作，提供 CRUD 接口。Service 层通过它来获取数据，从而屏蔽底层的存储细节。
- **`model/` (数据模型)**
  - **`entity/`**：**持久化对象 (PO)**。直接映射数据库表结构，通常带有 ORM 标签（如 `gorm:"primaryKey"`）。
  - **`dto/`**：**数据传输对象 (DTO)**。定义 API 接口的输入（Request）和输出（Response）结构。**作用**：解耦数据库结构与 API 接口，避免数据库字段变更直接影响前端。
- **`config/` (配置管理)**
  - **职责**：加载和映射配置。
  - **详解**：将 `yaml/env` 配置文件读取并反序列化为 Go Struct，供全局使用。

------

### 2. 程序入口 (`cmd/`)

- **`cmd/server/main.go`**
  - **职责**：应用程序的**启动入口**与**依赖注入 (DI) 容器**。
  - **详解**：不包含业务逻辑，只负责“组装”组件。例如：初始化 DB 连接 -> 初始化 Repo -> 初始化 Service -> 初始化 Handler -> 注册 Router -> 启动 HTTP Server。

------

### 3. 公共库 (`pkg/`)

- **`pkg/`**
  - **职责**：**通用的、无业务侵入的**工具库。
  - **详解**：这里的代码可以被**任何** Go 项目引用（不仅仅是本项目）。例如：通用的加密库、时间处理工具、Logger 封装。

------

### 4. 接口定义 (`api/`)

- **`api/`**
  - **职责**：Schema (契约) 定义文件。
  - **详解**：存放 `.proto` 文件（gRPC 定义）或 `openapi.yaml`（Swagger 文档）。这是前后端协作或服务间调用的“契约”。

------

### 5. 工程化与运维 (`configs/`, `scripts/`, `Makefile`)

- **`configs/`**：存放静态配置文件（如 `config.prod.yaml`），定义环境特定的参数（DB 地址、端口）。
- **`scripts/` & `Makefile`**：自动化运维工具。用于标准化常用的命令（如 Build, Test, Deploy, Lint），简化开发流程。

------

### 总结：数据流向

在一个标准的 HTTP 请求中，代码执行的流向是**单向依赖**的：


`Router` (路由匹配) 
⬇️ 
`Handler` (解析协议)
⬇️ 
`Service` (处理业务) 
⬇️ 
`Repository` (读写数据) 
⬇️ 
`Database` (实际存储)



**原则**：上层可以调用下层，但下层严禁反向调用上层（例如 Repository 不能调用 Service）。