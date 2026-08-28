---
trigger: always_on
description: [English](AGENTS.en.md) | **简体中文**
---

# Creeper — Agent 指南

[English](AGENTS.en.md) | **简体中文**

Windows 上的 C++17 隐写套件：把文件加密后藏进 PNG / MP3 / WAV。权威规格在 `docs/PROMPT_ENCODER.md` / `docs/PROMPT_TESTER.md`（编码/测试任务书），结果与已知缺陷在 `docs/TEST_REPORT.md`，产品文档在 `docs/使用说明书.md` / `docs/技术报告.md`。

## 构建与测试

- 构建：`src/build.bat`（ASCII-only）一次构建三个 exe 到 `res/`（发布包），需 w64devkit 的 `g++` 在 PATH。GUI exe 用 `-mwindows`，CLI 不用。
- 测试前必须先构建；临时文件放 `tests/tmp/`：
  - `python tests/test_crypto.py`（加密往返 + 健壮性）
  - `python tests/test_png.py` / `python tests/test_mp3.py` / `python tests/test_wav.py`（隐写往返）
  - `python tests/test_cross.py`（C++ ↔ Python 信封互解，关键）
  - `powershell -ExecutionPolicy Bypass -File tests\test_gui.ps1`（GUI 冒烟，会弹窗 3 秒并杀进程）
  - `powershell -ExecutionPolicy Bypass -File tests\test_gui_about.ps1`（关于弹窗真实点击链路，副本 exe）
  - `powershell -ExecutionPolicy Bypass -File tests\test_gui_quality.ps1`（隐藏窗「编码质量」OCR 验证）
  - 一键全量：`tests\run_all.bat`（9 套件串行，任一失败退出非 0）
- 测试依赖 Python 3.14 + Pillow + numpy + cryptography；`tests/tester.py` 的 `ROOT` 已相对化（仓库可移动）且 `summary()` 自动归档结果到 `tests/results/`；`tests/envelope.py` 为信封参考实现本地副本（test_cross 优先本地）；测试宿主在 `assets/`（`img.png` / `msc.mp3` / `test.wav`，`tests/gen_hosts.py` 统一生成，缺失时各自自动生成：W0 调 `gen_wav.py`）。
- 已知问题别当新 bug 报：OBS-2 已修复（`crypto.cpp` DEFLATE 升级 LZ77 + 三路块编码：固定/动态哈夫曼/存储块取最短，与 Python zlib 差距收敛，极端重复文本 1.9× 系 LZ77 匹配策略差异非缺陷）。历史缺陷 BUG-1（ID3v2 标签大小 +10 字节）已修复；OBS-1（`has` 对空文件返回 0）已随无魔数改造正式化（空文件 = 无载荷 = 0，合理语义）。改动相关代码须同步更新 TEST_REPORT.md。
- GUI 窗口为固定尺寸（不可缩放/最大化），尺寸按系统 DPI 缩放（`common_ui.cpp` `ui_run`，逻辑 1000×640 × `GetDpiForSystem()/96` 物理像素，超出工作区退回并居中）；改动布局时保证主界面铺满客户区（`vp-2` + `NoResize|NoMove|NoCollapse`），控件 label 一律放左侧（`Text` + `##` 隐藏 label），勿改回可自由缩放样式。

## 硬约束

- **禁止修改 `third_party/stb/` 和 `third_party/imgui/`，禁止引入新依赖**（只许系统库 + 现有第三方）。
- 项目是 git 仓库（BSD 3-Clause）；`res/`（发布包：3 个 exe + 说明书 md 副本）是交付物（已被 .gitignore 排除，不进版本库），别删别乱动，exe 变更后需同步 res/ 与便携版（便携版 zip 已废弃不打包）；文档交付物 = `docs/` 下的 markdown 源（`使用说明书.md` / `技术报告.md`），不产出 PDF。

## 文件格式（改任何一处必须保持与参考实现字节级兼容）

- **加密信封**（`crypto.h/cpp`）：41 字节头 `CREEPER1` + ver 0x01 + salt(16) + nonce(12) + ct_len 大端(4)，后接 AES-256-GCM 密文（含 16B tag）；PBKDF2-HMAC-SHA256 600,000 次迭代；seal 前先 DEFLATE 压缩（RFC1950 zlib，Python zlib 可解）。**必须与 Python 信封参考实现（`tests/envelope.py` 本地副本）互解**——test_cross 依赖此兼容。信封头 magic 字符串在源码中以 XOR 0x55 字节数组形式存在（`xstr()` 运行时还原），exe 内无 `CREEPER1` 明文。
- **PNG**（`png_steg.cpp`）：只嵌 RGB 三通道各 1 bit（alpha 保持原样）；隐写头**无魔数** = name_len(2B BE) + UTF-8 原始文件名 + env_len(4B BE)（**无 seed 字段**——散布种子由密码派生：`crypto_steg_seed(password, "creeper-seed")` = HMAC-SHA256(password, tag) 前 4B，轻量派生，seed 非机密）；头+信封整体按 seed 散布（xorshift64 + 线性探测，**纯内容无关序列**——曾尝试"纹理掩码自适应嵌入"，但 ±1 修改影响邻居 diff 达 2，任何内容阈值都有模糊带导致掩码翻转、序列无法重放，数学上不可行，已放弃）；**载荷存在性判定 = 密码正确时 GCM 认证通过**（`png_has_payload(path, password)` 完整解析 + 认证，密码错/无载荷均返回 false），误报概率 2^-128。**嵌入用 ±1（LSB matching，非强制 LSB）**：位不匹配时像素 ±1（方向随机，边界单方向）；嵌入后用未承载像素做直方图配对补偿（把修改后直方图拉回原形），避免直方图阶跃特征。**自对抗重嵌**：补偿后算 RGB 汇总直方图 L1，`L1 > 修改像素数×15%` 则按 undo 快照恢复原图重嵌（最多 3 次；undo 记录不可设硬上限，截断会导致恢复不全残留脏像素）；写盘前**重放提取序列逐位自检**（不一致报错）。提取端只读承载位 LSB，不受补偿影响。**填充率上限**：`png_embed(..., fill_limit_pct)` 默认 15%（防统计检测），超限报 `payload too large: exceeds N%...`；CLI 可用 `--cap N` 覆盖（仅 PNG/WAV 生效）。
- **MP3**（`mp3_steg.cpp`）：MPEG 帧头辅助位方案——只改写每帧头 3 个辅助位（private/copyright/original，掩码 0x01/0x08/0x04，3 bit/帧），音频数据区与 ID3 标签区逐字节零改动；位流 = name_len(16bit) | name(UTF-8) | env_len(32bit) | env，**无 magic**，**末尾填充 0 位到 3 的倍数**（否则帧按 3 位消费会截断末尾 1-2 位导致 GCM 认证失败）；**帧头辅助位整体 XOR 密码派生 keystream**（`crypto_steg_seed(password, "creeper-ks")` 派生 seed，`FrameKs` 每帧消费 3 位，帧序 = 扫描序；**只在帧层消费一次**——曾把 ks 放进 BitStream 双重 XOR 导致两端阶段错位、自检失败）——密码错则解析出垃圾 → GCM 认证失败，与"无载荷"不可区分；帧扫描：同步 0xFFE、Layer III（layer==1）、version≠1、bri 1..14、sri ≤2，帧长 = nslot×bitrate×1000/samplerate+padding（nslot=144/72），坏帧逐字节找同步；embed 写完**回读逐位自检**。旧 GEOB 方案（filename=`creeper`）已被取代，勿回改。
- **WAV**（`wav_steg.cpp`）：PCM（8/16-bit）无损 LSB——只改 data 区每样本最低 **depth 位**（depth=1：±1，LSB matching，边界单方向；depth=2/3：低 2/3 bit 重写，差 ≤3/≤7，仅 16-bit 宿主），RIFF/fmt 区与样本高位零改动；隐写头 = **depth(1B)** + name_len(2B BE) + name + env_len(4B BE)，无魔数无 seed 字段，seed = `crypto_steg_seed(password, "creeper-uaz")`（注意：XOR 数组还原值是 creeper-uaz，注释/文档勿再写 creeper-wav，数组不可改否则旧文件全废）；头+信封整体按 seed 散布（xorshift64 + 线性探测，纯内容无关序列）；存在性 = GCM 认证（误报 2^-128）；**解析新格式优先（首字节=1/2 合法），失败回退 v1.0 旧格式（无 depth 字段）**——旧文件新代码可解，新文件旧代码视为无载荷；**填充率上限默认 15%**（`--cap` 覆盖）；`--depth 2|3`（CLI）/「位深」下拉（GUI audio 隐藏窗）→ 容量 ×2/×3；写盘前重放逐位自检。容量 = 样本数×depth/8 字节（44.1kHz stereo 60s depth=1 ≈ 661KB，15% ≈ 99KB/分钟；depth=2/3 ×2/×3）。
- CLI：`creeper_cli seal|open|embed|extract|has|split|unsplit`，参数 UTF-8；
  - `split <payload> <password> <outdir> <host...> [--cap N] [--depth N]`：大文件拆分到多宿主（先整体 seal 再切分密文）；输出 `宿主_已转换.ext`
  - `unsplit <outdir> <password> <host...>`：多宿主合并还原（**宿主顺序无关**，按块内编号拼接）
  - 分片协议（`split_steg.cpp`）：每宿主隐写流 = 标准头（name_len+name+env_len，WAV 前多 1B depth）+ env = `magic(4B) + index(2B BE) + count(2B BE) + chunk_len(4B BE) + chunk`；magic = `crypto_steg_seed(password, "creeper-split")` 大端（密码派生、无明文特征，静态扫描无可命中特征）；单块过不了 GCM 认证（"无载荷"），必须收齐所有块按 index 拼接后认证还原——单宿主截获无意义；容量分配 = 每宿主 `floor(容量×填充率) − 头开销` 顺序填装，MP3 按 100%（辅助位方案无填充率概念）`has <host> <password>` 输出 `1`/`0`（密码错或无载荷均为 0）；`embed ... [--cap N]`（N=0..100，默认 15，仅 PNG/WAV 生效）`[--depth 1|2|3]`（仅 WAV，2/3 = 每样本 2/3 bit 容量 ×2/×3，仅 16-bit 宿主；非 WAV 宿主带 --depth 报错）；一切失败 → stderr 英文报错 + 非 0 退出。源码主文件名为 `cli_main.cpp`（避免 exe 内出现 `creeper` 字样的源文件名）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamzeneks/Creeper](https://github.com/iamzeneks/Creeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
