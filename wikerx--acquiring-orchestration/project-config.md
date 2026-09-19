---
trigger: always_on
description: 当前项目 `acquiring-orchestration` 是跨境收单支付系统后端工程，包含商户 OpenAPI、收单支付、代付、渠道适配、后台管理、商户后台、网关、定时任务、公共组件等模块。
---

# AGENTS.md

# 跨境收单支付系统后端协作规范

## 项目定位

当前项目 `acquiring-orchestration` 是跨境收单支付系统后端工程，包含商户 OpenAPI、收单支付、代付、渠道适配、后台管理、商户后台、网关、定时任务、公共组件等模块。

本项目涉及资金安全、交易状态一致、幂等、防重放、签名加密、日志审计和多服务协同。所有 AI 编码助手在修改代码、文档或配置说明时，必须优先保障：

1. 安全
2. 资金准确
3. 状态一致
4. 幂等
5. 可追踪
6. 可维护

---

## 当前模块地图

### 公共组件

* `component-library/component-core`：统一返回模型、基础异常、认证上下文、密码与 token 工具。
* `component-library/component-web`：Web 通用配置、统一异常处理、操作日志切面、内部鉴权拦截。
* `component-library/component-db`：MyBatis Plus 公共实体、Mapper、认证与 RBAC 支撑、ISO 字典能力。
* `component-library/component-security`：OpenAPI JWT、签名、加密、密钥、重放保护工具。
* `component-library/component-redis`、`component-http`、`component-mq`、`component-job`：Redis、HTTP、RocketMQ、轻量级任务调度共享契约基础封装。

### 业务模块

* `channel-library`：渠道适配抽象，包含收单渠道和代付渠道。
* `service-gateway`：网关服务。
* `service-admin`：管理后台服务。
* `service-merchant`：商户后台服务。
* `service-checkout`：Hosted Checkout 收银台服务。
* `service-openapi`：商户开放接口入口服务。
* `service-payment`：收单支付核心服务。
* `service-payout`：代付核心服务。
* `service-job`：轻量级任务调度中心与定时任务服务。

### 当前真实背景说明

* `service-payment` 和 `service-payout` 当前仍偏骨架或模拟实现，不应误判为已经具备完整交易核心能力。
* 后续不能继续在模拟实现类中堆复杂支付或代付业务逻辑。
* 前端仓库独立维护，不在本规范的主要修改范围内。

---

## 修改前后输出要求

### 修改前必须说明

1. 准备修改的模块
2. 准备修改的文件
3. 修改原因
4. 风险点
5. 验证方式

### 修改后必须说明

1. 实际修改的文件
2. 影响范围
3. 风险点
4. 建议测试用例

---

## 总体原则

1. 不允许为了“代码好看”随意改变业务逻辑。
2. 不允许一次性大范围重构多个模块。
3. 不允许一次性格式化整个仓库。
4. 不允许未经明确要求修改外部接口字段、接口路径、签名规则、加密规则、状态码。
5. 不允许删除看似无用但可能被反射、配置、网关路由、定时任务、MyBatis、Spring 扫描使用的代码。
6. 支付系统优先保证安全、资金准确、幂等、状态一致、日志可追踪。
7. 不要生成大量 AI 风格的模板化代码和模板化注释。
8. 不要为小改动新增 Markdown 报告。
9. 不要为了统一目录而做与当前任务无关的大规模搬迁。
10. 不要把模拟实现继续演化成正式支付核心。

---

## 模块边界规则

### `service-openapi`

`service-openapi` 是商户开放接口入口，负责：

* 商户 JWT 鉴权
* 请求体解密
* 参数校验
* 防重放
* 商户基础权限校验
* 响应 `data` 加密
* 调用内部 `payment` / `payout` 服务

禁止：

* 在 OpenAPI Controller 中写支付核心业务
* 在 OpenAPI 层直接落支付交易主单
* 在 OpenAPI 层直接完成渠道扣款
* 绕过 `@VerificationAndProcessing` 新增商户对外 API
* 把完整卡号、CVV、密钥、JWT、私钥写入日志
* 返回未加密的成功 `data` 给商户侧接口

所有商户对外接口原则上必须：

* 使用 `POST`
* 走 `/api/rest/{domain}/{version}` 路径
* 使用 `@VerificationAndProcessing`
* 请求体加密
* 响应 `data` 加密
* 保留版本兼容能力
* 明确校验分组
* 明确幂等口径

### `service-payment`

`service-payment` 是收单支付核心服务，后续应承载：

* 支付交易主单
* 交易操作单
* 授权、请款、撤销、冲正、退款
* 渠道路由
* 风控编排
* 渠道请求与响应记录
* 交易状态机
* 幂等控制
* MQ 事件发布
* 对账、清分、结算所需基础数据

当前 `PaymentTransactionServiceImpl` 属于模拟实现，不允许继续在该类中堆完整支付核心逻辑。

后续应逐步拆分为：

* 交易受理应用服务
* 交易状态机
* 幂等服务
* 渠道路由服务
* 渠道调用服务
* 交易仓储服务
* 交易事件发布服务

### `service-payout`

`service-payout` 是代付核心服务，后续应承载：

* 代付申请
* 代付审核
* 代付渠道路由
* 代付状态机
* 渠道回调
* 退汇处理
* 幂等控制
* 代付流水与账务状态

当前代付实现偏模拟，不允许继续在模拟类中堆完整代付逻辑。

### `service-admin`

`service-admin` 是管理后台服务，新增管理端接口时优先使用：

```text
api
application
service
service.impl
dto
mapper
entity / DO
converter
```

不要再新增新的 `controller` 包。已有旧包如需调整，必须小步迁移，不得一次性大范围搬迁。

### `service-merchant`

`service-merchant` 是商户后台服务，后续风格应逐步对齐 `service-admin`：

```text
api
application
service
service.impl
dto
mapper
entity / DO
converter
```

已有 `controller` 包如被修改，可在任务范围内逐步迁移到 `api`，但不要为了统一目录大范围改动。

### `component-library`

`component-library` 只能放跨服务复用的基础能力，不允许变成业务垃圾桶。

可以放：

* 统一返回模型
* 基础异常
* 工具类
* Web 通用配置
* 安全加密组件
* Redis / MQ 基础封装
* MyBatis 基础配置
* 通用认证上下文

不应该放：

* 具体支付交易业务
* 某个渠道的特殊逻辑
* 某个后台页面的业务逻辑
* 某个商户功能的私有规则

### `channel-library`

`channel-library` 只放渠道适配抽象和通用模型。

禁止：

* 把平台交易状态机写在渠道库
* 把商户业务规则写在渠道库
* 把管理后台规则写在渠道库
* 直接暴露渠道原始响应给商户

渠道响应进入平台后，必须映射为平台统一状态、统一错误码、统一失败原因分层。

---

## 包结构和命名规则

优先使用：

```text
api
application
service
service.impl
dto
vo
entity
mapper
converter
config
support
security
client
```

不要随意新增：

```text
handler2
biz
manager
processor
helper
temp
test
new
old
```

除非有明确职责说明。

### 类型命名规则

```text
外部接口入参：xxxRequest
外部接口出参：xxxResponse
内部服务调用入参：xxxCommand 或 xxxClientRequestDTO
内部服务调用出参：xxxResult 或 xxxClientResponseDTO
查询条件：xxxQuery
后台页面展示：xxxVO
内部传输：xxxDTO
数据库实体：xxxDO
领域实体：xxxEntity
枚举：xxxEnum
```

禁止把 `DO` / `Entity` 直接作为外部接口入参或出参。

---

## 注释和文档规则

代码注释以 `docs/standards/coding-standard.md` 为准。该规范要求的类级 Javadoc 模板必须保留，
但模板中的 `@description` 必须写清真实职责、分层边界和关键约束，不能只写空泛描述。

### 注释允许与禁止

禁止继续生成：

* 每个字段都写无意义注释
* 每个构造器都写模板注释
* 每个简单 getter/setter 都写注释
* 缺少真实职责说明的模板头
* “当前负责衔接，后续可扩展”等无实际约束的套话
* 与代码命名完全重复的注释

允许保留或新增：

* 业务规则说明
* 支付状态流转说明
* 幂等设计说明
* 金额精度和舍入说明
* 加密/签名安全边界说明
* 兼容历史接口的原因
* 不允许删除或修改的特殊逻辑说明

### 代码注释粒度

* 每个生产类、接口、枚举、record 必须有符合 `coding-standard.md` 模板的类注释，说明业务职责和所在层级。
* `public` 方法必须有方法注释，说明用途、关键参数、返回值和关键副作用。
* 复杂私有方法、关键分支、安全边界、状态流转、数据编排必须补简洁说明。
* DTO、VO、DO 字段注释应说明业务含义、单位、格式、敏感性和是否允许为空；禁止只重复字段名。
* 修改代码时必须同步更新注释。

### 文档文件规则

* 不要为小改动新增散落 Markdown。
* 优先更新现有 `docs` 下最接近的文档。
* 不要生成“扫描报告”“修复报告”“临时总结”类文件，除非用户明确要求。
* 不要把 PRD 内容长期写进代码仓库的工程约束文档。

---

## OpenAPI 安全规则

商户对外 API 必须满足：

1. 必须使用 `POST`
2. 必须经过商户身份认证
3. 必须经过请求体解密
4. 必须经过参数校验
5. 必须经过防重放校验
6. 成功响应的 `data` 必须加密
7. 不允许返回敏感明文
8. 不允许绕过 `@VerificationAndProcessing` 新增商户接口
9. 不允许在日志打印完整请求体明文
10. 不允许打印完整卡号、CVV、JWT、`merchantKey`、私钥、API Key

当前 OpenAPI 安全相关类包括：

```text
@VerificationAndProcessing
OpenApiHeaderInterceptor
OpenApiRequestBodyAdvice
OpenApiResponseBodyAdvice
OpenApiPayloadDecoder
OpenApiRequestArgumentResolver
OpenApiJwtReplayProtectionService
MerchantJwtVerifier
OpenApiPayloadCrypto
MerchantSecurityService
```

修改这些类时必须单独说明影响范围和回归测试场景。

---

## 渠道回调和商户通知规则

渠道回调接口不能直接套用商户 OpenAPI 的 `@VerificationAndProcessing`，但必须有自己的安全机制。

渠道回调必须具备：

* 渠道维度签名校验
* 渠道 IP 白名单
* 回调原文保存
* 回调幂等
* 渠道订单号与平台订单号映射
* 状态流转校验

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wikerx/acquiring-orchestration](https://github.com/wikerx/acquiring-orchestration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
