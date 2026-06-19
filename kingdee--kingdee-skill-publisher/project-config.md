---
trigger: always_on
description: |
---


# 金蝶云苍穹业务Skill发布器

这是一个元Skill（Meta Skill），用于快速发布基于金蝶云苍穹开放平台的业务Skill。它简化了从API集成到Skill发布的全流程。

---

## ⛔ 强制执行规则（最高优先级）

> **本 Skill 的所有操作（凭证验证、API 查询、API 详情获取、Skill 生成）都必须通过执行 Python 脚本完成。AI 不得自行编写、猜测或拼凑任何业务文件内容。**

### 必须遵守

1. **凭证验证** → 必须执行 `publisher.setup_credentials()` Python 代码
2. **API 列表查询** → 必须执行 `publisher.search_apis()` Python 代码
3. **API 详情获取** → 必须执行 `publisher.api_client.get_api_detail()` Python 代码
4. **Skill 生成** → 必须执行 `publisher.create_skill_from_api()` 或 `SkillGenerator.generate_skill()` Python 代码
5. **所有 API URL** → 必须来自服务器 `getDetail` 接口返回的 `urlformat` 字段，禁止猜测或拼凑

### 绝对禁止

- ❌ **禁止 AI 自行编写 SKILL.md、api_call.py 等业务文件**（必须由 Python 脚本生成）
- ❌ **禁止 AI 猜测 API 的 URL 地址**（必须从服务器获取）
- ❌ **禁止 AI 猜测 API 的请求参数和返回参数**（必须从 getDetail 获取）
- ❌ **禁止跳过 Python 脚本直接用 write_to_file 创建 Skill 文件**

### 执行方式

所有 Python 代码必须通过 `execute_command` 工具执行，示例：

```bash
cd /path/to/kingdee-skill-publisher && python3 -c "
from scripts import KingdeeSkillPublisher
publisher = KingdeeSkillPublisher()
publisher.setup_credentials(server_url=..., app_id=..., app_secret=..., account_id=..., user=...)
result = publisher.search_apis(appid_number='xxx', keyword='yyy')
print(result)
"
```

---

## 核心功能

### 1. 凭证管理
- 引导用户配置金蝶服务器连接信息
- 存储和验证API认证凭证（user、appId、appSecret、accountId）
- 按 `login.do` 协议获取并缓存 `access_token`
- 在Token失效时自动重新获取

### 2. API探索与选择
- 自然语言查询金蝶API清单
- 支持按模块、功能、API编码搜索
- 展示API文档细节和参数信息

### 3. Skill自动生成
- 基于选定的API自动生成Skill
- 智能包装参数校验和错误处理
- 生成Skill的README和使用说明

### 4. 发布与测试
- 打包Skill文件（.skill格式）
- 生成测试用例
- 提供Skill安装指引

## 工作流

```
┌─────────────────────────────────────────┐
│ 1. 配置凭证与连接                          │
│    - Server URL                          │
│    - user / appId / appSecret / accountId│
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│ 2. 询问 appid_number                     │
│    - 必须询问用户提供应用编码              │
│    - 未提供则停止执行                      │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│ 3. 搜索API列表                            │
│    - 调用 queryByApp 获取API清单           │
│    - 展示API名称、编码、描述               │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│ 4. 用户选择API                            │
│    - 展示API列表供用户选择                 │
│    - 必须等待用户明确选择                  │
│    - 未选择则停止执行                      │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│ 5. 获取API详情 (getDetail)                │
│    - 使用API ID调用详情接口                │
│    - 获取请求头、请求参数、返回参数         │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│ 6. 封装Skill                              │
│    - 根据详情生成请求头处理代码            │
│    - 生成参数验证和响应解析逻辑            │
│    - 生成完整Skill结构                     │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│ 7. 测试与发布                             │
│    - 生成测试用例                         │
│    - 打包 Skill                           │
│    - 提供安装指引                         │
└─────────────────────────────────────────┘
```

## 快速开始

### 第一步：初始化配置

当用户开始使用此Skill时，引导进行以下操作：

**需要收集的信息（根据认证方式选择）：**

**经典认证（默认，auth_type="login_do"）：**
```
金蝶服务器配置：
├─ 服务器 URL（必须）
│  示例：https://xxx.kingdee.com/ierp
│
├─ appId（必须）
│  由金蝶开放平台第三方应用申请
│
├─ appSecret（必须）
│  需妥善保管，不应暴露
│
├─ accountId（必须）
│  租户账套ID
│
└─ user（可选，默认admin）
   当前操作用户
```

**增强型Token认证（auth_type="oauth2"）：**
```
金蝶服务器配置：
├─ 服务器 URL（必须）
│
├─ client_id（必须）
│  示例：${YOUR_CLIENT_ID}
│
├─ client_secret（必须）
│  AccessToken认证密钥，示例：${YOUR_CLIENT_SECRET}，需妥善保管
│
├─ username（必须）
│  用户名或手机号，示例：${YOUR_USERNAME}
│
├─ accountId（必须）
│  租户账套ID
│
└─ language（可选，默认zh_CN）
   语言设置
```

> `nonce` 和 `timestamp` 由客户端自动生成，无需配置。

**配置验证步骤：**
1. 检查服务器连通性
2. 根据 `auth_type` 调用对应的Token接口获取 `access_token`
3. 使用 `Authorization: Bearer {access_token}` 调用测试接口验证权限
4. 列出可用的API清单（验证查询能力）

### 第二步：探索与查询API（⚠️ 必须执行 Python 脚本）

> ⛔ 必须通过 `execute_command` 执行 `publisher.search_apis()` Python 代码查询，禁止 AI 自行构造 HTTP 请求。

本Skill使用金蝶官方的 `queryByApp` 接口按应用编码查询API列表，确保返回结果与当前应用实际发布的API保持一致。

**接口路径**: `GET /kapi/v2/open/openapi_apilist/queryByApp`

**⚠️ 重要：查询前必须获取 appid_number**

在调用API查询前，**必须在对话框中询问用户提供 `appid_number`**（应用编码）。

- 如果用户提供了 `appid_number`：继续执行查询
- **如果用户未提供 `appid_number`：停止执行，不再进行后续任务**

**询问示例**：
```
我需要查询API列表，请提供应用编码（appid_number）。
例如：basedata、ar、ap、pm 等
```

用户可以通过多种方式查询API：

**方式1：按应用编码获取完整列表**
- 用户必须显式传入 `appid_number`
- `pageNo` 默认值为 `1`
- `pageSize` 默认值为 `10`

**方式2：对结果做本地过滤**
- 关键词过滤：匹配 API 编码、名称、描述
- 模块过滤：基于返回记录中的编码或 URL 做最佳努力过滤

**查询返回信息：**
- API编码 (apiCode) - 用于API调用
- API名称 (apiName) - 中文描述
- 请求方式 (method) - GET/POST
- API描述 (apiDescription) - 功能说明
- 所属模块 (module) - ar/ap/pm等
- API版本 (version) - 版本号
- API状态 (status) - 发布/草稿等
- 请求参数 (requestParams) - 完整的参数定义
- 返回参数 (returnParams) - 返回字段定义
- 错误码 (errorCodes) - 业务错误码

**示例查询结果：**

```python
# 注意：appid_number 必须询问用户提供
result = publisher.search_apis(
    appid_number="basedata",  # 用户提供的应用编码
    keyword="凭证"
)

# 输出示例：
{
    "success": True,
    "total": 5,
    "apis": [
        {
            "apiCode": "ar_getBillDetail",
            "apiName": "应收凭证详情",
            "method": "GET",
            "module": "ar",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kingdee/kingdee-skill-publisher](https://github.com/kingdee/kingdee-skill-publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
