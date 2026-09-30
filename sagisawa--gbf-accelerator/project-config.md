---
trigger: always_on
description: > 本规范为本项目最高约束与唯一事实来源 (Canonical Source of Truth)。
---

# GBF-Accelerator 核心架构与安全治理规范 (AGENTS.md)

> 本规范为本项目最高约束与唯一事实来源 (Canonical Source of Truth)。
> 改动前必须对照自查; 违背 P0 规则或测试诚信法则属严重故障, 必须无条件立即回滚。

---

## 0. 项目核心定位与叙事原则

1. **定位与架构**:
   - 定位: **高性能本地静态资源缓存与透明代理工具**。
   - 架构: v2.0 **Go 原生单静态二进制** (零 Python 运行时依赖)。
   - 核心 (`engine/`): `proxy` (转发/静态命中), `cache` (RAM LRU + 磁盘持久化), `control` (控制面), `telemetry` (指标/日志), `updater` (自更新); 适配层: `cert`, `sysproxy`, `startup`, `desktop`。
   - 前端: React SPA 内嵌 `engine/ui`。历史 Python 实现已废除, 规范以 Go 源码为准。
2. **目标与协议分流**:
   - 本地缓存 + HTTP/2 多路复用加速静态素材; 温和调度削峰填谷, 降低突发并发与 CDN 负载。
   - HTTP/2 仅限静态 CDN 上游, 禁改协商行为; 动态 API 严格按原协议透明转发, 当前上游基线为 HTTP/1.1 Keep-Alive 池。
3. **合规红线**:
   - 严禁宣称"100%不封号"、"免除官方处罚"或"规避风控检测"; 严禁表述为"抹平特征"或"对抗检测"。
   - 动机严格立足"工程减负、流量削峰填谷、透明稳定"。
4. **文风准则**:
   - 言简意赅、客观严谨; 严禁营销夸张词 ("彻底解决"、"绝对零阻塞"、"100%保证"、"起飞"、"全网最强"、"秒杀"、"完美"); 坚持中立工程术语与量化指标。

---

## P0 级: 不可逾越的绝对安全红线 (Violations are Critical Bugs)

### 1. 业务语义绝对透明 (Business Semantic Transparency)
- **转发范围与协议基线**: 非静态请求 (`/rest/`, `/quest/`, `/party/`, `/user/`, `/deck/`, `/gacha/`, `/casino/`, `/mypage/` 等) 经专用 `api_client` 透明转发。当前动态 API 上游基线为 HTTP/1.1 Keep-Alive；任何未来协议栈调整必须经过专项审计与 benchmark，并且不得改变业务语义、重试规则和透明转发约束。
- **业务零干预**: 除逐跳头 (`Connection`, `Transfer-Encoding` 等), **严禁修改上游状态码、实体正文、Cookie 或业务 Header**。
- **Content-Encoding 限制**: 仅底层解压且客户端无法解码时技术剔除, 保证解压字节语义一致, 严禁扩大篡改。
- **严禁 Mock 与动态缓存**: `MOCK_PATHS = ()` 保持为空, 严禁构造本地伪造 200; 动态响应严禁写入磁盘或 RAM 缓存。

### 2. 官方探测绝对穿透
- `/ob/r` (反作弊心跳) 与 `/rest/error/js` (前端错误上报) 作为标准动态 API 100% 穿透 Cygames; 严禁本地拦截/丢弃/伪造。

### 3. 双重约束安全重试机制 (Dual-Constraint Safe Retry)
- **POST/PUT/DELETE 坚决零重试**: 写请求 (攻击/技能/召唤/体力等) `max_attempts = 1`, **绝对禁止自动重试**, 杜绝"技能双发 / 状态不一致"。
- **GET 重试双重限制**:
  1. 仅限预审只读幂等接口 (`RETRYABLE_API_PATHS` 白名单);
  2. 仅建连前空闲 TCP 断开或 Stale Connection (`ConnectError`, `RemoteProtocolError`) 时允许最多 1 次静默快速重连。

### 4. 响应头零指纹污染 (Zero Header Pollution)
- 严禁向客户端返回自定义代理头 (`X-Proxy-Cache`, `X-Cache-Source`, `X-Acceleration-*`)。
- 动态响应头经 `forward_upstream_response` 原样还原, 严格多行保留每条 `Set-Cookie`, 禁逗号折叠合并。

### 5. 静态资源防篡改与缓存完整性 (Byte-for-Byte Integrity)
- 磁盘与 RAM 缓存素材 (`.js`, `.css`, 图频等) 必须为 Akamai CDN 原始字节流; 严禁注入作弊/挂机/DOM 脚本。
- 补丁清理 (Quarantine) 限"结构化空函数 (`void 0` / 空体) + 异常上下文"精准特征, 严禁误伤合法 `void 0`。

---

## P0 级: v2.0 架构核心不变量 (Network Plane & Transaction Invariants)

### 1. 双平面拓扑与 AllowLAN 边界
- **8124 Data Plane**:
  - `AllowLAN=false` 绑 `127.0.0.1`; `AllowLAN=true` 绑 `0.0.0.0`。
  - 负责代理流量, 向 LAN 提供 `/ca.crt`, `/proxy.pac` 与移动端引导页。
- **8125 Control Plane**:
  - **永远仅绑 `127.0.0.1` (Strictly Loopback)**。
  - 管理接口 (`/api/config/apply`, `/api/cache/*`, `/api/cert/*`, `/api/sysproxy/*`, `/api/startup/*`, `/api/update/*` 等) 仅限 Loopback。
  - 严禁因 `AllowLAN=true` 改绑非 Loopback; 严禁改 Origin 或白名单向 LAN 暴露 8125。
- **AllowLAN 语义**:
  - 仅控制 8124 是否接受非 Loopback 连接; 绝不开放 Control Plane 与后台。
  - `AllowLAN` 本身不得自动修改系统防火墙；Windows 防火墙若需适配，仅允许通过独立、用户明确触发的防火墙操作配置。
  - 防火墙适配必须严格限定 `Private + LocalSubnet + TCP + 当前代理端口 + Inbound + Allow`，不得开放 Public 或 Any。

### 2. Listener 生命周期与热重载
- **重绑顺序**:
  - 同端口变更: 串行 `Candidate -> Close old -> Listen new -> (成功 switch gen / 失败 rollback old)`, 禁未释放重绑同 TCP 地址。
  - 不同端口: `Listen new -> 成功 -> retire old` 缩短中断。
- **Generation 保护 Accept Loop**:
  - 每个 `serveLoop` 绑定唯一 `generation`; `gen != currentGeneration` 退役立即退出, 禁继续 accept。
  - 关闭旧 gen listener 属正常生命周期, 禁记高等级异常。
- **Anti-Spin 退避机制**:
  - 当前 gen 在 `Accept()` 遇临时错误受控退避 (默认 ~20ms), 禁无延时 busy spin; 修改策略须有测试/Benchmark 依据。
- **保护 Established 连接**:
  - `net.Listener.Close()` 仅影响未来 `Accept()`; 已建立 `net.Conn` (Established 连接) 严禁因 reload 或 gen 切换中断, 须由原 handler 跑完生命周期。
  - 新增热重载必须包含 existing-connection 回归测试。
- **Reload API 恢复性**:
  - 覆盖新绑失败、旧回滚失败、并发 Stop/Reload; 新绑失败旧 listener 尽可能恢复, 禁新旧双活。
  - 回滚失败记明确 high-level 错误, 禁静默返回 success。

### 3. 配置事务模型 (Configuration Transaction)
- **Candidate 数据隔离**:
  - 遵循 `Current Config -> Candidate copy -> Patch Candidate`。
  - Candidate 为纯内存快照, 禁直接修改生效配置、Listener、系统代理、自启动或缓存状态。
- **网络配置强事务顺序**:
  - 涉及 `allow_lan`, `listen_port`, `control_port` 严格遵循:
    `Candidate -> Config Commit / Save -> Network Rebind -> Post-Commit Runtime Sync`。
  - 先持久化配置再修改 listener；任一 listener 重绑失败必须回滚配置与已重绑 listener；严禁磁盘与运行时分裂 (`config.json = new, runtime = old`)。
- **Commit() 与 Update() 语义分离**:
  - `Commit(candidate)` 为强事务主路径 API; `Update(fn)` 仅历史兼容, 禁在网络事务主路径使用。
- **commitMu 锁作用域与禁止回调重入**:
  - 保证并发 Commit 串行, 禁交错 Save 与回滚覆盖。
  - **关键约束**: 严禁持有 `commitMu` 时执行可能重入配置系统的外部回调 (callback)。
  - 标准模式: `lock -> update memory -> save -> snapshot callbacks -> unlock -> invoke callbacks` (Callback 默认不得持有内部锁)。
- **Post-Commit 运行时同步**:
  - 系统代理、自启动、缓存配置 (`base_dir` / `ram_cache_limit_mb`) 与注册表属 Post-Commit best-effort sync。
  - 提交成功后执行, 失败不破坏已成网络事务亦不伪装成功; 日志明确区分 Commit success 与 Runtime Sync failure。

---

## P1 级: 资源调度与网络负载控制 (Resource & Scheduling Guardrails)

### 1. Prefetch 调度层平滑与主动避让
- **后台异步与调度平滑**:
  - Prefetch 完全在后台 worker 异步执行, 严禁侵入前台主请求管线。
  - 调度器 (Scheduler / Pacer) 分发任务, 基线采用 **15~35ms 随机抖动平滑**削峰填谷, 严禁循环硬编码机械 `sleep(ms)`。
  - 低负载或空队列零延迟; 调整策略须附 Benchmark 证明不加重 CDN 负担。
- **动态避让**:
  - 当存在活动中的前台动态 API 或前台静态资源请求时，Prefetch 主动暂停，避免与前台请求竞争 CPU、磁盘 I/O 和上游连接资源。

### 2. 连接池保守水线 (Connection Pool Conservatism)
- 静态客户端 `asset_client` 基于 HTTP/2 多路复用, 连接池基线保持 `asset_max_connections <= 32`, `asset_max_keepalive <= 16` 保守水线。
- 调高上限须提供充分基准测试报告, 证明不造成 TCP 握手风暴、连接抢占或前台 API 延迟退化。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sagisawa/GBF-Accelerator](https://github.com/Sagisawa/GBF-Accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
