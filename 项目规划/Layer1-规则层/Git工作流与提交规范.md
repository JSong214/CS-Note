# Git 工作流与提交规范

> **文档定位**：Layer 1 — 规则层（每次开新项目必读）
> **适用场景**：独立全栈开发者，覆盖分支策略、提交规范、代码审查、自动化钩子全流程

---

## 一、分支策略（简化版 GitFlow）

### 1.1 分支结构

```
main          ← 永远指向生产可部署的代码（受保护，禁止直接 push）
  └── develop ← 日常集成分支，功能开发完成后合入此处
        ├── feature/user-register     ← 每个垂直切片/功能一个分支
        ├── feature/post-like
        ├── feature/payment-flow
        ├── fix/login-token-expire    ← Bug 修复分支（从 develop 分出）
        └── hotfix/security-patch     ← 紧急修复（从 main 分出，修完合回 main + develop）
```

### 1.2 各分支职责与生命周期

| 分支类型 | 命名规范 | 来源 | 合入目标 | 生命周期 |
|---|---|---|---|---|
| `main` | 固定 | — | — | 永久 |
| `develop` | 固定 | `main` | — | 永久 |
| `feature/*` | `feature/<功能名>` | `develop` | `develop` | 功能完成后删除 |
| `fix/*` | `fix/<问题描述>` | `develop` | `develop` | 修复完成后删除 |
| `hotfix/*` | `hotfix/<紧急问题>` | `main` | `main` + `develop` | 修复完成后删除 |
| `release/*` | `release/v<版本号>` | `develop` | `main` + `develop` | 发版完成后删除 |

### 1.3 分支操作示例

```bash
# ✅ 开始一个新功能
git checkout develop
git pull origin develop
git checkout -b feature/user-register

# ✅ 功能完成，合回 develop
git checkout develop
git pull origin develop
git merge --no-ff feature/user-register  # 保留合并节点，方便追溯
git branch -d feature/user-register
git push origin develop

# ✅ 紧急修复生产 Bug
git checkout main
git pull origin main
git checkout -b hotfix/fix-payment-null-pointer
# ... 修复 ...
git checkout main
git merge --no-ff hotfix/fix-payment-null-pointer
git tag -a v1.0.1 -m "fix: 修复支付空指针问题"
git push origin main --tags

git checkout develop
git merge --no-ff hotfix/fix-payment-null-pointer
git push origin develop
git branch -d hotfix/fix-payment-null-pointer
```

### 1.4 分支保护规则（在 GitHub/GitLab 配置）

```
main 分支：
  ✅ 禁止直接 push
  ✅ 要求 Pull Request 审查（即使是独立开发者，也强制 PR 流程）
  ✅ 要求 CI 检查通过
  ✅ 禁止强制推送

develop 分支：
  ✅ 禁止直接 push
  ✅ 要求 CI 检查通过
```

---

## 二、Commit 消息规范（Conventional Commits）

### 2.1 格式定义

```
<type>(<scope>): <subject>

[可选的 body：详细说明]

[可选的 footer：关联 Issue、Breaking Change]
```

### 2.2 Type 类型一览

| Type | 用途 | 示例 |
|---|---|---|
| `feat` | 新功能 | `feat(auth): 完成用户注册接口及前端表单` |
| `fix` | Bug 修复 | `fix(order): 修复订单状态流转越权问题` |
| `test` | 测试相关 | `test(user): 补充注册接口悲观路径测试` |
| `docs` | 文档更新 | `docs(api): 更新 Swagger 文档至 v1.2` |
| `refactor` | 重构（不改功能） | `refactor(handler): 提取统一错误处理中间件` |
| `perf` | 性能优化 | `perf(query): 为 users 表 email 字段添加索引` |
| `style` | 代码格式（不影响逻辑） | `style: 统一换行符为 LF` |
| `chore` | 构建/工具配置 | `chore(ci): 配置 GitHub Actions 自动化测试` |
| `revert` | 回滚 | `revert: revert feat(auth): 完成用户注册接口` |

### 2.3 Scope 作用域建议

```
# 后端 scope
auth / user / order / payment / file / middleware / db / config

# 前端 scope
login / dashboard / form / router / store / components / api

# 工程 scope
ci / docker / nginx / deps / build
```

### 2.4 Good vs Bad Commit 示例

```bash
# ❌ 糟糕的 Commit（信息无意义）
git commit -m "fix"
git commit -m "update"
git commit -m "修改了一些东西"

# ✅ 优秀的 Commit（清晰、可追溯）
git commit -m "feat(user): 新增用户头像上传功能，支持 JPG/PNG，限制 2MB"
git commit -m "fix(auth): 修复 Refresh Token 在并发请求下重复消耗的竞态问题"
git commit -m "perf(query): 添加 orders 表 (user_id, status) 复合索引，优化订单列表查询"

# ✅ 带 footer 关联 Issue
git commit -m "fix(payment): 修复微信支付回调签名验证失败

当前环境签名算法与微信新版 API 不匹配。
升级签名算法为 RSA-SHA256。

Closes #42"
```

---

## 三、Pre-Commit 自动化检查

### 3.1 配置（前端 husky + lint-staged）

```bash
# 安装（在项目根目录）
npm install --save-dev husky lint-staged

# 初始化 husky
npx husky init
```

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."

# 前端检查
cd frontend && npx lint-staged

# 后端检查
cd ../backend
go vet ./...           # Go 静态分析
go build ./...         # 确保可以编译
go test ./... -short   # 跑快速单元测试（跳过集成测试）

echo "✅ All checks passed!"
```

```json
// package.json (frontend)
{
  "lint-staged": {
    "src/**/*.{ts,vue}": [
      "eslint --fix",
      "prettier --write"
    ],
    "src/**/*.{css,scss}": [
      "prettier --write"
    ]
  }
}
```

### 3.2 Commit Message 格式校验

```bash
# .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# 使用 commitlint 校验格式
npx --no -- commitlint --edit "$1"
```

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'test', 'docs', 'refactor',
      'perf', 'style', 'chore', 'revert'
    ]],
    'subject-max-length': [2, 'always', 100],
    'subject-empty': [2, 'never'],
  }
}
```

---

## 四、Pull Request 规范

### 4.1 PR 标题格式

```
<type>(<scope>): <简短描述>

# 示例：
feat(user): 完成用户注册、登录、Token 刷新完整流程
fix(payment): 修复微信支付回调幂等性问题
```

### 4.2 PR 描述模板

```markdown
## 变更内容
<!-- 简要说明本次 PR 做了什么 -->

## 关联需求/Issue
<!-- 关联的需求文档章节或 Issue 编号 -->
Closes #

## 测试情况
- [ ] 单元测试覆盖关键逻辑
- [ ] 集成测试通过
- [ ] 手动测试了以下场景：
  - 场景 1：...
  - 场景 2：...

## 截图（如有 UI 变更）
<!-- 粘贴 Before/After 截图 -->

## Checklist
- [ ] 代码符合 Global Rules 规范
- [ ] 无调试代码残留（console.log / fmt.Println）
- [ ] 敏感信息未硬编码
- [ ] 相关文档已更新（如 API 文档、README）
```

### 4.3 Squash Merge 策略

```bash
# 合并到 develop/main 时，使用 Squash and Merge
# 将 feature 分支的多个 commit 压缩为一个干净的提交
# 保持主分支历史简洁可读

# 在 GitHub 设置中：
# Settings → General → Merge button → 勾选 "Allow squash merging"
# 默认提交消息使用 PR 标题
```

---

## 五、标签与版本管理

### 5.1 版本号规范（语义化版本 SemVer）

```
v<MAJOR>.<MINOR>.<PATCH>

MAJOR：不兼容的 API 变更
MINOR：向后兼容的新功能
PATCH：向后兼容的 Bug 修复

示例：
v1.0.0  → 首次生产发布
v1.1.0  → 新增用户头像功能
v1.1.1  → 修复头像上传 Bug
v2.0.0  → 数据库结构重构，不兼容升级
```

### 5.2 打标签流程

```bash
# 发布前，确保 develop 合入 main 并通过 CI
git checkout main
git pull origin main

# 打标签（附带发布说明）
git tag -a v1.2.0 -m "release: v1.2.0

新增功能：
- 用户头像上传
- 订单导出 Excel

Bug 修复：
- 修复登录页在 Safari 上的样式问题"

git push origin v1.2.0
```

---

## 六、.gitignore 标准配置

```gitignore
# ========== 环境变量（绝对不能提交）==========
.env
.env.local
.env.production
.env.staging
*.secret
*.key
*.pem

# ========== 依赖目录 ==========
node_modules/
vendor/           # 如果使用 go mod vendor

# ========== 构建产物 ==========
dist/
build/
*.out
*.exe
*.dll

# ========== IDE 配置 ==========
.vscode/settings.json
.idea/
*.swp
*.swo
.DS_Store

# ========== 日志文件 ==========
*.log
logs/

# ========== 测试覆盖率 ==========
coverage.out
coverage.html

# ========== 数据库相关 ==========
*.db
*.sqlite
dump.sql
backup_*.sql

# ========== Docker ==========
docker-compose.override.yml
```

---

## 七、快速参考卡

```
日常开发流程：
1. git checkout develop && git pull           # 同步最新代码
2. git checkout -b feature/<功能名>           # 创建功能分支
3. [开发 + 提交，遵循 Conventional Commits]
4. git push origin feature/<功能名>           # 推送到远端
5. 创建 PR → develop（填写 PR 模板）
6. CI 通过 → Squash Merge → 删除分支

紧急 Bug 修复：
1. git checkout main && git pull
2. git checkout -b hotfix/<问题>
3. [修复 + 提交]
4. 合入 main（打 tag）→ 合入 develop
```

---

*文档版本：v1.0 | 创建日期：2026-04-13 | 归属：Layer 1 — 规则层*
