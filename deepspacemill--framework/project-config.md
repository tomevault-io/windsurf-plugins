---
trigger: always_on
description: 本文档面向 AI coding agent 和开发者，修改或定制 **末语（Moyu）视觉小说框架** 前务必阅读。
---

# Moyu Visual Novel Framework — Agent 指南

本文档面向 AI coding agent 和开发者，修改或定制 **末语（Moyu）视觉小说框架** 前务必阅读。

## 本项目是什么

基于 **末语（Moyu）** 引擎的 TypeScript + React 视觉小说框架。运行于 native（Windows / macOS / Linux / Android / iOS，通过 QuickJS）与 web（WebAssembly）两种平台。

- **渲染器**：`@momoyu-ink/kit` 的自定义 React reconciler（**不是 react-dom**）
- **JSX 元素**：`<container>` / `<sprite>` / `<text>` / `<video>` 等 Moyu intrinsic elements（**不支持 HTML 元素**）
- **状态管理**：Valtio proxy
- **动画**：react-spring（通过 `@momoyu-ink/kit` 重导出）
- **Schema**：Zod（命令定义 + `commands.schema.json` 生成）

> SDK 细节见 `@momoyu-ink/kit` 的 `AGENTS.md`。

---

## 关键约束

1. **不支持 HTML 元素**：写 `<div>` / `<span>` 会崩。JSX 元素必须是 kit 暴露的 intrinsic element。
2. **不支持 react-dom**：入口用 kit 的 `createRoot()`。
3. **优先复用 kit**：导航 / 事件 / 音频 / 动画 / stage 管理都在 kit 里。新增抽象前先查 kit。
4. **资源路径相对 `assets/`**：`<sprite src="image.png" />` 指 `assets/image.png`。

---

## 项目结构

```
src/
├── index.tsx          # 入口：ready 事件 → createRoot → <Main>
├── router.ts          # createStackNavigator：pages + overlays，含类型注册
├── error.tsx          # 全局 ErrorBoundary fallback
├── actors/            # Stage 下的 actor（可视 + headless）
│   ├── background.tsx # 背景 + fade transition + tint spring
│   ├── character.tsx  # 立绘（position / scale / tint / fade 动画）
│   ├── textbox.tsx    # 文本框 + typewriter + 功能按钮
│   ├── selection.tsx  # 选项菜单（期间用 skip/auto blocker 阻断）
│   ├── bgm.tsx        # BGM（headless）
│   ├── voice.tsx      # 语音（headless，参与 auto ticket）
│   ├── sfx.tsx        # 一次性音效（headless；skip 期间丢弃）
│   └── sound.tsx      # 命名声道音效（headless）
├── commands/
│   ├── commands.ts    # Zod schema（含 x-i18n / x-asset-kind / format 等 meta）
│   └── handlers.ts    # 命令响应层：更新框架状态与流程控制，让 Stage / actor 执行效果
├── components/        # UI 组件：button / dialog / checkbox / slider / select / frame / notification
├── hooks/
│   ├── useBacklog.ts  # Backlog 滚动 + 滚动条
│   ├── useButton.ts   # 按钮状态机（idle / hover / press）
│   └── useSaveLoad.ts # 存读档槽位管理
├── pages/
│   ├── title.tsx      # 标题画面
│   ├── stage.tsx      # 游戏主舞台：注册命令 + 装配 actor
│   └── menu.tsx / settings.tsx / saveload.tsx / backlog.tsx  # overlay 页
├── state/
│   ├── game.ts        # GameState（valtio proxy）+ resetGameState()
│   ├── ui.ts          # UIState：通知队列、confirm 弹窗入口
│   └── settings.ts    # 设置（持久化到 scenario permanent variable）
└── utils/
    ├── mergeEvent.ts
    └── scenarioGameState.ts  # gameState ⇌ scenario variable 序列化/恢复
```

---

## 架构

### App 生命周期

```
引擎发 'ready' 事件
  → index.tsx 调 createRoot().render(<Main>)
    → ErrorBoundary 包 <Navigation /> + <Notification />
      → router.ts 的 createStackNavigator（initialPage: 'title'）
        → Title 页
```

`index.tsx` 同时在 `beforeunload` 上挂了退出确认，会调 `system.quit`。

### Stage 两层架构

Stage 是剧情驱动 UI 的核心页面，采用严格的双层设计：

- **Layer 1 — Command handler**（`commands/handlers.ts`）：负责响应 scenario 命令，更新框架状态与流程控制，让 Stage / actor 执行出对应效果。当前 handler 主要改 `gameState`、调用 `control.setWaiting()` / `control.hold()` / `control.record()`，并在少量需要与 backlog / 存档保持一致的路径上先同步 scenario 变量快照。
- **Layer 2 — Actor**（`actors/*.tsx`）：React 组件，`useSnapshot` 订阅 `gameState`，承担副作用（音频、动画、节点命令、auto ticket）。

换句话说，handler 负责描述“命令发生后，框架状态应该变成什么、流程应该停在哪里”；actor 负责把这些状态变化落实成真实的视觉、音频和引擎行为。

**Stage 单例跨 HMR 保活**：`pages/stage.tsx` 把 stage 实例挂到 `globalThis.__MOYU_FRAMEWORK_STAGE__`，防止 Fast Refresh 重建 stage 后 scenario 事件监听错乱或 runtime 状态丢失：

```typescript
// pages/stage.tsx
function getStageSingleton(): StageInstance {
  const g = globalThis as StageGlobal;
  g.__MOYU_FRAMEWORK_STAGE__ ??= createStage();
  return g.__MOYU_FRAMEWORK_STAGE__;
}

const stage = getStageSingleton();

export function Stage() {
  const params = useNavigationParams<StageParams>();
  const stories = useMemo(() => [params.story], [params.story]);

  useEffect(() => {
    const unregs = registerStageHandlers(stage);
    return () => unregs.forEach((fn) => fn());
  }, []);

  useScenario(stories, params.story, params.entry, params.isNewGame);

  return (
    <StageContextProvider stage={stage}>
      <BackgroundActor />
      <CharacterActor />
      <TextBoxActor onButtonClick={/* ... */} />
      <SelectionActor />
      <BGMActor />
      <VoiceActor />
      <SfxActor />
      <SoundActor />
    </StageContextProvider>
  );
}
```

Stage 同时处理顶层输入：

- 点击 / 触摸结束时，若 overlay 可见或选项菜单可见，则不推进剧情。
- 若 textbox 当前不可见，则本次点击会先重新显示 textbox，并消费掉这次输入，不会继续推进剧情。
- 在可推进状态下，Stage 会先让 `tryInterrupt()` 消费输入（例如完成打字机）；只有没有 interrupt callback 接管时才会 `nextLine()`。
- `mousedown` / `touchstart` 会停止 skip；窗口失焦会同时停止 auto 和 skip。
- 在 stage 页面且没有 overlay 时，滚轮向下会停止 auto 并打开 backlog overlay。

### 流程控制（GameControl）

Handler 收到的 `control` 对象：

| 方法 | 作用 |
|------|------|
| `control.hold()` | 暂停直到用户点击 / 按键；若在 skip 中且当前 dispatch `unskippable()` 则会停止 skip |
| `control.setWaiting(ms, skippable)` | 定时等待，到期或可跳过时由用户 / skip 触发推进 |
| `control.nextLine()` | 立即推进（少用） |
| `control.unskippable()` | 标记本次 dispatch 不可跳过 |
| `control.record(meta)` | 写入 backlog 历史，返回 id |

**默认行为**（handler 不调任何 control 方法）：自动推进到下一行。

### 状态管理

**`gameState`**（`state/game.ts`，valtio proxy）—— 剧情可见、会被序列化到存档的状态：

```typescript
gameState.story       // { title }
gameState.background  // { src, fadeTime, tint?, skippable }
gameState.character   // { presets, characters[], currentSpeaker?, autoTintEnabled, autoTint }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeepSpaceMill/framework](https://github.com/DeepSpaceMill/framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
