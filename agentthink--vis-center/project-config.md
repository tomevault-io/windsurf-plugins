---
trigger: always_on
description: > 本文件是 AI 助手在操作本项目前的**强制前置阅读**。开工前先读本文件，明确角色、约定与红线。
---

# AGENTS.md — CommandCenter 项目指南

> 本文件是 AI 助手在操作本项目前的**强制前置阅读**。开工前先读本文件，明确角色、约定与红线。
> 优先级：本文档 > 项目已有代码风格 > 通用最佳实践。

## 项目角色

你是本项目（Windows 窗体 C#/.NET Framework 应用）的**资深开发/维护工程师**，负责按用户需求改代码、修 bug、沉淀约定。改动必须**可编译、可运行、风格统一**，并在关键改动后更新 `CHANGELOG.md`。

## 技术栈

- .NET Framework **4.7.2** WinForms（非 .NET Core/.NET 5+，勿引入其语法/API），C# 语言版本 `LangVersion=7.3`
- 通讯：**NModbus 3.0.83**（汇川 PLC Modbus TCP）；相机走基恩士 TCP 无协议通信（自写 TcpClient）
- 序列化：**Newtonsoft.Json**（配置/型号）
- **依赖策略（重要）**：第三方库拷在 `CommandCenter/libs/` 目录由 csproj `<Reference HintPath>` 直接引用，**不依赖 NuGet restore**，离线可编译。新增第三方库请同样"拷 dll 进 libs 再引用"。

## 铁律（违反即返工）

1. **文件编码 UTF-8**。禁止 `Add-Content`/`Out-File` 默认编码写中文（会成 GBK）。写文件用 write 工具；新增中文文件后自查：`[IO.File]::ReadAllText(path, UTF8).Contains("预期中文")` 要能命中。
2. **不提交运行时数据与机密**：`Config/*.json`（appconfig 等运行时生成）、`Logs/`、`bin/`、`obj/` 一律 gitignore，绝不入库。
3. **改动后必须构建验证**（命令见下），禁止提交编译不过的代码。
4. **不主动 commit/push**，除非用户明确要求；提交前先 `git status` + `git diff` 确认只包含预期改动。
5. **代码注释要详细，让小白能看懂**：关键方法/流程/边界/配置依赖写清"做什么 + 为什么 + 怎么改"，杜绝 `i++ // 自增` 式废话。参考本仓库 `Services/ProductionCoordinator.cs` 与 `Models/AppConfig.cs` 头部注释风格。

## 代码约定

- 类/方法/属性 PascalCase；私有字段 `_camelCase`；接口前缀 `I`。
- 控件命名匈牙利前缀：`lbl`/`btn`/`txt`/`nud`/`cmb`/`pnl`/`grid`。
- **界面文件头注释必须带 ASCII 布局图**（`Views/*.cs`、`Dialogs/*.cs` 类 XML 注释里，用 `┌─┐│└┘` 画），框内标注控件名与关键交互点，必须与实际布局一致。AI 无法看图，全靠这张文本图。
- **串口/枚举配置值的存储约定**：停止位存字符串 `"1"`/`"15"`/`"2"`；校验位存标准枚举名 `None/Odd/Even/Mark/Space`。读写两端大小写兼容。参考 `Services/ScannerService.StopBitsFromString` / `ParityFromName`。
- **OK/NG 现场习惯（必须）**：**OK = 绿色、NG = 红色**（矩形框 + 文字同色），颜色名可在 `appconfig.json` 的 `display.okColorName/ngColorName` 里配。
- **管理员登录（V1.9.0）**：点"系统设置"每次都要登录管理员账号（`Security.AdminEnabled=true` 时，MainForm.OpenSettings 校验），**密码只存 SHA-256 哈希、不存明文**（`Utils/SecurityUtil.HashPassword`）。账号维护全部在**登录对话框**里完成：登录面板校验，改密码面板（验证原密码 → 新密码两次一致且 ≥6 位 → 保存写盘）；**系统设置窗体不放管理员区**，保持纯业务配置。**"记住密码"用 Windows DPAPI 加密存 `%LOCALAPPDATA%\CommandCenter\`**（绑定当前 Windows 用户，拷走无效；`SecurityUtil.Save/Load/ClearRememberedLogin(bool isDev, …)`，`isDev=false` 存管理员文件 `remembered_login.dat`、`isDev=true` 存开发者文件 `remembered_login_dev.dat`），**管理员/开发者记录互斥**：登录任一账号成功会把另一角色的记住文件一并清除（`LoginForm.BtnLogin_Click`），改密码也清开发者记录，防止跨角色回填残留。绝不在配置文件里存可回填的明文密码。新增安全类配置走 `SecurityConfig`，勿引入明文密码字段。
- **开发者账号 + 功能测试（V1.12.0）**：除管理员外还有开发者账号（`SecurityConfig.DevEnabled/DevUser/DevPasswordHash`，默认 `dev`/`dev123`）。MainForm.OpenSettings 登录后按 `login.Role` 分流：`Admin` → 系统设置 SettingsForm，`Developer` → 功能测试 DevTestForm。**功能测试窗体约定（必须遵守）**：① 只做通讯手动验证、不产生任何配置改动；② **复用主窗体传入的 `_plc`/`_cameras`/`_scanners` 实例、绝不新建 TcpClient/串口/连接**（内部 EnsureConnected 惰性建连缓存复用；扫码枪为设备主动推码，只订阅 `SerialNumberScanned` 事件收码、不重复 Open，可调 `SendTrigger()` 手动重发触发指令），关闭时不 Dispose 这些服务；③ 所有网络 IO 走后台线程 + SafeInvoke 回 UI（红线同 UI 禁 IO）；④ 开发者密码不支持界面修改（改密码面板仅服务管理员）。新增测试入口若需连设备，先找 MainForm 是否已有该服务实例，有了就传引用复用。**T2 取图存图（V1.12.24）**：`btnTriggerRead`（"触发+判定T2（取图存图）"）触发成功后复用主窗体传入的 `_imageStore` 与相机配置（`FtpUploadDir`）扫该相机 FTP 目录取最新 jpeg+iv4p → `picTestShot` 闪图 → `SaveImageFilePair` 存进主窗体配置存图目录（**点位固定 1**、判定 OK/NG、打开窗体时 SN 快照），结果/路径进日志；点 T1 只验证触发链路不取图存图。
- **扫码枪触发指令（V1.12.1，基恩士 SR 无协议）**：Tcp 模式下扫码枪**不是连上就回数据**，上位机须先发一条打开激光/开始读取的指令（`ScanConfig.TriggerCommand`，默认 `LON`）才读码。`ScannerTcpService.TryConnect` 每次连接/重连成功后**自动发送一次**（发送时自动补 `\r\n` 帧结束符），配置留空则不发送。`IScanner.SendTrigger()` 供界面手动重发。串口扫码枪上电即读码、无需触发（串口实现 SendTrigger 为空操作）。改动扫码枪通讯必须同步 `docs/CommandCenter.md` 的"扫码枪"章节与默认配置。
- **UI 线程禁做网络 IO（V1.0.1 血泪）**：轮询/连接/读写 PLC 与相机一律放后台线程（`System.Threading.Timer`），TCP 连接必须 `BeginConnect + WaitOne` 强制超时。禁止在 UI 线程同步 `TcpClient.Connect` 或 `ReadHoldingRegisters`——对不可达 IP 会冻结整个界面（表现为"点按钮半天才响应"）。
- **显示窗口矩阵用 TableLayoutPanel 百分比等分**：窗口数量由 `display.rows/columns` 配置，所有窗口尺寸由容器等分自动保持一致，禁止写死像素布局。
- **显示窗口矩阵统一模型（V2.12.1，取代并合并 V2.12.0"自适应"）**：窗口总数**恒** = 各相机按型号
  点位表 `ProgramsFor(型号)` 条目和（`DisplayConfig.WindowCountFor` / `ResolveLayout` 统一计算，
  列=min(7,总数)、行=ceil(总数/列)、点数≤7 单行铺满），`AutoFitCameraStarts` 返回各相机窗口起始序号
  （"前上相机后下相机"分组），主窗体 BuildWindowGrid / 设置页预览 / 协调器 / WindowPointForm **共用同一套
  计算，禁止各层再各写一套**。**勾选"自适应"只决定行列是否自动算**：`AutoFit=true` 时行/列输入框置灰
  （行列自动铺排）；不勾时手填行/列只当"排列宽度/期望行数"，放不下**自动补行**，窗口总数仍=点位和，
  两种模式所见完全一致。**存图点位统一 = 相机点位号（StationNo）**（上下相机各自从 1 起会重复，靠
  ImageStore 归档子目录 **`{相机}` 层隔开**——`SubDirs` 默认含 `{相机}`，旧配置加载自动补，绝不拿
   WindowStationMap/windowIndex 当存图点位）；手动点位编辑（编辑点位/交换位置/恢复默认）在 WindowPointForm
  里两种模式**都可编辑（V2.13 恢复）**：结果按型号分表存 `DisplayConfig.WindowPointMaps`
  （`WindowPointItem{CameraIndex,StationNo}`，默认=前上相机后下相机铺排、不编辑行为与旧版零差异；
  `ResolveWindowPointMap` 按型号查表、长度≠窗口总数时回退默认；`ConfigStore.EnsureWindowPointMaps`
  加载/保存自动对齐）。**编辑规则**：编辑点位候选=当前型号各相机点位表已有点位、自动排除已被其他窗口
  占用的组合（同"相机+点位"只对应一个窗口，`ProductionCoordinator.TryResolveActiveWindow` 据此反查
  唯一窗口）；**交换位置任意两窗口可互换（含跨相机，V2.13.1 放开）**——窗口↔点位映射本来就是
  "归属相机+点位号"二元组（`WindowPointItem{CameraIndex,StationNo}`），上相机·点位3 与下相机·点位3
  是不同点位，反查键=(相机,点位) 在两窗口互换后仍唯一（值集合不变），故跨相机交换不会让反查混乱；
  交换只改"窗口↔点位"对应（写回 WindowPointMaps），**不改各相机点位表/程序映射 ModelStationPrograms**；
  恢复默认=重置该型号出厂铺排+全部窗口重新启用。设置页勾选自适应仅置灰行/列输入框并弹 ToolTip
  明示"自适应只影响行列形状、不影响点位编辑"。
   **默认型号（V2.12.3）**：`AppConfig.ProductModel` 默认 **"U171"**（非空），无配置文件首次启动也
   按该型号点位表铺出对应窗口（U171=上18+下4=22 窗），不会因型号空串把窗口塌成 1 个（此前 `Load()`
   无文件分支直接 new AppConfig() 连相机列表也是空的，窗口=0→兜底 1 个的回归根因）；`ConfigStore.Load`
   把"空段兜底+数组对齐"抽成 `ApplyDefaults`，有/无配置文件统一走。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentthink/vis-center](https://github.com/agentthink/vis-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
