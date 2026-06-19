---
trigger: always_on
description: 途牛旅行统一助手（推荐优先使用）- 通过 tuniu CLI 统一调用机票、酒店、门票、火车票、邮轮、度假产品等旅行服务。适用于用户询问航班、酒店、景点门票、火车票、邮轮以及跟团游、自助游、自驾游等度假相关需求的场景。【优先级说明】当同时安装了 tuniu-flight/tuniu-hotel/tuniu-ticket/tuniu-train/tuniu-cruise 等单独服务 skill 时，请优先使用本 skill，它整合了所有服务能力且调用方式更简洁。
---


# 途牛旅行助手

当用户询问航班、酒店、景点门票、火车票、邮轮、度假产品（跟团/自助/自驾/当地游等）等旅行服务时，使用此 skill 通过 tuniu CLI 调用途牛服务。

## 运行环境要求

本 skill 通过 **shell exec** 执行 **tuniu CLI** 命令调用途牛服务。**运行环境必须安装 tuniu-cli**，否则无法调用服务。

### 安装 tuniu-cli

```bash
# npm 全局安装（推荐）
npm install -g tuniu-cli@latest

# 或使用 npx 临时调用
npx tuniu-cli --version
```

## 配置要求

### 必需配置

- **TUNIU_API_KEY**：途牛开放平台 API key，用于认证

用户需在 [途牛开放平台](https://open.tuniu.com/mcp/login) 注册并获取上述密钥。

```bash
export TUNIU_API_KEY=your_api_key
```

## 速查表

### 意图识别（用户说什么 → 用什么工具）

| 用户意图关键词 | server | 首选工具 | 必填参数 |
|---------------|--------|----------|----------|
| 航班/机票/飞机 | `flight` | `searchLowestPriceFlight` | `departureCityName`, `arrivalCityName`, `departureDate` |
| 酒店/住宿/民宿 | `hotel` | `tuniu_hotel_search` | `cityName` |
| 门票/景点门票 | `ticket` | `query_cheapest_tickets` | `scenic_name` |
| 火车票/高铁/动车 | `train` | `searchLowestPriceTrain` | `departureCityName`, `arrivalCityName`, `departureDate` |
| 邮轮/游轮 | `cruise` | `searchCruiseList` | `departsDateBegin`, `departsDateEnd` |
| 度假/跟团/自助游/自驾游/旅游线路 | `holiday` | `searchHolidayList` | 无单一必填（建议 `keyWord` 和/或结构化条件；若传出游日期则 `departsDateBegin` 与 `departsDateEnd` 需成对） |

### 基本命令格式

```bash
tuniu call <server> <tool> -a '<JSON参数>'
```

| 参数 | 说明 |
|------|------|
| `server` | 服务名称：`ticket`、`hotel`、`flight`、`train`、`cruise`、`holiday` |
| `tool` | 工具名称，如 `query_cheapest_tickets`、`searchLowestPriceFlight` 等 |
| `--args` 或 `-a` | 工具输入参数，必须是合法的 JSON 字符串 |

**重要**：`--args` 的值必须是 JSON 格式，且用引号包裹。中文可直接写入，无需转义。无参数时用空对象：`-a '{}'`

### 服务工具链路

| 服务 | 完整流程（搜索→详情→下单） |
|------|---------------------------|
| `flight` | `searchLowestPriceFlight` → `multiCabinDetails` → `saveOrder` → `cancelOrder` |
| `hotel` | `tuniu_hotel_search` → `tuniu_hotel_detail` → `tuniu_hotel_create_order` |
| `ticket` | `query_cheapest_tickets` → `create_ticket_order` |
| `train` | `searchLowestPriceTrain` → `queryTrainDetail` → `bookTrain` → `cancelOrder` |
| `cruise` | `searchCruiseList` → `getCruiseProductDetail` → `getCruiseCabinAndRoom` → `saveCruiseOrder` |
| `holiday` | `searchHolidayList` → `getHolidayProductDetail` → `getHolidayBookingRequiredInfo`（可选，预订说明）→ `saveHolidayOrder` |

### 常用辅助命令

| 命令 | 用途 |
|------|------|
| `tuniu list` / `tuniu list <server>` | 列出服务/工具 |
| `tuniu help <server> <tool>` | 查看参数说明 |
| `tuniu schema --output json` | 获取完整 Schema |
| `tuniu discovery refresh && tuniu discovery list` | 检查新服务 |
| `tuniu call ... -d` | 调试模式 |
| `tuniu skill version` | 查看已安装 skill 版本 |
| `tuniu skill install [--agent/--dir]` | 安装/更新 skill 到指定 Agent 或目录 |

---

## 服务发现触发条件

当遇到以下情况时，**必须**先执行 `tuniu discovery refresh && tuniu discovery list`：

1. **用户需求不在已知服务列表中**（如签证、租车、度假套餐等）
2. **tuniu list 返回的服务不包含用户需要的功能**
3. **工具调用返回"工具不存在"错误（退出码 102）**
4. **首次使用 tuniu-cli 时**（确保获取最新服务列表）

```bash
tuniu discovery refresh && tuniu discovery list
```

执行后重新检查服务列表，再决定下一步调用。若仍无法满足用户需求，才告知用户当前平台暂不支持该功能。

---

## Skill 版本与更新说明

`tuniu-cli` 提供 **skill** 子命令，用于维护本助手在各 AI Agent 目录下的安装与版本查看，与业务调用（`tuniu call`）相互独立。

**使用场景简述**

- **`tuniu skill version`**：在已配置多台 Agent（如 Cursor、Claude 等）时，检查各目录下已安装的 skill 版本、来源与安装时间；便于确认是否与文档站最新包一致。
- **`tuniu skill install`**：需要**安装或更新**本 skill 时使用。默认仅写入 `~/.agents/skills/tuniu-cli/`；通过 `--agent` 可指定单个、多个（逗号分隔）或 `all`（全部内置支持的 Agent）；`--dir` 可额外指定自定义 skills 根目录。
- **`npm install` / `npm ci`**：安装 `tuniu-cli` 时若启用脚本，**postinstall** 可能已根据本机存在的 Agent 父目录自动复制内置 skill；若需与线上一致或显式更新，仍建议执行 `tuniu skill install`。

更完整的参数与示例见：`tuniu skill install --help`。

---

## 隐私与个人信息（PII）说明

预订功能会将用户提供的**个人信息**（联系人姓名、手机号、乘客姓名、证件号等）通过 tuniu CLI 发送至途牛远端服务，以完成订单创建。使用本 skill 即表示用户知晓并同意上述 PII 被发送到外部服务。请勿在日志或回复中暴露用户个人信息。

## 适用场景

- 机票搜索、舱位查询、机票预订
- 酒店搜索、详情查询、酒店预订
- 景点门票查询、门票预订
- 火车票车次查询、车次详情、火车票预订
- 邮轮产品搜索、团期查询、邮轮预订（兼容"游轮"说法）
- 度假产品搜索、团期价格日历、度假预订（兼容跟团、自助游、自驾游、当地游等表述）
- **动态服务发现**：当用户旅行需求超出上述服务范围时，通过 discovery 功能检查是否有新服务上线

## 动态服务发现

途牛 CLI 支持动态发现新服务。**触发条件见上方 服务发现触发条件 章节**，满足条件时执行：

```bash
tuniu discovery refresh && tuniu discovery list
```

**服务发现默认开启**。如不确定，可先执行 `tuniu discovery status` 确认；若返回 `启用: 否`，手动开启：

```bash
export TUNIU_DISCOVERY_ENABLED=true
```

| 命令 | 用途 |
|------|------|
| `tuniu discovery status` | 查看启用状态、缓存状态、服务数量 |
| `tuniu discovery list` | 获取当前可用服务列表（失败时回退静态配置/缓存） |
| `tuniu discovery refresh` | 强制刷新缓存，获取最新服务列表 |

> 工具调用返回退出码 102 时，先执行 `tuniu discovery refresh && tuniu schema --output json`，再重试调用。

### 最佳实践

1. **初始化时**：执行 `tuniu discovery status` 确认服务发现状态（默认开启）
2. **遇到新需求时**：先执行 `tuniu discovery refresh` 刷新缓存，再 `tuniu discovery list` 查看最新服务
3. **获取新服务能力**：执行 `tuniu schema --output json` 获取最新工具定义
4. **降级处理**：如果 discovery 服务不可用，会自动回退到静态配置

## 各服务详细说明

### 1. 机票服务 (flight)

**触发词**：航班、机票、飞机、某地到某地航班、查机票、机票价格

#### 1.1 航班搜索 (searchLowestPriceFlight)

**支持 6 种查询模式**：
- **默认低价查询**：不传 searchType
- **TIME 时间范围查询**：searchType="TIME"，按出发/到达时间筛选
- **PRICE 价格区间查询**：searchType="PRICE"，按价格区间筛选
- **NEAR_GO 周边出发**：searchType="NEAR_GO"，查询出发地周边机场
- **NEAR_BACK 周边到达**：searchType="NEAR_BACK"，查询目的地周边机场
- **TRANSFER 中转查询**：searchType="TRANSFER"，查询中转航班

**必填参数**：`departureCityName`、`arrivalCityName`、`departureDate`（YYYY-MM-DD）

**翻页**：传相同城市日期参数 + `pageNum`（2=第二页，3=第三页…）

```bash
# 默认低价查询

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuniucorp/tuniu-cli](https://github.com/tuniucorp/tuniu-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
