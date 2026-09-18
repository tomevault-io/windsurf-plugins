---
trigger: always_on
description: 本文件是写给 **AI Agent（DSH / Codex / Claude Code 等）**看的。人类读者请看 [README.md](README.md)。
---

# AGENTS.md — 给 AI Agent 的安装与操作说明

本文件是写给 **AI Agent（DSH / Codex / Claude Code 等）**看的。人类读者请看 [README.md](README.md)。

仓库：`https://github.com/Guzhou2002/Fairy-DSH-Optimized`
上游（非本分支作者）：`https://github.com/Chengzhibense/Fairy-DSH`

---

## 1. 你的任务边界

用户让你"装 Fairy"时，默认只需要装 **3 个安全插件**：

| 包 | 能否由你自动安装 | 说明 |
| --- | --- | --- |
| `dsh-fairy-visual` | ✅ **可以** | 核心 UI，无外部服务依赖 |
| `dsh-balance-meter` | ✅ **可以** | 余额挂件，无外部服务依赖 |
| `dsh-fairy-voice` | ⚠️ **能装，但朗读不会响** | 朗读需要用户本机跑 GPT-SoVITS（`127.0.0.1:9880`）+ 参考音频。**你无法替他启动 TTS 服务或生成参考音频** |
| `dsh-fairy-startup` | ❌ **不要装** | 每次启动清空用户的会话选择，会丢掉他没结束的对话 |
| `dsh-browser-dock` | ❌ **不要装** | 暴露控制 token、页面截图落盘、takeover 硬编码 macOS 路径 |

**除非用户明确点名要后两个、并且你已把风险讲清楚，否则不要装。**

---

## 2. 标准安装流程

### 第 0 步：确认环境

```powershell
where pnpm      # 没有就让用户先装：npm install -g pnpm
where dsh
dsh --version
```

### 第 1 步：安装（一条命令）

```powershell
dsh plugin --profile web add `
  https://github.com/Guzhou2002/Fairy-DSH-Optimized/releases/latest/download/dsh-fairy-visual.tgz `
  https://github.com/Guzhou2002/Fairy-DSH-Optimized/releases/latest/download/dsh-fairy-voice.tgz `
  https://github.com/Guzhou2002/Fairy-DSH-Optimized/releases/latest/download/dsh-balance-meter.tgz
```

**要点**：

- `--profile web` 是默认 profile 名；**先问用户用的是哪个 profile**，不要假设
- URL 一律用 `releases/latest/download/<包名>.tgz` —— 这是**永久地址**，带版本号会 404
- 每个包自带 `dsh.bundle.patch` 声明，所以**不需要**你手写 `cordis.patch.yml`
- 只装某一个就把其余 URL 删掉即可

### 第 2 步：验证（必做）

```powershell
dsh --profile web --dump-config
```

成功标志：输出里能看到对应插件条目，且**没有报错**。

### 第 3 步：告诉用户手动做两件事

你**无法**替他完成这两步，必须明确提示：

1. **重启 DSH**（插件在启动时装载）
2. **设置 → Fairy → 打开「启用」开关** —— 不开的话视觉和朗读都不出现，这是设计如此

---

## 3. 卸载

**优先让用户双击 `uninstall.cmd`**（**必须**和 `uninstall.ps1` 放同一个文件夹 —— 那个 `.cmd`
是个纯 ASCII 启动器，逻辑在 `.ps1` 里）。它会备份配置 → 卸插件 → **还原「新会话默认预设」**
→ 删预设目录 → 扫历史残留 → 校验 profile → 把保留的用户数据路径打给用户。

> 🔴 **这条别漏**：卸载**必须**还原 `settings.yaml` 里的 `agent-presets.default`
> （优先用 `~/.dsh/.fairy-persona/default-preset-backup.json`，没有就回落 `standard`）。
> 不还原的话，默认值还指着已被删掉的 `fairy` → **「点新建会话没反应」**，卸载反而把机器弄坏。

会打命令才用手工：

```powershell
dsh plugin --profile web remove dsh-fairy-visual dsh-fairy-voice dsh-balance-meter
.\uninstall.ps1 -CleanBundle   # 老版本（0.2.x）留过受管块 / package.json 有残留行时才需要，会先备份
```

> 🔒 **卸载不删用户数据**：`~/.dsh/fairy-voice/`（参考音频 / 朗读设置 / 语音简报 API Key）原样保留，
> 只在最后把路径打给用户，删不删由他自己决定。

另一个做法是双击仓库根目录的 `install.cmd` 重新安装（它是安装器也是修复器）。

---

## 4. 验收清单

装完请逐条自检，并在回复里告诉用户哪些通过了：

- [ ] `dsh plugin --profile web add` 退出码为 0
- [ ] `dsh --profile web --dump-config` 能看到插件条目、无报错
- [ ] 已提示用户**重启 DSH**
- [ ] 已提示用户**打开「启用」开关**
- [ ] 若装了 `fairy-voice`：已告知用户**朗读需要本机 GPT-SoVITS**，否则按钮是灰的
- [ ] 若用户想要人设：已告知去 **设置 → Fairy → 打开发「Fairy 人设预设」**

---

## 5. 故障判定表（**先拿判别信号，再动手**）

> **用法**：按「判别信号」这一列去比对。**表里没列的原因不要猜** —— 命中 §5.5 任一条就停下来问用户。
> `🛑 停` = 你不许自己试，把**信号原文 + 这一行判断**贴给用户，由他决定。

### 5.0 三条铁律（违反必出事）

1. **不要"统一版本"**：`@deepseek-ai/dsh-settings` 必须保留插件自己 pin 的 `0.1.1-rc.2`。
   提升成宿主的 `0.1.2-rc.1` → 插件在**导入阶段直接失败**。
2. **不要碰正在运行的 DSH**：GUI 监听 `127.0.0.1:3080`。清理进程前先按端口确认归属（曾差点误杀）。
3. **改 `.cmd` / `.ps1` 之前先读 §6.1 / §6.2**：编码错了安装必然失败，而且在开发机上**复现不出来**。

### 5.1 装不上（安装阶段）

| 判别信号 | 判定 | 你该做什么 |
| --- | --- | --- |
| 下载 `.tgz` 卡住 / 超时 / `Failed to connect to github.com:443 after 21xxx ms` | 国内直连 GitHub 常超时 | 让用户挂代理；或改用 `install.cmd`（它有网络自查）。**别默认"绕开代理"** —— 若梯子在跑、代理也配着，"绕开代理"反而会因 schannel 取不到凭据而失败 |
| `schannel: SEC_E_NO_CREDENTIALS` | 三查：`git config --get-regexp proxy` / 注册表 `ProxyServer` / 梯子端口在不在监听 | **先确认梯子在跑，再考虑权限**。详见 `docs\交接摘要.md` §9 坑 15 |
| `dsh: pnpm failed` | pnpm 没装，或网络 | `where pnpm`；缺就 `npm install -g pnpm`，重试 |
| 报 **404** | URL 带了版本号 | 只能用 `releases/latest/download/<包名>.tgz`（**不带版本号**，这是永久地址） |
| 双击安装器**一闪就没了** | 报错随窗口一起消失 | 让用户在**命令行**里跑同一个 `.cmd` 看输出 |
| 🔴 满屏 `'xx' is not recognized as an internal or external command`，**但标题和部分 echo 又显示正常** | **安装器编码坏了**（不是用户操作错）：`.cmd` 存成了 UTF-8，在 `chcp=936` 的机器上 `rem` 注释行被从中间切开当命令执行 | 让用户敲 `chcp` 确认（**936 = 会中招**，65001 = 不会）。若是本仓库要重新发版 → 按 §6.1 用 GBK 写回 |
| `node_modules` 里只有一个 Junction、没有 `@deepseek-ai` 相关目录 | 用了 `link:` 形态 —— **`link:` 不装依赖** | 换成 `file:` / `.tgz`，或先手动装依赖 |
| `plugin add` 成功，但 `dump-config` 里**没有**插件条目 | 包没声明 `dsh.bundle.patch` / 缺 `cordis.patch.yml` | 🛑 **停**：说明包本身有问题，**别手写 `cordis.patch.yml` 去补** |
| 路径解析失败 | 用了 Windows 反斜杠 | `link:` 路径一律用**正斜杠** |
| 用户后来删了本地 `.tgz`，pnpm 就找不到包 | 本地 tgz 会把**绝对路径** `file:C:/.../x.tgz` 写进 profile | 改用**远程 URL** 或 npm 包 |
| 设置里找不到 Fairy **人设预设** | 包内 `.agent-presets/` **不会**被自动复制到 `$DSH_HOME\.agent-presets\` | 手动复制；`install.cmd` 会做这一步 |
| 从很老的版本升级上来，出现重复注册 / `Fairy-DSH managed block` | `0.1.x` 老写法的遗留块 | 🛑 **停**：不要自动清理。先让用户备份 profile，确认后再动 |

### 5.2 装上了但没反应（加载 / 启动阶段）

| 判别信号 | 判定 | 你该做什么 |
| --- | --- | --- |
| 界面一切照旧，毫无变化 | 两种可能，**按顺序排除**：① 没重启 DSH ② 「启用」没开（默认 `false`，设计如此） | 让用户**重启** + **设置 → Fairy → 打开「启用」** |
| 重启后设置里仍然没有 Fairy | 插件没挂上 | `dsh --profile web --dump-config`，看有没有 `fairy-visual` 条目 |
| 加载失败，报 **`settingsNamespace` 不存在** | 依赖被提升成了宿主的 `0.1.2-rc.1` | 🛑 **绝不要"统一版本"**。`settingsNamespace` 只是"校验命名空间格式后原样返回字符串"，与宿主 `register(ns, schema)` 兼容；保留插件自己的 `node_modules` 即可 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Guzhou2002/Fairy-DSH-Optimized](https://github.com/Guzhou2002/Fairy-DSH-Optimized) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
