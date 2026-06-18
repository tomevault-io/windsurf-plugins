---
trigger: always_on
description: 把任意代码仓库转成可在 Obsidian 中搜索的知识库。结合 graphify 与 code2prompt 索引符号、按模块打包源码到 Markdown。当用户想"把代码放进 Obsidian"、"在 Obsidian 里导航仓库"、"做一个代码 wiki"、"把代码库存到 Obsidian 以后再看"，或者询问 graphify/code2prompt 与 Obsidian 组合用法时触发。匹配短语："用 Obsidian 看代码"、"把代码库整理成知识库"、"code vault"、"代码导航"。
---


# code-vault — 把任意代码仓库变成 Obsidian 知识库

## 这个 skill 做什么

给定一个代码仓库，本 skill 会在用户的 Obsidian vault 内，按模块构建一份 Markdown 知识库：

```
~/Documents/Obsidian Vault/<repo-name>/
├── 00-Index.md                 ← 模块目录
├── .code-vault/
│   ├── config.yaml             ← 用户可编辑的模块定义
│   └── modules.conf            ← 自动生成的 glob 列表（机器可读）
└── modules/
    ├── <module-a>/
    │   ├── README.md           ← 人工撰写的概览
    │   ├── code.md             ← 提取的源码（来自 code2prompt）
    │   └── flow.md             ← 调用关系（来自 graph）
    └── ...
```

## 调用方式

用户在 Claude 中输入以下任一形式：

- `/code-vault <repo-path>` — 为某个仓库初始化一个新 vault
- `/code-vault <repo-path> refresh` — 重建所有模块
- `/code-vault <repo-path> build <module>` — 重建单个模块
- `/code-vault <repo-path> list` — 显示已检测到的模块

`<repo-path>` 必须是绝对路径。

## 前置依赖

继续之前请先确认这些工具已安装。如果缺失，先装再继续。

| 工具 | macOS 安装 | Linux 安装 | 用途 |
|------|-----------|-----------|------|
| `code2prompt` | `pip install code2prompt` | `pip3 install code2prompt` | 提取源码 → markdown |
| `graphify` | `brew install graphify` | `npm i -g graphify-ts` | 构建符号图 |
| `python3` | 系统自带（或 `brew install python`） | 系统自带 | 模块检测、flow 分析 |
| Obsidian vault | macOS 默认：`~/Documents/Obsidian Vault/` | 自选路径，设置 `OBSIDIAN_VAULT` | 输出目标 |

如果用户的 Obsidian vault 不在默认位置，先问一下再继续。

**验证安装：**

```bash
which code2prompt graphify python3
code2prompt --help 2>&1 | head -3
graphify help 2>&1 | head -3
```

如果 macOS 上明明 `brew install` 过了却报 `graphify: command not found`，
检查一下 `/opt/homebrew/bin/graphify`（Apple Silicon）或
`/usr/local/bin/graphify`（Intel）。

## 常见坑（首次运行前必读）

这些都是真正会踩到的坑，每条都是血泪教训：

1. **`code2prompt` 不支持 `**` glob。** 像 `src/**/*.cpp` 这样的模式会被解读成
   "任何包含 `src/` 的路径"，导致过度匹配，把 `platform/`、`third_party/` 也
   一起拉进来。**修法：** 永远用具体的扁平 glob：
   - ❌ `src/**/*.cpp`
   - ✅ `src/app/*.cpp src/app/*.h src/framework/*.cpp ...`

   当你不写 `config.yaml` 时，`detect-modules.py` 会自动生成这种扁平 glob；
   只有要自定义时才需要手写。

2. **YAML 的 glob 模式必须加引号。** 行首的 `*.sh` 会被解析成 alias 引用，让
   PyYAML 直接报错。**修法：** 始终加引号：
   ```yaml
   patterns:
     - "*.sh"
     - "src/**/*.cpp"   # 仍然不要用 **，见上一条
   ```

3. **在大仓库 + vendored SDK 上 graphify 会静默返回 0 个符号。**
   症状：`Indexed 5448 files, 0 symbols, 0 relationships`。
   原因：项目自己的代码和几千个三方头文件（如
   `platform/rv1106/prebuilt/include/...`）一起解析，解析器直接被噎住。
   **修法：** 用 skill 自带的辅助脚本：
   ```bash
   ~/.claude/skills/code-vault/scripts/refresh-graph.sh /path/to/repo
   ```
   它会临时隐藏 `platform/`、`third_party/`、`vendor/`、`third/`、`oss/`、
   `node_modules/`，跑完 `graphify build .` 后自动恢复。

4. **graphify 不需要 `compile_commands.json`。** graphify 用的是 tree-sitter
   风格的解析器，直接读源文件。别浪费时间去配 cmake 交叉编译工具链只为
   喂 graphify。

## 输出目录自动创建

所有入口（Claude 斜杠命令、`code-vault.sh`、`init.sh`、`refresh.sh`、
`build-module.sh`）在 vault 目录不存在时都会自动创建。用户**不需要**预先
`mkdir` 任何东西。

| 创建什么 | 何时 | 由哪个脚本 |
|---------|------|-----------|
| `$OBSIDIAN_VAULT`（父目录） | 未设置 / 路径不存在 | `init.sh`、`refresh.sh`、`code-vault.sh build` |
| `<vault>/.code-vault/` | vault 存在但子目录缺失 | 任意入口 |
| `<vault>/modules/` | vault 存在但子目录缺失 | 任意入口 |
| `<vault>/modules/<module>/` | 逐模块构建时 | `build-module.sh` |

`init.sh` 中的路径解析顺序：
1. `$OBSIDIAN_VAULT` 环境变量 —— **即使路径不存在也尊重用户的设置**，会创建出来。
2. `detect_obsidian_vault`（自动扫描 `~/Documents/Obsidian Vault` 等）
3. 默认 `$HOME/Documents/Obsidian Vault`（不存在则创建）

**仍会硬失败（无法自动解决）**：
- `modules.conf` 缺失 → 提示"先跑 init.sh"。模块定义必须由 init 生成，
  构建阶段没法凭空造。
- `code2prompt` / `graphify` 未安装 → 必须用户自己装（见上方安装命令）。

## 工作流（Agent 应遵循的步骤）

### Step 1：解析参数 & 校验

1. 从用户消息中提取 `<repo-path>` 和任何子命令。
2. 确认路径存在且是目录。
3. 确认前置依赖已安装（`which code2prompt`、`which graphify`、`which python3`）。
4. 定位 Obsidian vault：
   - 默认：`~/Documents/Obsidian Vault/`
   - 或扫描常见位置：`~/Obsidian*`、`~/Documents/*obsidian*`
   - 仍未找到则问用户。

### Step 2：选择 vault 名字

- 默认用 `<repo-path>` 的 basename（如 `tesla-vending-new` → `tesla-vending-new`）。
- 如果 Obsidian vault 里已有同名目录，问：覆盖 / 合并 / 另选名字？

### Step 3：构建符号图

```bash
cd <repo-path>
graphify build .
```

如果 `graphify-out/graph.json` 已存在则跳过（增量构建）。

### Step 4：检测模块

跑自动检测器：

```bash
python3 ~/.claude/skills/code-vault/scripts/detect-modules.py \
    <repo-path> \
    <vault-path>/<repo-name>/.code-vault/modules.conf
```

这会生成一个 `modules.conf`，每个模块一行，格式为：
```
module-name|Title::Description|glob1 glob2 glob3
```

### Step 5：让用户自定义（可选但建议）

自动检测完成后，把检测到的模块给用户看。用户可以：
- 编辑 `.code-vault/config.yaml` 做精细控制（模板在
  `~/.claude/skills/code-vault/templates/config.yaml`）
- 或者直接说"可以，继续"，用自动检测的结果

如果用户要自定义，从 YAML 重新生成 `modules.conf`。

### Step 6：逐模块跑构建

对每个模块跑通用 builder：

```bash
~/.claude/skills/code-vault/scripts/build-module.sh \
    <repo-path> \
    <vault-path>/<repo-name> \
    <module-name> \
    <vault-path>/<repo-name>/.code-vault/modules.conf
```

脚本做的事：
1. 从 `modules.conf` 读取对应条目
2. 跑 `code2prompt -f <globs>` → `code.md`
3. 从 `code.md` 解析文件列表（单一信息源）
4. 用 Python 生成 `flow.md`：关键符号 + 外部调用者（只 grep 源码扩展名）
5. 从模板生成 `README.md`

### Step 7：生成 00-Index.md

所有模块都构建完后，从 `modules.conf` 生成索引页：

```bash
python3 ~/.claude/skills/code-vault/scripts/gen-index.py \
    <vault-path>/<repo-name>/.code-vault/modules.conf \
    <vault-path>/<repo-name> \
    > <vault-path>/<repo-name>/00-Index.md
```

### Step 8：汇报

告诉用户：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hy546880109/code-vault](https://github.com/hy546880109/code-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
