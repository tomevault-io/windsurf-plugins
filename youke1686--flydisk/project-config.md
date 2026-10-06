---
trigger: always_on
description: 本文件供 AI 编码代理（Agent）阅读，用于快速了解本仓库的结构、技术背景与工作规则。在执行任何任务前，请先完整阅读本文件并严格遵守其中的规则。
---

# AGENTS.md

本文件供 AI 编码代理（Agent）阅读，用于快速了解本仓库的结构、技术背景与工作规则。在执行任何任务前，请先完整阅读本文件并严格遵守其中的规则。

***

## 一、项目概述

RamCacheDisk 是一个 **本地硬盘加速软件**：用内存（L1）+ SSD（L2）两级缓存加速机械盘上的热数据，
对上层提供一块"经过缓存的盘"，而不是把整个数据搬进内存的传统 RAMDisk。

### 当前状态（2026-09-27）：项目已转向阶段二

| | 阶段一（已归档） | 阶段二（进行中） |
| --- | --- | --- |
| 对外形态 | WinFsp 挂载的虚拟盘（缓存代理型文件系统） | 自研 iSCSI target 提供的**块设备**（本地固定盘） |
| 上层看到 | 可被设备层探针一眼认出 | 真实设备栈给出的盘（`bus=iSCSI`） |
| 代码位置 | `废\阶段一-WinFsp文件系统层\` | `RamCacheDisk\`（**单项目**：界面 + `Engine\` 引擎层） |

- **形态已实测通过（2026-09-27）**：用现成 demo 把 `J:`（整块机械盘）脱机后 export，原 NTFS 原样挂回
  （`fsutil fsinfo volumeinfo` 与真盘 **24 项逐行一致**，阶段一缺的那一串全部回来），**原神关联 + 启动成功**。
  零搬运、原数据原地跑。详见 `阶段二-iSCSI块设备形态.md` §6。
- **为什么弃用 WinFsp**：阶段一撞上**驱动层硬边界**——`src/sys/devctl.c` 只把"设备类型带 `0x8000` 位 +
  `METHOD_BUFFERED`"的请求转发到用户态，`src/sys/volinfo.c` 的卷特性位只从 `VolumeParams` 的 10 个位拼出。
  实测后果：原神启动器"关联游戏"按钮灰掉、直启 `YuanShen.exe` 卡死。完整证据链与源码级根因见
  `阶段二-iSCSI块设备形态.md` §1（该文同时记录了官方 `NTFS-Compatibility` 口径与实测的偏差）。
- **复用什么**：两级缓存的"机制"全留（非托管内存池 / LRU 时间桶 / L2 淘汰结构 / 容器与账本 / 统计 / 日志），
  **键要换**（路径 + 文件内块号 → 全局块号）；外围基本不动；宿主与文件系统层重写。§4 有逐项清单。
- **当前形态（2026-09-27，M2 代码已落地）**：**单项目**——`RamCacheDisk`（WinForms，`requireAdministrator`）
  里既有界面（`Form1`/`Settings`/`Inspector`/`RemoteForm`/`SelectDiskForm`）也有引擎（`Engine/`：`CacheService`、
  `SsdCacheService`、`LogService`、`TargetService`、`CachedPhysicalDisk`、`PhysicalDiskHandle`、`RemotePeer`、
  `RemoteProtocol`、`IBlockSource`）；`RamCacheDisk.Core` 与
  `RamCacheDisk.Service` 两个项目已整体退役到 `废\阶段一-IPC与SCM\`。**编译与实测由用户执行**（见第五章）。

### 阶段一形态（已归档，仅作背景）

核心思路：不是传统的 RAMDisk（把数据全部放进内存），而是一种 **"缓存代理型文件系统"**：

- 软件挂载一个虚拟磁盘，其目录结构与被加速的机械硬盘完全一致；

- 虚拟盘本身不存储任何实际数据，仅展示原硬盘中的文件与目录；

- 访问虚拟盘中的文件时，先查询内存缓存：

  - **命中** → 直接用内存数据响应；

  - **未命中** → 从原机械硬盘读取数据响应，并将热点块写入内存缓存；

- 原始数据始终以机械硬盘为准，断电安全，内存仅保留热点文件副本。

### 进程架构（阶段二：**单进程**）

**一个进程**：`RamCacheDisk.exe`（WinForms，管理员运行），界面与引擎同程序集。启动按钮一次性完成
"校验 → 整盘脱机 → 独占打开 → 起 iSCSI target"，停止按钮反向收尾（顺序见下）。

- **界面层（`RamCacheDisk`）**：`Form1`（启停 + 状态栏 + 菜单，**用户可见的过程信息都由它自己说**——
  主界面日志框、状态栏、弹窗）、`Settings`（iSCSI 监听端口 / 两级缓存参数）、`RemoteForm`（「远程」对话框）、
  `SelectDiskForm`（「选择硬盘」，本地/远程共用）、`Inspector`（每秒直读 `TargetService.CreateSnapshot()`）；
- **引擎层（`RamCacheDisk.Engine`）**：`TargetService`（生命周期与启动校验）、`CachedPhysicalDisk`
  （`DiskAccessLibrary.Disk` 的 4 个成员实现：缓存读 / 写透传 / 写命中刷新 + 块级失效）、`CacheService`（L1 门面）、
  `SsdCacheService`（L2）、`PhysicalDiskHandle`（脱机 / 独占 / 原始扇区 P/Invoke）、`RemotePeer` + `RemoteProtocol`
  （远程形态）、`IBlockSource`（块源抽象）、`LogService`（**只落盘**的诊断账本）；
- **模型层（`RamCacheDisk.Models`）**：`DiskConfig`（配置）、`CacheStats`（统计快照）、`ConfigService`、`ServiceConstants`。

**为什么不再拆双端**：阶段一拆"UI 进程 + LocalSystem 服务"的唯一硬理由是"让挂载的盘符对所有会话全局可见"；
而 iSCSI 盘的盘符是 `disk.sys` 枚举出的**系统级磁盘**（与插一块 USB 硬盘无异），天生对所有会话可见。
⇒ 该约束随形态一起消失，IPC 与 SCM 服务控制一并退役（见 `废\阶段一-IPC与SCM\说明.md`，M4 若要服务化再取回）。

**停止顺序（不可颠倒）**：**先确认没有发起程序还连着——有连接就拒绝停止**（2026-09-30 起）→ 停 target →
关闭 LUN 后端 → 等在途命令跑完（上限 2 秒）→ 冻结 L2 → 落盘账本 → 关句柄（**盘保持脱机**）。
拒绝的判据是"本机监听端口上还有 ESTABLISHED 的 TCP 连接"（库的会话表是 internal，见 `TargetService.GetActiveConnectionCount`）；
其中"冻结"是为了让账本成为一个不再变化的一致快照；
"不联机"是 2026-09-27 定下的口径，理由与两条兜底恢复路径见 `阶段二-iSCSI块设备形态.md` §3.6。

**关机 / 重启 / 注销**（界面层 `CloseReason.WindowsShutDown` 分支，2026-10-03 起）：用户走不到"先点停止加速、
再关窗口"那条路，故 `Form1.OnFormClosing` 会补一次 `TargetService.TryStopForShutdown()`。它与 `Stop()` 的
**唯一区别是不做"有连接就拒绝停止"的守卫**：关机时 L2 账本**必须落盘**（那是 L2 的全部价值），而落盘要求先停
target 让缓存表静止——所以即便发起程序登不出去（盘仍被占用），也照样**中止 iSCSI 服务、掐断会话**，然后落账本。
代价（已知且接受）：那块盘会在关机流程里被"意外移除"，挂着的 NTFS 可能丢掉尚未下发的写；依据是 Windows 本来
就会在关机时拆掉这块盘，这里只是把这一刻提前。运行中点「停止加速」则**绝不**这么干（用户操作、盘还要继续用）。
该路径**绝不外抛**（抛出去会被 `ThreadException` 兜住并弹窗，反卡关机），收尾过程只落 `debug-security.log`。

### 缓存设计要点

- **非托管内存池**：通过 `Marshal.AllocHGlobal` 直接向操作系统申请大块内存，完全绕过 .NET GC，避免缓存增大导致程序卡顿与内存碎片；

- **按块缓存**：内存划分为固定 **4KB Slot（插槽）**；以 **64MB Slab** 为单位向系统申请内存。
  4KB 块与设备扇区的对应关系是"配置项 `SectorsPerBlock` × `BytesPerSector`"，启动时校验必须等于 4096
  （512 字节扇区的盘填 8，4Kn 盘填 1）；

- **块号索引**：**一张 `ConcurrentDictionary<块号, 槽位+时间轮节点>`**，键是全局块号（`LBA / SectorsPerBlock`）。
  块设备下没有文件，所以没有"文件表 / `CacheEntry` / `BitArray`"这些东西（那是阶段一文件系统形态的产物）；
  写透传后的处置是**按块**的（W1，2026-09-30）：整块被写请求完整覆盖 ⇒ `UpdateBlock` 就地刷新已有的副本
  （省掉下次读的回源），残缺的首尾块才 `InvalidateRange(块号, 1)` 作废；

- **基于系统内存占用的阈值管理**：

  - `EvictionThreshold`（默认 0.8）：系统内存占用超过该比例 → 触发 LRU 淘汰；

  - `StopCachingThreshold`（默认 0.9）：系统内存占用超过该比例 → 新数据不再缓存，直接透传；

  - 安全约束：`EvictionThreshold` 必须小于 `StopCachingThreshold`，否则缓存退化为仅透传；

  - `SsdConservativeThreshold`（默认 0.80）：**L2 的**占用率水位（与上面两个同族同单位）。占用 =（数据槽 +
    空洞槽）/ 容器槽数；未达该线时"激进"（回源块一律落位），达到后转"保守"（只收 ghost 命中的块）。
    引擎内部按**可用槽位**判定，本值是它的补数（占用 80% ⇔ 可用 20%），换算见 `SsdCacheService` 构造器；
    越界夹到 `[0.50, 0.99]` 并落盘。检查器/设置里的"保守线"就是它（2026-10-01 从写死常量提为配置项）；

- **淘汰策略**：两级淘汰，判据**刻意不同**（见 `关于内存缓存的进一步讨论.md` §6/§7）：
  - **L1（内存）**：LRU——「时间桶 + 自排序队列」，在 `AcquireSlot` 里按需增量摘一批（O(K)，不需要后台线程）；
  - **L2（SSD）**：单条 **M 环（FIFO-reinsertion）+ ghost**（2026-09-30 起；原 S3-FIFO 的 S 队列已删除——
    它的"一次性访问过滤"职责与 **L1 完全重复**）。准入只有"**回源读到整块**"一个入口：可用槽位还在保留量
    之上（**激进期**）就回源块一律落位，不足之后（**保守期**）只收 ghost 命中的块——分界值见
    `SsdConservativeThreshold`。L1 命中对 L2 完全静默。
    **两者必须正交**：L1 管"近期性"（LRU），L2 管"跨重启的长期保留"（M 环 + ghost）。
    2026-09-30 之前靠"每次访问（含 L1 命中）都要走 L2 准入"去维持这条正交性，实践下来是反效果：
    L1 服务得了的块灌进 L2 只会把 M 挤空（§7.10 的实测数字），现在的做法是 L1 命中**完全不进 L2**；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youke1686/FlyDisk](https://github.com/youke1686/FlyDisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
