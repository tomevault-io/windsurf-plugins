---
trigger: always_on
description: 本文档作为 AI Agent 在本工作区（photools 代码库根目录）的**唯一核心持久化记忆、技术规范与架构约束文件（Single Source of Truth）**。后续所有新增功能、架构演进、代码重构或测试均必须严格遵守本文档。
---

# 📸 photools 项目持久化记忆与核心架构规约 (GEMINI.md)

本文档作为 AI Agent 在本工作区（photools 代码库根目录）的**唯一核心持久化记忆、技术规范与架构约束文件（Single Source of Truth）**。后续所有新增功能、架构演进、代码重构或测试均必须严格遵守本文档。

---

## 1. 核心定位与用户画像

- **用户画像**：Golang 程序员与尼康单反/微单摄影师；
- **系统核心定位**：专注于相机 RAW/JPG 照片导入电脑后的 **GPS 轨迹匹配、智能时间插值推算、离线高精逆地理编码中文地名清洗与规范化拍摄日期归档**；
- **语言规约**：所有交互、思维推理、终端输出、用户日志与异常提示**统一强制使用简体中文**；
- **系统边界与非目标**：
  - 本系统不是完整 DAM，不管理全部摄影资产生命周期；
  - 本系统不是后期调色软件，不承担 Lightroom/Capture One 等工具的职责；
  - 默认不包含云同步、评分筛选、非 ExifTool 独立 Exif 写入等脱离核心定位的功能。

---

## 2. 摄影资产模型与元数据写入规则

- **资产主文件优先模型 (Primary Asset Model)**：
  - 同 basename 的配套文件是一个独立的拍摄单元；
  - 核心操作遵循 **“主文件优先（PrimaryPath）”** 抽象：若存在 `RAW`，以 `RAW` 为主决策源并自动同步至伴随的 `JPG` 与 `XMP`；若仅有单 `JPG` 或单 `RAW`，该文件本身即作为完整主文件，平等享受 GPX 匹配、GPS 插值、逆地理与归档能力；
  - 若存在同 basename 的伴随文件（`XMP`、`ACR`、`WAV` 等），作为伴随文件整体同步维护并一同归档。
- **时间与轨迹匹配**：
  - 默认信任照片内的 `DateTimeOriginal` 与 `OffsetTimeOriginal`；
  - 默认 `geosync=0`，但必须支持显式时间偏移补偿；
  - 多个 `GPX` 文件必须逐个传参，禁止拼接成无效字符串。
- **写入顺序与二次校验**：
  - 双格式时必须先写主文件 `RAW`，再同步至 `JPG` 和 `XMP`；单文件时直接写入主文件并同步 `XMP`；
  - 写入成功判定**绝不能仅依赖 ExifTool 退出码**，必须包含二次读取校验，至少确认主文件上存在有效 `GPSPosition`。

---

## 3. 四大能力插件与分阶段屏障调度器 (Capability Architecture)

系统采用接口解耦的插件化架构，严禁将 GPX 匹配、推算、逆地理、归档杂糅在同一个单体函数中。

### 3.1 四大核心能力插件
1. **`gpx_matching` (Priority 10 · 阶段 1)**:
   - 依赖 ExifTool 进行 GPX 轨迹时间轴精准匹配；
   - 写入 RAW 经纬度并二次校验，无污染同步至伴随 JPG/XMP。
2. **`gps_interpolate` (Priority 15 · 阶段 2)** ✨:
   - **极速时间分桶与二分查找索引 (`AnchorIndex`)**：
     - 内存按拍摄日期 `YYYY-MM-DD` 分桶维护时间有序锚点切片，通过 `sort.Search` 二分查找在 $O(\log K)$ 纳秒级（~200ns）定位前后最邻近机位；
     - 批次首轮统一提取拍摄时间与坐标，彻底杜绝在检索循环中重复启动外部 ExifTool 子进程；
     - 双向锚点采用球面大圆时间权重线性插值，单向锚点采用同机位近邻继承，推算后动态合入索引供同批次后序照片自愈继承；
     - 默认推算窗口 15m（支持 `--interpolate-window` 自定义），开启 `--allow-no-gps` 时超出窗口安全跳过而不阻断流水线。
3. **`reverse_geocode` (Priority 20 · 阶段 3)**:
   - 基于 3D KD-Tree 离线高精空间索引；
   - 写入国家、省份、城市、区县、风景区 POI 中文元数据（IPTC/XMP），支持 IPTC Extension 完整结构化位置模型 (`LocationCreated` / `LocationShown`) 与 Lightroom 分层关键词标签树 (`XMP-lr:HierarchicalSubject`)；
   - **绝对幂等性与标签清洗铁律 (Idempotency & Clean-Merge)** ✨：
       - 严禁使用破坏幂等性的无脑追加（`+=`），必须采用 ExifTool 官方标准的“单次指令内原子置空并全量重写”模式（先置空
         `-XMP-lr:HierarchicalSubject=` 再重新赋值）；
       - 必须通过 `domain.CleanAndMergeLocationTags` 精准剥离旧地理层级树（`中国|` / `China|`），同时 **100%
         完整保护摄影师自定义业务标签与题材关键词**（如 `题材|人像|户外`、`扫街` 等），并使用 Set 机制保序去重；
       - 坐标缺失区县/POI 时必须显式置空相关标签，彻底杜绝跨机位纠偏时产生的“历史地名缝合怪”；
       - 写入后必须通过 `VerifyLocationTags` 进行二次回读断言，确认标签真实落盘且绝无重复项。
   - **离线地理数据包终端安装规约**：为保证地理数据库来源准确性、网络完整性与数据源校验，离线数据包严禁在 GUI 客户端内隐式下载安装，必须由使用者在终端（CLI）中显式执行 `photools geodata install <target>` 安装。GUI 仅负责状态探测、命令复制与 3D KD-Tree 坐标反查测试。
4. **`date_archive` (Priority 100 · 阶段 4)**:
   - 依据原始拍摄日期规范化重命名；
   - 原子安全归档至 `Processed/YYYY/MMDD/`（破坏性移动插件，必须具备最低优先级），完整保留 `.nef.xmp` / `.jpg.xmp` 复合侧车后缀。

### 3.2 阶段流转、调度器与性能契约
- **多阶段平滑交接与就绪判定**：
  - Dry-Run 预检判定：前序插件良性跳过（如已有 GPS 跳过插值）绝不阻断后序插件（如逆地理写入），只要整条链路无阻塞 Warning 且包含有效执行阶段，资产即判定为 `ReadyCount` 就绪；
  - 阶段 1 未命中 GPX 轨迹时，若流水线配置了阶段 2 插值插件，编排器平滑交接，严禁提前硬熔断。
- **全生命周期 `sync.Once` 与零冗余进程机制**：
  - 各插件 `Init()` 强制采用 `sync.Once` 缓存环境自检与 ExifTool 探针结果，预检与执行阶段毫秒级直通；
  - 离线地理数据包与 KD-Tree 建树全局单例装载一次，杜绝重复 I/O 与建树开销；
  - **ExifTool Stay-Open 常驻守护进程池 (`DefaultRunner`)**：生产环境默认启用常驻进程池（`StayOpenPool`），通过 `exiftool -stay_open True -@ -` 消除重复 `fork/exec` 子进程开销，单次读取开销从 ~30ms 骤降至 1~2ms，并具备进程崩溃自愈与优雅退出能力。
- **软降级容错 (`--allow-no-gps`)**：彻底无 GPS 照片在逆地理阶段良性跳过（不产生阻塞 Issue），安全进入阶段 4 按拍摄日期规范归档；
- **并发与待处理报告**：按 basename 资产组并发处理，处理中断或待补资产自动生成详尽的 Markdown 原因清单（默认 `~/.logs/photools/inbox_pending_report_latest.md`）；
- **全量实时中文日志流落盘 (Real-Time Log Streaming)**：
  - 流水线执行期间产生的全部中文实时事件流（含插件自检、阶段流转、每张照片推算/打标/归档进度与异常）强制实时流式落盘至用户主目录全局日志中心 `~/.logs/photools/photools_latest.log` 与带时间戳的 `~/.logs/photools/photools_YYYYMMDD_HHMMSS.log`，彻底杜绝在照片工作区目录产生 `Logs/` 垃圾文件；
  - 采用毫秒级带时区时间戳格式（`[HH:MM:SS.mmm] [LEVEL] [阶段] 消息`），并在任务结算时自动追加 Execution Summary 与异常清单；
- **扁平原地模式与就地保存 (Flat Mode & In-Place)**：支持忽略传统 `Inbox/` -> `Processed/YYYY/MMDD/` 分层，直接指定源目录就地扫描、就地逆地理/打标签、并原地规范化重命名，且同一目录下重命名严格免自冲突；
- **智能元数据分层模型与四档策略 (`SidecarPolicy` · `--sidecar-policy`)** ✨：
  1. `smart`（默认/推荐 · 智能分层模式）：
     - **第二层修正事实 (GPS/时间修正)**：写入 RAW EXIF 头部 + 伴随 JPG 内嵌 + 同步 `.xmp` 侧车（附带 Photools 溯源指纹）；
     - **第三层派生信息 (中文地名/分层标签)**：RAW 严格只读仅写 `.nef.xmp` 侧车，JPG 交付格式直接内嵌写入；
     - 黄金平衡点：RAW 永久拥有标准 GPS 事实，地名由 XMP 承载，JPG 跨设备即开即看。
  2. `sidecar_only`（纯 XMP 侧车模式）：RAW 与 JPG 均不触碰原图，所有修正与派生元数据严格输出为独立 `{file}.xmp` 侧车；
  3. `embed_and_sidecar`（双写同步模式）：直接修改原图内嵌 EXIF，同时维护配套的 `.xmp` 侧车文件；
  4. `embed_only`（纯原图内嵌模式）：直接修改原图内嵌 EXIF，不产生或更新任何 `.xmp` 侧车文件。
- **伴随文件扩展名白名单 (`CompanionExtensions` · `--companion-exts`)** ✨：支持针对 RAW/JPG 主文件配套的伴随文件（如 `wav` 录音、`acr` 调色、`exf`、`xmp` 等）进行拓扑发现、原子同步流转与规范重命名归档；
- **统一设置抽象与会话覆盖 (Session & Plugin Settings)**：
  - 核心配置由 `internal/config/schema.go` 统一定义，全局参数与插件专属参数清晰正交解耦；
  - TUI 中按 `[s]` 调出全局设置，光标选中插件按 `[o]` 调出插件专属设置，输入路径支持 `[Tab]` 智能补全与多候选轮转；
  - 运行时基于 `SessionConfig` 动态覆盖，支持 `Enter` 会话生效与 `Ctrl+S` 持久化写入 `~/.config/photools/plugins.json`。
- **插件自描述与自配置契约 (Self-Describing & Configurable Capability)**：
  - 插件接口强制实现 `SupportedOptions() []OptionSpec` 与 `Configure(opts map[string]any) error`；
  - 插件自身完全内聚掌管自己的可配置选项、默认值、说明与预设候选值（Choices）；
  - TUI 界面与 Pipeline 构建器基于元数据纯动态驱动，彻底杜绝针对具体插件的外部硬编码 `switch-case`。

---

## 4. 核心工具包单元测试 100% 闭环规范 (AI 必遵)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentchyu/photools](https://github.com/vincentchyu/photools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
