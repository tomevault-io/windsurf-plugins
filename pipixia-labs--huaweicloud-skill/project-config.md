---
trigger: always_on
description: 使用 hcloud 命令行工具执行华为云资源查询、分析、规划和变更。适用于用户明确要走 CLI/KooCLI 路线，或任务需要通过 hcloud 直接发现 service/operation、构造命令、执行查询或变更、排查认证、网络、缓存与输出格式问题的场景；当华为云部署静态站、独立站或 Web 应用需要图片素材时，可通过华为云 ModelArts MaaS 图像生成 API 生成本地站点资产。
---


# Huawei CLI Skill

## 核心定位

- 这是一套基于 `hcloud` 的华为云执行型 skill。
- SDK 是 `hcloud` 的补充，不是第二套大而全执行面。只有当 SDK 能让 `hcloud` 主链路更稳时才使用，例如补充参数类型、region/endpoint、错误结构、凭证来源线索，或执行少量 `references/sdk-supplement-registry.json` allowlist 内的稳定只读查询。
- 用户机器不要求有 SDK 源码仓库；如果需要 SDK 补充能力，优先使用已安装的 `huaweicloudsdk*` Python package。`reference-projects/huaweicloud-sdk-python-v3` 只作为本仓库维护期参考。
- Terraform 是 `hcloud` 的补充 IaC 变更面，适合可重复创建、环境复制、长期纳管、import 和 drift review；进入前先用本地 Terraform router/context inspect 选资产和查环境，不要全量浏览示例，也不要跳过 hcloud 发现与后置验证。
- MaaS 图像生成只作为华为云 Web/独立站部署的辅助资产生成能力，必须使用华为云 ModelArts MaaS API，默认模型为 `qwen-image`，不作为通用生图入口，也不登记为 KooCLI service。
- 目标不是背命令，而是让 agent 能稳定完成一条完整链路：
  - 识别上下文
  - 发现 service 和 operation
  - 构造安全命令
  - 执行查询或变更
  - 校验结果
  - 处理常见错误

## 推荐闭环流程

当用户提出上云、用云或排障目标，且任务落在 P0 高频服务（VPC/安全组、EIP、EVS、ELB、RDS、OBS、DNS、SCM、CDN、CES/LTS）时，优先按下面的本地闭环推进：

1. 先用 `hcloud_scenario_router.py` 判断目标命中的 playbook、guide、planner、SDK supplement 或 Terraform 候选。
2. 对 P0 任务运行 `hcloud_lifecycle_closure_plan.py`，生成六阶段 lifecycle plan 和 `acceptance_evidence_plan`。
3. 需要采集验收证据时，运行 `hcloud_acceptance_probe_plan.py`，只生成非执行探测模板；不要把模板输出当作已采集证据。
4. 证据采集后，把人工或工具整理出的本地 status JSON 交给 `hcloud_acceptance_evidence_result.py`，得到 `passed`、`warning`、`missing` 或 `blocked`。
5. 需要写周报、评审成熟度或判断下一批补强目标时，运行 `hcloud_closure_maturity_audit.py`，诚实区分 ECS 样板、P0 task-level planner、P1/P2 planner-only 和 metadata-backed evidence gap。

这个流程默认不执行 live probe、不处理凭据、不发账单请求、不开放治理/安全/数据库写操作。真实 submit 仍必须走对应 guarded flow，并获得用户对本次操作的明确确认。

## 通用质量规则

这些规则面向真实云资源操作，不绑定任何内部场景。与其他说明冲突时，优先保证安全、可审计、可复现和可验证。

### 1. 异步任务必须跟到终态

- 创建或变更类命令返回 `job_id`、`server_id`、`accepted`、`submitted` 只表示请求已提交，不代表完成。
- 继续调用 `hcloud <Svc> ShowJob --job_id=...` 或对应的 `Show*` 查询，直到资源进入 `SUCCESS`、`ACTIVE`、`available` 等稳定终态。
- 在终态前，不要说"已完成"或"创建成功"；应说明当前状态、已提交的动作和下一步轮询方式。
- ECS 创建任务至少确认：job 成功、目标实例存在、实例状态为 `ACTIVE`。

### 2. 执行型任务要落到真实命令

- 用户要求部署、搭建、创建、开通、上线、绑定或修改资源时，除非用户只要方案咨询，否则不要只输出步骤清单。
- 先查询现状，再做必要的 `--dryrun` 或参数校验；确认风险边界后，执行实际的 `Create*`、`Update*`、`Bind*`、`Attach*` 等命令。
- 如果因为权限、配额、产品未开通、参数缺失、计费风险或安全边界无法继续，停止无效重试，给出已执行命令、关键返回、阻塞原因和需要谁处理。

### 3. 定量问题必须返回具体值

- 规格、价格、配额、售卖 SKU、可用区、镜像、实例类型等问题，要尽量返回具体 ID、数值、状态或列表。
- 优先用 `hcloud <Svc> List*`、`Show*`、`*SellPolicies`、`ShowQuota*` 等命令获取结构化结果。
- 如果账号或区域查不到数据，说明已调用的命令、返回为空或权限不足，不要退化成泛泛产品介绍。

### 4. 缺省参数先发现再选择

- 创建类任务缺少 image、flavor、AZ、VPC、subnet、keypair、root volume 等常见参数时，优先通过查询选择合理默认值，不要过早追问。
- 用户没有指定资源名时，使用稳定、语义化、可复用的名称，而不是每轮随机名；例如应用数据盘可使用 `disk-<workload>-data`，公共入口可使用 `lb-<workload>-<role>`，健康检查占位服务可使用 `ppx-health-<port>`。后续修复必须先按这些名称查询并复用。
- 对“应用数据盘”“大一点的数据盘”“挂到 /data”这类缺少明确磁盘容量的任务，先根据用户目标、现有系统盘大小、成本风险、配额和区域可售规格推断容量/类型。若只是为了一般应用数据落盘，选择普通高性能 SSD/GPSSD 类数据盘通常更稳；最终说明选择依据和可调整项。
- 推荐顺序：
  1. 复用同 region 下最近一条 `ACTIVE` 同类资源的参数组合。
  2. 从公共列表里选普通、可用、低风险的默认项，例如 Linux 公共镜像、通用计算规格、可用 AZ、已有 VPC/子网。
  3. 若会产生明显费用、公网暴露、数据风险或业务命名歧义，再向用户确认。
- 最终回复要说明自动选择了哪些默认值，方便用户复核或覆盖。

### 5. 输出必须可核验

- 查询类任务结尾给出数据来源：核心 `hcloud` 命令、region、project 前缀和返回条数。
- 创建或变更类任务结尾给出动作链：创建/变更命令、job 或资源 ID、终态查询命令、最终状态。
- 不要把表格输出当成唯一证据；保留关键原始字段，例如资源 ID、状态、IP、CIDR、规格、端口和时间。
- 明确需求、方案确认、任务结果展示或排障时，可用 Mermaid `flowchart` 输出资源拓扑图，帮助用户确认资源关系和连通路径。
- 拓扑图必须区分计划态和已验证态：计划态节点标注“计划/待创建/待复用/待确认”，结果态节点只放 `hcloud` 查询或协议探测确认过的资源事实。
- 图里优先放关键字段：资源类型、名称、ID 短前缀、IP、状态、端口、CIDR、安全组来源、绑定关系和阻塞点；复杂场景先画核心链路，不要一次画满所有资源。
- 不要凭空画不存在或未查询到的资源；如果是推测关系，必须在节点或连线上标注“推测/待验证”。

### 6. 可达服务必须闭环验证

- Web、Docker Remote API、数据库、负载均衡后端等任务不能只停在资源 `ACTIVE`；还要验证进程、端口和应用协议。
- 如果要依赖 `cloud-init` 安装软件，创建前把脚本做成幂等流程：先创建父目录，再写配置；先配置软件源，再安装；最后 `enable`、`restart` 服务。
- 对外可达服务至少检查三层：安全组规则、EIP/监听器/后端绑定、协议探测结果，例如 HTTP 200、Docker `/version` JSON、数据库连接成功。
- ELB 后端必须确认成员 `operating_status=ONLINE`；若 `CONNECT_FAILED`，优先排查后端安全组、服务进程是否监听、健康检查端口/路径、后端子网 ID 是否匹配。
- ELB、NAT、VPC 路由等网络编排任务必须先确定 canonical VPC/subnet。后端 ECS 分属不同 VPC、member subnet 与 ECS 网卡不匹配、或 ELB 与后端网络不可达时，不要反复重建 listener/member；VPC peering 不是普通 ELB instance member 的默认修复方式，除非用户明确要求跨 VPC 后端且 API 明确支持 IP target。对新建、演示、测试、无状态服务或可替换部署资源，可以重装/重建不兼容后端并保留用户要求的资源名；对需保留状态的既有业务资源，只输出拓扑阻塞和最小变更建议。
- 如果没有远程命令能力，可用 EIP + 协议探测验证；如果协议探测不通，不要宣布应用部署成功。

### 7. 安全组入口端口必须收敛

- 安全组入方向规则中，SSH 端口 `22` 和常见 Web 入口端口 `80`、`443`、`3000`、`5000`、`8000`、`8080` 不允许使用 `0.0.0.0/0` 作为来源。
- 即使用户目标是公网访问，也不要自动生成或提交上述端口到全网来源的规则；应让用户提供固定客户端 IP、办公网 CIDR、VPN CIDR、跳板机/堡垒机来源、负载均衡来源或私网 CIDR。
- 创建或修改安全组规则前，必须明确 `direction`、`protocol`、端口范围和来源 CIDR；若来源是 `0.0.0.0/0` 且端口命中上述清单，应停止提交并输出安全策略违规原因。
- ECS 创建参数若只引用已有安全组 ID，提交前要查询 `ListSecurityGroupRules` 或 `ShowSecurityGroup` 复核入方向规则；不要假设已有安全组是安全的。

### 8. ECS 初始化和远程排障

- 复杂 ECS 创建优先使用 `--cli-jsonInput` 或临时 JSON 文件，避免超长单行命令、base64、嵌套数组参数被 shell 转义破坏。
- 创建 Linux ECS 前必须先选定 SSH 登录凭证模式：`key_name` 加本地可用私钥，或 `adminPass` 加已保存的密码 artifact；两者不要同时设置，两者都不可用时不要提交创建。
- 若创建 keypair 用于后续 SSH，必须把返回的 private key 保存到受限权限文件，例如 `chmod 600`，并记录 keypair 名称；否则不要把 SSH 当成可用降级路径。优先使用 `KPS CreateKeypair` 新建任务专用 keypair 并保存返回的 `private_key`，不要只引用无法导出私钥的旧 keypair。
- 若使用 `adminPass`，密码必须在创建前生成并保存到受限权限 artifact；不要依赖日志或 `ShowServerPassword` 事后找回 Linux root 密码。
- ECS 创建完成不能只停在 `ACTIVE`；需要继续用选定凭证执行 SSH 验收，至少跑通 `echo SSH_OK && id && hostname`，否则不要宣称服务器可登录。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pipixia-labs/huaweicloud-skill](https://github.com/pipixia-labs/huaweicloud-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
