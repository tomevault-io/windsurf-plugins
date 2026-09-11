---
trigger: always_on
description: 本文件用于给后续 AI / Codex 说明独立版 Crimson Desert 模组加载器 `cdmm` 的运行方式、底层原理、已踩坑点，以及 Format 3 模组尚未完成的范围。所有沟通默认使用中文，所有新增或修改文件使用 UTF-8 编码。
---

# AGENTS.md

本文件用于给后续 AI / Codex 说明独立版 Crimson Desert 模组加载器 `cdmm` 的运行方式、底层原理、已踩坑点，以及 Format 3 模组尚未完成的范围。所有沟通默认使用中文，所有新增或修改文件使用 UTF-8 编码。

## 版本号规范

- 项目根目录 `version.txt` 是 VFS、Physical 与废弃兼容加载器发布版本号的唯一来源。
- VFS 成品文件名、控制台标题、错误提示以及 Windows PE 的 FileVersion/ProductVersion 必须由 `version.txt` 派生，禁止在 Python 或 PowerShell 中重复硬编码版本号。
- 简洁版本（例如 `v3`、`v1.1`）在 PE 元数据中依次补零为四段数字（例如 `3.0.0.0`、`1.1.0.0`）。

## VFS 成品打包强制基线

- VFS 正式成品禁止使用 Nuitka `--onefile`。2026-07-12 已实机确认：相同业务代码打成单文件后，Nuitka 会先释放到临时目录再启动，Crimson Desert 的 Steam / ASI / 保护层启动链会失败；改回 `--standalone` 目录版后可正常进入游戏。
- VFS 目录版固定结构如下，外层 EXE 只负责转发到真实 core：

```text
cdloader-VFS-v<版本>.exe
cdloader/
  cdloader-vfs-core.exe
  cdmm/private/vfs_runtime/
```

- `tools/vfs_launcher.py::resolve_game_dir()` 必须同时检查 core 自身目录和父目录。目录版真实进程位于 `游戏根目录\cdloader\cdloader-vfs-core.exe`；如果只检查 `executable_dir()`，会错误提示“未识别到游戏根目录”，即使外层 EXE 已正确放在游戏根目录。
- 后续不得再次删除 `for candidate in (exe_dir, exe_dir.parent)` 的父目录识别逻辑。
- 每次 VFS 目录版打包至少运行 `test/test_vfs_launcher_command.py`，其中必须保留“core 位于一级发布目录时解析回游戏根目录”的回归测试。
- 2026-07-12 实机确认：修复父目录识别后的目录测试包可正常进入游戏。单文件模式不可作为回退方案；VirusTotal/Nexus 是否放行不能凌驾于实际可启动性。

## Physical 实体加载强制基线

- Physical 是 Windows 10 等无法稳定使用 native VFS runtime 用户的兼容入口。它只复用 VFS 的模组合成与 DMM-like 分包构建结果，然后把产物真实写入游戏目录；启动游戏时禁止调用 `nppvfs_launcher.exe`、`--asi-load`、`vfs_runtime.dll` 或任何 VFS Hook/注入链路。
- Physical 与 VFS 必须共用 `services/vfs_loader.py::build_vfs_package()` 生成的最终产物，禁止恢复旧 `services/loader.py::apply_loader()` 单数字 overlay 管线。相同游戏版本、语言、mods 和加载顺序下，两种模式的 `nppv3_*`、`nppvoice`、`nppgen`、`nppsa` PAZ/PAMT 字节以及 PAPGT 注册顺序必须一致。
- `nppv3_*` 保存已拆分的 Format 3 表结果，`nppvoice` 保存 `.wem` 语音 loose，`nppgen` 保存传统 JSON/通用表补丁，`nppsa` 保存其余 loose 与 `.cdmod` 文件替换。`nppsa` 很大通常是大量 PAC、DDS、PAA、UI、模型或动画资源合成后的正常结果，不能因为目录大就重新塞回原始编号包。
- 模组提供 `files/0012/...` 或根部 `0012/...` loose 文件时，`0012` 只用于定位原版 PAMT entry；最终修改内容写入 `nppsa`，严禁改写游戏原始 `0012/0.paz` 或 `0012/0.pamt`。这一规则适用于所有原始 `0000-0035` 目录。
- 模组自带完整 `NNNN/0.paz + NNNN/0.pamt` 时属于 standalone archive，必须保留完整包并分配安全的新四位数字目录（通常从 `0037` 起）。因此 Physical 同时出现 `npp*` 与 `0037/0040/0043` 等目录是合法结构：前者是合成分包，后者是 standalone，不得误判为旧数字 overlay。
- Physical 从始至终不得覆盖游戏原始编号 PAZ/PAMT。它对游戏源文件的覆盖只允许是 `meta/0.papgt`，存在 DDS/PATHC 变化时再覆盖 `meta/0.pathc`；`npp*` 和新分配的四位数字目录均为加载器新增文件。两个 meta 文件写入前必须由 `VanillaStore` 保存当前游戏版本的 vanilla 备份。
- Physical 必须使用 `Transaction` 事务提交 npp/standalone/meta。新文件全部提交成功后，才能清理状态中记录的废弃 `npp*`、standalone 或 v9.2 早期 `0039/0040` 单数字 overlay；不得先删旧产物再构建，避免失败后游戏既无法恢复也无法启动。
- `.cdloader/state.json` 必须记录 `physical_output_files`、`physical_output_dirs` 和 `standalone_dirs`；`.cdloader/physical_mode_state.json` 必须记录模组指纹、加载器版本、PALOC 语言、游戏 EXE 修改时间及实体输出快照。缓存命中时直接原生/Steam 启动，不重复构建或复制大型分包。
- 游戏更新、Steam 校验、语言变化、模组新增/删除/修改/排序变化或实体输出缺失时，Physical 缓存必须失效。游戏更新后只能把 Steam 实际替换的新版本 meta 刷新为 vanilla 备份，严禁把仍处于 Physical 状态的旧实体 meta 收录成新 vanilla。
- 用户删除全部 mods 后再次运行 Physical，必须恢复 vanilla PAPGT/PATHC 并清理全部加载器新增目录。`--revert` 同样必须恢复正确游戏版本的 vanilla meta、删除所有记录的 `npp*` 与 standalone、清除 Physical 模式锁；禁止仅删除 `physical_mode_state.json` 冒充恢复。
- Physical active/pending 状态必须阻止 VFS 构建和启动，因为游戏实体 meta 已被修改；只有完整 `--revert` 成功后才能解除互斥锁并重新允许 VFS。
- Physical 冷构建必须复用 VFS 的控制台阶段回调，并额外显示实体文件名、大小、事务提交、旧目录清理和状态保存。耗时阶段至少每 3 秒输出一次“仍在处理”，避免低性能电脑用户误以为程序卡死。
- Physical 与 VFS 两个外层 EXE 共用 standalone 目录版 `cdloader/cdloader-vfs-core.exe`。版本号、PE 元数据和文件名仍只能由 `version.txt` 派生；每次修改 Physical 至少运行 `test/test_physical_launcher.py`、`test/test_vfs_launcher_command.py`、完整 `test/` 与 `ruff check .`。

## 2026-09-01 普通 apply 废弃确认

- `services/loader.py::apply_loader()` 的旧单数字 overlay 管线已经废弃，不再作为正式加载方式，也不再扩展新功能。它暂时只为旧 CLI、历史脚本和已有恢复流程保留兼容性；后续不得让 VFS 或 Physical 回退复用该管线。
- 正式加载方式只有两种：VFS 使用虚拟映射启动；Physical 复用同一个 `services/vfs_loader.py::build_vfs_package()` 构建结果，再以事务方式实体写入。两者必须继续共用扫描、排序、目标解析、缺失目标策略、DMM-like 分包及 PAPGT 顺序，禁止维护两套业务规则。
- 游戏更新后，传统 JSON 与 `.cdmod file-replacement` 声明的目标若已不在当前 PAMT，统一由 `services/missing_target_policy.py` 过滤为 warning；其他错误仍为致命错误。VFS、Physical 和废弃兼容 apply 使用同一个策略，显式严格模式仍可阻止加载。
- 旧普通加载的菜单和调用结果必须保留 `[DEPRECATED]` / “已废弃”提示，引导用户使用 `cdloader-VFS`，无法使用 native VFS runtime 时改用 `cdloader-Physical`。不得重新把旧 apply 描述为推荐入口。
- 本次 2.0.02 Kliff 女性中文配音更新验证：旧包有 6 个 WEM 目标被当前 `0035` 删除；默认策略正确跳过后，VFS BuildOnly 成功生成 `nppvoice`、`nppgen`、`nppsa`，完成二阶段缓存复核，热构建可正常命中缓存。用户反馈目前未发现问题。
- 本次代码回归基线：完整 `test/` 为 `536 passed`，`ruff check .` 通过。该结果是构建/回归与用户当前使用反馈记录，不应扩大表述为所有模组、所有机器均已完成实机验证。

## 项目定位

- 当前目录：`T:\python_pro\cdmm`
- 这是独立命令行加载器，不是完整 GUI 管理器 `T:\python_pro\CrimsonDesert-UltimateModsManager`。
- 目标游戏通常位于：`G:\SteamLibrary\steamapps\common\Crimson Desert`
- 模组目录通常位于：`G:\SteamLibrary\steamapps\common\Crimson Desert\mods`
- 当前独立版已经验证：传统 JSON byte patch、`files/NNNN/...`、根部 `NNNN/...`、`files/gamedata/...`、根部游戏路径 loose files、DDS PATHC 更新、standalone `0.paz + 0.pamt`、meta 统一重建、以及 `iteminfo.pabgb` 的一类 Format 3 嵌套字段窄支持可以正常组合加载。
- 2026-07 起，`Format 3` 不再只是一层临时桥接；当前已经拆成“解析层 + 能力声明层 + 运行时分发层 + table writer”结构，后续扩展必须沿这个结构继续推进，不要把新逻辑重新塞回单一大函数里。

## 本机运行约定

- PowerShell 7：
  `C:\Program Files\PowerShell\7\pwsh.exe`
- 优先使用项目虚拟环境 Python：
  `T:\python_pro\cdmm\.venv\Scripts\python.exe`
- 备用 Python：
  `E:\python\UV\uvpython\cpython-3.10.18-windows-x86_64-none\python.exe`
- 用户入口通常是：
  `T:\python_pro\cdmm\run_cdmm.bat`
- `run_cdmm.bat` 只做启动包装，实际逻辑走 `run_cdmm.ps1`，避免 `.bat` 中文编码问题。
- 当前已支持打包为单体控制台 exe：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuhVIP/CD_Mods_Loader_VFS](https://github.com/liuhVIP/CD_Mods_Loader_VFS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
