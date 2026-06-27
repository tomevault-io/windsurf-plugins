---
trigger: always_on
description: `tick-tui` 是 Tick 任务管理系统的命令行 TUI 客户端，基于 Go + bubbletea。
---

# CLAUDE.md

## 项目概述

`tick-tui` 是 Tick 任务管理系统的命令行 TUI 客户端，基于 Go + bubbletea。
设计理念是 lazygit 风格的窄窗口（约 40 字符宽），全程同一画面，没有任何弹框 / modal / popup —— 所有编辑就地内联。

数据直接读写本地 markdown 文件（路径由首次启动 wizard 决定，默认 fallback `~/tick/tasks.md`），**不再依赖任何服务端**。
推荐放在 Obsidian vault 里，手机端在 Obsidian 里手敲新任务即可，tick-tui 启动时自动补全 metadata。

我（开发者）个人的实际路径是 `~/Documents/hoard/tick/tasks.md`（hoard = 我的 Obsidian vault），仅作为本文档中举例参考；新用户的路径由 wizard 选定。

## 架构

```
cmd/tick/main.go           入口：加载配置、建 store、跑 bubbletea
cmd/seed/main.go           dev 工具：灌假数据（--days/--avg/--out）
internal/store/
  types.go                 Feature / TodayResponse / ProjectItem
  markdown.go              parser / serializer / 文件 IO / Store 接口实现
  markdown_test.go         单测
  stats.go                 GetCompletionsByDate：跨 tasks.md+archive.md 按日期计数
  stats_test.go            单测
internal/config/
  config.go                读 / 写 ~/.config/tick/config（key=value，行内 ` #` 注释）
  config_test.go
internal/setup/
  detect.go                Obsidian vault 检测（读 obsidian.json）
  detect_test.go
  strings.go               wizard 自己的 i18n 表（与主屏 i18n 解耦）
  wizard.go                首次启动 / O 修改文件夹的 bubbletea 子 model
  wizard_test.go
internal/i18n/
  i18n.go                  TUI 双语字符串表 + Lang 类型 + 星期/月份本地化
  i18n_test.go             单测
internal/tui/
  model.go                 Model + 状态机常量 + buildRows + 项目分组排序 + lang/strings 字段
  update.go                Update：消息分发、store tea.Cmd、按键 handler、l 切换语言
  view.go                  View：列表渲染、padBetween、scrollWindow（走 m.strings 表）
  editor.go                ComputeGhostText / renderTitleWithGhost / renderProjectField
  styles.go                lipgloss 样式集中
  keys.go                  bubbles/key 绑定 + footerShortHelp（用 m.strings）
  stats.go                 renderBars30 / renderHeatYear 纯渲染函数（接 i18n.TUIStrings）
  stats_test.go            stats 渲染断言（EN + ZH 双语断言）
  update_test.go           关键状态机单测（含 stats/settings/lang 切换）
internal/watcher/
  watcher.go               fsnotify-based tasks.md 监听
```

依赖：`charmbracelet/bubbletea` v1.3 · `bubbles` v1.0 · `lipgloss` v1.1 · `atotto/clipboard`。

## 数据模型

```go
type Feature struct {
    ID          string  // 8-char hex (e.g. "a3k7m2x9"); empty = not yet assigned
    Title       string
    ProjectName *string
    IsDone      int     // 0/1
    CompletedAt *string // YYYY-MM-DD; nil 表示未完成
    CreatedAt   string  // YYYY-MM-DD
}
```

## 文件格式

紧凑 ASCII 单行，字段顺序自由（解析时位置无关）：

```
- [ ] buy milk @home +2026-05-01 [a3k7m2x9]
- [x] write report @work +2026-04-29 *2026-04-30 [b1d4e5f0]
- [ ] 买菜 @家庭 +2026-05-01 [c2f3a4b5]
```

| 部分 | 含义 | 必填 |
|---|---|---|
| `- [ ]` / `- [x]` | 状态 | 是 |
| 描述文本 | task title（含 CJK） | 是 |
| `@project` | 可选项目（`@` + 非空白） | 否 |
| `+YYYY-MM-DD` | 创建日 | 否（缺则 sweep 时补 today） |
| `*YYYY-MM-DD` | 完成日（仅 `[x]` 行） | 否（缺则 sweep 时补 today） |
| `[ID]` | 8 字符 hex 随机 ID，**强制行尾** | 否（缺则 sweep 时随机分配） |

ID 用 8 字符 hex（`crypto/rand` 4 字节）。**为什么不用自增数字**：手机插件 + Mac CLI 双向同步时，两边都按"max + 1"会撞 ID（实际遇到过两条任务 [63] 同 ID 导致 mark-done 走错行）。8 hex = 32 bit ≈ 40亿种，碰撞概率近 0。

正则 `\s\[([a-zA-Z0-9]{1,16})\]\s*$` — 接受 1-16 字符，兼容旧数字 ID 直到下次 sweep 自动重写为 hex。`[3 个]` 这类 CJK 描述不会误匹配（中文不是 alphanumeric）。

如果两条行碰巧同 ID（迁移残留 / 极端碰撞），sweep 会给第二条重新 roll 一个。

## 双文件 + 7 天滚动归档

```
<wizard 选定的目录>/
  tasks.md       ← undone + 过去 7 天的 done — 任意时刻 < 350 行 / 35 KB
  archive.md     ← 7 天前的 done；append-only；TUI 列表不读它
```

mark-done / undone 是**就地操作**（仅修改 tasks.md），不跨文件移动。

每次 `loadTasks()` 跑一次"被动 sweep"：
1. 缺 `[ID]` → 分配 `genID()`（8 hex chars）
2. 重复 `[ID]` → 第二条重新 roll
3. 缺 `+date` → 补 `+today`
4. 状态 `[x]` 但缺 `*date` → 补 `*today`（手机端手动勾选语义补丁）
5. 状态 `[x]` 且 `*date < today-7d` → 移到 archive.md

效果：手机端在 Obsidian 里手敲 `- [ ] 任务 @项目` 一行，sync 到 Mac，tick-tui 启动 → 自动补成完整一行写回。

## TUI today 语义

- `pending = tasks.md 中所有 [ ] 行`
- `done section = tasks.md 中 [x] 且 *date == today 的行`（TodayResponse.Done）
- `done section 续` = tasks.md 中 [x] 且 *date == yesterday 的行（TodayResponse.DoneYesterday）
  - 两者共用同一个 "── done ──" separator，yesterday 行尾显示 dim 的 `-1d` 标记
  - row.daysAgo: 0 = today/pending，1 = yesterday done
- 历史完成（2-6 天前 + archive.md）TUI 列表不读

## Stats 30-day drill-down

### 选中模式（drill-down）

- 进入 modeStats30 默认**未选中**（无右 panel，首屏）
- 按 `←` 一次 → 选中 today，进入 drill-down；右 panel 显示该日 task list
- 再按 `←` → selectedDate -= 1d；超出 bars 窗口左边界时 statsWindowEnd -= 1（窗口左移），无界往前
- `→` 反向；selectedDate 不超过 statsEnd（today）
- `↑` / `k` / `↓` / `j` 滚动 task panel（selectedScroll）
- `esc` 第一次：清 selectedDate，回首屏；第二次：退 stats 回 modeList
- `s` 切 30 天：重置 selectedDate / selectedTasks / selectedScroll / statsWindowEnd

### Task panel

- 最多显示 10 条；超出显示 `↑ 上方 X 条` / `↓ 还有 X 条`
- 切到新日期时 selectedScroll 重置为 0
- task 行格式：`· @proj title`（proj 为空时 `· title`）；title 截断到 panel 宽度（lipgloss.Width 安全）

### 布局

- `width >= 70`（`wideStatsWidth`）→ 宽布局：bars 占 `barsAreaWidth=36` 列，右侧 panel 占剩余
- `width < 70` → 窄布局：panel 在 bars 下方单列堆叠

### Streak 算法

- 从 today 倒数，遇到第一个 count == 0 的日停止
- 使用 `statsData`（最近 30 天），streak >= 30 显示 `🔥 30+`
- 在 `statsLoadedMsg` 到达时（加载完成后）由 `computeStreak()` 计算一次，缓存到 `m.streak`
- 标题行右侧显示 `🔥 N 天`（ZH）/ `🔥 Nd`（EN）

### 数据来源

- `store.GetTasksOnDate(d)` 扫 tasks.md + archive.md，返回指定日的所有完成任务，按 ID 升序稳定排序
- 每次 selectedDate 变化触发 `cmdLoadTasksOnDate`；stale response 用 `sameDay()` helper 按 Y/M/D 比较丢弃（不依赖字符串 format，避开 time-of-day drift）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [al4danim/tick-tui](https://github.com/al4danim/tick-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
