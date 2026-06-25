---
trigger: always_on
description: 编码器校准完成后的速度环 PI 调参规程
---


# 速度环 PI 调参指南

> **前提**: 电流环校准 + KTH71 ANLC + 电角度零点标定 **全部完成**
> 未完成编码器校准就切速度模式, 电角度错误会导致力矩方向不对, PI 无论怎么调都没用。

---

## 一、Live Watch 变量清单 (复制粘贴)

开始前先把以下变量全部加到 Live Watch，后续操作直接改值。

### M1 速度环调参

```
┌─ 变量名 ──────────────── 用途 ─────────────────┐
│ g_FollowM1M2              主从随动开关 (=0)     │
│ g_VofaMotorSel            VOFA 电机选择 (=1)    │
│ g_VofaSrc                 VOFA 数据源 (=1)      │
├─ 控制 ─────────────────────────────────────────┤
│ g_M1_CtrlMode             0=开环, 1=速度        │
│ g_M1_RPM_Cmd              目标转速 (RPM)        │
│ g_M1_Iq_Ref_mA            开环力矩 (默认50mA)   │
├─ 速度 PI ──────────────────────────────────────┤
│ g_M1_SpeedPI              展开看 kp/ki/积分/输出 │
│ g_M1_RpmRampRate          斜坡速率 (默认2RPM/ms)│
├─ 反馈 ─────────────────────────────────────────┤
│ g_Enc1_SpeedFilt          编码器滤波转速 (RPM)   │
└─────────────────────────────────────────────────┘
```

### M2 速度环调参

```
┌─ 变量名 ──────────────── 用途 ─────────────────┐
│ g_FollowM1M2              主从随动开关 (=0)     │
│ g_VofaMotorSel            VOFA 电机选择 (=0)    │
│ g_VofaSrc                 VOFA 数据源 (=1)      │
├─ 控制 ─────────────────────────────────────────┤
│ g_M2_CtrlMode             0=开环, 1=速度        │
│ g_M2_RPM_Cmd              目标转速 (RPM)        │
│ g_M2_Iq_Ref_mA            开环力矩 (默认100mA)  │
├─ 速度 PI ──────────────────────────────────────┤
│ g_M2_SpeedPI              展开看 kp/ki/积分/输出 │
│ g_M2_RpmRampRate          斜坡速率 (默认2RPM/ms)│
├─ 反馈 ─────────────────────────────────────────┤
│ g_Enc2_SpeedFilt          编码器滤波转速 (RPM)   │
└─────────────────────────────────────────────────┘
```

> `Iq_Ref_mA` 切速度模式后不生效, 留着方便随时切回开环对比。
> `SpeedPI` 是结构体, Live Watch 展开可看 kp/ki/integ_limit/out_limit/integral/output。

---

## 二、速度环架构概览

```
  g_M1_RPM_Cmd / g_M2_RPM_Cmd ──→ [斜坡限速] ──→ rpm_target
                                                      │
                                                      ▼
                    ┌──────────────────────────────┐
                    │  SpeedPI_Run (1kHz, TIM3)    │
                    │                              │
                    │  error = rpm_target - speed   │
                    │  P = kp × error              │
                    │  I += ki × error (三重限幅)   │
                    │  output = P + I → Iq (mA)    │
                    └──────────────────────────────┘
                                  │
                                  ▼
                     g_M1_Idq_Ref.q / g_M2_Idq_Ref.q (Q15)
                                  │
                                  ▼
                      电流环 PI (17kHz, ADC ISR)
                                  │
                                  ▼
                              HRTIM PWM
```

**关键特征:**
- 运行频率: 1kHz (TIM3 ISR)
- 输入: RPM 误差
- 输出: Iq 电流指令 (mA)
- 斜坡限速: `g_M1_RpmRampRate` / `g_M2_RpmRampRate` RPM/ms (默认 2, 即 0→1000RPM 需 500ms)
- Anti-windup: 积分限幅 + 输出反算法退饱和 + 输出限幅 (三重保护)

---

## 三、参数含义

`SpeedPI_Float_t` 结构体 (Live Watch 展开 `g_M1_SpeedPI` 或 `g_M2_SpeedPI`):

- **kp** — 比例增益, 单位: mA/RPM
  - 含义: 每 1 RPM 误差产生多少 mA 力矩电流
  - 默认: 0.5
  - 直觉: kp=0.5 → 100RPM 误差产生 50mA Iq

- **ki** — 积分增益, 单位: mA/(RPM·ms)
  - 含义: 每 1 RPM 误差持续 1ms 积分贡献多少 mA
  - 默认: 0.01
  - 作用: 消除摩擦/齿槽导致的稳态转速偏差

- **integ_limit** — 积分限幅 (mA)
  - 默认: 100
  - 作用: 防堵转时积分飙到天花板 → 松开后巨大过冲

- **out_limit** — 输出限幅 (mA)
  - 默认: 300
  - 作用: 最大允许力矩电流, 保护电机不过热
  - 硬上限: `M1_IQ_LIMIT_MAX_MA` / `M2_IQ_LIMIT_MAX_MA` = 600 (M1/M2 各自独立定义)

- **integral** — 积分当前值 (只读诊断, VOFA I4)
- **output** — 输出当前值 (只读诊断, VOFA I5)

**斜坡相关:**
- **g_M1_RpmRampRate / g_M2_RpmRampRate** — 斜坡速率, 单位: RPM/ms, 默认 2, 范围 1~100
- **g_M1_RPM_Cmd / g_M2_RPM_Cmd** — 目标转速, 范围 ±1000 RPM

---

## 四、调参操作步骤

### 3.0 准备

1. 确认 VOFA 连接, `g_VofaSrc = 1` (NORMAL)
2. `g_VofaMotorSel` = 0(M2) 或 1(M1), 选择要调的电机
3. 当前模式为开环: `g_M1_CtrlMode = 0` / `g_M2_CtrlMode = 0`

### 3.1 切换到速度模式

Live Watch 设置:
```
g_M1_RPM_Cmd  = 0  / g_M2_RPM_Cmd  = 0       (先给 0, 防止突然起跳)
g_M1_CtrlMode = 1  / g_M2_CtrlMode = 1       (切速度模式)
```

> 切入瞬间 PI 积分自动清零, 斜坡目标从当前速度开始 — 不会有突变。

### 3.2 给目标转速, 观察 VOFA

```
g_M1_RPM_Cmd = 200  / g_M2_RPM_Cmd = 200      (从低速开始)
```

**VOFA 通道含义 (速度模式):**
- **I0** — 原始转速 (RPM), 有噪声
- **I1** — 滤波转速 (RPM), PI 实际用的反馈
- **I2** — 斜坡后目标 (RPM), PI 实际跟的指令
- **I3** — 速度误差 (RPM), = I2 - I1
- **I4** — PI 积分值 (mA), 观察积分动态
- **I5** — PI 输出 Iq (mA), 最终力矩指令

**健康波形特征:**
- I1 (滤波速度) 平滑跟上 I2 (斜坡目标)
- I3 (误差) 稳态时接近 0 (允许 ±5 RPM)
- I4 (积分) 稳态后收敛到一个小值 (补偿摩擦)
- I5 (Iq) 稳态后稳定, 不振荡

### 3.3 先调 kp (纯比例, 关积分)

```
g_M1_SpeedPI.ki = 0  / g_M2_SpeedPI.ki = 0      (暂时关掉积分)
g_M1_SpeedPI.kp = 0.5 / g_M2_SpeedPI.kp = 0.5   (默认起点)
g_M1_RPM_Cmd    = 200 / g_M2_RPM_Cmd    = 200
```

**观察 I1 (滤波速度) 的响应:**

- **响应太慢 (爬了好几百 ms 还没到)** → kp 增大, 试 1.0, 2.0
- **响应快但振荡 (I1 在目标附近来回摆)** → kp 过大, 减半
- **响应快且平滑, 有稳态偏差** → kp 合适, 下一步加 ki 消偏差

**kp 粗调参考范围 (步进电机):**
- NEMA08 (20mm): 0.3 ~ 1.0
- NEMA17 (42mm): 0.5 ~ 2.0
- NEMA23 (57mm): 0.5 ~ 3.0

> 步进电机因高电感, kp 范围偏小。三相伺服因低电感, kp 可大得多。

### 3.4 加 ki (消除稳态误差)

kp 调好后, 逐步加 ki:

```
g_M1_SpeedPI.ki = 0.005  / g_M2_SpeedPI.ki = 0.005   (从小值开始)
```

**观察:**
- I3 (速度误差) 稳态是否趋近 0
- I4 (积分) 是否缓慢收敛到一个稳定值

**调整方向:**
- **误差消除太慢 (>2 秒才到 0)** → ki 加倍, 试 0.01, 0.02
- **过冲后回摆 (I1 超过目标再落回)** → ki 太大, 减半
- **稳态震荡 (积分 I4 在 ± 来回)** → ki 远过大, 降到 1/4

**ki 典型范围:**
- 一般: 0.005 ~ 0.05
- 高摩擦负载: 可适当加大, 但 integ_limit 要配合

### 3.5 调 integ_limit (积分抗饱和)

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TigerBruce/Double_SPET_FOC](https://github.com/TigerBruce/Double_SPET_FOC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
