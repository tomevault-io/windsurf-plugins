---
trigger: always_on
description: > 这份文件是给为 Marina 贡献代码的 AI agent(Claude Code / Cursor / Codex / 其他)看的。
---

# AGENTS.md — Marina 项目 AI Agent 工作说明书

> 这份文件是给为 Marina 贡献代码的 AI agent(Claude Code / Cursor / Codex / 其他)看的。
> 你正在 YOLO 模式下工作,大部分时间不需要打扰开发者。但有少数情况你必须立刻停下来。
> 仔细读完整份文件再开始工作。

文档版本:1.3 · 最后更新:2026-05-16

> **v1.3 变更**:与软件定义书 v1.6 / ADR-013 对齐 — 附录 C "关闭弹确认"硬规则增加 Linux 例外脚注:仅当 `lifecycleModel === 'no-persistence'` + 最后一个窗口 + 仍有非 exited session 时,允许弹 `<LastSessionConfirm />`。1.2 节边界 3 例子同步加例外注脚。
> **v1.2 变更**:产品改名 EasyTerm → **Marina**(对齐软件定义书 v1.5,ADR-012)。所有"产品现状"维度的 EasyTerm 字样替换为 Marina;commit message 示例等"历史/惯例"维度的 EasyTerm 保留作为风格参考。`%APPDATA%\EasyTerm\` 全部改为 `%APPDATA%\Marina\`(Electron 由 `productName` 自动派生)。
> **v1.1 变更**:与软件定义书 v1.3 / CP-4 勘误回合对齐 — CP-4 完成标志改 7 套主题;附录 D 新增"勘误回合工作纪律";4.5 章明确"勘误回合修复"也是检查点工作流的一部分。

---

## 0. 你必须先读的文件

按顺序读完以下文件,你才有足够的上下文开始工作:

1. **`docs/软件定义书.md`** — 整个产品的设计共识。读懂第 2 章(设计哲学)和第 8 章(状态机),否则你写的代码会偏离产品方向。
2. **本文件(`AGENTS.md`)** — 你现在在读的。
3. **`docs/ipc-protocol.md`** — IPC 协议规格(若存在)。如果不存在,在你开始实现 IPC 之前需要先和开发者一起定。
4. **`README.md`** — 项目的对外说明。

如果上述任何文件不存在或内容残缺,**立刻停下来通知开发者**,不要自己脑补。

---

## 1. 你的工作模式:YOLO 模式

### 1.1 什么是 YOLO 模式

你被授权在不打扰开发者的前提下,**端到端**完成 Marina 的构建。这意味着:

* **可以自主决策**:文件命名、代码组织、变量名、内部 API 设计、测试用例选择、调试策略
* **可以自主执行**:写代码、跑构建、跑测试、提交 git commit、安装(限定范围内的)依赖
* **不需要每一步问开发者**:写完一个函数不用问、起一个新文件不用问、修复一个 bug 不用问

**你的默认行为是"继续干活"**,不是"等指示"。

### 1.2 YOLO 的三条边界

YOLO 不是"想干啥干啥"。以下三条边界**永远不能越过**,越过即停止:

#### 边界 1:破坏性操作前必须停

包括但不限于:
* 删除文件(除非是你刚才自己创建的临时文件)
* `git push --force` 任何操作
* 修改 `.git/` 目录
* 删除或修改 `~/AppData/Roaming/Marina/` 下的任何文件(即使是测试)
* 卸载 npm 包
* 修改系统注册表(Windows)
* 调用任何"清理"、"重置"、"reset"、"clean"类命令对实际数据生效

遇到这些操作前,**停下来,描述你想做什么,等开发者确认**。

#### 边界 2:超出技术栈的依赖

只允许使用 `软件定义书.md` 第 10.1 节列出的技术栈。如果你觉得需要新加一个 npm 包(例如某个工具库、UI 库、状态管理库等),**停下来问开发者**。

理由:agent 历史经验里经常引入冷门 / 已废弃 / 安全风险的包,这种决策必须人来做。

允许的例外:
* `@types/*` 类型定义包,可自由安装
* 明显的小工具(如 `uuid`、`debounce`),先在对话里告知一声再装
* 测试相关包(jest 等,见第 5 章)

#### 边界 3:产品哲学

`软件定义书.md` 第 2 章的四条哲学原则和第 13.2 节"永远不做"列表是**红线**。如果你发现某个功能实现起来"如果加个 XXX 会简单很多",而那个 XXX 出现在"永远不做"列表里,**不要做,停下来问**。

例如:
* 如果你想加"应用内快捷键来切换 session",**停**(违反"鼠标优先")
* 如果你想加"主窗口"概念来简化 session 持久化,**停**(违反"窗口平等")
* 如果你想给关闭窗口加确认对话框,**停**(违反"窗口零成本开关")
  * **唯一例外(v1.6,软件定义书 ADR-013)**:Linux 上"最后一个窗口关闭 + 仍有非 exited session"时弹同一个 `<LastSessionConfirm />`。非 Linux / 非最后窗口 / 全 exited session 时仍然禁止弹任何确认。详见软件定义书第 2 章 Linux 平台例外脚注与 ADR-013。

### 1.3 YOLO 的成功标准

如果你做到了下面所有事,这次 YOLO 就是成功的:

* [ ] 严格按 `软件定义书.md` 实现所有 V1 必做功能(5.1 节)
* [ ] 没有违反"永远不做"列表(13.2 节)
* [ ] 在每个检查点(见第 4 章)按规格停下来,等开发者测试
* [ ] 后端关键模块有自动化测试覆盖(见第 5 章)
* [ ] 代码注释充分,出问题时开发者能调试(见第 2 章)
* [ ] 所有 commit 历史清晰,git bisect 可用(见第 6 章)
* [ ] 应用能正确打包成 Windows 安装包,在干净的 Windows 11 上能跑

---

## 2. 代码注释要求(关键)

### 2.1 为什么注释要求高

正常项目里,代码 review 是质量保证。**这个项目没人 review 你的代码**。
当你犯错时,开发者要在事后接手调试。如果你的代码他读不懂,他就只能扔掉重写。

所以你的代码必须满足:**任何一个有 TypeScript / Electron 经验的人,在没读过你思考过程的前提下,能在一小时内理解任何一个文件在做什么、为什么这么做、出问题时该看哪里。**

### 2.2 必须写注释的地方

#### 文件头注释(每个 .ts / .tsx 文件必须有)

```typescript
/**
 * @file session-manager.ts
 * @purpose 管理所有 PTY 会话的生命周期(创建、活跃/空闲检测、墓地、销毁)
 *
 * @关键设计:
 * - 每个 Session 在守护进程内是单例,owner_window_id 可为 null
 * - PTY 进程退出后进入"墓地"5 分钟,期间用户可恢复
 * - 字节流通过 IPC 推送给 owner window,无 owner 时仍写 scrollback
 *
 * @对应文档章节:软件定义书.md 第 5.1.2、8.3 节
 *
 * @不要在这里做的事:
 * - 不要解析 OSC 1337(那是 cwd-tracker.ts 的职责)
 * - 不要持久化 session(session 不持久化,设计如此)
 * - 不要管理 path 归属(那是 path-manager.ts 的职责)
 */
```

#### 函数注释(公开函数必须有,私有复杂函数必须有)

```typescript
/**
 * 创建一个新 session 并启动 PTY。
 *
 * @param pathId 该 session 启动时的工作目录所对应的 path id
 * @param templateId 启动模板 id,从 templates.json 读
 * @param ownerWindowId 创建该 session 的窗口 id,设为初始 owner
 * @returns 创建的 SessionInfo
 *
 * @throws 'PathNotFound' 如果 pathId 不存在
 * @throws 'TemplateNotFound' 如果 templateId 不存在
 * @throws 'PtySpawnFailed' 如果 node-pty 启动失败(常见原因:cwd 不存在、shell 不存在)
 *
 * @副作用:
 * - 创建 node-pty 子进程
 * - 把 session 加入 path 的 session 列表
 * - 触发 path 状态机:可能让 path 进入"临时"分类
 * - 广播 pathTreeUpdated 和 sessionStateChanged 事件
 *
 * @常见问题排查:
 * - 如果 PTY 启动后立即退出 → 检查 shell 路径、cwd 权限
 * - 如果 OSC 1337 hook 不生效 → 检查 templateId 对应的 shell hook 文件是否注入
 */
async function createSession(
  pathId: string,
  templateId: string,
  ownerWindowId: string
): Promise<SessionInfo> {
  // ...
}
```

#### 复杂逻辑必须有"为什么这么写"的注释

不是写"这行代码做什么",而是写"为什么这么做,不那么做"。

```typescript
// 我们用 setImmediate 而不是 process.nextTick,因为 nextTick 优先级太高,
// 会饿死后续的 PTY data 事件,导致字节流堆积。这是踩过坑的。
setImmediate(() => {
  this.broadcastPathTreeUpdate();
});

// node-pty 的 onData 回调可能在 PTY 已经标记 destroyed 之后还触发一次
// (经验:Windows ConPTY 的关闭是异步的)。所以这里要 guard。
if (this.destroyed) return;
```

#### 状态机相关代码必须有状态图引用

任何涉及到状态转移的代码,在该函数顶部用注释画出 mini 状态图,或引用文档:

```typescript
/**
 * Session 状态转移逻辑。
 *
 * 状态机参见 软件定义书.md 第 8.3 节(v1.2 ADR-008 后)。
 * 简化:
 *   active <--有/无字节流--> idle
 *   active/idle --PTY 进程退出--> exited (灯显灰,scrollback 保留,无时限)
 *   exited --用户右键关闭 / 应用退出--> destroyed
 *   注:exited 不可回到 active(无重启路径,见 ADR-008)
 */
```

### 2.3 不要写的注释

* `// 设置 x 为 5` ← 重复代码本身,删掉
* `// TODO: fix this later` ← 要么修,要么写明白具体要修什么、为什么不现在修
* 注释掉的代码 ← 直接删,git 里有历史

### 2.4 错误信息要详细

任何你 throw 的 Error 必须包含:
* 出错的操作名
* 出错的关键参数值
* 可能的原因(至少猜两个)
* 建议的下一步

```typescript
// 不好
throw new Error('Failed to create session');

// 好
throw new Error(
  `[SessionManager] Failed to spawn PTY for templateId="${templateId}" cwd="${cwd}". ` +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liyue-Cheng/marina](https://github.com/Liyue-Cheng/marina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
