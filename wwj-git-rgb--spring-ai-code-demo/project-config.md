---
trigger: always_on
description: 本规则适用于DAS系统内所有REST API的开发工作，包括：
---

## 1. 文档说明

### 1.1 适用范围

本规则适用于DAS系统内所有REST API的开发工作，包括：
- 内部API（使用"/api"前缀）
- 对外OPEN API（使用"/openapi"前缀）
- 第三方集成API接口

### 1.2 约束强度分类

| 强度级别 | 关键词 | 说明 |
| -------- | ------ | ---- |
| **强制性** | MUST、SHALL、REQUIRED | 必须严格遵守，违反将导致API不符合规范 |
| **推荐性** | SHOULD、RECOMMENDED | 强烈建议遵守，有助于提升API质量 |
| **禁止性** | MUST NOT、SHALL NOT | 绝对禁止的行为或做法 |
| **可选性** | MAY、OPTIONAL | 可根据具体场景选择是否采用 |

### 1.3 使用指导

- 开发者**必须**在API设计阶段检查相关规则
- 代码审查时**应该**对照本规则进行检查
- 测试阶段**推荐**基于本规则制定测试用例
- 对于历史接口，**不建议**仅为遵循规则而进行重大变更

---

## 2. 基础原则

### 2.1 设计理念

- **一致性优先**：确保所有API遵循统一的设计模式
- **易用性导向**：让API简单直观，便于前后端协作
- **标准化实施**：遵循REST架构风格和HTTP协议标准
- **向后兼容**：保证API版本间的兼容性和稳定性

### 2.2 核心原则

- API**必须**遵循RESTful设计原则
- URL**必须**表示资源而非动作
- HTTP方法**必须**准确表达操作语义
- 响应格式**必须**保持结构化和一致性
- 错误处理**必须**提供明确的问题定位信息
- **必须**使用强类型设计，禁止使用Map<String, Object>
- **必须**使用统一的错误码枚举
- **必须**使用统一的请求和响应基类

---

## 3. R1 - URL与版本设计规则

### 3.1 URL结构规范

#### R1.1 基础结构【强制性】

**规则描述**：所有API的URL**必须**遵循以下结构格式

```
{schema}://{host}:{port}/{prefix}/{version}/{collection}[/{id}]
```

**字段说明**：
- `{schema}` - 协议（http或https）
- `{host}` - 服务域名或IP地址  
- `{port}` - 服务端口
- `{prefix}` - 接口标识前缀（内部接口使用"/api"，对外接口使用"/openapi"）
- `{version}` - 接口版本（格式：vX.Y）
- `{collection}` - 集合名称，必须为复数名词
- `{id}` - 资源唯一标识符

**正确示例**：
```http
https://das.com/api/v1.0/agents
https://das.com/api/v1.0/agents/JHF8UE6H5W34D
https://das.com/openapi/v2.1/devices/12345/sensors
```

**错误示例**：
```http
# 错误：使用单数名词
https://das.com/api/v1.0/agent
# 错误：缺少版本号
https://das.com/api/agents  
# 错误：使用动词
https://das.com/api/v1.0/getAgents
```

#### R1.2 URL长度限制【推荐性】

**规则描述**：URL长度**应该**不超过2000个字符

**检查方法**：在API设计时验证完整URL长度

#### R1.3 资源标识符规范【强制性】

**规则描述**：
- 资源标识符**必须**唯一且稳定
- **禁止**在标识符中使用"/"字符（如业务需要，必须使用"\"转义）
- **推荐**使用有意义的索引字段（IP、邮箱等）

### 3.2 版本控制规范

#### R1.4 版本格式【强制性】

**规则描述**：
- **必须**支持显式版本控制
- 版本号**必须**使用major.minor格式（如v1.0, v2.1）
- 版本定义**必须**在/api路径之后

**正确示例**：
```http
https://das.com/api/v1.0/agents
https://das.com/api/v2.1/devices
```

#### R1.5 版本兼容性【强制性】

**规则描述**：
- **必须**保留历史版本接口，除非确认无客户端使用
- 不更新版本的情况下**可以**新增属性
- **禁止**删除或修改现有属性

---

## 4. R2 - HTTP方法与状态码规则

### 4.1 HTTP方法使用规范

#### R2.1 方法语义映射【强制性】

**规则描述**：HTTP方法**必须**准确表达操作语义

| 方法 | 操作描述 | 成功状态码 | 幂等性 |
| ---- | -------- | ---------- | ------ |
| **GET** | 获取资源当前值 | 200 OK | ✓ |
| **POST** | 创建新资源或提交操作 | 201 Created | ✗ |
| **PUT** | 更新资源（完整替换） | 200 OK | ✓ |
| **PATCH** | 更新资源（部分更新） | 200 OK | ✗ |
| **DELETE** | 删除资源 | 204 No Content | ✓ |

**正确示例**：
```http
GET /api/v1.0/agents           # 获取探针列表
POST /api/v1.0/agents          # 创建新探针
GET /api/v1.0/agents/12345     # 获取特定探针
PUT /api/v1.0/agents/12345     # 完整更新探针
PATCH /api/v1.0/agents/12345   # 部分更新探针
DELETE /api/v1.0/agents/12345  # 删除探针
```

#### R2.2 PATCH方法特殊规范【强制性】

**规则描述**：对不存在资源的PATCH请求**必须**返回409 Conflict状态码

### 4.2 状态码使用规范

#### R2.3 标准状态码【强制性】

**规则描述**：**必须**使用标准HTTP状态码，**必须**返回精确对应的状态码

**2xx 成功状态码**：
- `200 OK` - GET、PUT、PATCH操作成功
- `201 Created` - POST创建资源成功  
- `202 Accepted` - 请求已接受，用于异步操作
- `204 No Content` - DELETE操作成功

**4xx 客户端错误**：
- `400 Bad Request` - 请求语法错误或参数无效
- `401 Unauthorized` - 未提供认证凭据或认证失败
- `403 Forbidden` - 已认证但无访问权限
- `404 Not Found` - 请求资源不存在
- `405 Method Not Allowed` - HTTP方法不被允许
- `409 Conflict` - 请求冲突（如PATCH不存在的资源）
- `422 Unprocessable Entity` - 请求格式正确但语义错误
- `429 Too Many Requests` - 请求频率超限

**5xx 服务器错误**：
- `500 Internal Server Error` - 服务器内部错误
- `503 Service Unavailable` - 服务暂时不可用

#### R2.4 异步操作状态码【推荐性】

**规则描述**：异步操作**应该**返回202状态码并提供任务跟踪信息

**示例**：
```http
HTTP/1.1 202 Accepted
{
  "task": {
    "href": "/api/v1.0/tasks/12345",
    "id": "12345"
  }
}
```

---

## 5. R3 - 请求响应格式规则

### 5.1 请求头规范

#### R3.1 标准请求头【推荐性】

**规则描述**：**推荐**使用以下标准请求头，使用时**必须**保持一致

| 请求头 | 类型 | 说明 |
| ------ | ---- | ---- |
| Accept | Content type | 请求的响应内容类型，推荐：application/json |
| Accept-Language | Language code | 响应语言偏好（如支持国际化） |  
| Accept-Charset | Charset type | 响应编码，默认UTF-8 |
| Content-Type | Content type | 请求体媒体类型（PUT/POST/PATCH） |

### 5.2 响应格式规范

#### R3.2 成功响应格式【强制性】

**规则描述**：
- 成功响应**必须**是单个JSON对象
- **必须**包含名为"data"的键
- 单个资源时data**必须**是JSON对象
- 多个资源时data**必须**是JSON数组（可为空数组）

**正确示例**：
```json
// 单个资源
{
  "data": {
    "agentType": "soc",
    "softVersion": "3.0.1",
    "config": "",
    "version": 1611054006000
  }
}

// 多个资源
{
  "data": [
    {
      "agentType": "soc",
      "softVersion": "3.0.1"
    },
    {
      "agentType": "ainta", 
      "softVersion": "1.1.3"
    }
  ]
}

// 空结果
{
  "data": []
}

// 操作结果
{
  "data": 1
}
```

#### R3.3 错误响应格式【强制性】

**规则描述**：
- 错误响应**必须**是单个JSON对象
- **必须**包含名为"error"的键
- error对象**必须**包含"code"和"message"字段

**Error对象结构**：
| 字段 | 类型 | 必需 | 说明 |
| ---- | ---- | ---- | ---- |
| code | String | ✓ | 服务端定义的错误代码 |
| message | String | ✓ | 用户可读的错误描述 |
| target | String | - | 错误的具体目标 |
| details | Error[] | - | 详细错误数组 |
| innerError | InnerError | - | 更具体的内部错误 |

**正确示例**：
```json
{
  "error": {
    "code": "BadArgument",
    "message": "Previous passwords may not be reused",
    "target": "password",
    "innerError": {
      "code": "PasswordError",
      "innerError": {
        "code": "PasswordReuseNotAllowed"
      }
    }
  }
}
```

#### R3.4 响应内容类型【强制性】

**规则描述**：
- 默认响应格式**必须**是application/json
- JSON属性名**必须**采用驼峰命名规范

---

## 6. R4 - 强类型设计规范

### 6.1 基础类型规范

#### R4.1 强类型设计【强制性】

**规则描述**：
- **禁止**使用Map<String, Object>作为参数或返回值

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wwj-git-rgb/spring-ai-code-demo](https://github.com/wwj-git-rgb/spring-ai-code-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
