---
trigger: always_on
description: 面向 AI coding agent 的项目说明。人类读者请看 [README.md](./README.md) / [README.zhCN.md](./README.zhCN.md)。
---

# AGENTS.md

面向 AI coding agent 的项目说明。人类读者请看 [README.md](./README.md) / [README.zhCN.md](./README.zhCN.md)。

## 1. 项目是什么

**Riven Mirror（极镜）** —— Warframe 的裂罅 MOD（Riven Mod）分析与配装计算器。

核心价值不在 UI，而在 `src/warframe/` 下那套**游戏伤害数学模型**：识别玄卡词条、计算加成、穷举最优 MOD 组合、模拟对敌伤害曲线。UI 只是这套模型的展示层。

改动业务逻辑前请先理解 `src/warframe/modbuild.ts` 与 `src/warframe/codex/`，不要凭直觉改公式。

## 2. 技术栈

| 领域 | 选型 |
| --- | --- |
| 框架 | Vue 3（`<script lang="ts">` + **类组件**） |
| 类组件 | `vue-facing-decorator`（不是 vue-property-decorator） |
| 构建 | Vite + `@vitejs/plugin-vue` + `@vitejs/plugin-vue-jsx` |
| 语言 | TypeScript（`strict: false`，legacy 装饰器） |
| UI 库 | Element Plus（全量注册） |
| 状态 | Vuex 4 |
| 路由 | Vue Router（history 模式） |
| i18n | vue-i18n（**legacy 模式**，模板里用 `$t` / `$d`） |
| 样式 | Less |
| 图表 | ECharts 6（按需 `use()` 注册） |
| 桌面端 | Electron + electron-builder（`vite-plugin-electron`） |
| PWA | `vite-plugin-pwa`（injectManifest，自定义 `sw.js`） |

## 3. 命令

包管理器是 **bun**（锁文件 `bun.lock`），不要用 npm/yarn：

```bash
bun install          # 装依赖（含 data 子模块请先 git submodule update --init）
bun run dev          # 开发服务器
bun run build        # 生产构建 -> dist/
bun run type-check   # vue-tsc 类型检查（构建不做类型检查，改完请手动跑）
bun run electron:dev # Electron 开发
bun run electron:build
```

数据子模块（`data/`，独立 git 仓库 `riven-mirror-data`，同样用 bun）：

```bash
cd data
bun install
bun run fetch      # 从 DE 官方 API + Wiki 抓原始数据 -> tmp/（需要外网）
bun run build      # 生成 dist/*.json 与 dist/weapons.data（protobuf 二进制）
bun run build:all  # clean + fetch + build
```

注意：`data` 的 `build` 在缺少 `tmp/` 抓取数据时会跳过依赖原始数据的步骤（STEP2/STEP3），只重新编码 protobuf。

## 4. 目录地图

```
src/
├── warframe/            ★ 领域核心，纯 TS，不依赖 Vue
│   ├── codex/           静态游戏数据 + 数据类（武器/战甲/MOD/敌人/词条…）
│   │   ├── *.data.ts    手工维护的数据表（体积大，改动请谨慎）
│   │   ├── weapon.ts    WeaponDatabase：加载 protobuf 武器数据
│   │   └── index.ts     Codex 门面 + 统一 re-export
│   ├── modbuild.ts      ★ 配装计算基类（加成叠加、面板计算）
│   ├── gunmodbuild.ts / meleemodbuild.ts / warframebuild.ts / companionbuild.ts
│   │                    各类型配装的具体实现
│   ├── rivenmod.ts      ★ 裂罅 MOD 解析 / 评分 / 生成
│   ├── status.ts        异常状态（触发/叠层）
│   ├── translate.ts     游戏术语本地化（Translator）
│   └── worldstat.ts     世界状态（警报/裂缝）
├── components/          通用组件（33 个）+ vse/（技能编辑器子组件）
├── views/               页面级组件
│   ├── build/           配装编辑器（BaseBuildEditor.ts 是共享基类）
│   ├── buildview/       配装只读展示
│   └── music/           Shawzin 乐器编辑器（Tone.js）
├── store/               Vuex：modules/{mod,user,theme,build}.ts（class 形式的 Module）
├── i18n/                lang/*.json + plugin.ts（动态加载语言包）
├── search/              全局搜索 + 拼音首字母匹配
├── service/             外部服务（HMT 统计、wiki 抓取、user API）
├── less/                全局样式，ele/ 下是 Element 主题覆写
├── main.ts              应用装配入口
└── background.ts        Electron 主进程

data/                    git submodule：数据抓取与构建流水线
└── dist/                产物，被 src 直接 import
```

## 5. 关键约定（必须遵守）

### 5.1 组件一律写成类组件

```vue
<script lang="ts">
import { Vue, Component, Prop, Watch } from "vue-facing-decorator";
import Child from "@/components/Child.vue";

@Component({ components: { Child } })
export default class MyView extends Vue {
  @Prop() value: string;        // props
  count = 0;                    // data
  get double() { return this.count * 2; }   // computed
  @Watch("value") onChange() {} // watch
  mounted() {}                  // 生命周期
}
</script>
```

- **不要**改写成 `defineComponent` / `<script setup>` / Composition API。全项目 73 个组件风格统一，混用会让维护成本爆炸。
- 一个方法上**不要**叠多个 `@Watch`，`vue-facing-decorator` 不支持；写 `@Watch("a", ...)` 分开或在方法里合并逻辑。
- 基类（如 `BaseBuildEditor`）必须用 `@Component` 装饰，子类通过 `extends` 继承。

### 5.2 Vuex 通过 `this.$store` 访问

本项目**不用** `vuex-class`（它不兼容 Vue 3）。写法：

```ts
get bigScreen(): boolean { return this.$store.getters.bigScreen; }
setBuild(build: ModBuild) { return this.$store.dispatch("setBuild", build); }
```

### 5.3 i18n

- 模板里用 `$t("key")`、`$d(date, "short")`。
- TS 里用 `import { i18n } from "@/i18n"` 这个 **Proxy 单例**（`i18n.t()` / `i18n.te()`），它包了 vue-i18n 实例，游戏术语翻译走 `Translator`。
- 语言包在 `src/i18n/lang/`，切换语言走 `changeLocale()` 动态 import。

### 5.4 特殊 import（Vite 插件支撑）

```ts
import data from "../../../data/dist/weapons.data";   // -> 资源 URL（assetsInclude）
import proto from "../../../data/src/proto/weapon.proto"; // -> protobufjs Root（自定义插件）
```

`.proto` 由 `vite.config.ts` 里的 `protoPlugin()` 在编译期内联源码、运行时 `protobuf.parse()`，产出的默认导出可直接 `proto.Weapons.decode(...)`。

### 5.5 Element Plus 注意点

- `el-dialog` 用 `v-model`（不是 `:visible.sync`）。
- 具名插槽用 `<template #footer>`（不是 `slot="footer"`）。
- `el-radio` / `el-option` 的选项值用 `value`（不是 `label`）。
- 文字按钮是 `link`（不是 `type="text"`）。
- `this.$message` / `this.$confirm` / `this.$notify` 仍可用（全量安装注册了 globalProperties）。

### 5.6 代码风格

Prettier：`printWidth: 160`、双引号、分号、`arrowParens: "avoid"`。缩进 2 空格，LF，UTF-8。

改完代码跑 `npx prettier --write <file>`，不要手动对齐。

## 6. 数据流

```
DE 官方 MobileExport API ─┐
Warframe Wiki (fandom)  ─┼─> data/ bun run fetch ─> data/tmp/*.json
灰机 Wiki（中文词条）    ─┘
                              │
                              ├─ bun run build ─> data/dist/weapons.data (protobuf)
                              │                   data/dist/{mods,weapons,disposition,zh-*}.json
                              ▼
        src/warframe/codex/weapon.ts  WeaponDatabase.loadDataOnline()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pa001024/riven-mirror](https://github.com/pa001024/riven-mirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
