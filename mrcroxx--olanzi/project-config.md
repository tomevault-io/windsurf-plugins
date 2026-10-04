---
trigger: always_on
description: > 本文件是**项目级记忆**，每次会话开始时自动加载。修改约定前请先读这里。
---

# AGENTS.md · 项目 Memory

> 本文件是**项目级记忆**，每次会话开始时自动加载。修改约定前请先读这里。

---

## 1. 语言约定

| 场景 | 语言 |
|---|---|
| 与用户对话 | **中文** |
| 代码注释 | 中文 |
| 提交信息 | **英文**（仓库公开，历史应对英文读者可读） |
| 文档 | **中英双语（强制，见 §2）** |

---

## 2. ⭐ 文档双语强制要求

**`README` 和 `docs/` 下的每一份文档都必须有中英两个版本。**

### 命名规范

⭐ **英文是默认入口，文件名不带语言后缀。** 中文版加 `.zh.md`。

| 英文版（默认，无后缀） | 中文版 |
|---|---|
| `README.md` | `README.zh.md` |
| `docs/01-ulanzi-studio-scope.md` | `docs/01-ulanzi-studio-scope.zh.md` |
| `docs/NN-<ascii-slug>.md` | `docs/NN-<ascii-slug>.zh.md` |

- 文件名**一律用 ASCII slug**（不要用中文文件名）
- 编号 `NN` 中英一致，便于对照
- 两版是**同级目录下的兄弟文件**，不另建子目录
- ⚠️ **不要再新增 `.en.md`** —— 无后缀的那份就是英文版。仓库公开，
  默认入口必须是英文，中文是可选语言

### 硬性规则

1. **新增文档必须同时产出中英两份。** 只写一份视为未完成。英文用无后缀文件名。
2. **修改任一语言版本后，必须同步另一版本。** 不允许长期漂移。
3. **两版的 Markdown 结构必须一一对应**：标题层级、表格列数、代码块、列表层级全部一致。
4. **每份文档头部都要有文档集导航**，中文版链中文、英文版链英文。
5. **中英互链**：英文版头部放 `[中文](....zh.md)`，中文版头部放 `[English](....md)`（指向无后缀的英文版）。
6. **改完必须跑校验**：

   ```bash
   python3 tools/check_docs.py
   ```

   它检查 7 项：中文版是否存在（含孤立的 `.zh.md` 反向检查）、两版结构是否一一对应
   （标题层级 / 代码块数 / 表格形状）、**十六进制数值是否一一保真**、互链是否正常、
   `docs/` 下每份文档是否有 `📚` 导航行、
   **英文版散文是否残留中文**（围栏代码块与行内代码中的中文是合法引用，豁免）、
   相对链接是否全部可解析。**不通过就是没做完。**

### 翻译规范 —— 什么**不**翻译

以下内容**原样保留**，一个字都不要改：

- 十六进制 / 字节序列（`01 06 50 04`、`0xFFF1`、`ca ba a5 ca …`）
- 代码块中的**代码本身**（Python / bash / 汇编指令）
- 命令与参数（`--set-key`、`python3 vibekey.py --keys`）
- 文件路径（`~/ulanzi-re/raw/…`）
- 符号名（`+[MessageHelper setDeviceButtonShortcutFunctionMessage:…]`）
- 错误码（`0xE00002E2`、`kIOReturnNotPermitted`）
- markdown 表格结构与对齐
- emoji 标记（`✅ ⚠️ ⭐ ⌨ ⟳ ❌`）
- 证据强度标签 `[CONFIRMED]` / `[INFERRED]`（本来就是英文）

#### ⚠️ 代码块里的中文分两类，别搞混

| 类型 | 处理 | 例 |
|---|---|---|
| **真实程序输出 / 终端记录** | **一字不改**，中文原样留着（它是证据） | `13:36:28.960 按键 ⌨ 旋钮 按下 PrintScreen 178 ms`、`← 未使用` |
| **说明性记法里的中文占位符** | **翻译** | `<类型\|sign<<7> <键码> ×num` → `<type\|sign<<7> <key code> ×num` |
| **代码注释** | 翻译注释，保留 `;` `#` `//` 等标记与代码本身 | `; 帧头 01 06 50 04` → `; frame header 01 06 50 04` |
| **CLI 参数的元变量** | 翻译 | `--log 文件` → `--log FILE` |

> 判据：**这段中文是"设备/程序真的吐出来的"还是"作者写给人看的说明"？**
> 前者留，后者翻。终端记录是证据，改动它就破坏可审计性。

#### 英文文档里引用程序的中文输出

`vibekey.py` 的输出**就是中文**。英文文档在描述"你会看到什么"时，
**应当用行内代码原样引用中文**，这样读者才能和屏幕上的输出对上：

```markdown
| Category | `按键` (key) / `旋钮` (knob) |
The symptom `✗ 打不开 … 无权限` means the input-monitoring permission is missing.
```

校验器**只检查散文里残留的中文**，行内代码与围栏代码块中的中文一律豁免 ——
所以引用原文不会被判定为漏译。

### 术语表（保持一致）

| 中文 | English |
|---|---|
| 键 / 按键 | key |
| 旋钮 | knob |
| 旋钮按下 | knob press |
| 旋钮 → 右拧 / ← 左拧 | knob twist → right / ← left |
| 控件 | control |
| 厂商通道 | vendor channel |
| 输入接口 | input interface |
| 帧 | frame |
| 明文 / 密文 | plaintext / ciphertext |
| 保活 | keepalive |
| 心跳 | heartbeat |
| 按键配置 / 可编程按键表 | key configuration / programmable key table |
| 设备端 | on-device / device-side |
| 逆向 | reverse engineering |
| 抓包 | capture |
| 反汇编 | disassembly |
| 符号表 | symbol table |
| 验证记录 | verification log |
| 职责边界 | scope / responsibility boundary |
| 踩坑 | pitfalls |
| 实测 | measured / verified |
| 待验证 | to be verified |
| 固件 | firmware |
| 电量 | battery |
| 降噪 | noise reduction |
| 指示灯 | indicator light |
| 组合键 | key combination |
| 修饰键 | modifier key |
| 时延 / 时长 | duration |

### 文档集

| # | English（默认） | 中文 | 内容 |
|---|---|---|---|
| — | `README.md` | `README.zh.md` | 项目总览、快速上手 |
| 01 | `docs/01-ulanzi-studio-scope.md` | `.zh.md` | Studio 职责边界 |
| 02 | `docs/02-vibekey-protocol.md` | `.zh.md` | 协议：TEA / 帧格式 / 命令表 / 按键表 |
| 03 | `docs/03-tool-manual.md` | `.zh.md` | `vibekey.py` 工具手册 |
| 04 | `docs/04-methodology.md` | `.zh.md` | 逆向方法论 |
| 05 | `docs/05-verification-log.md` | `.zh.md` | 验证记录 |

---

## 3. 项目地图

```
olanzi/
├── AGENTS.md                      ← 本文件（项目记忆，不翻译）
├── README.md / README.zh.md       ← 入口（英 / 中）
├── vibekey.py                     ← 工具本体（单文件，零第三方依赖）
├── tools/
│   └── check_docs.py              ← 文档双语一致性校验
└── docs/
    ├── 01-ulanzi-studio-scope.md  (+ .zh.md)
    ├── 02-vibekey-protocol.md     (+ .zh.md)
    ├── 03-tool-manual.md          (+ .zh.md)
    ├── 04-methodology.md          (+ .zh.md)
    ├── 05-verification-log.md     (+ .zh.md)
    └── evidence/                  ← 原始证据留档（不翻译）
```

**逆向中间产物在 `~/ulanzi-re/`**（反汇编、符号表、62 MB 解码日志），
属于临时工作区，**不入库、不翻译**。

---

## 4. 关键技术不变量

> 新会话不需要重新推导这些，直接用。完整细节见 `docs/02-vibekey-protocol.zh.md`。

### 设备

- **AU05**（Vibe Key），VID `0xFFF1` / PID `0x00DD`，序列号 `202606031150`
- ⚠️ **设备唯一标识不公开**：`deviceSn` 与 `flashId` 完整值一律写成 `<REDACTED>`，
  `docs/evidence/` 日志里用 `xx` 掩码（保持字节数与偏移）。序列号保留作示例。
- 控件：**3 个键（上下排列）+ 1 个旋钮 + 1 个电源键**
- 接口 2 = 标准 HID（Consumer `0x01` / Mouse `0x02` / Keyboard `0x03`）
- 接口 3 = 厂商私有（Usage Page `0xFFFC`，Report ID `0x55`）
- **电源键不发报文**（设备硬件处理）

### 加密

- **TEA / ECB / 8 字节分组 / 32 轮**（不是 XTEA）
- delta `0x9E3779B9`，解密 sum 起始 `0xC6EF3720`
- 密钥 `ca ba a5 ca 6d 8a 2a bc ba 9e 5a ca ca 8b b8 9b`
- 验证：`TEA_Enc(00×8) == 38 90 c4 99 a3 60 aa ad`

### 帧

- **明文 64 字节，只发前 63 字节**（第 64 字节位置被 report ID 占用）
- **解密只解 7 个分组（56 字节）**，末 7 字节是不可解填充
- 帧头 `cmd = frame[0] & 0x1F`；回复标记 `frame[0] & 0x80` 或 `frame[3] & 0x10`

### 控件映射（出厂值）

| 控件 | index | 键码 |
|---|---|---|
| 键 1（上） | 0 | `0x01` ErrorRollOver（**无效码**） |
| 键 2（中） | 1 | `0x28` Enter |
| 键 3（下） | 2 | `0x29` Esc |
| 旋钮 按下 | 3 | `0x46` PrintScreen |
| 旋钮 → 右拧 | 4 | `0x4F` RightArrow |
| 旋钮 ← 左拧 | 5 | `0x2A` Backspace |

### 按键配置读写

```
读:  → 01 06 50 01 <index>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrCroxx/olanzi](https://github.com/MrCroxx/olanzi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
