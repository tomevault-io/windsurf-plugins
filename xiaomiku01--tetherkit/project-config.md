---
trigger: always_on
description: > 本文件是给 AI agent（以及接手的人类）看的**工作记忆**。
---

# AGENTS.md —— TetherKit 实现备忘

> 本文件是给 AI agent（以及接手的人类）看的**工作记忆**。
> 每完成一个提交都要更新对应章节。开始任何工作前先读一遍本文件，避免重复踩坑。

---

## 1. 项目一句话

macOS **用户态** RNDIS 驱动：USB 侧用 libusb 与 RNDIS 设备（Android 手机 USB 网络共享等）
通话，网卡侧用 `feth` 虚拟网卡对 + BPF 直接读写原始以太帧，把设备变成一张系统可见的网卡。

---

## 2. 已实测确认的环境事实（**不要重复验证，直接引用**）

实测机器：macOS 26.5.1 (Darwin 25.5.0)、Apple Silicon arm64、Apple clang 21.0.0。

### 2.1 工具链

| 事实 | 结论 | 验证方式 |
|---|---|---|
| Apple clang 21 的 C++23 | **可用**：`std::expected`、`std::byteswap`、`std::format`、`jthread`、`stop_token`、`latch`、`counting_semaphore` 全部通过编译+运行 | 实测编译运行 |
| `std::expected` / `std::byteswap` 在 `-std=c++20` 下 | **不可用**，是 C++23 库特性。项目因此定为 C++23 | 实测编译报错 |
| `std::format` 浮点格式化 | 依赖 libc++ 的 `std::to_chars`，带 availability 标注，**部署目标必须 ≥ macOS 13.3**，否则编译失败 | 实测 `-mmacosx-version-min=13.0` 报 `'to_chars' is unavailable: introduced in macOS 13.3` |
| `std::hardware_destructive_interference_size` | Apple libc++ **未提供**。必须自己定义缓存行常量 | 实测编译报错 |
| **缓存行大小** | **128 字节**（不是 64！）`hw.cachelinesize: 128`。SPSC 队列的 false-sharing 填充必须按 128 对齐 | `sysctl hw.cachelinesize` |
| L1D 缓存 | 65536 字节 | `sysctl hw.l1dcachesize` |
| CPU | 10 逻辑核，其中 **4 个性能核**（`hw.perflevel0.logicalcpu = 4`）。数据路径线程需要用 QoS 争取性能核 | `sysctl hw.ncpu hw.perflevel0.logicalcpu` |
| `-mcpu=apple-m1` / `-mcpu=native` | 都可用；但默认**不开**（见 `cmake/Optimizations.cmake` 里的取舍说明） | 实测编译 |
| ninja | **未安装**，用默认 Unix Makefiles 生成器 | `which ninja` |
| CMake | 4.3.3 | `cmake --version` |

### 2.2 libusb

| 事实 | 结论 |
|---|---|
| 版本 | 1.0.30.12037，`/opt/homebrew/opt/libusb` |
| pkg-config | 可用。注意头文件目录是 `.../include/libusb-1.0`（非常规），`FindLibUSB.cmake` 已处理 |
| hotplug | `libusb_has_capability(LIBUSB_CAP_HAS_HOTPLUG)` 返回 **1**，支持 |
| 链接依赖 | darwin 后端需要 `IOKit`、`CoreFoundation`、`Security` 三个 framework |
| **本机 USB 设备** | 立项时为 **0**（`libusb_get_device_list` 与 `ioreg -c IOUSBHostDevice` 都是 0 条），架构因此按「USB 逻辑必须能用 mock 后端离线测试」来设计。后来接上过真实 RNDIS 设备做验证（见第 6 节），但**这个设计约束仍然有效** —— 不要假设跑测试时一定有设备 |

### 2.3 BPF（Darwin）

| 事实 | 结论 |
|---|---|
| **零拷贝 BPF** | **不存在**。SDK 的 `net/bpf.h` 里**没有** `BIOCSETZBUF` / `BIOCGETZMAX` / `BIOCROTZBUF`（FreeBSD 有，Darwin 没有）。只能用经典 BPF：大 `BIOCSBLEN` + 批量 `read()` |
| 可用 ioctl 全集 | `BIOCGBLEN`(102R) `BIOCSBLEN`(102WR) `BIOCSETF`(103) `BIOCFLUSH`(104) `BIOCPROMISC`(105) `BIOCGDLT`(106) `BIOCGETIF`(107) `BIOCSETIF`(108) `BIOCSRTIMEOUT`(109) `BIOCGRTIMEOUT`(110) `BIOCGSTATS`(111) `BIOCIMMEDIATE`(112) `BIOCVERSION`(113) `BIOCGRSIG`(114) `BIOCSRSIG`(115) `BIOCGHDRCMPLT`(116) `BIOCSHDRCMPLT`(117) `BIOCGSEESENT`(118) `BIOCSSEESENT`(119) `BIOCSDLT`(120) `BIOCGDLTLIST`(121) `BIOCSETFNR`(126) |
| `struct bpf_hdr` | `{ struct BPF_TIMEVAL bh_tstamp; bpf_u_int32 bh_caplen; bpf_u_int32 bh_datalen; u_short bh_hdrlen; }` —— 遍历时**必须**用 `bh_hdrlen` 而非 `sizeof(bpf_hdr)` |
| 对齐宏 | `BPF_ALIGNMENT = sizeof(int32_t) = 4`，`BPF_WORDALIGN(x) = ((x)+3) & ~3` |
| 设备节点 | `/dev/bpf0..3` 存在（Darwin 按需克隆更多节点） |
| **批量写** | ✅ **支持**（macOS 14+）：私有 ioctl `BIOCSBATCHWRITE = _IOW('B',143,int) = 0x8004428f`。缓冲格式与 read 对称（连续的 `bpf_hdr + 帧`，每条按 `BPF_WORDALIGN` 对齐），前置条件是 `BIOCSHDRCMPLT=1`。macOS 13 及更早**没有**此 ioctl → 必须运行时特性探测，失败回落逐帧 write |
| 私有 ioctl 编号（实测核对） | `BIOCSBATCHWRITE`=0x8004428f、`BIOCSNOTSTAMP`=0x80044291（关时间戳，省每帧一次 microtime）、`BIOCSWRITEMAX`=0x8004428c、`BIOCSDIRECTION`=0x8004428a、`BIOCSHEADDROP`=0x80044280。macOS 26 把它们从 `net/bpf.h` 挪到了 `net/bpf_private.h`（**SDK 未提供该文件**） |
| `BIOCSBLEN` 上限 | `sysctl debug.bpf_bufsize_cap` = **33554432（32 MiB）**。超限**不报错**，静默截断并通过 `_IOWR` 把实际值**写回参数** |
| `read()` 缓冲长度 | **必须精确等于** `bd_bufsize`，否则 `bpfread` 开头就返回 `EINVAL`。必须用 `BIOCSBLEN` 的写回值 |
| `BIOCSHDRCMPLT` | **必须设为 1**。=0 时 `bpfwrite` 会剥掉前 14 字节重建帧头（源 MAC 被驱动改写）；=1 才走 `DLIL_OUTPUT_FLAGS_RAW` 原样透传。批量写也硬性要求它是 1 |
| 单帧写入长度上限 | `BPF_WRITE_LEEWAY = 18`，`hdrcmplt=1` 时整帧长度必须 ≤ 接口 MTU + 18（MTU=1500 → 1518） |
| `BIOCSRTIMEOUT` 分辨率 | **10 ms**（内核存 `tvtohz(tv)-1` 个 tick，实测 `kern.clockrate hz=100`）。传 `{0,0}` 会变成**永久阻塞** |
| 最优读取模型 | `BIOCIMMEDIATE=1` + **专用线程阻塞 `read()`**，不要用 kqueue。immediate 下每来一包就唤醒，`read()` 醒来时一次性交付期间累积的全部包（低速低延迟、高速自动大批量，行为类似 NAPI），每批只一次系统调用；kqueue 的就绪判据完全一样却要多一次 `kevent()` |
| `/dev/bpf` 克隆节点 | **不存在**（实测 `ls /dev/bpf` → No such file）。必须遍历 `/dev/bpf%d`：`EBUSY` 试下一个、`ENOENT` 到上限。打开当前最后一个节点时内核会按需再造一个。上限 `sysctl debug.bpf_maxdevices` = 256 |
| `access_bpf` 组 | macOS **没有**（那是 FreeBSD 的做法，实测 `dscl . -list /Groups` 无匹配）。`/dev/bpf*` 是 `0600 root:wheel`，只能 root |

### 2.4 feth（if_fake）

| 事实 | 结论 |
|---|---|
| 是否存在 | **存在**，`sysctl net.link.fake.*` 可见 |
| 关键 sysctl | `net.link.fake.max_mtu = 2048`、`tx_headroom = 32`、`buflet_size = 512`、`qset_cnt = 4`、`link_layer_aggregation_factor = 96` |
| 创建权限 | **需要 root**：非 root 下 `ifconfig feth0 create` 返回 `SIOCIFCREATE2: Operation not permitted` |
| `struct ifdrv` | **不在**公开 SDK 的 `net/if.h` 中，必须自行声明。带 `#pragma pack(4)`，LP64 下 `sizeof == 40`（偏移 16/24/32）。**大小参与 ioctl 编号计算**，算错就得到不存在的 ioctl 号 → 已用 `static_assert` 钉死。实测 `SIOCSDRVSPEC = 0x8028697b`、`SIOCGDRVSPEC = 0xc028697b` |
| `net/if_fake_var.h` | **不在**公开 SDK 中。`struct if_fake_request` = `uint64_t reserved[4]`（32 字节，**内核校验必须全零**）+ 128 字节 union，**总 160 字节**。`IF_FAKE_S_CMD_SET_PEER = 1`、`IF_FAKE_G_CMD_GET_PEER = 1` |
| 私有 ABI 的版本风险 | **极低，无需降级到 ifconfig**：`if_fake_var.h` 在 xnu-7195(macOS 11) → xnu-12377(macOS 26) 的所有发布 tag 下文件 md5 完全相同，从未变动。Apple 自己的 `ifconfig fethN peer fethM` 走的就是这套 ABI |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XiaoMiku01/TetherKit](https://github.com/XiaoMiku01/TetherKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
