### 📂 推荐的项目目录结构概览

这个结构采用了**功能分层**与**模块化**相结合的设计思路。

```
project-root/
├── .husky/                 # Git Hooks 配置 (如 commit-msg 校验)
├── .vscode/                # VSCode 统一配置 (确保团队编辑器设置一致)
├── public/                 # 静态资源 (不经过构建工具处理，如 favicon, index.html)
├── src/                    # 源代码目录
│   ├── api/                # 接口管理 (API请求层)
│   ├── assets/             # 静态资源 (经过构建工具处理，如 images, fonts)
│   ├── components/         # 通用组件库 (非业务相关)
│   │   ├── common/         # 全局基础组件 (如 Button, Input)
│   │   └── business/       # 全局业务组件 (如 UserAvatar, GlobalSearch)
│   ├── config/             # 全局静态配置 (如常量, 枚举, 菜单配置)
│   ├── directives/         # 自定义指令
│   ├── hooks/              # 组合式函数 / Hooks (逻辑复用层)
│   ├── layout/             # 布局组件 (如 Sidebar, Header, Footer)
│   ├── router/             # 路由配置
│   ├── store/              # 全局状态管理 (Pinia / Redux)
│   ├── styles/             # 全局样式文件 (Sass/Less/CSS)
│   ├── types/              # TypeScript 类型定义 (*.d.ts)
│   ├── utils/              # 工具函数 (纯函数，不包含业务逻辑)
│   ├── views/              # 页面级组件 (路由对应的页面)
│   ├── App.vue             # 根组件
│   └── main.ts             # 入口文件
├── tests/                  # 测试文件 (Unit / E2E)
├── .env.development        # 开发环境配置
├── .env.production         # 生产环境配置
├── .eslintrc.js            # Linter 配置
├── .prettierrc             # 格式化配置
├── package.json            # 依赖管理
├── tsconfig.json           # TypeScript 配置
└── vite.config.ts          # 构建工具配置 (或 webpack/next)
```

------



### 🔍 核心目录深度解析

为了让你更好地理解每个目录的职责，以下是详细的划分逻辑和使用建议：



#### 1. `src/api/` (网络层)

不要在组件中直接写 `axios.get`。将所有后端接口调用封装在这里。

- **建议结构：** 按业务模块划分文件，例如 `user.ts`, `order.ts`, `product.ts`。
- **Senior Tip:** 创建一个核心的 `request.ts` 或 `interceptor.ts` 来处理拦截器（Token 注入、统一错误处理）。



#### 2. `src/components/` vs `src/views/` (视图层)

这是新手最容易混淆的地方。

- **`components/` (木偶组件):** 这里的组件应该是**纯粹的 UI**，不包含具体的路由逻辑或复杂的 API 调用。它们通过 props 接收数据，通过 emit 抛出事件。
  - *Common:* 整个项目通用的基础 UI，如“二次封装的 Table”。
  - *Business:* 涉及特定业务逻辑但会被多处复用的组件，如“用户选择器弹窗”。
- **`views/` (智能组件/页面):** 对应路由的页面。它们负责调用 API，获取数据，组装 components，并处理页面级逻辑。



#### 3. `src/hooks/` (逻辑层 - 现代前端核心)

存放 Vue 的 Composables 或 React 的 Hooks。

- **作用：** 剥离组件中的业务逻辑。如果你的组件代码超过 300 行，通常意味着你需要提取逻辑到 hooks 中。
- **示例：** `useTableQuery` (处理表格的分页和搜索), `useTheme` (切换主题), `useCountdown` (倒计时)。



#### 4. `src/utils/` (工具层)

存放通用的纯函数。

- **特点：** 输入确定，输出必定确定，不依赖组件实例。
- **示例：** `formatDate.ts` (时间格式化), `validate.ts` (正则校验), `storage.ts` (LocalStorage 封装)。



#### 5. `src/store/` (状态层)

不要把所有数据都放 Store。

- **准则：** 只有**跨页面/跨组件层级**共享的数据才放入 Store（如用户信息、权限列表、全局设置）。页面内部的状态请使用组件内的 `ref/state` 或 `hooks`。



#### 6. `src/types/` (类型层 - TypeScript)

- **建议：** 存放后端的实体模型定义（Model Interface）和全局通用的类型定义。
- **目录：** 可以建立 `api.d.ts` (接口请求响应类型) 和 `entity.d.ts` (数据库实体类型)。

------



### 💡 资深开发者的 3 个最佳实践

1. **统一导出 (Barrel Export):** 在每个文件夹（如 `components/common`）下创建一个 `index.ts`，将内部组件统一导出。
   - *Bad:* `import MyButton from '@/components/common/MyButton/MyButton.vue'`
   - *Good:* `import { MyButton, MyInput } from '@/components/common'`
2. **路径别名 (Path Aliases):** 务必在 `tsconfig.json` 和构建配置中设置 `@` 指向 `src` 目录。避免使用 `../../../../` 这种地狱路径。
3. **模块化 Views:** 如果 `views` 下的某个页面非常复杂（例如 `Dashboard`），请在 `views/Dashboard/` 下建立私有的 `components/` 文件夹。不要把只属于 Dashboard 的组件放到全局 `src/components` 中，这能保持全局目录的整洁。