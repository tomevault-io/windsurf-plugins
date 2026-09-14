---
trigger: always_on
description: > 2026-09-14 用户授权发布1.1.0至Likely7/Veyra-NRVideo：包含普通版Smooth Motion开启说明、允许自主叠加的策略及此前采集格式扩展；更新双语README与教程、完整便携包及对应源码。沿用已批准七个增强运行文件与patched FFmpeg，不新增驱动文件或SDK入Git。Smooth Motion由NVIDIA App管理，用户已反馈本机有效稳定，不宣称驱动算法内置、所有设备验收或叠加必然更好。发布完成并核实资产后按用户要求关机。
---

# Veyra 项目 Agent 执行规则

> 2026-09-14 用户授权发布1.1.0至Likely7/Veyra-NRVideo：包含普通版Smooth Motion开启说明、允许自主叠加的策略及此前采集格式扩展；更新双语README与教程、完整便携包及对应源码。沿用已批准七个增强运行文件与patched FFmpeg，不新增驱动文件或SDK入Git。Smooth Motion由NVIDIA App管理，用户已反馈本机有效稳定，不宣称驱动算法内置、所有设备验收或叠加必然更好。发布完成并核实资产后按用户要求关机。

> 2026-09-14 用户确认 Smooth Motion 在本机实验版有效且稳定，决定普通版仅提供 NVIDIA App 开启方法与注意事项：只用驱动补帧时选择软件“关闭补帧”；换回内部 DLSS/XeSS 时由用户在 NVIDIA App 关闭 AI 插帧。允许用户自行叠加驱动与内部补帧，不检测、不拦截、不强制互斥，也不自动修改驱动配置。叠加效果未验证，不宣称更好；软件统计/截图/导出不冒充含驱动生成帧。本条取代此前 Smooth Motion 实验方案的强制互斥与软件内驱动配置管理要求；保留原实验分支作历史存档。本轮未授权新发布。

> 2026-09-13 用户授权修复持续播放后重连额度不恢复并发布1.0.2。连续30秒有解码进展、帧间无1秒断档后恢复本轮3次重试；会话累计计数/源epoch不回退，短暂出帧不续次数。包含此前手动占用重试、首帧等待和日志保留修复。沿用七个运行组件及patched FFmpeg；不把软件恢复改善宣传成Wi-Fi断流已根治。

> 2026-09-13 用户授权发布1.0.1：采集音频10ms缓冲协商与输入诊断修复合入主线、更新双语说明和完整便携包。沿用1.0.0七文件运行组件，FFmpeg slice补丁和对应源码要求不变。用户日志约半秒音频落后与旧缓冲缺口高度一致；修复后的反馈者实卡结果尚未取得，不宣称所有设备延迟已消除。

> 2026-09-13 用户授权整合已完成修复分支、替换透明 Logo，并向 Likely7/Veyra-NRVideo 发布正式应用版本 1.0.0。沿用七个已批准增强运行文件；保留 GPU DIS 为可选实验功能，NVOF 默认。必须使用带 PS5 H.264 slice 补丁的 FFmpeg，附对应源码、构建记录、补丁与开源许可证。正式应用版本号不改变 NR、DLSS FG、XeSS、GPU DIS、PSN/HDR 等功能的实验边界。SDK、DLL、模型、凭据、配置及测试媒体仍不得新增进入源码 Git。

> 2026-09-13 PS5 H.264 硬解细条修复：本机 FFmpeg n9.0.1 增加 `scripts/ffmpeg/ps5-h264-slices.patch`（32→256 slices），同一真实 PS5 AU 软/硬解已全图比较一致；另修正硬解 SRV 双帧覆盖。后续构建/发布不得无意退回未打补丁的 FFmpeg。对应源码应使用实际 patched tree，并携带 `veyra-local-build.json` 及补丁/重编译说明；原 vcpkg SPDX 仅代表底包。详见 `docs/PS5_HARDWARE_STRIP_REPAIR_2026-09-13.md`。本次只做本地修复，未授权新发布；NVIDIA 运行组件不变。

> 2026-09-12 用户授权合并 PS5 开发版并发布 0.0.5 至 Likely7/Veyra-NRVideo，更新双语 README 和完整便携包。沿用七个已批准增强运行文件，串流开源依赖附对应源码、固定版本、补丁及许可证。SDK、凭据、个人配置、测试媒体不进入源码或便携包。实验功能边界如实说明。

> 2026-09-12 用户授权实施 PS5 HDR、PSN 登录与主机保留方案（docs/PS5_HDR_PSN_HOST_PLAN_2026-09-12.md）。仅对本次经过显式HDR输入/输出契约的PS5路径扩展旧SDR限制；文件/采集/导出不得全局放开HDR防错检查。原生HDR旁路显示与“HDR转SDR后增强”必须区分，不能将实验NR的SDR结果冒充原生HDR增强。PSN凭据及主机密钥只在用户数据目录DPAPI加密保存，不提交Git。当前状态按对应施工记录，不由本授权推断测试通过。

> 2026-09-12 用户授权默认关闭的“低延迟模式”：实时预览可选 NR→SR→FG，取代本文件对该选项的固定SR先行限制。NR在源尺寸（或明确的实时内部尺寸）执行，残差/保护合成后再进入SR；共享现有图、光流和生命周期，不另造播放循环。默认仍SR→NR→FG，图片/视频导出保持原有完整处理。不得保证所有素材延迟下降；提示可能增加拖影/边缘瑕疵，真实屏幕撕裂不能简单归因于算法顺序。见 docs/NR_BEFORE_SR_PREVIEW_2026-09-12.md。

> 2026-09-11 用户授权执行实时调度修复（方案见 `docs/REALTIME_AV_SCHEDULING_REPAIR_PLAN_2026-09-11.md`，开工前存档点 `checkpoint/av-scheduling-2026-09-11`，隔离分支施工）：实时文件/采集**预览**在持续欠速时允许按真实 PTS 跳过已解码源帧的增强/呈现机会（先省补帧工作，再省原帧增强），稳态音频不停机、媒体时间保持一倍速、跳帧均匀覆盖时间线；源 PTS/序号/duration 保持真实，历史按断点 reset。该例外只限实时预览：视频导出、图片处理、暂停单帧查看仍必须完整处理，不得把丢帧扩大到导出，也不得把预览跳帧伪装成未丢帧。不得以停音频/丢 PCM/偷偷放慢声音制造同步。

> 2026-09-11 用户授权发布 0.0.4：当前音画同步与音频连续性修复推送到 `Likely7/Veyra-NRVideo`，更新中英文 README 并发布完整便携包。运行组件沿用 0.0.3 的七文件白名单及双 NR 原件身份，不新增或替换运行时；社区版继续记录 `HashMismatch`。源码 Git 禁止 SDK/DLL/模型，软件允许用户替换 DLL 的决定不变。

> 2026-09-11 用户授权发布当前0.0.3：源码推送到`Likely7/Veyra-NRVideo`，完整便携Release包含当前双NR运行版本和直播实验选项。此前用户指定的RTX40/50社区DLL可原样进入本次Release的`runtime/experimental/nr-community/`，身份沿用下条SHA256、310.8.0.0、165840496字节、`HashMismatch`，必须逐文件列入manifest并明确社区修改版；这是本次发布范围的扩展，不允许自动加入其他版本或修改文件。源码Git仍绝不包含SDK/DLL/模型。软件不恢复运行时哈希锁。OBS捕获说明按用户要求仅补README，不继续改软件提示。

> 2026-09-11 用户指定RTX40/50社区NR运行时接入：允许把用户明确提供的`E:/Ai/mg/DLSSNR-DLL-Options-310.8.0.0/Community-RTX40-RTX50/nvngx_dlssnr.dll`原样复制到忽略的`runtime_local/nvidia/nr-community/`进行本机切换测试。该文件SHA256为`984BEE0F775C277D5829B8FD6775D53A7B0F75396C852B3AAF06A18375F81014`，签名状态`HashMismatch`，必须标注社区修改/实验，不能称为有效NVIDIA签名原版。用户指定此文件取代旧“仅根目录原版可本机加载”的限制；不授予Agent修改/重签名文件的权限，不自动扩大Release默认白名单，不上传DLL到源码Git。

> 2026-09-11 用户发布与替换决定：0.0.2发布到`Likely7/Veyra-NRVideo`，源码与Release二进制继续分离。用户明确要求去掉运行时校验、允许自行替换DLL：不得用固定哈希、签名或manifest拒绝用户替换的运行时；保留绝对路径加载、API存在性及初始化结果检查，替换不保证ABI/硬件兼容。下文相冲突的启动校验/禁止用户替换要求被本条取代。发布者制作的默认Runtime Pack仍按固定来源、哈希、签名与许可证审计；组件清单供用户查看，不作为软件加载锁。禁止将SDK/运行时/模型提交源码Git的规则不变。

> 2026-09-10 用户废弃旧 Loop：`loop/`、`scripts/loop-gate.ps1`、`CONTROL_HASHES.json` 及早期 Phase 排队/控制哈希停工规则全部退出当前执行流程，仅保留作历史记录。不得因旧 Loop 状态、STOP、控制哈希或旧文档顺序阻塞用户明确要求的修复，也不得更新旧清单来制造通过。当前任务以最新用户指令、对应修复计划和 `docs/WORKLOG.md` 为准。构建、针对性回归及必要的 `scripts/gates/delivery.ps1` 直接执行；单次测试最多 300 秒，并非全部测试累计 300 秒。运行时身份、源码/二进制隔离、真实验证与发布授权规则继续生效。

> 2026-09-07 新用户决定：默认明确标注的实时档（4K输入可用1080内部处理），保留原生4K可选；视频导出仍原生4K。不得要求以本机原生4K NR达到60fps作为本次默认档门槛，也不得把实时档冒充native4K。最终统一软件短测为 `scripts/gates/delivery.ps1`（phase5–7本次合同的合并检查），实卡仍由用户验收。其他安全/许可规则不变。

> 2026-09-06 用户授权接管修订：当前推进、五分钟短测与用户实卡验收以 `docs/ACTIVE_DELIVERY_PLAN.md` 为准，取代下文旧的严格串行施工/30分钟测试/未接设备阻塞全部交付规则。历史记录不是当前通过证明。
> 2026-09-09 用户发布决策：Veyra 采用“完整开箱即用的实验 Runtime Pack”发布策略。用户明确接受该 Pack 可能被 GitHub 下架或被权利方要求移除的风险，并授权将经过身份校验的指定实验运行时作为 **GitHub Release 资产** 随 Veyra 用户包分发。该例外不允许把 NVIDIA SDK、运行时、模型、头文件、样例或压缩包提交进源码仓库、Git LFS、Git 历史或 C++ 源文件；不允许修改、重签名、伪装或从游戏/驱动缓存提取二进制。详情以“Release Runtime Pack 规则”为准。

本文件对在本目录工作的所有 Agent 生效。不要只扫标题；开工前必须完整阅读：

1. `README.md`——当前状态和唯一入口；
2. 当前任务对应的 `docs/*PLAN*.md`——当前施工方案；旧 `VEYRA_AGENT_EXECUTION_PLAYBOOK_V1.md` 仅作技术契约与历史参考；
3. `VEYRA_PRODUCT_SPEC_V1.md`——产品与技术边界；
4. `docs/COMPETITOR_AUDIT_2026-09-03.md`——竞品事实、许可证和可迁移启发；
5. 当前阶段的 `docs/WORKLOG.md`——已完成、失败记录和下一步。

旧 Loop 不再要求阅读、同步或运行。最新用户指令优先于历史方案；仍适用的技术、运行时身份和许可证约束以本文件为准。

## 不可擅自改变的决定

- V1 是 Windows x64、C++20、Win32、D3D12 项目。
- V1 直接调用 NGX；不要同时接入 Streamline。Streamline 仅作未来替换方案和文档参考。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Likely7/Veyra-NRVideo](https://github.com/Likely7/Veyra-NRVideo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
