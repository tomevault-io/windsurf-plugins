---
trigger: always_on
description: enableSR = (fisrEnable == 2 || fisrEnable == 4)  = true
---

# CLAUDE.md

## 简介

一个 **KernelSU WebUI 模块**，编辑 V3 版 Joyose 云控数据库
（`/data/user/0/com.xiaomi.joyose/databases/SmartP.db` 与 `teg_config.db`）。
UI 跑在 KernelSU 的 WebView 里；模块通过 `bin/joyose-edit.sh` 作为唯一的
特权入口（子命令白名单 — 永远不把用户输入拼进 shell）。

**重点支持的路径是 MIFISR（骁龙 8 Elite 2 / 小米 17 系列）**。其他路径
（15 系列 AFME/FRC/FSR、红米独显 Novatek）**项目维护者没有实机**，代码基于
样本 DB 形状 + Joyose 反编译推断保留，**不保证实际跑得通**。

---

## Joyose 的 boot 决策树（`com.xiaomi.joyose.enhance.a.c()`）

实机选哪条增强上下文由 MIUI device_features + vendor 系统属性决定，按顺序：

```text
d() 先看 FeatureParser.getString("support_dual_dpu")
   == "NT"                   → NovaTekEnhanceContext    (红米独显)
   == "CUSTOMIZE"             → k.b (CustomizeEnhanceContext)   ← 17 系列走这条
   否则 d() 返回 false

e() 再看 FeatureParser.getString("game_enhance_feature_name")
   == "game_enhance_fisr"    + game_turbo_api_version >= 1 → q.i (MIFISREnhanceContext)
   == "game_enhance_frc_sr"   → com.xiaomi.joyose.enhance.b (combiner)
   == "MIFI_SR"              + ro.vendor.display.hyperos.miDualDPU_gamebox_version==3 → v.b
   否则 e() 返回 false

兜底：按顺序查 ro.vendor.display.iris_x7.support / ro.vendor.xiaomi.sr.support /
ro.vendor.gpp.frc.support，第一个为 true 的决定走 o.a / v.d / m.a。
```

**17 Pro Max 和 17 Ultra 都声明 `support_dual_dpu=CUSTOMIZE`**，所以**都走 `k.b`**
（不是我们早前猜的 `q.i`）。"q.i" 路径是给声明 `game_enhance_fisr` feature 的机型
准备的，实机没见过。

---

## MIFISR 路径详解（17 系列，已实机验证）

### 三层阻塞结构

游戏助手悬浮窗能否显示"插帧/超分"勾选框、改 DB 能否生效，取决于三层条件都满足
（星铁 VK 还有一条额外的 Layer 4 守卫，见下方单独小节）：

**Layer 1：MIUI securitycenter UI 层能力判定**

`com.miui.securitycenter` 的 `GameBoxVisionEnhanceUtils.needInitService()` 会查：

- `ro.vendor.gpp.frc.support`（决定 `isDeviceSupportFRC`）
- `ro.vendor.xiaomi.sr.support`（决定 `isSupportResolution`）
- 某个 DPU 检查（决定 `isSupportDualDPU`；17 / 15 系列硬件无 DPU，永远 false）

**任何一个 vendor 属性不为 `"true"`，游戏助手本地直接拒绝渲染画质增强面板，Joyose
和 DB 再怎么配都白费。** MifisrView 顶部 banner 读这两个属性做红/黄/绿诊断。

17 PM 原厂两个 vendor 属性都没设；17 Ultra 两个都是 true。对 17 PM 用户：模块**不会
自动改** vendor 属性（那是系统级写操作，超出本模块职责），UI 给出 `resetprop` 命令
让用户自己决定翻不翻。

**Layer 2：Joyose `customize_game_params.*` bean 存在性**

`k.b.getPictureEnhanceSupportType(pkg)` 的第一行：

```java
if (e.k(ctx).e(pkg) == null) return new int[2];   // [0,0] → UI 勾选框全空
```

`k.e.e(pkg)` 查 `k.e.f3250c` HashMap，这个 map 由 `k.e.w()` 解析
`customize_game_params` 下面的 **5 个并列数组**填充，任一数组里出现
`"<pkg>_xxx"` 条目即可让 `e(pkg)` 返回非 null：

- `dp_fi_config` —— DMI 策略的 srcFps→targetFps 映射
- `dp_sr_config` —— DPQ 策略的超分配置
- `game_mifisr_config` —— MIFISR 策略的合体配置
- `gfrc_config` —— GFRC（骁龙老 FRC 兼容）
- `mfrc_config` —— MFRC（同上）

每条字符串格式 `<pkg>_<载荷>`（parser 强制 `split("_")` 长度 = 2，所以包名里不能有 `_`）。

**Layer 3：`fisr_config` 的 `feature` 声明 + `support_game_mode` 位图**

`k.e.u(pkg, featureName, true)`（被 `k.b.getPictureEnhanceSupportType` 和
`k.e.q` 复用）：

```java
for (q.b bVar : this.f3249b.h2()) {              // 所有 fisr_config 组
    if (bVar.f().contains(pkg) || bVar.f().contains("OTHER")) {
        boolean has = bVar.j().contains(featureName);   // feature 列表含 "FI"/"SR"/"FISR"
        String mode = bVar.k(featureName);              // 对应 feature 的 support_game_mode
        if (mode != null) {
            String[] parts = mode.split("#");            // "1#1" → ["1", "1"]
            // MGAME(均衡) → 取 [0]，TGAME(性能) → 取 [1]
            int idx = "MGAME".equals(currentMode) ? 0 : 1;
            return has && parts[idx] == "1";
        }
        return has;
    }
}
```

**关键**：`support_game_mode` 是 **2 位 bitmap**（`[MGAME位, TGAME位]`，不是模式 ID 列举）。
**`1` = 该模式启用本 feature，`0` = 禁用**。实机对照 Ultra 原厂下发 `"0#1"`：均衡读
`parts[0]=0` → 禁用，性能读 `parts[1]=1` → 启用，正好对应 `k.e.u` 里
`parts[idx] == 1 ? true : false` 的代码逻辑（用户实测 2026-04-22 确认：`"0#1"`
均衡没选项 / 性能有选项）。本模块 `mifisrStandard()` 默认 `"1#1"`（两模式都开）。

**模式来源：`h0.p(ctx).q()` 读的是 `IFeedbackControl.isEnableOptimizeGame()`**
（bound 到 `com.miui.powerkeeper.FeedbackControlService`），true → `"TGAME"` 即
性能档，false → `"MGAME"` 即均衡档，默认 true。所以没绑上 FeedbackControl 时
默认按 TGAME 读 `parts[1]`。

### policy 字段完整清单（2026-04-23 反编译核对）

所有 `fisr_config.enhance_config[i].enhance_policy_config[j]` 的字段，Joyose 的
`q.g.b()` parser 处理了哪些、哪些被实际消费：

| 字段                       | Joyose 读取点               | 作用                               | 缺失行为                                                                                          |
| -------------------------- | --------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------- |
| `feature`                  | `k.e.u` / `q.d.c`           | "FI"/"SR"/"FISR"/... 关键判定      | 必须                                                                                              |
| `strategy`                 | `k.e.i` / `k.e.h`           | 绑 strategy 实例名                 | 必须                                                                                              |
| `support_max_refresh`      | `q.d.f` → `l.i.r`           | `X#Y` 两档 **Hz 整数上限**（见下） | **默认 60**（两档都 clamp）；官方下发普遍 `60#120`                                                |
| `support_game_mode`        | `k.e.u`（带 `z2=true`）     | 2-bit mode 启用 bitmap             | 非 `X#Y` 格式 silently 忽略 mode 位                                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuKongA/JoyoseEdit](https://github.com/YuKongA/JoyoseEdit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
