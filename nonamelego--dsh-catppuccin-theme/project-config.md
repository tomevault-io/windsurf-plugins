---
trigger: always_on
description: 给本仓库的 Agent（以及人类维护者）的项目速览与发版 SOP。开始改代码前请先读「项目定位与核心目标」与「项目身份速览」，**惯例是这里踩过坑的集结点**；文末「与报告人协作」「本机测量资产」两节是本轮（issue #12 / #13）新增的。
---

# AGENTS.md — dsh-catppuccin

给本仓库的 Agent（以及人类维护者）的项目速览与发版 SOP。开始改代码前请先读「项目定位与核心目标」与「项目身份速览」，**惯例是这里踩过坑的集结点**；文末「与报告人协作」「本机测量资产」两节是本轮（issue #12 / #13）新增的。

## 项目定位与核心目标（判断「该不该做」的准绳）

**核心目标是「适配」，不是「设计配色」**：按 [Catppuccin 官方源配色](https://github.com/catppuccin/catppuccin)（`palette.json` v1.8.0）把 DeepSeek Harness 的官方 `--dsw-*` token 体系**完整**映射成四个风味。**官方色板是准绳，不是教条**——官方取值在 DSH 的实际用法下真不成立时（有证据）可以改，走规则 1；**玻璃质感（Glassmorphism）是叠加在正确配色之上的视觉皮肤，属附带目标**——它只改材质，不改配色。

权威数据源（由 `scripts/generate-palettes.mjs` 读取，均在仓库外的缓存）：

| 文件 | 作用 |
|---|---|
| `<缓存>/.cache/dsh-ref/dsw-tokens.json` | 官方 design-platform.css 解析出的 `--dsw-*` token 全表——**要全覆盖，不许漏 token** |
| `<缓存>/.cache/dsh-ref/catppuccin-palette.json` | 官方 Catppuccin 色板 v1.8.0——**默认取色来源**（官方取值在 DSH 下确实不成立时按规则 1 的例外流程偏离） |

> 路径基准：脚本里写的是 `join(__dirname, '..', '..', '.cache', 'dsh-ref')`（**相对 `scripts/`，往上两级**），
> 本机实测 = `D:\Vibe-Coding\.cache\dsh-ref\`（2026-09-24 校验：目录存在，含 `dsw-tokens.json`、
> `catppuccin-palette.json` 与各 tag 的 `design-platform-*.css`）。换机器时改成自己的缓存盘即可。
| 产物 `src/client/palettes.ts` | 生成物；同源产物还有 `themes/`（`pnpm gen:themes`）与 shiki token 表 |

> **两层「官方」不要混淆**：(a) **Catppuccin 源色板** —— 取色来源；(b) **DSH 官方 token 的取值与语义** —— 被适配对象。改 (b) 是本插件存在的理由（例如整族重映射 `brand-primary` 到风味蓝），不需要理由；改 (a) 才需要证据（规则 1）。

### 由此推出的四条硬规则

1. **以官方色板为默认来源，偏离需要证据。** 每个 `--dsw-*` 的取值默认是官方 26 色本身、或由它们 `color-mix` 出来的中间值；**不得凭观感发明色相**（「这个蓝有点闷，换 sapphire」不是理由）。但当官方取值**在 DSH 的实际用法下确实不成立**时——例如该色被 DSH 用在对比度不达标的场景、或该 token 在 DSH 里被用在官方设计没有的场景——**可以改**，过下面三条即可：
   - ① 有可复核的证据（实测对比度数字 / 用户 issue / 上游设计差异）；
   - ② 已穷尽「不换色相」的手段（混色目标、层级档位）仍不达标；
   - ③ 在代码注释 + CHANGELOG 写明**与官方取值的差异及理由**，并补上锁定该决定的断言。
2. **改颜色只改映射表。** 调 `scripts/generate-palettes.mjs` → `pnpm gen:palettes` → 提交生成物；手改 `palettes.ts` 下次生成即被覆盖。
3. **对比度问题先换手段，再换颜色**：第一步永远是不换色相的调整——混色目标（`base` → `crust`）与层级档位（先例：issue #7、issue #11、VV）。仍不达标时按规则 1 的例外流程偏离并记录：已有两个**记录在案的有意偏离**——① Latte `--dsw-static-blue-900`：该步被 DSH 当作浅色徽章（hero「预览版」）上的**文字**用，通用浅色计划把它混向 `base` 会洗成淡色，故改混向 `text`（`generate-palettes.mjs` 内有完整注释）；② `brand-primary` 整族：DSH 官方品牌蓝与 Catppuccin 主题冲突，直接重映射为风味蓝（生成器头部注释说明）。
4. **玻璃层不得改变配色语义。** 玻璃只动 alpha / blur / rim / shadow；开关关闭时必须回到与官方色板逐 token 一致的纯色形态。**推论（issue #13 实测）**：本皮肤把页面地面强制成**纯色**，所以「背后只有地面」的玻璃面上加 `backdrop-filter` 是**恒等变换**——填充逐像素不变（隐藏该面内容后 A/B：气泡 **0/19184 像素**），代价却是 Chromium 每帧回读 backdrop（**半径不参与**；`will-change` / `contain` 绕不开，只有 `none` 免提升）。**新增 blur 前先问一句「这个面背后是不是只有纯色地面」**，`tests/glass-css.spec.ts` 双向锁定（地面之上的面不得有 blur、覆盖移动内容的必须保留）。唯一已知的可见副作用不是降级：面变成合成层后，**面内文字会重抗锯齿**（≤16/255 侧栏、≤64/255 气泡）。
   **推论二（issue #19 实测，同样必须记）：不要把 blur 加在「托管浮层」的容器上。** 带 `backdrop-filter` 的元素会成为其后代的 **backdrop root**，后代只能读到**该元素自己画的那层**（页面内容不在其中）；命令面板 `[data-trigger-menu]` 就挂在 `[data-composer-card]` 的 `conversation.input.overlay` 锚点里，且整块**悬在卡盒之外**，于是它自带的 `blur(40px)` 读空 ⇒ 底下的转录原样透出。**推论：材质必须整体搬到容器内部的 `::before` 材质面（填充 + blur 都要，别只搬 blur），容器本体保持零绘制**——这正是上游 `MenuSurface.module.css` 的做法（`isolation: isolate` 外壳 + `z-index:-1` 材质面，其注释原话就是「keeps nested menus free of an ancestor backdrop root」）。`isolation: isolate` **不是** backdrop root，可以放心用。护栏 = `tests/glass-css.spec.ts` 的 `backdrop-root hygiene` 一组；判定手法与实测数字见 `skills/css-backdrop-root-probe` 与 `docs/issue-19-palette-blur.md`。

### 用这条准绳做过的判定（可引用的先例）

- 2026-09-15 **驳回** MM（暗色主 accent 提亮）：驳回理由**不是「官方色不许动」**，而是它拿不出「官方取值在 DSH 下不成立」的证据（`#89b4fa` / `#8caaee` 在 `crust` 上远高于 AA，属审美偏好而非缺陷），且 `brand-primary` 已被 `tests/palettes.spec.ts:105` 锁成契约。想要更亮的主操作色走 **K（token 覆盖）**，不走 palette。
- 2026-09-15 **立项** VV（success / warn tertiary 对比度）：实测 3.18~4.37 低于 AA，就是「官方取值在本项目不成立」的那类证据；修法仍走第一步（只换混色目标、色相不变）——规则 1 与规则 3 各占一半。

两次判定的完整理由见 `docs/plugin-improvements.md`（「六、2026-09-15 复核」）。

## 项目身份速览：三套名字，不要混淆

这个仓库同时存在三套不同的「名字」，改动前必须先分清（2026-08 仓库改名时踩过一次坑）：

| 名字 | 值 | 说明 |
|---|---|---|
| GitHub 仓库名 | `NoNameLeGo/dsh-catppuccin-theme` | 已从 `dsh-catppuccin` 301 改名，GitHub 自动跳转旧链接 |
| npm 包名 | `@nonamelego/dsh-catppuccin` | **不要改成 `-theme`**——安装/更新检查代码都依赖它（`src/update-check.ts` 的 `PACKAGE_NAME`、`cordis.patch.yml` 的 `name`、README 里的安装/升级命令） |
| 插件 ID / 显示名 | `dsh-catppuccin` | `src/index.ts:54` 的 `export const name`；运行时的 Cordis 插件身份，不可改 |

- 代码/配置里 `grep dsh-catppuccin` 命中的大多数**都不该改**。只有**完整的 GitHub URL**（`github.com/NoNameLeGo/dsh-catppuccin`，不带 `-theme`）才需要更新为新仓库名。
- npm 包名、插件 ID、本地路径 `link:D:\Vibe-Coding\dsh-catppuccin`（README 里的本地开发命令）一律保持原样。
- `git remote` 的 `origin` 指向 `https://github.com/NoNameLeGo/dsh-catppuccin-theme.git`。

## 上游形态与维护核心（2026-09-22 定）

**维护核心 = 官方的 web + desktop。** 官方上游是 monorepo
[`deepseek-ai/deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness)：

| 上游位置 | 是什么 | 与本插件的关系 |
|---|---|---|
| `apps/web` | Web GUI（`dsh web` / `web` profile） | **核心**：`--dsw-*` token 体系与玻璃层的适配对象 |
| `apps/desktop` + `apps/desktop-host` | 官方 Electron 桌面壳（`private: true`；**已作为客户端发行**，2026-10-02 本机装机版 = `@deepseek-ai/dsh-desktop@0.2.0-rc.2`，仍在开发迭代），启动 `$DSH_HOME/profiles/desktop` | **核心**：装/升走**应用内插件界面**（见下「两个壳两条路」），**不是** `--profile desktop` 那条命令 |
| `apps/cli` → npm `@deepseek-ai/dsh` | CLI：profile 启动 / `dsh plugin` 转发 pnpm | 安装与更新检查的宿主；⚠️ **对 `desktop` 这个 profile 名会硬拒**（见下） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NoNameLeGo/dsh-catppuccin-theme](https://github.com/NoNameLeGo/dsh-catppuccin-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
