---
trigger: always_on
description: > 面向原创内容站长的自托管虚拟资源销售系统。
---

# Mercury — Vibe Coding Rules

> 面向原创内容站长的自托管虚拟资源销售系统。
> 资源商品交付类型：`file`（文件）/ `card`（卡密）/ `wiki`（在线知识库）  （积分支付购买）
> 现金充值业务：`points`（充值购买积分）/ `vip`（充值购买会员）  （微信/支付宝支付）
> 插件期功能（一期不做）：优惠券、签到、推广联盟、外链网盘、SaaS 插件市场。

---

## 1. 架构

**三层结构**

| 层 | 职责 |
|---|---|
| Kernel | 零业务逻辑基础设施：配置、JWT、网关、EventBus、WebSocket Hub、插件 KV 沙箱 |
| Modules | 核心业务：user / product / order / delivery / upload / wiki / ai / blockchain / settings |
| Plugins | 进程隔离外围扩展（HashiCorp go-plugin + gRPC），崩溃不影响主进程 |

**不可违反的约束**
- 使用 `uber-go/fx` 依赖注入，禁止全局 `init()` / 单例变量。
- 模块间禁止直接 `import`：异步用 `kernel/eventbus`，同步只读用内核共享接口（在 `kernel/auth` 中定义）。
- 多表写操作必须在 Service 层用 `db.Transaction`，杜绝扣款未发货。
- 所有用户可见文本（错误、label、推送）使用**简体中文**。
- 单文件不超过 300 行，按职责切割。

---

## 2. 目录结构

```
Mercury/
├── cmd/main.go                  # 唯一入口，仅做 fx.App 组装
├── kernel/
│   ├── auth/                    # JWT 生成/验证，Admin/Buyer 中间件，UserReader 共享接口
│   ├── config/                  # Bootstrap(YAML) + Runtime(DB 动态) 双层配置
│   ├── db/                      # GORM 初始化，自动建库，连接池
│   ├── redis/                   # Redis 客户端
│   ├── eventbus/                # Pub/Sub 事件总线（内存实现，可换 Redis）
│   ├── ws/                      # WebSocket Hub（订阅 EventBus，推送给在线用户）
│   ├── middleware/              # 限流、CORS、TraceID、审计日志、Panic恢复、Gzip
│   └── gateway/                 # 路由注册、插件注册表、插件 KV 代理
├── modules/
│   ├── user/                    # 注册/登录/资料/VIP状态/封禁；Admin：用户列表/手动授VIP
│   ├── product/                 # 资源商品CRUD+分类；PriceCalculator（查VIP专属积分价格）
│   ├── order/                   # 订单模块：微信/支付宝现金充值订单 (`recharge_orders`) + 积分消费兑换订单 (`points_orders`)
│   ├── delivery/                # 订阅 points_orders 的交付事件，编排卡密、文件或Wiki授权的派发
│   ├── upload/                  # 小文件上传 + 大文件三步分片（init/chunk/merge）
│   ├── wiki/                    # 空间/节点/阅读权限/DRM水印；Admin：空间和节点CRUD
│   ├── ai/                      # 向量化/RAG/SSE对话（可选，未配置则静默旁路）
│   ├── blockchain/              # 纯内核存证底座（定义 DDL 表、存证审计列表 API 及 Hook 事件分发，一期不开发链插件）
│   └── settings/                # system_settings 运行时配置读写；Admin：仪表盘/报表/审计日志
├── pkg/                         # 纯工具包（response/errcode/pagination/hash/validator）
├── sdk/proto/ + plugin_host/    # 插件 gRPC 契约
├── plugins/                     # 独立可执行插件进程
├── docs/                        # swag 生成的 Swagger
├── docker-compose.yml
└── go.mod
```

每个模块内部：`model/ → dto/ → repository/ → service/ → handler/`，依赖方向单向向下。

---

## 3. 数据库 Schema（19 张表，权威定义）

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE kernel_users (
    id             BIGSERIAL PRIMARY KEY,
    username       VARCHAR(64)  UNIQUE NOT NULL,
    email          VARCHAR(255) UNIQUE NOT NULL,
    password       VARCHAR(255) NOT NULL,
    role           VARCHAR(20)  NOT NULL DEFAULT 'buyer',  -- admin / buyer
    vip_level      INT          NOT NULL DEFAULT 0,
    vip_expire     TIMESTAMPTZ,
    points         BIGINT       NOT NULL DEFAULT 0,        -- 核心虚拟资产：积分余额
    is_banned      BOOLEAN      NOT NULL DEFAULT FALSE,
    wallet_address VARCHAR(128) UNIQUE,                    -- 预留 Web3 公钥
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE product_categories (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(64) NOT NULL,
    slug       VARCHAR(64) UNIQUE NOT NULL,
    sort_order INT         NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE products (
    id            BIGSERIAL PRIMARY KEY,
    category_id   BIGINT      REFERENCES product_categories(id) ON DELETE SET NULL,
    title         VARCHAR(255) NOT NULL,
    description   TEXT,
    price         BIGINT       NOT NULL,                   -- 价格单位：积分
    delivery_type VARCHAR(32)  NOT NULL,                   -- file/card/wiki (商品兑换只支持这三种类型)
    cover_url     VARCHAR(512),
    delivery_cfg  JSONB        NOT NULL DEFAULT '{}',
    -- delivery_cfg 规范：
    --   file:   { "file_id": "xxx" }
    --   card:   { "pool_id": "xxx" }  -- 仅支持单订单交付一张卡密，批量由插件扩展
    --   wiki:   { "space_id": 1, "days": 30 }  -- days=0为永久授权，days>0为到期天数
    validity_days INT          NOT NULL DEFAULT 0,         -- 0=永久，>0=购买后可再访问天数
    status        SMALLINT     NOT NULL DEFAULT 1,         -- 1=上架 0=下架
    is_consigned  BOOLEAN      NOT NULL DEFAULT FALSE,     -- 预留：代销标记
    is_original   BOOLEAN      NOT NULL DEFAULT FALSE,     -- 是否原创商品（勾选则需确认原创协议，承担侵权法律责任）
    original_stmt TEXT,                                    -- 原创责任声明承诺书文本快照（仅在 is_original=true 时有效）
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_products_fts ON products
    USING gin(to_tsvector('simple', title || ' ' || COALESCE(description,'')));

CREATE TABLE product_vip_prices (
    id         BIGSERIAL PRIMARY KEY,
    product_id BIGINT      NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    vip_level  INT         NOT NULL,
    price      BIGINT      NOT NULL,                       -- VIP专属价，单位：积分，0=免费
    UNIQUE(product_id, vip_level)
);

-- 现金充值订单（用户法币支付购买 积分 或 会员）
CREATE TABLE recharge_orders (
    id              BIGSERIAL PRIMARY KEY,
    order_no        VARCHAR(64)  UNIQUE NOT NULL,
    user_id         BIGINT       NOT NULL REFERENCES kernel_users(id),
    recharge_type   VARCHAR(20)  NOT NULL,                 -- points (买积分) / vip (买会员)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lip5201/Mercury-Digital](https://github.com/lip5201/Mercury-Digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
