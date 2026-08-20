---
trigger: always_on
description: 在华为 MateBook E Go（Snapdragon 8cx Gen 3 / sc8280xp，代号 gaokun）上跑原生 AOSP，
---

# 项目：MateBook E Go (sc8280xp / gaokun) 移植 Android

## 目标

在华为 MateBook E Go（Snapdragon 8cx Gen 3 / sc8280xp，代号 gaokun）上跑原生 AOSP，
最终目标是能稳定运行 arm64 手游。

**当前阶段：Stage 6 M13 完成 — ★内核已进入 OTA 范围并实机验收 13/13；★传感器开机自启（不再需要脚本）；★许可证转 GPL-3**（每次开工时更新这一行）

> **★★ Stage 6 M13（2026-08-20）：按 Android 分区规范把内核纳入 OTA。**
>
> 用户明确要求"遵守 Android 分区规范"，所以不再让内核留在 OTA 之外。
> **终局是自研 EFI 加载器**（读 misc 选槽 + 解析 boot 镜像），本轮完成前两段。
> - ★**内核 `CONFIG_QCOM_FASTRPC=y` 实机验证**：`#18` 内核起来后
>   `/dev/fastrpc-*` 四个节点都在而 `/proc/modules` 是 **0 行** ——
>   不再需要 insmod，第 13 个「=m 坑」关闭。
> - ★**`CONFIG_EFI_ZBOOT`**：`Image` 39,422,464 → `vmlinuz.efi` **14,021,120**
>   字节（37→13 MB，2.8 分之一），`PE32+ EFI application`，
>   **经 systemd-boot 实机启动成功**。这是能在 300 MiB 的 ESP 上放两份内核的前提。
> - ★**真 `boot_a`/`boot_b`**（各 64 MiB，nvme0n1p5/p6），`boot` 进
>   `AB_OTA_PARTITIONS`，`TARGET_NO_KERNEL := false` + header **v2**
>   （一个分区装齐 kernel+ramdisk+dtb；v3+ 会强制引入 vendor_boot）。
>   构建日志实证：`--partition_names boot:product:system:system_ext:vendor`、
>   `Using generator FullUpdateGenerator() for partition boot`。
>   空间来自**删除 userdata-old**（用户授权），余 63.9 GiB 未分配；GPT 已备份。
> - ★**ESP 改成每槽独立目录** `slot_a/` `slot_b/`，两个 BLS 条目各指自己的目录
>   —— 于是 postinstall 只写"刚刷好的那个槽"，**绝不碰正在运行的内核**，
>   回滚天然安全。实机启动验证通过。
> - ★**过渡期的那一步**：`gaokun3-bootimg-extract`（include 树内 `<bootimg.h>`，
>   不手抄偏移）+ `gaokun3-ota-postinstall.sh`。argv[1] 是目标槽位整数 ——
>   出处 `postinstall_runner_action.cc:355-357`，不是猜的。
>   ⚠️ `POSTINSTALL_OPTIONAL=false` 是故意的：宁可 OTA 失败，也不能出现
>   "新 system + 旧内核"。
> - ★**两个解包实现交叉校验**（设备上的 C++ 与安装器里的 python）：
>   同一个 boot.img，三个文件 sha256 逐个一致。偏移抄错的典型后果是
>   "magic 通过但内容错位"，只有对校验和才看得出来。
>
> **本轮拆掉/发现的坑（都值得记）**：
> - ⚠️★ `boot_control` HAL 里的 `EnsureSlotProbeHints()` **已删除**。它把
>   `by-name/boot_a|b` symlink 到 `/dev/null` 好让 libboot_control 数出 2 槽；
>   现在那两条路径是真实分区，一旦它抢在 ueventd 之前跑成功，
>   update_engine 就会把 boot 镜像写进 `/dev/null`，**静默毁掉整个更新**。
> - ⚠️★ `deploy-from-ubuntu.sh` 的 `.bak-prev` 备份**已去掉** —— 那正是把
>   296 MiB 的 ESP 吃到 95%（63 MB 全是备份）的元凶，再部署一次就会写出被
>   截断的内核。**现在"另一个槽就是备份"**，这才是 A/B 的意义。
> - ⚠️★ 变量名是 **`BOARD_INCLUDE_DTB_IN_BOOTIMG`**（BOOTIMG 连写）。写错不会被
>   忽略，而是在 **lunch 阶段**炸成 `Don't have a product spec for:
>   'lineage_gaokun3'` —— 极易误判成 manifest 没拉全设备树。
>   ★**判据**：grep 一个变量名**返回空**本身就是信号。
> - ⚠️★ 必须用 `TARGET_PREBUILT_KERNEL`。Lineage 的 `kernel.mk:171-176` 另有一条
>   "扫 PRODUCT_COPY_FILES 里 dest=kernel"的分支，但那段写的是
>   `$(ifeq kernel,$(_dest), ...)` —— **`ifeq` 不是 make 函数**，整个展开成空，
>   那条路根本不生效（Lineage 自己的 bug）。设了它之后再用
>   `PRODUCT_COPY_FILES` 拷一份会撞 `overriding commands for target .../kernel`。
> - ⚠️★ `PRODUCT_OTA_ENFORCE_VINTF_KERNEL_REQUIREMENTS := false`。有了真 boot.img
>   这道检查才生效，而它不认识主线 7.2：`No kernel entry found for kernel
>   version 7.2 at kernel FCM version 202504`（Minimum LTS 6.12.0），连
>   `7.2.0-rc2-gaokun3+` 都解析不了。
> - ⚠️★ `BOARD_KERNEL_CMDLINE` **早已与实机 BLS 条目漂移**（此前它没有消费者，
>   漂了没人发现）：原写 `deferred_probe_timeout=30` 而实际是 10，且缺
>   `veritymode`/`flash.locked`/`verifiedbootstate`/`iommu.*` 四项。已对齐。
>   **教训：没有消费者的配置一定会漂。**
> - ⚠️ 编内核**两处**都要 `CROSS_COMPILE=aarch64-linux-gnu-`：编译时不带会在
>   `prepare0` 报 `gcc: unrecognized command-line option '-mlittle-endian'`；
>   `olddefconfig` 不带会用**宿主 gcc** 评估 `CC_HAS_*`，静默改掉一批符号。
> - ⚠️ `bacon` **不重建 super.img** —— 发布安装包要单独 `m superimage`。
> - ⚠️★ `scripts/kernel-config-android.sh` 里有 5 项（`PM_WAKELOCKS`/`CPUSETS_V1`/
>   `MEMCG_V1`/`UCLAMP_TASK(_GROUP)`）**一直没被应用**：它们写在续行里，而前面
>   是**普通 `#` 注释** —— shell 里注释会终止续行，命令提前结束，只留一句
>   `--enable: command not found`。现有机器没出问题纯属侥幸（.config 里早就有）；
>   新树上会重现 cgroup v1 开机失败。已拆成独立命令并补进断言（37→44 项）。
>
> ★★**新 ROM 已部署并验收**（`ro.build.date = Thu Aug 20 11:06:26 UTC 2026`）：
> `bash scripts/verify-kernel-ota.sh` **13 项全过 / 0 失败** —— 内核、
> FASTRPC=y（4 个节点 / 0 模块）、boot_a 的 Android 镜像、bootctl 2 槽、
> 解包器与 postinstall 钩子、ESP 两槽文件齐、以及★**传感器开机自启**
> （`hexagonrpcd` 由镜像拉起，框架里 Z≈9.88，`gaokun3-ssc-test` 3 秒 151 条）。
> **`scripts/sensors-up-android.sh` 那套手动步骤从此不需要了。**
>
> ⚠️★**运维：不要让构建机干等大文件传输 —— 用 R2 中转。** 本轮直连
> 构建机→救援机只有 1 MB/s（2.7 GB 要 45 分钟，而机器按分钟计费）；
> 改成 zstd 压到 1.2 GB → 上传 R2 **27 秒（43 MB/s）** → 停机 → 本机拉 →
> 局域网推到救援机 **51 秒（23 MB/s）**。R2 出站免费。
> - ⚠️★ **别在上传前去探对象**：我先 curl 了一次那个还不存在的路径，
>   Cloudflare 把 **404 缓存住了**，上传成功后照样 404，
>   加个 `?cb=…` 才拿得到。差一点误判成"上传失败"。
> - ⚠️★ **沙箱代理会掐断到构建机的 ssh**：连续几次 `Connection closed by
>   ...port 22`，一度以为机器挂了（Azure 查询显示 running）。
>   同一条命令绕开沙箱就通 —— 长任务与大流量的 ssh 一律绕沙箱。
> - ⚠️ 家里到 Cloudflare 只有 **71 KB/s**（救援机 WiFi 直连），
>   所以"让设备自己从 R2 拉"这条路走不通，得经本机中转。
>
> ⬜ **还欠**：端到端 OTA 实测；
> 第 3 段的 **EFI 加载器**（读 misc + 解析 boot 镜像 + 装 initrd/DTB 协议，
> 用 systemd-boot 的 `efi` 指令 chainload，起不来就选别的条目 —— 安全阀）。
> ⬜ 另记用户提议：**把救援 Ubuntu 换成/补上 Android recovery**。我的判断是
> 补充而非替代 —— recovery 也是 boot 镜像格式（systemd-boot 同样读不了），
> 而它的主要能力 `adb sideload` 已被系统内 OTA 覆盖；而 `sgdisk`/`resize2fs`/
> sshd 这些**本轮重新分区真正用到的东西** recovery 给不了。
> 合理的优化是给 Ubuntu 瘦身（最小 rootfs 1–2 GiB，现在 24.6 GiB）。

> **★★ Stage 6 M12（2026-08-20）：sensors HAL 落地，自动旋转接上。**
>
> ★**实机验证**（`dumpsys sensorservice`）：
> ```
> Active sensors: SH3001 Accelerometer (handle=0x00000001, connections=2)
> SH3001 Accelerometer: last 50 events
>      1 (ts=612.371743828, wall=17:42:46.858) -0.04, 0.05, 9.88,
> ```
> 两个消费者正是自动旋转的 `WindowOrientationListener$AccelSensorJudge`
> 与 `FaceDownDetector`。★**框架还自动融合出 Game Rotation Vector /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vahiru/gaokun-android](https://github.com/vahiru/gaokun-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
