---
trigger: always_on
description: > 本文件为 AI 编码助手（Copilot、Cursor、Claude 等）提供项目上下文，帮助其在本仓库中做出准确、符合规范的代码决策。
---

# NG-ALAIN — Agent 协作指南

> 本文件为 AI 编码助手（Copilot、Cursor、Claude 等）提供项目上下文，帮助其在本仓库中做出准确、符合规范的代码决策。

---

## 项目概述

**NG-ALAIN** 是一个基于 [Angular](https://angular.io/) + [NG-ZORRO Antd](https://ng.ant.design/) 的企业级中后台前端脚手架，遵循 Ant Design 设计价值观。当前版本：**v21.x**（Angular 21，Zoneless 模式）。

- 文档站点：<https://ng-alain.com/zh>
- 源码仓库：<https://github.com/ng-alain/ng-alain>
- 组件库仓库：<https://github.com/ng-alain/delon>

---

## 技术栈

| 层次 | 技术 |
|------|------|
| 框架 | Angular 21（Standalone + Zoneless） |
| UI 组件库 | ng-zorro-antd 21 |
| 业务组件库 | @delon/abc、@delon/chart |
| 主题系统 | @delon/theme（LESS 变量、CSS 工具集） |
| 表单引擎 | @delon/form（JSON Schema 动态表单） |
| 权限控制 | @delon/acl |
| 用户认证 | @delon/auth（SimpleToken / JWTToken） |
| 缓存 | @delon/cache |
| Mock | @delon/mock |
| 工具库 | @delon/util |
| 样式 | LESS |
| 测试框架 | Vitest（`vitest.config.ts`） |
| 代码规范 | ESLint + Prettier + Stylelint |
| 提交钩子 | Husky（pre-commit lint-staged） |
| 脚手架工具 | Angular CLI Schematics（`ng-alain` schematic） |

---

## 目录结构

```
ng-alain/
├── _mock/                        # Mock 数据规则（@delon/mock）
├── _cli-tpl/                     # 自定义 CLI 模板
├── public/                       # 静态资源（直接复制到产物）
├── src/
│   ├── main.ts                   # 应用入口
│   ├── styles.less               # 全局样式入口
│   ├── style-icons.ts            # 按需引入的 Ant Design 图标
│   ├── style-icons-auto.ts       # 自动生成的图标引用
│   ├── app/
│   │   ├── app.ts                # 根组件
│   │   ├── app.config.ts         # 全局 providers 配置（ApplicationConfig）
│   │   ├── core/                 # 核心服务（仅全局单例，严禁在此写业务）
│   │   │   ├── i18n/             # 国际化加载服务（I18NService）
│   │   │   ├── net/              # HTTP 拦截器（DefaultInterceptor）
│   │   │   ├── startup/          # 启动服务（StartupService）
│   │   │   └── index.ts          # 核心导出
│   │   ├── layout/               # 布局组件
│   │   │   ├── basic/            # 基础布局（含顶栏、侧边菜单）
│   │   │   ├── blank/            # 空白布局
│   │   │   └── passport/         # 登录/注册布局
│   │   ├── routes/               # 业务路由模块
│   │   │   ├── routes.ts         # 根路由配置
│   │   │   ├── dashboard/        # 仪表盘示例
│   │   │   ├── delon/            # @delon 组件示例
│   │   │   ├── passport/         # 登录/注册页
│   │   │   ├── exception/        # 异常页（403/404/500）
│   │   │   └── ...               # 其他业务模块
│   │   └── shared/               # 共享资源
│   │       ├── shared-imports.ts # 高频共享组件集合（直接 import 导出）
│   │       ├── shared-delon.module.ts  # @delon 共享模块
│   │       ├── shared-zorro.module.ts  # NG-ZORRO 共享模块
│   │       └── index.ts          # 共享导出
│   ├── assets/                   # 本地静态资源（图片、i18n JSON 等）
│   ├── environments/             # 环境变量
│   └── styles/                   # 全局 LESS 样式
├── angular.json                  # Angular 工作区配置
├── tsconfig.json                 # TypeScript 根配置
├── vitest.config.ts              # Vitest 测试配置
├── eslint.config.mjs             # ESLint 配置
└── stylelint.config.mjs          # Stylelint 配置
```

---

## 常用命令

```bash
# 启动开发服务器
npm start                         # 等价于 ng serve -o

# 构建生产包
npm run build                     # 等价于 ng build

# 代码检查
npm run lint                      # TS + LESS 全量 lint
npm run lint:ts                   # 仅 TypeScript
npm run lint:style                # 仅 LESS

# 运行测试
npm test                          # Vitest 交互模式
npm run test-coverage             # 生成覆盖率报告

# 主题工具
npm run color-less                # 生成色彩 LESS 变量
npm run theme                     # 生成主题 CSS（dark/compact）
npm run icon                      # 更新按需图标列表
```

### CLI Schematics（新增页面）

```bash
# 新增业务模块
ng g ng-alain:module <module-name>

# 新增列表页
ng g ng-alain:list <page-name> -m=<module-name>

# 新增编辑页（模态框）
ng g ng-alain:edit <page-name> -m=<module-name>

# 新增详情查看页
ng g ng-alain:view <page-name> -m=<module-name>

# 新增空白页
ng g ng-alain:empty <page-name> -m=<module-name>
```

---

## 架构约定

### 组件风格

- 项目使用 **Standalone Components**（Angular 17+ 风格），新建组件统一使用 `standalone: true`。
- 变更检测使用 **Zoneless**（`provideZonelessChangeDetection()`），**禁止**引入 `zone.js`。
- 样式文件使用 **LESS**（`angular.json` 中已配置 `inlineStyleLanguage: less`）。
- 组件默认**不生成**单独的 `.less` 样式文件（在 `angular.json` 的 schematics 中配置），如需自定义样式，手动创建并在组件中引用。

### 路由约定

- 所有业务路由通过**懒加载**（`loadChildren`）方式注册于 `src/app/routes/routes.ts`。
- 受认证保护的路由统一使用 `authSimpleCanActivate` + `authSimpleCanActivateChild` 守卫。
- 路由守卫 `startPageGuard` 用于处理起始页跳转逻辑。

### 共享模块

- `shared-imports.ts`：整个项目高频使用的 Standalone 组件/指令/管道直接导出，在各业务页面按需 import。
- `shared-delon.module.ts`：@delon 系列模块的聚合（用于 NgModule 模式的导入）。
- `shared-zorro.module.ts`：NG-ZORRO 模块的聚合。
- **原则**：某个模块/组件在 **2 个以上**业务页面中使用，才应放入 shared；否则在使用处直接 import。

### HTTP 与服务端交互

- 使用 `@delon/theme` 提供的 `_HttpClient`（封装了通用错误处理与参数序列化）代替原生 `HttpClient`。
- 拦截器注册顺序（`app.config.ts`）：`authSimpleInterceptor` → `defaultInterceptor`。
- `DefaultInterceptor`（`src/app/core/net/default.interceptor.ts`）负责：统一请求前缀、处理 HTTP 异常、处理业务异常（如 `{ code: 0, msg: '...' }`）。
- 开发环境跨域通过 `proxy.conf.js` 配置 Angular CLI 代理解决，**不要**在生产代码中绕过 CORS。

### 启动初始化

- `StartupService.load()` 在 `APP_INITIALIZER` 中执行，用于加载菜单、用户信息、权限数据等应用级数据。
- 若需要在启动前请求接口（如获取菜单），应在 `StartupService` 内完成，**不要**在组件的 `ngOnInit` 中做全局性初始化。

### Mock 数据

- Mock 规则统一放在 `_mock/` 目录下，以模块为单位分文件。
- Mock 仅在开发/测试环境下生效（通过 `environment.ts` 中的 `mock` 配置控制）。
- 确保 Mock 数据的接口结构与后端真实接口保持一致。
- 完成联调后及时注释或删除对应 Mock 规则。

### ACL 权限


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ng-alain/ng-alain](https://github.com/ng-alain/ng-alain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
