---
trigger: always_on
description: **接口地址**: `/api/v1/agent/count/status`
---

# Agent接口

## 获取在线和不在线的agent数量

**接口地址**: `/api/v1/agent/count/status`

**请求方式**: `GET`

**请求数据类型**: `application/x-www-form-urlencoded`

**响应数据类型**: `*/*`

**接口描述**: 获取Agent在线和离线数量统计

### 请求参数

暂无

### 响应状态

| 状态码 | 说明 | schema |
|--------|------|--------|
| 200 | OK | R«Map«string,int»» |

### 响应参数

| 参数名称 | 参数说明 | 类型 | schema |
|----------|----------|------|--------|
| data | 统计数据对象 | object | |
| flag | 请求是否成功 | boolean | |
| message | 响应消息 | string | |

### 响应示例

```json
{
  "data": {
    "online": 5,
    "offline": 2
  },
  "flag": true,
  "message": ""
}
```

## 通过agent id删除agent

**接口地址**: `/api/v1/agent/delete/{agentId}`

**请求方式**: `DELETE`

**请求数据类型**: `application/x-www-form-urlencoded`

**响应数据类型**: `*/*`

**接口描述**: 仅支持单一删除Agent

### 请求参数

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
|----------|----------|----------|----------|----------|--------|
| agentId | agent ID | path | true | string | |

### 响应状态

| 状态码 | 说明 | schema |
|--------|------|--------|
| 200 | OK | R«string» |

### 响应参数

| 参数名称 | 参数说明 | 类型 | schema |
|----------|----------|------|--------|
| data | 操作结果 | string | |
| flag | 请求是否成功 | boolean | |
| message | 响应消息 | string | |

### 响应示例

```json
{
  "data": "Agent删除成功",
  "flag": true,
  "message": ""
}
```

## 下载Agent

**接口地址**: `/api/v1/agent/download`

**请求方式**: `GET`

**请求数据类型**: `application/x-www-form-urlencoded`

**响应数据类型**:`*/*,application/java-archive`

**接口描述**: 下载iast-agent.jar

### 请求参数

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
|----------|----------|----------|----------|----------|--------|
| fileName | 文件名(agent或engine) | query | true | string | |
| serverUrl | 服务器URL(下载engine时必需) | query | false | string | |

### 响应状态

| 状态码 | 说明 | schema |
|--------|------|--------|
| 200 | OK | 文件流 |

### 响应参数

暂无

## 获取agent与服务器通信key

**接口地址**: `/api/v1/agent/get-agent-key`

**请求方式**: `GET`

**请求数据类型**: `application/x-www-form-urlencoded`

**响应数据类型**: `*/*`

**接口描述**: 获取Agent与服务器通信的密钥

### 请求参数

暂无

### 响应状态

| 状态码 | 说明 | schema |
|--------|------|--------|
| 200 | OK | R«string» |

### 响应参数

| 参数名称 | 参数说明 | 类型 | schema |
|----------|----------|------|--------|
| data | 通信密钥 | string | |
| flag | 请求是否成功 | boolean | |
| message | 响应消息 | string | |

### 响应示例

```json
{
  "data": "your-agent-key",
  "flag": true,
  "message": ""
}
```

## 查询所有agent列表

**接口地址**: `/api/v1/agent/list`

**请求方式**: `GET`

**请求数据类型**: `application/x-www-form-urlencoded`

**响应数据类型**: `*/*`

**接口描述**: 支持主机名、state、agent版本筛选

### 请求参数

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
|----------|----------|----------|----------|----------|--------|
| hostname | 主机名 | query | false | string | |
| state | 状态(0=离线,1=在线) | query | false | string | |
| version | agent版本 | query | false | string | |

### 响应状态

| 状态码 | 说明 | schema |
|--------|------|--------|
| 200 | OK | R«List«Agent实体»» |

### 响应参数

| 参数名称 | 参数说明 | 类型 | schema |
|----------|----------|------|--------|
| data | Agent列表 | array | Agent实体 |
| &emsp;&emsp;agentId | agentId | string | |
| &emsp;&emsp;cpuUsage | cpu使用率 | number(double) | |
| &emsp;&emsp;createTime | 创建时间 | string | |
| &emsp;&emsp;detectionStatus | 上线状态,1开启,2关闭 | integer(int32) | |
| &emsp;&emsp;hostname | 主机名 | string | |
| &emsp;&emsp;jdkVersion | jdk版本 | string | |
| &emsp;&emsp;lastActiveTime | 最后活跃时间 | string | |
| &emsp;&emsp;memoryUsage | 内存使用率 | number(double) | |
| &emsp;&emsp;os | 操作系统类型 | string | |
| &emsp;&emsp;process | 应用进程号 | string | |
| &emsp;&emsp;project | 所属项目信息 | 应用实体 | |
| &emsp;&emsp;&emsp;&emsp;id | id | integer | |
| &emsp;&emsp;&emsp;&emsp;level | 应用重要性 | integer | |
| &emsp;&emsp;&emsp;&emsp;name | 应用名 | string | |
| &emsp;&emsp;&emsp;&emsp;tag | 标签 | string | |
| &emsp;&emsp;&emsp;&emsp;userId | 用户ID | integer | |
| &emsp;&emsp;serverPath | 应用路径 | string | |
| &emsp;&emsp;state | 上线状态,1开启,2关闭 | integer(int32) | |
| &emsp;&emsp;version | agent版本 | string | |
| flag | 请求是否成功 | boolean | |
| message | 响应消息 | string | |

### 响应示例

```json
{
  "data": [
    {
      "agentId": "agent-123",
      "cpuUsage": 0.15,
      "createTime": "2023-06-15T08:30:00Z",
      "detectionStatus": 1,
      "hostname": "server-01",
      "jdkVersion": "1.8.0_341",
      "lastActiveTime": "2023-06-15T10:45:00Z",
      "memoryUsage": 0.45,
      "os": "Linux",
      "process": "12345",
      "project": {
        "id": 1,
        "level": 1,
        "name": "Sample Project",
        "tag": "web",
        "userId": 1
      },
      "serverPath": "/opt/tomcat/webapps",
      "state": 1,
      "version": "1.0.0"
    }
  ],
  "flag": true,
  "message": ""
}
```

## 分页查询所有agent

**接口地址**: `/api/v1/agent/list/page`

**请求方式**: `GET`

**请求数据类型**: `application/x-www-form-urlencoded`

**响应数据类型**: `*/*`

**接口描述**: 支持主机名、state、agent版本筛选的分页查询

### 请求参数

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
|----------|----------|----------|----------|----------|--------|
| pageNum | 页码 | query | true | integer(int32) | |
| pageSize | 每页数量 | query | true | integer(int32) | |
| hostname | 主机名 | query | false | string | |
| state | 状态(0=离线,1=在线) | query | false | string | |
| version | agent版本 | query | false | string | |

### 响应状态

| 状态码 | 说明 | schema |
|--------|------|--------|
| 200 | OK | R«PageInfo«Agent实体»» |

### 响应参数

| 参数名称 | 参数说明 | 类型 | schema |
|----------|----------|------|--------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keven1z/simpleIAST](https://github.com/keven1z/simpleIAST) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
