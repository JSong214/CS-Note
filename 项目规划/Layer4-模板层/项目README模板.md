# [项目名称]

> [一句话描述项目是什么，解决什么问题]
>
> **线上地址**：https://yourdomain.com
> **API 文档**：https://yourdomain.com/swagger/index.html

---

## 目录

- [项目简介](#项目简介)
- [技术栈](#技术栈)
- [快速启动（开发环境）](#快速启动开发环境)
- [目录结构](#目录结构)
- [常用命令](#常用命令)
- [环境变量说明](#环境变量说明)
- [数据库迁移](#数据库迁移)
- [测试](#测试)
- [部署](#部署)
- [API 文档](#api-文档)
- [已知问题](#已知问题)
- [联系方式](#联系方式)

---

## 项目简介

[2-4 句话描述：
- 项目背景和目标
- 主要功能模块
- 服务的用户群体]

---

## 技术栈

### 后端

| 技术 | 版本 | 用途 |
|---|---|---|
| Go | 1.22+ | 服务端语言 |
| Gin | v1.9+ | HTTP 框架 |
| PostgreSQL | 16+ | 主数据库 |
| Redis | 7+ | 缓存 / Session |
| GORM | v2 | ORM |
| golang-migrate | — | 数据库迁移 |

### 前端

| 技术 | 版本 | 用途 |
|---|---|---|
| Vue | 3.x | 前端框架 |
| TypeScript | 5.x | 类型安全 |
| Pinia | 2.x | 状态管理 |
| Element Plus | 2.x | UI 组件库 |
| Axios | 1.x | HTTP 客户端 |
| Vite | 5.x | 构建工具 |

### 基础设施

| 技术 | 用途 |
|---|---|
| Docker + Docker Compose | 容器化 / 本地开发环境 |
| Nginx | 反向代理 / 静态文件服务 |
| GitHub Actions | CI/CD 自动化 |
| Let's Encrypt | SSL 证书 |

---

## 快速启动（开发环境）

### 前置依赖

确保本地已安装以下工具：

- [Docker](https://www.docker.com/) & Docker Compose（≥ 24.0）
- [Go](https://go.dev/) 1.22+
- [Node.js](https://nodejs.org/) 20+ & npm

### 一键启动

```bash
# 1. 克隆仓库
git clone <repository-url>
cd <project-name>

# 2. 复制并配置环境变量
cp backend/.env.example backend/.env.development
cp frontend/.env.example frontend/.env.development
# 编辑 .env.development，填写必要的配置项

# 3. 启动依赖服务（数据库 + Redis）
docker-compose up -d db redis

# 4. 执行数据库迁移
make migrate

# 5. 启动后端（另开终端）
cd backend && make dev

# 6. 启动前端（另开终端）
cd frontend && npm install && npm run dev
```

启动完成后访问：
- 前端：http://localhost:3000
- 后端 API：http://localhost:8080
- Swagger 文档：http://localhost:8080/swagger/index.html

### 一键启动（Docker 全套）

```bash
# 使用 Docker Compose 启动全部服务（适合演示/测试）
docker-compose up -d

# 访问：http://localhost:3000
```

---

## 目录结构

```
.
├── backend/                 # Go 后端
│   ├── cmd/
│   │   └── server/          # 程序入口
│   ├── internal/
│   │   ├── handler/         # HTTP 处理器（路由层）
│   │   ├── service/         # 业务逻辑层
│   │   ├── repository/      # 数据访问层
│   │   ├── model/           # 数据库模型
│   │   ├── dto/             # 请求/响应 DTO
│   │   └── middleware/      # 中间件
│   ├── migrations/          # 数据库迁移文件
│   ├── configs/             # 配置文件
│   ├── Dockerfile
│   ├── Makefile
│   └── go.mod
│
├── frontend/                # Vue 3 前端
│   ├── src/
│   │   ├── api/             # API 请求封装
│   │   ├── components/      # 组件
│   │   ├── views/           # 页面
│   │   ├── stores/          # Pinia 状态管理
│   │   ├── router/          # 路由配置
│   │   └── utils/           # 工具函数
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml       # 本地开发环境
├── docker-compose.prod.yml  # 生产环境
├── nginx/
│   └── conf.d/
│       └── app.conf         # Nginx 配置
└── Makefile                 # 全局命令入口
```

---

## 常用命令

```bash
# ========== 开发 ==========
make dev              # 启动开发环境（后端热重载）
make frontend-dev     # 启动前端开发服务器

# ========== 测试 ==========
make test             # 运行所有测试（后端 + 前端）
make test-backend     # 仅运行后端测试（含覆盖率）
make test-frontend    # 仅运行前端测试
make test-e2e         # 运行端到端测试（需要 Playwright）

# ========== 代码质量 ==========
make lint             # 代码风格检查（go vet + ESLint）
make fmt              # 自动格式化代码

# ========== 数据库 ==========
make migrate          # 执行待执行的 Migration
make migrate-down     # 回滚最近一次 Migration
make migrate-create NAME=add_xxx_to_users  # 创建新 Migration 文件

# ========== 构建 ==========
make build            # 构建生产镜像
make build-backend    # 仅构建后端二进制
make build-frontend   # 仅构建前端静态文件

# ========== 部署（需配置）==========
make deploy-staging   # 部署到预发布环境
make deploy-prod      # 部署到生产环境（需审批）
```

---

## 环境变量说明

参考 `backend/.env.example` 文件：

| 变量名 | 必填 | 示例值 | 说明 |
|---|---|---|---|
| `APP_ENV` | ✅ | `development` | 运行环境：development / staging / production |
| `APP_PORT` | ✅ | `8080` | HTTP 服务端口 |
| `DB_HOST` | ✅ | `localhost` | 数据库主机 |
| `DB_PORT` | ✅ | `5432` | 数据库端口 |
| `DB_USER` | ✅ | `dev` | 数据库用户名 |
| `DB_PASSWORD` | ✅ | — | 数据库密码（生产环境通过 CI/CD 注入） |
| `DB_NAME` | ✅ | `myapp_dev` | 数据库名称 |
| `REDIS_URL` | ✅ | `redis://localhost:6379` | Redis 连接地址 |
| `JWT_SECRET` | ✅ | — | JWT 签名密钥（至少 32 位随机字符串） |
| `JWT_EXPIRE_MINUTES` | ✅ | `15` | AccessToken 有效期（分钟） |
| `REFRESH_TOKEN_EXPIRE_DAYS` | ✅ | `7` | RefreshToken 有效期（天） |
| `SENTRY_DSN` | ❌ | — | 错误追踪（生产环境建议配置） |

> ⚠️ 生产环境的敏感变量（DB_PASSWORD、JWT_SECRET 等）通过 CI/CD 平台的 Secrets 注入，绝不写入文件并提交到代码库。

---

## 数据库迁移

本项目使用 `golang-migrate` 管理数据库版本。

```bash
# 执行所有未执行的 Migration（升级）
make migrate

# 回滚最近一次 Migration（降级）
make migrate-down

# 查看当前版本
make migrate-version

# 创建新 Migration 文件
make migrate-create NAME=create_orders_table
# 会生成：
# backend/migrations/000002_create_orders_table.up.sql
# backend/migrations/000002_create_orders_table.down.sql
```

Migration 文件位于 `backend/migrations/`，每个变更都有对应的 `.up.sql`（升级）和 `.down.sql`（回滚）文件。

---

## 测试

### 测试覆盖率目标

| 层级 | 目标覆盖率 |
|---|---|
| 后端 Service 层 | ≥ 80% |
| 后端 Handler 层 | ≥ 60% |
| 前端关键组件 | ≥ 60% |

### 运行测试

```bash
# 后端：运行所有测试 + 输出覆盖率报告
cd backend && go test ./... -race -coverprofile=coverage.out
go tool cover -html=coverage.out  # 在浏览器查看覆盖率详情

# 前端：运行单元测试
cd frontend && npm run test:unit

# E2E 测试（需安装 Playwright）
cd frontend && npx playwright install
npm run test:e2e
```

---

## 部署

### 生产环境架构

```
用户
 │
 ▼
Nginx（SSL 终止 + 反向代理）
 ├── / → 前端静态文件（/var/www/frontend/dist）
 └── /api/ → 后端服务（:8080）
              ├── PostgreSQL
              └── Redis
```

### 手动部署步骤

```bash
# 1. 在服务器上拉取最新代码
git pull origin main

# 2. 构建并启动服务
docker-compose -f docker-compose.prod.yml build
docker-compose -f docker-compose.prod.yml up -d

# 3. 执行数据库迁移
docker-compose -f docker-compose.prod.yml exec backend make migrate

# 4. 验证服务状态
docker-compose -f docker-compose.prod.yml ps
curl http://localhost:8080/api/v1/health
```

### 自动部署（推荐）

推送到 `main` 分支后，GitHub Actions 会自动：
1. 运行所有测试（后端 + 前端）
2. 通过后，触发生产环境部署（需要人工审批）

---

## API 文档

启动服务后访问：
- **开发环境**：http://localhost:8080/swagger/index.html
- **生产环境**：https://yourdomain.com/swagger/index.html

API 遵循统一响应格式，详见 `docs/统一响应封装设计文档.md`。

---

## 已知问题

| 问题描述 | 优先级 | 影响范围 | 预计修复版本 |
|---|---|---|---|
| [已知问题1] | P2 | [影响] | v1.1 |
| [已知问题2] | P3 | [影响] | 待排期 |

---

## 联系方式

**开发者**：[姓名]
**邮箱**：[邮箱地址]
**质保期**：[截止日期]（质保期内功能性 Bug 免费修复）

---

*文档版本：v1.0 | 最后更新：YYYY-MM-DD*
