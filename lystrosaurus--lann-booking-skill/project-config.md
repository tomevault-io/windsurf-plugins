---
trigger: always_on
description: 提供蘭泰式按摩（Lann Thai Massage）的门店查询、SPA服务查询和在线预约功能。支持泰式古法按摩、精油护理、草本热敷等专业服务预约，覆盖上海、杭州、成都等多城市门店。
---


# 蘭泰式按摩预约技能 (Lann Booking Skill)

## 概述

本 Skill 为 AI 助手提供蘭泰式按摩（Lann Thai Massage）的完整预约能力，包括：
- **门店查询**：检索全国 75+ 门店信息（名称、地址、电话、交通指引、经纬度）
- **服务查询**：检索 28+ 专业泰式按摩与 SPA 服务项目
- **创建预约**：通过 MCP 协议或直接 API 调用完成在线预约

**关键词**：泰式按摩、预约、SPA、Lann、lann、蘭、兰泰、古法按摩、精油护理、草本热敷

## 集成模式

本 Skill 支持三种调用模式，AI 应根据运行环境自动选择：

### 模式 1：MCP 客户端模式（推荐）
- **场景**：本地或远程部署 `lann-mcp-server`
- **连接方式**：
  - stdio：通过标准输入输出连接本地 MCP Server
  - streamableHttp：通过 HTTP 流式连接远程 MCP Server
- **优势**：符合 Model Context Protocol 标准，支持工具发现、类型安全

### 模式 2：远程 MCP 服务模式
- **场景**：直接连接已部署的 MCP 服务
- **Endpoint**：`https://open.lannlife.com/mcp`
- **协议**：streamableHttp
- **优势**：无需本地部署，开箱即用

### 模式 3：直连 API 模式（降级方案）
- **场景**：无 MCP 环境时的备选方案
- **API Endpoint**：`https://open.lannlife.com/mcp/book/create`
- **方法**：HTTP POST
- **Content-Type**：`application/json`
- **优势**：简单直接，兼容传统 HTTP 客户端

## 能力定义 (Intents)

### Intent 1: 查询门店 (query_stores)
**触发条件**：用户询问门店信息、附近门店、特定地区门店等

**支持的操作**：
- 列出所有可用门店
- 按城市筛选（上海、杭州、成都、深圳、苏州、武汉、宁波）
- 按关键词模糊匹配（门店名称、地址、地铁站）
- 获取单个门店详细信息（地址、电话、交通指引、经纬度）

**数据来源**：`org_store.json`（75 家门店）

**示例对话**：
- User: "上海有哪些门店？"
- User: "淮海路附近有门店吗？"
- User: "花木店的地址和电话是什么？"
- User: "离地铁 2 号线最近的门店是哪个？"

### Intent 2: 查询服务 (query_services)
**触发条件**：用户询问服务项目、按摩类型、SPA 内容等

**支持的操作**：
- 列出所有可用服务项目
- 按服务名称模糊匹配
- 按服务描述关键词搜索
- 按时长筛选（60分钟、90分钟、120分钟等）
- 按服务类型分类（古法按摩、精油护理、特色 SPA 等）

**数据来源**：`prod_service.json`（28 项服务）

**示例对话**：
- User: "有哪些按摩服务？"
- User: "90 分钟的服务有哪些？"
- User: "传统古法按摩包含什么内容？"
- User: "推荐一个缓解肩颈疲劳的服务"

### Intent 3: 创建预约 (create_booking)
**触发条件**：用户明确表达预约意图并提供必要信息

**必填参数**：
| 参数名 | 类型 | 校验规则 | 说明 | 示例 |
|--------|------|----------|------|------|
| mobile | string | `/^1[3-9]\d{9}$/` | 11 位中国大陆手机号 | `"13812345678"` |
| storeName | string | 必须与 `org_store.json` 中 `name` 字段完全一致 | 门店名称 | `"淮海店"` |
| serviceName | string | 必须与 `prod_service.json` 中 `name` 字段完全一致 | 服务项目名称 | `"传统古法全身按摩-90分钟"` |
| count | number | `1 <= count <= 20` | 预约人数 | `2` |
| bookTime | string | ISO 8601 格式，且晚于当前时间 | 预约开始时间 | `"2024-01-15T14:00:00"` |

**工作流程**：
1. **意图识别**：确认用户想要创建预约
2. **参数收集**：检查是否提供所有必填参数
   - 若缺少手机号：提示用户提供
   - 若缺少门店：调用 `query_stores` 展示可选门店，让用户选择
   - 若缺少服务：调用 `query_services` 展示可选服务，让用户选择
   - 若缺少人数：默认为 1 人，或询问用户
   - 若缺少时间：询问用户期望的预约时间
3. **参数校验**：
   - 验证手机号格式
   - 验证门店名称是否在 `org_store.json` 中存在（模糊匹配后需用户确认）
   - 验证服务名称是否在 `prod_service.json` 中存在（模糊匹配后需用户确认）
   - 验证人数范围（1-20 人）
   - 验证时间格式（ISO 8601）且晚于当前时间
4. **构造请求**：组装符合 API 规范的 JSON 请求体
5. **选择调用模式**：
   - 优先使用 MCP 模式（如果可用）
   - 降级使用直连 API 模式
6. **执行调用**：发送预约请求
7. **处理响应**：
   - 成功：返回预约 ID、时间、门店和服务信息
   - 失败：根据错误码给出友好提示

**示例对话**：
- User: "我要预约淮海店的传统古法全身按摩，2 人，明天下午 2 点，手机号 13812345678"
- User: "帮我预约花木店的泰式精油全身护理，1 人，后天上午 10 点"
- User: "我想预约一个肩颈按摩，有什么推荐？"

## 数据源说明

### 门店数据结构 (org_store.json)
```json
{
  "name": "门店名称",
  "address": "详细地址",
  "telephone": "联系电话",
  "traffic": "交通指引",
  "longitude": 经度,
  "latitude": 纬度
}
```

**关键字段**：
- `name`：**唯一标识符**，创建预约时必须与此字段完全一致
- `address`：详细地址，用于地理位置匹配
- `telephone`：门店电话，用于用户咨询
- `traffic`：交通指引，包含地铁线路和出口信息
- `longitude` / `latitude`：经纬度坐标，可用于距离计算

**门店分布**：
- 上海市：约 60+ 家（黄浦区、浦东新区、静安区、徐汇区、长宁区、闵行区、宝山区、普陀区、杨浦区、嘉定区、松江区、青浦区等）
- 杭州市：7 家（奥体印象城店、EFC 店、滨江银泰店、来福士店、杭州中心店、东站万象汇店、黄龙万科店）
- 成都市：4 家（万象城店、优品道店、太古里旗舰店、银泰中心 in99 店）
- 其他城市：武汉（1 家）、苏州（2 家）、深圳（1 家）、宁波（1 家）

### 服务项目数据结构 (prod_service.json)
```json
{
  "name": "服务名称",
  "desc": "服务描述"
}
```

**关键字段**：
- `name`：**唯一标识符**，创建预约时必须与此字段完全一致
- `desc`：服务详细描述，用于关键词匹配和用户理解

**服务分类**：
1. **传统古法按摩系列**（6 项）：90-120 分钟，推、拉、蹬、摇、踩等手法
2. **泰式精油护理系列**（7 项）：60-120 分钟，植物精油 + 泰式手法
3. **特色护理系列**（8 项）：椰香按摩、轻体 Spa、水光焕肤等
4. **快速/专项服务系列**（5 项）：肩颈版、精华版、深度拉伸等
5. **其他**：泊兰泰

## 工作流指南 (AI Assistant Workflow)

### Step 1: 意图识别
分析用户输入，判断属于以下哪种意图：
- **查询门店**：包含"门店"、"地址"、"电话"、"附近"、"哪里有"等关键词
- **查询服务**：包含"服务"、"按摩"、"SPA"、"项目"、"有什么"等关键词
- **创建预约**：包含"预约"、"预订"、"订一个"、"帮我约"等关键词

### Step 2: 信息补全
如果用户意图是创建预约，但信息不完整：

**缺少门店时**：
1. 读取 `org_store.json`
2. 根据用户提到的地区、地标、地铁线等关键词进行模糊匹配
3. 如果匹配到多个候选，列出前 5 个供用户选择
4. 如果未提到任何地区信息，询问用户期望的城市或区域

**缺少服务时**：
1. 读取 `prod_service.json`
2. 根据用户需求（如"缓解肩颈疲劳"、"全身放松"）匹配服务描述
3. 推荐 3-5 个相关服务，附上时长和简要说明
4. 让用户选择或进一步说明需求

**缺少时间时**：
1. 询问用户期望的日期和时间
2. 提供相对时间解析（如"明天下午 2 点"转换为 ISO 8601 格式）
3. 确保时间晚于当前时间

**缺少手机号时**：
1. 提示用户提供 11 位中国大陆手机号
2. 强调手机号用于门店联系确认

### Step 3: 参数校验
在调用 API 之前，必须完成以下校验：

**手机号校验**：
```python
import re
def validate_phone(mobile):
    return bool(re.match(r'^1[3-9]\d{9}$', mobile))
```

**门店名称校验**：
```python
def validate_store(store_name, stores):
    # 精确匹配
    if store_name in [s['name'] for s in stores]:
        return True, store_name
    # 模糊匹配
    candidates = [s['name'] for s in stores if store_name in s['name'] or s['name'] in store_name]
    if len(candidates) == 1:
        return True, candidates[0]
    elif len(candidates) > 1:
        return False, f"找到多个匹配门店：{', '.join(candidates)}，请指定具体门店"
    else:
        return False, f"未找到门店'{store_name}'，请使用以下门店名称之一：{', '.join([s['name'] for s in stores[:10]])}..."
```

**服务名称校验**：
```python
def validate_service(service_name, services):
    # 精确匹配
    if service_name in [s['name'] for s in services]:
        return True, service_name
    # 模糊匹配
    candidates = [s['name'] for s in services if service_name in s['name'] or s['name'] in service_name]
    if len(candidates) == 1:
        return True, candidates[0]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lystrosaurus/lann-booking-skill](https://github.com/lystrosaurus/lann-booking-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
