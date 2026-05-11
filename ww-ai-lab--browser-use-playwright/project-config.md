---
trigger: always_on
description: Web UI 开发规范，包含架构概览、项目结构、开发规范、样式开发规范等
---

# Web UI 开发规范

## 架构概览

### 技术栈
- **后端**: FastAPI + Pydantic
- **前端**: HTML5 + Bootstrap 5 + Vanilla JavaScript
- **模板引擎**: Jinja2
- **样式**: 本地Bootstrap + 自定义CSS

### 项目结构
```
src/web/
├── app.py                    # FastAPI主应用
├── template_manager.py       # 模板管理器
├── templates/               
│   ├── base.html            # 基础模板
│   ├── index.html           # 主页模板
│   ├── workflow_detail.html # 详情页模板
│   └── workflow_edit.html   # 编辑页模板
└── static/
    ├── css/
    │   ├── bootstrap-local.min.css  # 本地Bootstrap
    │   └── custom.css               # 自定义样式
    └── js/
        └── workflow-editor.js       # 编辑器脚本
```

## FastAPI 开发规范

### API路由设计
参考 [src/web/app.py](mdc:src/web/app.py) 的设计：

```python
# RESTful API 设计
@app.get("/api/workflows")                    # 获取工作流列表
@app.get("/api/workflows/{workflow_name}")    # 获取单个工作流
@app.put("/api/workflows/{workflow_name}")    # 更新工作流
@app.delete("/api/workflows/{workflow_name}") # 删除工作流
@app.post("/api/workflows/{workflow_name}/clean") # 清理工作流

# 页面路由
@app.get("/", response_class=HTMLResponse)                    # 主页
@app.get("/workflows/{workflow_name}", response_class=HTMLResponse) # 详情页
@app.get("/workflows/{workflow_name}/edit", response_class=HTMLResponse) # 编辑页
```

### 请求/响应模型
使用 Pydantic 模型定义API接口：

```python
class WorkflowUpdate(BaseModel):
    """工作流更新模型"""
    name: Optional[str] = None
    description: Optional[str] = None
    steps: Optional[List[Dict]] = None

class ApiResponse(BaseModel):
    """标准API响应"""
    success: bool
    message: str
    data: Optional[Dict] = None
```

### 错误处理
```python
from fastapi import HTTPException

try:
    workflow = recorder.load_workflow(workflow_name)
    if not workflow:
        raise HTTPException(status_code=404, detail="工作流不存在")
except Exception as e:
    logger.error("获取工作流失败", workflow_name=workflow_name, error=str(e))
    raise HTTPException(status_code=500, detail=str(e))
```

## 模板开发规范

### 模板继承结构
参考 [src/web/template_manager.py](mdc:src/web/template_manager.py) 的设计：

```jinja2
<!-- base.html - 基础模板 -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>{% block title %}Gaia RPA{% endblock %}</title>
    {% block extra_css %}{% endblock %}
</head>
<body>
    <nav><!-- 导航栏 --></nav>
    <main>{% block content %}{% endblock %}</main>
    {% block extra_js %}{% endblock %}
</body>
</html>

<!-- 子模板继承 -->
{% extends "base.html" %}
{% block title %}页面标题{% endblock %}
{% block content %}页面内容{% endblock %}
```

### 模板管理器使用
```python
from .template_manager import get_template_manager

# 初始化模板管理器
template_manager = get_template_manager()
template_manager.create_all_templates()
```

### 组件化设计
将复杂的UI组件拆分为可复用的模板片段：

```jinja2
<!-- 工作流卡片组件 -->
{% macro workflow_card(workflow) %}
<div class="card">
    <div class="card-body">
        <h5 class="card-title">{{ workflow.name }}</h5>
        <p class="card-text">{{ workflow.description or "无描述" }}</p>
        <div class="btn-group">
            <a href="/workflows/{{ workflow.name }}" class="btn btn-outline-primary btn-sm">
                <i class="bi bi-eye"></i> 查看
            </a>
            <a href="/workflows/{{ workflow.name }}/edit" class="btn btn-outline-secondary btn-sm">
                <i class="bi bi-pencil"></i> 编辑
            </a>
        </div>
    </div>
</div>
{% endmacro %}
```

## 前端JavaScript规范

### 模块化设计
参考 [src/web/static/js/workflow-editor.js](mdc:src/web/static/js/workflow-editor.js)：

```javascript
// 使用立即执行函数避免全局污染
(function() {
    'use strict';
    
    // 模块变量
    let stepCounter = 0;
    
    // 公共API
    window.WorkflowEditor = {
        init: init,
        addStep: addStep,
        saveWorkflow: saveWorkflow
    };
    
    function init() {
        stepCounter = document.querySelectorAll('.step-card').length;
        bindEvents();
    }
})();
```

### 异步请求处理
```javascript
async function saveWorkflow() {
    try {
        const response = await fetch(`/api/workflows/${workflowName}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(workflowData)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const result = await response.json();
        showSuccess('工作流保存成功！');
        
    } catch (error) {
        showError(`保存失败: ${error.message}`);
    }
}
```

### 事件处理
```javascript
// 使用事件委托处理动态元素
document.addEventListener('click', function(e) {
    if (e.target.matches('.btn-delete-step')) {
        handleDeleteStep(e);
    } else if (e.target.matches('.btn-move-up')) {
        handleMoveStepUp(e);
    }
});

// 防抖处理用户输入
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}
```

## 样式开发规范

### CSS组织结构
```css
/* 1. 重置和基础样式 */
/* 2. 布局相关 */
/* 3. 组件样式 */
/* 4. 工具类 */
/* 5. 响应式样式 */
```

### 命名规范
使用BEM命名方法论：

```css
/* 块(Block) */
.workflow-card { }

/* 元素(Element) */
.workflow-card__title { }
.workflow-card__description { }

/* 修饰符(Modifier) */
.workflow-card--featured { }
.workflow-card__title--large { }
```

### 响应式设计
```css
/* 移动优先 */
.workflow-grid {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WW-AI-Lab/browser-use-playwright](https://github.com/WW-AI-Lab/browser-use-playwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
