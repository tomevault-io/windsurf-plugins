---
trigger: always_on
description: 面向在本项目工作的开发者与编码 agent。游玩说明见 `README.md`。
---

# AGENTS.md

面向在本项目工作的开发者与编码 agent。游玩说明见 `README.md`。

---

## 项目当前形态

`fate-sandbox` 不是一张 prompt 卡；它是本地运行的互动叙事 runtime。

核心组成：

- `agents/`：GM prompt 模块。分工包括 system、context、rules、tool-policy、story-driver、render、style、input-guide、output-contract 等。
- `skills/start-game/`：新游戏初始化流程机。只负责新游戏/重新开始/创建角色，不负责续局或修档。
- `engine/core/`：确定性领域引擎。state、scene、actor、servant、economy、memory、secret、offscreen 等逻辑在这里落地。
- `tools/`：GM 领域事件工具。工具不是状态栏更新器，而是 GM 改变世界的接口。
- `data/`：型月世界数据、lookup 数据、campaign preset、timeline contract。
- `extensions/`：pi extension。玩家 UI panel、compaction policy、timeline subagent 注入都在这里。
- `.pi/agents/`：项目作用域子代理定义。必须保持 project-only 语义，不依赖 user-scope agent。
- `sessions/`、`state/`、`.pi/agent/`：运行产物/本地私有配置，不属于发布内容，不进 git。

---

## 宪章

本项目是跑在**自己机器上**的东西——没有用户兼容性包袱、没有遗留接口、没有「先这样后面再改」。每一次妥协都会留到下一次、再下一次，最终变成屎山。唯一能拦住这个螺旋的是：**从一开始就不妥协。**

本文件是工程纪律的单一权威源。违反宪章的代码不叫「能跑就行」，叫「不合格」。

### 硬切优先，schema 迁移兜底

项目没有用户兼容性负担。旧概念一旦被判定为错误，就必须从当前契约中消失：

- 不保留 alias、deprecated 字段、兼容 normalizer、旧工具入口或旧 engine public API。
- 不在工具描述、prompt、错误信息里写「不要使用旧字段」；提到旧字段本身就是继续教模型使用它。
- 不用运行时 fallback 读取新字段；state 只能先迁移到当前 schema，再进入业务逻辑。
- 唯一允许的兼容层是 persisted state schema migration。

State schema 变更必须 bump `schemaVersion`，并提供程序化逐版本迁移。迁移链必须是线性的 `v1 -> v2 -> v3`，每个函数只负责相邻版本；禁止写 `v1 -> current`、`v2 -> current` 这种 O(n²) 迁移矩阵。

### Prompt 不是防线

Prompt 负责引导，不能承担正确性。模型常犯错时，优先把约束下沉到 schema、tool boundary、normalizer、engine invariant、migration 和测试。只补一句 prompt 骂模型，等于没有修。

---

## 工具链基线

| 工具       | 配置                                                                                              | 不可绕过                           |
| ---------- | ------------------------------------------------------------------------------------------------- | ---------------------------------- |
| TypeScript | `tsconfig.json` — `strict` + `noUncheckedIndexedAccess` + `noUnusedLocals` + `noUnusedParameters` | `pnpm typecheck` 零错误才能 commit |
| oxlint     | `.oxlintrc.json` — `correctness` + `suspicious` + `typeAware` + 逐条显式                          | `pnpm lint` 零错误                 |
| oxfmt      | `.oxfmtrc.json` — import 分组排序                                                                 | `pnpm format:check` 零差异         |
| pnpm       | `pnpm@11.3.0`, `node>=24`, `packageManager` 钉死                                                  | 不用 npm/yarn                      |

任何绕过（`// @ts-ignore`、`// oxlint-disable-next-line`、`/* prettier-ignore */`）必须附带一行注释说明**为什么这里非绕过不可**。无注释的绕过视为蓄意违规。

---

## 类型系统戒律

### 零 `any`

`any` 是瘟疫。项目里不应出现。如果 pi SDK 的类型定义确实返回 `any`，在消费点立即窄化——写到类型守卫、写到 assert 函数里，不要扩散到业务代码。

```ts
// ❌ 不合格：把 any 传染出去
const data: any = pi.session.get("state");
return data.money;

// ✅ 正确：在边界窄化
const raw = pi.session.get("state");
const state = assertStateSchema(raw);
return state.money;
```

### `as` 断言必须有理由

类型断言不是「我知道这是什么」的声明，是「编译器不知道，我来告诉它」的覆盖。每次 `as` 都是一次信任链断裂。

```ts
// ❌ 不合格：静默绕过
const el = document.getElementById("root") as HTMLDivElement;

// ✅ 合格：断言后立即验证，或注释说明为什么安全
const el = document.getElementById("root");
if (!el || !(el instanceof HTMLDivElement)) throw new Error("root not found");
// 或
const state = raw as State; // safe: validated by assertStateSchema above
```

### 导出函数必须标注返回类型

公共 API 的返回类型是契约的一部分。让编译器推导是让契约变成「碰巧产生的副作用」。

```ts
// ❌ 不合格
export function getStatus() {
  return status();
}

// ✅ 合格
export function getStatus(): StatusSnapshot {
  return status();
}
```

### 歧视联合 > optional 字段 > `| undefined`

一个状态对象有 N 种形态 → 用 tagged union，不要靠 optional 字段的存在性区分。

```ts
// ❌ 不合格
type SceneResult = {
  settlement?: Settlement; // 只有 success 才有
  events?: Event[];
  error?: string; // 只有 failure 才有
};

// ✅ 合格
type SceneResult =
  | { kind: "success"; settlement: Settlement; events: Event[] }
  | { kind: "failure"; error: string };
```

---

## 叙事系统纪律

### 工具是领域事件，不是 MVU 状态栏

不要把工具设计成“把状态改一下”。工具必须表达世界里发生的事：

- `commit_turn`：非 Scene Beat lifecycle 的 canonical turn 提交入口；顶层 `time` 是必填 turn envelope。
- `progress_scene_beat`：玩家当前 Scene Beat 行动窗口的开启与收口；顶层 `time` 是必填 turn envelope。
- `update_economy`：有账户、有来源、有 reason 的资金事件；修账户名用 `rename-purse`，不要伪造 spend/gain。
- `update_actor_condition`：wound / affliction / outfit / tracked item 等可审计条件变化。
- `reveal_secret`：隐藏真名、宝具、动机的配置与揭示；不能用叙事直接泄密。
- `record_offscreen_event`：玩家视野外的真实后台事件；前台只能看到痕迹、传闻、梦境、异常投影或后果。

模型犯错时，优先把错误沉淀成：

1. 工具层归一化或拒绝；
2. 领域引擎 invariant；
3. 清晰错误信息，列出可用 id / summary / actor；
4. 回归测试。

不要只加 prompt 骂模型。

### 时间推进是 turn envelope，不是 scene event

每个 canonical turn 都必须推进 clock：

- `commit_turn.time` / `progress_scene_beat.time` 必填。
- 当前时间裁决只允许 `elapsed` 或 `travel`。
- 没有 `none`；短促对白、瞬间反应、换装、抬手格挡也至少 `elapsedMinutes: 1`。
- 地点移动用 `time.kind="travel"`，非移动耗时用 `time.kind="elapsed"`。
- Scene event 不承担时间推进；时间不是 `scene.kind` 的一个分支。
- `turnLog` 是审计账本，必须能看出每轮 `startedAt -> endedAt`。

如果 JSONL 显示 accepted tool call 大量没有推进时间，说明当前工具契约仍有逃生门；删契约，不要写提示。

### Public / secrets / player knowledge 分层

“玩家知道”不是 public state visibility。

必须区分：

| 层级              | 含义                        | 允许落点                                                  |
| ----------------- | --------------------------- | --------------------------------------------------------- |
| player-only       | 现实玩家知道；角色未必知道  | 不写 state；最多用于 GM 避免误剧透                        |
| protagonist-known | 玩家角色本人知道            | public actor identity / public memory，前提是剧情内成立   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lolo-s-Cosmos/fate-sandbox](https://github.com/lolo-s-Cosmos/fate-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
