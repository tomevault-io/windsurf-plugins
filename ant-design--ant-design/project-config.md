---
trigger: always_on
description: > 本文件为 AI 编程助手提供项目上下文和开发规范。
---

# Ant Design 项目开发指南

> 本文件为 AI 编程助手提供项目上下文和开发规范。

## 项目信息

- React 组件库，发布为 npm 包 `antd`
- 使用 TypeScript 和 React 开发
- 采用 CSS-in-JS 架构（基于 `@ant-design/cssinjs`）
- 支持 Design Token 主题系统、暗色模式、RTL 布局、SSR、国际化（150+ 语言）

### 项目结构

```
ant-design/
├── components/              # 组件源代码（84+ 组件）
│   ├── component-name/      # 单个组件目录
│   │   ├── ComponentName.tsx      # 主组件实现
│   │   ├── demo/                  # 演示代码（*.tsx 和 *.md）
│   │   ├── style/                 # 样式系统（index.ts / token.ts）
│   │   ├── __tests__/            # 单元测试
│   │   ├── index.en-US.md        # 英文文档
│   │   ├── index.zh-CN.md        # 中文文档
│   │   └── index.tsx             # 导出入口
│   ├── _util/                   # 共享工具函数库
│   ├── theme/                   # 主题系统
│   └── locale/                  # 国际化文本
├── tests/                       # 测试工具和共享测试
├── docs/                        # 站点文档
├── CHANGELOG.zh-CN.md           # 中文更新日志
└── CHANGELOG.en-US.md           # 英文更新日志
```

---

## 通用编码规范

- 判断数据类型时，优先使用 `components/_util/is.ts` 中已有的方法，例如 `isNumber`、`isString`、`isPlainObject`、`isFunction`、`isThenable`、`isPrimitive`、`isNonNullable`。
- 仅当 `components/_util/is.ts` 中没有合适方法，或当前场景需要更严格、更特殊的判断逻辑时，再使用内联 `typeof`、`instanceof` 等判断方式。

## 样式优先级规范

- 当组件根节点同时支持 ConfigProvider 的 `styles.root`、ConfigProvider 的 `style`、组件自身的 `styles.root` 和组件自身的 `style` 时，优先级从低到高为：ConfigProvider `styles.root` -> ConfigProvider `style` -> 组件 `styles.root` -> 组件 `style`；如在 util 层预合并根节点样式，预合并结果内部应保持前三者的同样顺序，组件自身 `style` 保持最终最高优先级。

---

## Demo 导入规范

- 常规 `components/**/demo/` 文件在引入 Ant Design 组件、组件内部模块、工具方法、变量、类型定义时，一律使用绝对路径导入，不使用相对路径导入。
- `components/**/demo/_semantic*.tsx` 属于语义文档专用 demo，是例外场景：允许通过相对路径引用 `.dumi/hooks/useLocale`、`.dumi/theme/common/*` 等站点侧辅助模块。
- `.dumi/` 目录内部的站点实现文件可按现有目录结构使用相对路径引用本目录模块；当引用仓库内 Ant Design 组件入口时，优先使用项目公开入口或已配置别名。
- 允许的导入形式应优先使用项目公开入口或已配置别名，例如：`antd`、`antd/es/*`、`antd/lib/*`、`antd/locale/*`、`@@/*`。
- `.dumi/*` 不是仓库通用的 TS 路径别名；如需引用 `.dumi` 内部模块，请按文件位置使用相对路径。
- 常规 demo 文件中，禁止使用 `..`、`../xxx`、`../../xxx`、`./xxx` 这类相对路径去引用组件实现、内部模块、方法、变量、类型，包含跨 demo、跨目录复用的场景。
- 常规 demo 与 `.dumi` 文件之间不要互相相对引用（`_semantic*.tsx` 等站点语义 demo 复用 `.dumi` 辅助模块除外）。如果需要复用少量逻辑，优先内联，或提取到可通过绝对路径访问的公共位置。

## Test 导入规范

- 本规范适用于 `components/**/__tests__/` 下的测试文件。
- 在这些目录下引入 Ant Design 组件，或引入组件内部模块、工具方法、变量、类型定义时，一律使用相对路径导入，不使用绝对路径导入。
- 测试文件应优先从当前组件目录、相邻内部模块或共享测试工具目录通过相对路径引用，例如：`..`、`../index`、`../xxx`、`../../_util/*`、`../../../tests/shared/*`。
- 禁止在 `__tests__` 目录下使用 `antd`、`antd/es/*`、`antd/lib/*`、`antd/locale/*`、`.dumi/*`、`@@/*` 这类绝对路径或别名路径去引用仓库内代码。
- 如需引用仓库外第三方依赖，仍按依赖包名正常导入，例如 `react`、`@testing-library/react`、`dayjs`。

---

## 文档规范

### API 表格格式

英文版：

| Property | Description | Type | Default | Version | [Global Config](/components/config-provider#component-config) |
| --- | --- | --- | --- | --- | --- |
| disabled | Whether the component is disabled | boolean | false | - | × |
| loadingIcon | (Only supports global configuration) Custom loading icon | ReactNode | - | - | 6.2.0 |
| type | Button type | `primary` \| `default` | `default` | - | ✔ |

中文版：

| 参数 | 说明 | 类型 | 默认值 | 版本 | [全局配置](/components/config-provider-cn#component-config) |
| --- | --- | --- | --- | --- | --- |
| disabled | 是否禁用 | boolean | false | - | × |
| loadingIcon | (仅支持全局配置) 自定义加载图标 | ReactNode | - | × | 6.2.0 |
| type | 按钮类型 | `primary` \| `default` | `default` | - | ✔ |

列说明：

- 参数：按字母顺序排列，忽略 className, style, onClick, onKeyDown 等通用属性, onChange, onClick 等事件回调放在最后
- 说明：简洁描述参数作用，如果仅支持全局配置需在描述中用括号注明
- 类型：使用 TypeScript 定义的类型
- 默认值：字符串用反引号，布尔/数字直接写，无默认值用 `-`
- 版本：新增属性需声明引入的版本号；上个大版本已存在属性标注 `-`；仅支持全局配置的属性标注 `×`
- 全局配置：支持全局配置的属性需标注版本号；上个大版本已支持的标注 `✔`；不支持全局配置的属性标注 `×`

### 文档锚点 ID 规范

- 中文标题必须手动指定英文锚点：`## 中文标题 {#english-anchor-id}`
- 锚点 ID 符合 `^[a-zA-Z][\w-:\.]*$`，长度不超过 32 字符
- FAQ 章节下的锚点必须以 `faq-` 为前缀
- 同一问题的中英文锚点保持一致

### 国际化规范

- 本地化配置文件：`components/locale/`，命名如 `zh_CN.ts`、`en_US.ts`
- 添加或修改本地化配置时，需同时修改所有语言文件
- 类型入口：`components/locale/index.tsx`

---

## PR 规范

### 标题与内容

- PR 标题始终使用英文，格式：`类型: 简短描述`
- PR 内容默认使用英文，可根据用户语言习惯决定使用中文或英文
- 示例：`fix: fix button style issues in Safari browser`

### PR 模板（必须使用）

- 英文模板：`.github/PULL_REQUEST_TEMPLATE.md`
- 中文模板：`.github/PULL_REQUEST_TEMPLATE_CN.md`
- 使用 `gh pr create` 创建 PR 时，必须手动填充模板内容

### 分支策略

- 新特性开发需基于 `feature` 分支，PR 目标分支也需为 `feature`
- 其余提交至 `master` 分支
- 分支命名规范：
  - 功能开发：`feat/description-of-feature`
  - 问题修复：`fix/issue-number-or-description`
  - 文档更新：`docs/what-is-changed`
  - 代码重构：`refactor/what-is-changed`

### PR 改动类型

- 🆕 新特性提交
- 🐞 Bug 修复
- 📝 文档改进
- 📽️ 演示代码改进
- 💄 样式/交互改进
- 🤖 TypeScript 更新
- 📦 包体积优化
- ⚡️ 性能优化
- 🌐 国际化改进

---

## Changelog 规范

### 适用范围

- 本节仅适用于用户明确要求收集/生成 changelog、准备 release PR、版本发布，或正在编辑 `CHANGELOG.en-US.md` / `CHANGELOG.zh-CN.md` 的场景。
- 普通功能、修复、文档、demo PR 不要求直接修改 `CHANGELOG.en-US.md` / `CHANGELOG.zh-CN.md`；代码 CR 时也不应仅因缺少 CHANGELOG 文件改动而提出 finding。
- 普通 PR 如需填写 PR 模板中的 Change Log，仅描述本 PR 对用户或开发者的影响，或按模板填写 `N/A` / `No changelog required` / `无需更新日志`。正式 CHANGELOG 由 release owner 在发布流程中统一整理。

### 核心原则

- 文件位置：`CHANGELOG.en-US.md` 和 `CHANGELOG.zh-CN.md`
- 必须同时提供中英文两个版本
- 忽略用户无感知的改动（内部重构、纯测试更新、工具链优化等）
- 描述"用户或开发者能感知到的变化"，而非"具体的实现细节"；不要用"传递某配置"、"修改 runtime"、"增加 metadata"等内部实现视角代替效果描述
- 涉及 Design Token 时，必须列出具体新增、修复或变更的 token 名称
- `rc-component` 或其他运行时依赖升级默认不作为 changelog；但如果带来用户可感知能力、行为变化、类型定义改进或包体积收益，必须单独核查并描述影响
- 尽量给出 PR 链接，并统一添加贡献者链接

### 格式规范

#### 条目格式


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ant-design/ant-design](https://github.com/ant-design/ant-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
