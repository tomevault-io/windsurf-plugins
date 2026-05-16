---
trigger: always_on
description: TX-5DR 数字电台项目指南。**请使用中文与用户沟通。**
---

# CLAUDE.md

TX-5DR 数字电台项目指南。**请使用中文与用户沟通。**

## 项目概述
Node.js 后端 + React 前端 + Electron 桌面应用，Turborepo + Yarn 4 管理 monorepo。

## 包结构
- **contracts**: Schema 和类型 → 详见 `packages/contracts/CLAUDE.md`
- **core**: 通信客户端 → 详见 `packages/core/CLAUDE.md`
- **server**: 后端服务（DigitalRadioEngine Facade + 子系统）→ 详见 `packages/server/CLAUDE.md`
- **web**: React 前端 → 详见 `packages/web/CLAUDE.md`
- **electron-***: 桌面应用 → 详见各包 CLAUDE.md

依赖: contracts → core → web/electron, core ↔ server

## 常用命令
```bash
# 开发
yarn dev                    # 浏览器模式（启动 server + web，默认 http://localhost:8076，冲突时递增）
yarn dev:electron           # Electron模式（启动 server + web + electron-main）

# 独立启动（用于调试）
yarn workspace @tx5dr/server dev    # 单独启动后端（4000端口）
yarn workspace @tx5dr/web dev       # 单独启动前端（8076起，冲突时递增）
yarn workspace @tx5dr/electron-main dev  # 单独启动Electron（需要先启动server和web）

# 构建
yarn build                  # 构建所有包
yarn build:package         # Electron打包
yarn lint                   # 代码检查
yarn test                   # 测试

# Docker
yarn docker:build          # Docker构建
docker-compose up -d        # 启动服务

# DXCC 数据
yarn generate:dxcc         # 生成/更新 core 的 DXCC 数据文件（可配合 BigCTY 手动下载目录使用）
```

若需要更新 DXCC 数据，优先使用手动下载的 BigCTY 目录本地生成，例如：
`node scripts/generate-dxcc-data.mjs --cty-dir=/Users/you/Downloads/bigcty-YYYYMMDD --arrl=https://www.arrl.org/files/file/DXCC/Current_Deleted.txt`

注意：该脚本只应在确认 BigCTY 数据包已手动下载后执行；不要依赖 country-files.com 页面里的旧下载直链。

## 技术栈
前端: React 18 + TypeScript + HeroUI + WebGL
后端: Fastify + Audify (RtAudio) + WSJTX + WebSocket
工具: Piscina 工作池 + Turborepo + XState v5 状态机

## 核心架构

### Profile 系统
Profile 是电台配置 + 音频配置的原子单元，由 `ProfileManager` 管理 CRUD 和激活。

**数据结构** (`contracts/radio-profile.schema.ts`):
- `id` / `name` / `description` — 标识信息
- `radio: HamlibConfig` — 电台连接配置（type + network/icomWlan/serial 三种子配置共存）
- `audio: AudioDeviceSettings` — 音频设备配置
- `audioLockedToRadio` — ICOM WLAN 时自动锁定为 true

**激活流程** (`ProfileManager.activateProfile`):
1. 安全停止引擎（超时 10s 兜底）
2. 切换 activeProfileId（原子操作）
3. 广播 `profileChanged` 事件
4. 始终启动引擎（使用新 Profile 配置，启动失败不影响切换）

### 电台连接层

**三层架构**:
```
IRadioConnection (统一接口)       ← connect/disconnect/setFrequency/setPTT/...
  ├─ IcomWlanConnection           ← ICOM IC-705 等 WiFi 直连
  ├─ HamlibConnection             ← Hamlib 网络或串口（通用电台）
  └─ NullConnection               ← 无电台模式（测试/纯监听）
RadioConnectionFactory            ← 工厂：根据 HamlibConfig.type 创建实例
PhysicalRadioManager              ← 编排器：连接启停 + 状态机驱动 + 事件转发
```

**连接配置** (`contracts/radio.schema.ts` — `HamlibConfig`):
- `type`: `'none' | 'network' | 'serial' | 'icom-wlan'`
- `network` / `icomWlan` / `serial` — 三种子配置对象共存，按 type 读取对应配置
- `transmitCompensationMs` — 发射时序补偿 (-1000~1000ms)

**电台 I/O 约束**:
- 所有底层 CAT/CI-V 访问必须经过连接对象自己的串行队列，禁止绕过连接层直接并发访问 rig handle
- `setFrequency` / `setMode` / `setPTT` 属于关键操作，必须保守串行执行
- “切频 + 切模式”必须走 `applyOperatingState(...)` 这类复合入口，避免拆成多个独立写操作
- meter、capability、频率监测属于低优先级轮询；关键操作进行中应直接跳过，不得抢占
- 低优先级轮询失败默认只记日志，不应单独作为断线依据
- 连接后的后台轮询必须在保守 bootstrap 完成后再启动

### 双状态机架构 (XState v5)

系统使用两个 XState v5 状态机分别管理引擎生命周期和电台连接，代码位于 `server/src/state-machines/`。

**引擎状态机** (`engineStateMachine.ts`):
```
IDLE ──START──→ STARTING ──onDone──→ RUNNING ──STOP──→ STOPPING ──onDone──→ IDLE
                  │ onError→IDLE       │ RADIO_DISCONNECTED/FORCE_STOP→STOPPING
```
- 4 状态: IDLE / STARTING / RUNNING / STOPPING
- 启动/停止失败均回 IDLE（context.error 记录错误）
- 电台断线 (`RADIO_DISCONNECTED`) 触发强制停止
- `EngineLifecycle` 子系统管理 ResourceManager 按优先级启停资源

**电台状态机** (`radioStateMachine.ts`):
```
DISCONNECTED ──CONNECT──→ CONNECTING ──onDone──→ CONNECTED
     ↑ 重试耗尽                                    │ CONNECTION_LOST
     └──────────────── RECONNECTING ←───────────────┘ (仅 wasEverConnected=true)
```
- 4 状态: DISCONNECTED / CONNECTING / CONNECTED / RECONNECTING
- **首次连接失败**: 直接回 DISCONNECTED + 错误通知（不自动重连）
- **运行中断线**: 仅当 `wasEverConnected=true` 时进入 RECONNECTING
- **重连策略**: 指数退避 [2s, 4s, 8s, 16s, 30s]，最多 5 次
- **健康检查**: CONNECTED 状态下每 3s 检查一次，连续失败触发断线

**两个状态机的协作**:
```
DigitalRadioEngine (Facade)
  └─ EngineLifecycle (engineActor: 引擎状态机)
       └─ ResourceManager.startup() 按优先级启动资源
            └─ PhysicalRadioManager (radioActor: 电台状态机)
                 └─ RadioConnectionFactory → IRadioConnection 实现
```
- 引擎启动时 ResourceManager 按优先级启动资源，radio 是第一个
- 电台断线 → PhysicalRadioManager 通知 EngineLifecycle → 引擎强制停止
- 重连成功 → 恢复断线前的运行状态

### 电台控制能力系统（Radio Capability System）

统一管理电台可控参数（天调、发射功率、AF增益、静噪等），屏蔽 Hamlib/icom-wlan 实现差异。

**新增一个能力，需改动以下 6 处：**

1. **`IRadioConnection.ts`** — 新增可选方法签名（`getXxx?`, `setXxx?`）
2. **`HamlibConnection.ts` / `IcomWlanConnection.ts`** — 实现读写方法
3. **`RadioCapabilityManager.ts`** — 在 `CAPABILITY_CONFIGS`、`READ_MAP`、`WRITE_MAP` 三个静态表中注册
4. **`contracts/radio-capability.schema.ts`** — 在 `CAPABILITY_IDS` 中追加新 ID
5. **`web/src/radio-capability/capability-descriptors.ts`** — 新增静态描述符（category/valueType/range/i18nKey 等）
6. **`web/src/radio-capability/capability-registration.ts`** — 注册前端组件（`registerCapabilityComponent`）；若需新建组件，放在 `components/` 下

**i18n**：在 `locales/{zh,en}/radio.json` 的 `capability` 节点下添加 `label` 和 `description`（description 会在面板中以 tooltip 展示）。

**前端组件**：通用滑块用 `NumberLevelCapabilityPanel`（rf_power/af_gain/sql 已复用）；开关类用 `TunerCapabilityPanel` 参考；surface 控件（工具栏露出）需额外实现 surface component 并在注册时传入第三参数。

**权限**：写命令统一受 `execute:RadioControl` 保护，无需单独添加权限。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boybook/tx-5dr](https://github.com/boybook/tx-5dr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
