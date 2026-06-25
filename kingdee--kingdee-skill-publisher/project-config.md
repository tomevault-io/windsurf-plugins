---
trigger: always_on
description: 本文档面向AI编程助手，提供项目的技术细节和开发指南。
---

# AGENTS.md - 金蝶云苍穹业务Skill发布器

本文档面向AI编程助手，提供项目的技术细节和开发指南。

---

## 项目概述

**金蝶云苍穹业务Skill发布器**（Kingdee Skill Publisher）是一个元Skill（Meta Skill）工具，用于基于金蝶云苍穹开放平台的API快速创建和发布业务Skill。

### 核心价值

- **降低门槛**: 将复杂的API集成简化为几行代码
- **加速开发**: 从API到Skill，自动化生成所有必要代码
- **保证质量**: 生成遵循最佳实践的生产级代码
- **便于维护**: 自动生成完整的文档和示例

### 技术栈

| 组件 | 技术 | 说明 |
|------|------|------|
| 编程语言 | Python 3.7+ | 核心实现语言 |
| HTTP客户端 | requests | API调用 |
| 配置管理 | python-dotenv | 环境变量管理 |
| 数据序列化 | PyYAML | YAML文件处理 |

### 项目结构

```
kingdee-skill-publisher/
├── SKILL.md                          # 元Skill定义（Claude使用）
├── README.md                         # 项目总览和快速开始
├── PROJECT_SUMMARY.md                # 项目详细总结
├── LICENSE.txt                       # MIT许可证
├── AGENTS.md                         # 本文件
│
├── scripts/                          # Python核心模块
│   ├── __init__.py                   # 主入口，导出KingdeeSkillPublisher
│   ├── api_client.py                 # API客户端（Token管理、请求）
│   └── skill_generator.py            # Skill生成器
│
├── tests/                            # 测试文件
│   └── test_api_client_token.py      # Token管理单元测试
│
└── references/                       # 参考文档
    ├── kingdee_api_spec.md           # 金蝶API规范详解
    ├── API_QUERY_GUIDE.md            # API查询接口指南
    ├── USAGE_WITH_OFFICIAL_API.md    # 官方API使用指南
    └── examples.json                 # 使用示例和配置模板
```

---

## 核心组件

### 1. KingdeeSkillPublisher (`scripts/__init__.py`)

主协调类，管理整个Skill发布流程。

**主要方法**:

```python
# 配置和验证凭证
setup_credentials(server_url, app_id, app_secret, account_id, user) -> Dict

# 搜索API（使用官方queryByApp接口）
search_apis(appid_number, keyword=None, module=None, page_no=1, page_size=10) -> Dict

# 基于API创建Skill
create_skill_from_api(api_info, skill_name=None, skill_description=None) -> Dict

# 发布Skill
publish_skill(skill_dir) -> Dict
```

### 2. KingdeeAPIClient (`scripts/api_client.py`)

处理与金蝶OpenAPI的所有交互。

**主要方法**:

```python
# 获取/刷新Token
get_token(force_refresh=False) -> str

# 调用金蝶API
call_api(api_code, method="POST", params=None, form_id=None, retry_count=3) -> Dict

# 查询API列表（使用官方queryByApp接口）
get_api_list(appid_number, search_keyword=None, module=None, page_no=1, page_size=10) -> Dict

# 自动分页获取所有API
get_all_apis(appid_number, module=None, page_size=100) -> List[Dict]

# 验证凭证有效性
validate_credentials() -> Dict
```

**认证流程**:
1. 通过 `HEAD {baseUrl}/kapi` 验证服务器连通性
2. 调用 `POST {baseUrl}/api/login.do` 获取 `data.access_token`
3. 业务调用使用 `Authorization: Bearer {token}` 请求头
4. API列表查询使用 `accesstoken: {token}` 请求头

### 3. SkillGenerator (`scripts/skill_generator.py`)

根据API信息自动生成完整的Skill文件。

**主要方法**:

```python
# 生成完整Skill
generate_skill(api_info, custom_name=None, custom_description=None) -> str

# 生成各组件文件
_generate_skill_md(skill_dir, skill_name, api_info, custom_description)
_generate_api_call_py(skill_dir, skill_name, api_info)
_generate_api_schema(skill_dir, api_info)
_generate_error_codes(skill_dir, api_info)
_generate_examples(skill_dir, api_info)
_generate_readme(skill_dir, skill_name, api_info)
_generate_init_files(skill_dir)
```

**生成的Skill结构**:

```
${skill_name}/
├── SKILL.md                    # Skill定义
├── README.md                   # 使用说明
├── LICENSE.txt                 # 许可证
├── scripts/
│   ├── __init__.py
│   ├── api_call.py            # API调用主逻辑
│   └── api_client.py          # API客户端（复制）
└── references/
    ├── api_schema.json        # API参数定义
    ├── error_codes.json       # 错误码对照表
    └── examples.json          # 使用示例
```

---

## 配置与环境变量

### 必需的环境变量

```bash
export KINGDEE_SERVER_URL="https://xxx.kingdee.com/ierp"
export KINGDEE_APP_ID="your_app_id"
export KINGDEE_APP_SECRET="your_app_secret"
export KINGDEE_ACCOUNT_ID="your_account_id"
export KINGDEE_USER="admin"  # 可选，默认admin
```

### 依赖安装

```bash
pip install requests python-dotenv PyYAML
```

---

## 工作流程

### 完整工作流程

```
1. 配置凭证
   └── setup_credentials() 
       ├── 验证服务器连接
       ├── 获取Token
       └── 验证权限

2. 搜索API
   └── search_apis(appid_number, keyword)
       ├── 调用get_api_list()
       │   └── GET /kapi/v2/open/openapi_apilist/queryByApp
       └── 本地过滤结果

3. 创建Skill
   └── create_skill_from_api(api_info)
       └── SkillGenerator.generate_skill()
           ├── 生成SKILL.md
           ├── 生成scripts/api_call.py
           ├── 生成scripts/api_client.py
           ├── 生成references/*.json
           ├── 生成README.md
           └── 生成LICENSE.txt

4. 发布Skill
   └── publish_skill(skill_dir)
       ├── 验证文件完整性
       └── 返回发布指引
```

### 用户交互规则（重要）

在使用此Skill时，必须遵循以下交互规则：

1. **查询API前必须获取 `appid_number`**
   - 必须在对话框中询问用户提供应用编码
   - 如果用户未提供，停止执行后续任务

2. **返回API列表后必须等待用户选择**
   - 展示API列表并询问用户选择
   - 必须等待用户明确选择一个API后才能继续
   - 如果用户未选择，停止执行

---

## 代码风格指南

### Python代码规范

1. **文档字符串**: 使用Google风格的docstring
   ```python
   def method(self, param: str) -> Dict[str, Any]:
       """
       方法描述
       
       Args:
           param: 参数说明
           
       Returns:
           返回值说明
       """
   ```

2. **类型注解**: 使用Python类型注解
   ```python
   from typing import Dict, Any, Optional, List
   
   def func(param: str) -> Optional[Dict[str, Any]]:
       ...
   ```

3. **错误处理**: 使用try-except块捕获异常，返回友好的错误信息
   ```python
   try:
       result = api_call()
       return {'success': True, 'data': result}
   except Exception as e:
       logger.error(f"操作失败: {e}")
       return {'success': False, 'message': str(e)}
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kingdee/kingdee-skill-publisher](https://github.com/kingdee/kingdee-skill-publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
