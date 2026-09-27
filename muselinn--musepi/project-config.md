---
trigger: always_on
description: The desktop GUI follows its own living spec — **`docs/gui-design.md`** (design/interaction standards) and **`docs/gui-implementation.md`** (RPC contracts, gotchas, verification workflow). Update them when you change GUI behavior. Key rules that bite:
---

# Development Rules

## GUI Development Rules (`packages/desktop-app`, `packages/guest-client`)

The desktop GUI follows its own living spec — **`docs/gui-design.md`** (design/interaction standards) and **`docs/gui-implementation.md`** (RPC contracts, gotchas, verification workflow). Update them when you change GUI behavior. Key rules that bite:

- **Modals must own the keyboard while open.** `DialogFrame` captures Escape on `document` in the capture phase (wins over handlers behind), moves focus into the dialog and restores it on close; confirm boxes confirm on Enter; the onboarding overlay advances on Enter / steps back on Escape. Never let a modal rely on the page behind keeping focus — the composer swallows Enter and sends a message.
- **`DialogFrame` is always mounted and driven by `open`** — conditional mounting (`{x && <DialogFrame/>}`) kills the exit animation. Same rule for prompt/confirm dialogs (`lib/prompt-dialog.tsx`): closing defers the promise resolution until the 180ms exit plays.
- **Small-content dialogs must use the compact style** (`gui-dialog--confirm`): auto-sized, `max-width: 380px`. The base `.gui-dialog` is a 600×420 settings box — a desc + two buttons floating in it reads broken.
- **Every hook must be declared before any early return** (`if (!open) return null`). A hook after a conditional return flips the hook count and crashes with "Rendered more hooks than during the previous render" (AnnouncementOverlay regression, fixed 2026-08-14).
- **Model identity is `provider/id`, never a bare id.** Two providers can serve the same bare id (opencode-go vs opencode-zen both offering `deepseek-v4-flash`): favorites, the DEFAULT pin, the selection state and role assignments all key on `provider/id`, and `session.setModel` carries `provider` so the daemon resolves the exact model. The daemon's model resolver already understands `provider/id` references.
- **Model selection is session-scoped** (TUI `/switch` parity): the in-chat composer's pick calls `session.setModel` for THAT session only. The welcome composer's resting preselect is the DEFAULT role (`modelRoles.default`) kept in its own `defaultModelId` app state — opening/switching sessions must NEVER write it, and `ModelSelector` resets its `userPicked` seed lock on session change (the composer stays mounted across switches, so a pick in session A must not freeze session B's selector on A's model). Seed precedence in session mode: live model (`contextUsage.model`) → session preselect → DEFAULT → list head.
- **Role thinking ladders are per-model.** The role rows' thinking select renders the resolved model's `getSupportedEfforts` (daemon `resolvedRoleModels.efforts`) — never a fixed seven-rung list; re-fetch the resolution after every role-model change (`applyRoleModels`).
- **CSS-only interactions stay CSS-only** (chroma group glow via CSS vars + hover; recap slide via sibling selectors) — no React state for pointer tracks.
- **i18n 词表按域拆分**（`guest-client/src/i18n/{zh-CN,en-US}/<domain>.ts`，TUI 在 `coding-agent/src/i18n/zh-CN/`）：改文案找对应域文件，禁止塞回单文件；en 域文件必须 `as const satisfies Record<ZhKey, string>`（缺/多 key 即编译错误，加 zh key 必须同步 en）；域间 key 重复 → barrel 模块加载抛错。插件/扩展文案走 `registerTranslations`（GUI 另有 `tLoose`），不直接改词表。架构见 `docs/i18n.md`。
- **Extension HMR / `registerComponent`**（P4 v1 + P5 v2，契约见 `docs/extensions-dev.md §6`）：扩展入口文件变更 → daemon watcher 500ms 内广播 `extensions.changed`（GUI `useSlotComponents`/`ExtensionsCenter`/`PluginsSection` 监听即刷，替代纯轮询）+ 对每个活跃会话按入口 mtime 对比执行 `reloadExtension`（忙会话挂起、`agent_end` 补做），完成发会话内事件 `extensions.reloaded`。GUI 组件渲染 = 文件变更后 ~1s；会话内工具 = 下次调用生效（旧名若未被新模块重注册则从注册表删除）。**子模块改动不热生效**（Bun 模块缓存只重键入口 specifier）——多文件扩展改子模块需 touch 入口。扩展内存态不迁移、在途副作用不回收、handler 重载存在 ~ms 双跑窗口（旧 handler 先清后推新）。新增扩展 API 必须同步更新 `docs/extensions-dev.md`。
- **Modes（预设）与扩展中心分类**（已归档：`docs/archive/modes-plan.md`；实现见 `coding-agent/src/presets/`）：预设 = 扩展白名单 + 提示词区块 + settings 覆盖，文件在 `~/.musepi/modes/<id>.json`（`presets/resolve.ts` 继承展开/校验、`prompts/composer.ts` 注入；入口 `--preset` CLI / GUI 欢迎页项目行 chip / 设置→智能体→预设）。**扩展中心 provider 并存**：`omp-plugins` = "OMP Extension Packages"（上游生态，勿改品牌名）、`musepi-extensions` = "MusePi Extensions"（自有扩展系统，`discovery/builtin.ts` 的 ExtensionModule/Extension 源标记）——新增自有扩展能力沿用 `musepi-extensions` provider，勿并入 native。

## Docs: 计划文档实现状态速查

> **权威状态在 `docs/` 各文档头部状态行**。本轮（2026-08-26）逐项核验后固化如下；改动功能时先更新对应计划文档状态行，再参考本文避免重复核验。
>
> 2026-09-17：下表**已完结**的一次性计划文档已移入 `docs/archive/`（含双语三件套），路径随之更新。归档规则与完整清单见 `docs/archive/README.md`；留在 `docs/` 顶层的是仍在约束实现的活文档。

各计划文档实现状态（2026-08-26 核对）：

| 文档 | 状态 | 关键实现位置 |
|---|---|---|
| `docs/archive/modes-plan.md` | ✅ v1+v2 已实现（2026-08-21 `7bff540c13`/`457039db31`） | `coding-agent/src/presets/resolve.ts`、`prompts/composer.ts`、`--preset` CLI、GUI 欢迎页 mode chip |
| `docs/archive/tui-trace-plan.md` | ✅ 已实现（2026-08-26） | `/trace` 叠加在 `/tree` 上；`modes/components/tree-selector.ts` 投影参数、`test/modes/components/trace-selector.test.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MuseLinn/MusePi](https://github.com/MuseLinn/MusePi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
