---
trigger: always_on
description: > High-signal guidance for OpenCode sessions working on this repository
---

# AGENTS.md
> High-signal guidance for OpenCode sessions working on this repository

## Project Overview
Forge Admin is a Vue3 + Spring Boot enterprise admin framework with microkernel plugin architecture:
- **Backend**: Java 17 + Spring Boot 3 + MyBatis-Plus + Sa-Token
- **Frontend**: Vue 3 + Naive UI + Vite + UnoCSS
- Core features: RBAC, multi-tenant, code generation, flow engine, message center

## Directory Structure
```
forge/                          # Backend root
├── forge-admin/                # Main application entry
├── forge-framework/            # Core framework
│   ├── forge-plugin-parent/    # Plugins (system/generator/job/message/flow)
│   └── forge-starter-parent/   # Starters (auth/cache/config/crypto etc.)
forge-admin-ui/                 # Frontend project
code-copilot/                   # AI coding assistant rules & change management
.opencode/                      # OpenCode configuration & rules
```

## Environment Requirements
- JDK 17+
- Node.js 18+
- pnpm 8+
- MySQL 8.0+
- Redis 6.0+

## Common Commands

### 渐进式开发流程（推荐）
> 遵循 No Spec No Code 原则，使用 `/` 命令触发渐进式开发流程
- `/spec-init` - 初始化项目上下文
- `/propose <需求描述>` - 创建变更提案
- `/apply <变更名>` - 执行编码
- `/fix <变更名>` - Review后修正
- `/review <变更名>` - 两阶段审查
- `/test <变更名>` - 生成测试
- `/archive <变更名>` - 归档沉淀

> 变更产物统一存放在 `code-copilot/changes/[变更名]/` 目录下

#### Backend
```bash
# Build entire project
cd forge && mvn clean install

# Run admin service
cd forge/forge-admin && mvn spring-boot:run

# Run flow service (optional)
cd forge/forge-flow && mvn spring-boot:run
```
> Service runs on `http://localhost:8080` by default

#### Frontend
```bash
# Install dependencies
cd forge-admin-ui && pnpm install

# Start dev server
pnpm dev

# Build for production
pnpm build

# Lint & fix
pnpm lint:fix
```
> Dev server runs on `http://localhost:5173` by default, default credentials: admin/123456

## Key Conventions (Follow These Strictly)
### 1. 渐进式Spec规范
- No Spec No Code：没有确认的Spec不准写代码
- Spec包含完整的背景、现状分析、功能点、业务规则、数据/接口变更、风险、测试策略
- 所有待澄清问题解决后才能进入/apply阶段
- Spec与代码冲突时，以Spec为准，先修Spec再改代码

### 2. Code Generation
- Use built-in `forge-plugin-generator` for CRUD modules, do not write boilerplate manually
- Frontend CRUD pages use `AiCrudPage` component (see `.opencode/instructions/code-rules.md` for usage)

### 3. Backend APIs
- Follow REST conventions: `GET /page` (list), `GET /{id}` (detail), `POST /` (create), `PUT /` (update), `DELETE /{id}` (delete)
- Return unified `RespInfo.success(data)` / `RespInfo.error(msg)`
- Use `@ApiDecrypt`/`@ApiEncrypt` annotations for sensitive interfaces
- **SQL 编写规范**：查询类 SQL 必须写在 Mapper XML 中，禁止在 Service 层使用 `LambdaQueryWrapper` 构建查询条件。原因：
  1. XML 中的 SQL 可被数据权限拦截器（`DataScopeInterceptor`）按 mapperMethod 精确匹配和改写
  2. XML SQL 更易于审查、优化和维护，复杂条件（如虚拟组织 ALL 后缀判断）在 XML 中表达更清晰
  3. 便于后续扩展（加字段、加条件、加索引提示等）无需改 Java 代码
- 仅以下场景允许使用 `LambdaQueryWrapper`：单表简单 CRUD（selectById、insert、updateById、deleteById 等 MyBatis-Plus 内置方法）

### 4. Database
- All business tables must include base fields: `id`, `tenant_id`, `create_by`, `create_time`, `create_dept`, `update_by`, `update_time`
- Use `utf8mb4` charset, `InnoDB` engine
- Create indexes for frequently queried fields, follow leftmost prefix rule for composite indexes
- Never commit database credentials to repo (local configs are in `.gitignore`)
- **租户ID规则**：SQL初始化数据中，业务数据（字典、配置等需被租户查询到的数据）的 `tenant_id` 必须设为 `1`（默认租户），**不能设为 `0`**。原因是项目的 `TenantLineInnerInterceptor` 会自动在所有查询中追加 `WHERE tenant_id = <当前登录用户租户ID>`，`tenant_id=0` 的数据对非零租户用户不可见。`sys_resource`（菜单/权限）表不在租户拦截范围内，其 `tenant_id` 保持 `0` 即可

### 5. Frontend
- API calls use `request` utility from `@/utils/request`
- Encrypted API calls use `postEncrypt` from `@/utils/encrypt-request`
- Use UnoCSS for styling, avoid unnecessary custom CSS
- Follow existing component patterns, use Naive UI components where possible

## Important Gotchas
- **First time setup**: Copy `application-dev.example.yml` to `application-dev.yml` in backend modules and configure local database/Redis
- Frontend: Copy `.env.example` to `.env.local` for local environment config
- No Spec No Code: Always refer to requirements specs first before writing code, follow code-copilot workflow for changes
- Never push to master branch directly, use feature branches
- All code must compile and pass existing tests before commit

## Reusable Components

### RegionTreeSelect — 行政区划树选择组件

**路径**: `forge-admin-ui/src/components/RegionTreeSelect.vue`

**功能**: 一次性加载指定 rootCode 下的完整区划树（含虚拟组织），虚拟组织节点不可选中，支持数据权限控制。

**用法**:
```vue
<!-- 基本用法（默认内蒙150000，启用数据权限） -->
<RegionTreeSelect v-model="form.regionCode" />

<!-- 自定义参数 -->
<RegionTreeSelect v-model="form.regionCode" root-code="150000" :data-right="true" />
```

**在 searchSchema 中使用**（搜索表单中的区划筛选）:
```javascript
{
  field: 'regionCode',
  label: '行政区划',
  type: 'treeSelect',
  props: { placeholder: '请选择行政区划', clearable: true, filterable: true },
  options: () => regionOptions.value,  // 需要页面调用 loadRegionOptions 加载
}
```

**Props**:
| Prop | Type | Default | 说明 |
|------|------|---------|------|
| modelValue | String/Number | undefined | 绑定值（区划编码） |
| rootCode | String | '150000' | 根区域编码 |
| dataRight | Boolean | true | 是否启用数据权限过滤 |
| placeholder | String | '请选择行政区划' | 占位文本 |
| clearable | Boolean | true | 是否可清除 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaomindong1996/forge-admin](https://github.com/yaomindong1996/forge-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
