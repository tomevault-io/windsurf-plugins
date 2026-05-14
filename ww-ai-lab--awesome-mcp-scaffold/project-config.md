---
trigger: always_on
description: Streamable HTTP 开发部署指南：基础 Streamable HTTP 设置、安全配置、容器化部署等
---

# Streamable HTTP 生产环境部署指南

## 概述
Streamable HTTP 是 MCP 推荐的生产环境传输协议，提供与负载均衡器和授权中间件的最佳兼容性，支持 HTTP2/H2C，并且具备内建 SSE 支持。

## 核心配置

### 1. 基础 Streamable HTTP 设置
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP(
    name="ProductionServer",
    stateless_http=True,  # 关键：启用无状态模式便于水平扩容
)

# 获取 FastAPI 应用实例
app = mcp.streamable_http_app()

# 生产环境运行
if __name__ == "__main__":
    mcp.run(
        transport="streamable-http",
        host="0.0.0.0",  # 监听所有接口
        port=8000,
        path="/mcp"      # 默认 MCP 端点路径
    )
```

### 2. 安全配置
```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],  # 生产环境限制来源
    allow_credentials=True,
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)

# 可信主机配置
app.add_middleware(
    TrustedHostMiddleware, 
    allowed_hosts=["yourdomain.com", "*.yourdomain.com"]
)
```

## 生产环境架构模式

### 1. 容器化部署
```dockerfile
# Dockerfile
FROM python:3.13-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY server/ ./server/
COPY config/ ./config/

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# 运行应用
CMD ["python", "-m", "server.main"]
```

### 2. Kubernetes 部署配置
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mcp-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mcp-server
  template:
    metadata:
      labels:
        app: mcp-server
    spec:
      containers:
      - name: mcp-server
        image: your-registry/mcp-server:latest
        ports:
        - containerPort: 8000
        env:
        - name: MCP_LOG_LEVEL
          value: "INFO"
        - name: MCP_CORS_ORIGINS
          value: "https://yourdomain.com"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

### 3. 负载均衡器配置
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mcp-server-service
spec:
  selector:
    app: mcp-server
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer

---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mcp-server-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
spec:
  tls:
  - hosts:
    - mcp.yourdomain.com
    secretName: mcp-tls-secret
  rules:
  - host: mcp.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mcp-server-service
            port:
              number: 80
```

## 监控和可观测性

### 1. 健康检查端点
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

@app.get("/health")
async def health_check():
    """基础健康检查"""
    return JSONResponse({"status": "healthy", "timestamp": datetime.utcnow()})

@app.get("/health/live")
async def liveness_check():
    """存活检查 - K8s liveness probe"""
    return JSONResponse({"status": "alive"})

@app.get("/health/ready")
async def readiness_check():
    """就绪检查 - K8s readiness probe"""
    # 检查依赖服务（数据库、Redis 等）
    dependencies_healthy = await check_dependencies()
    
    if dependencies_healthy:
        return JSONResponse({"status": "ready"})
    else:
        return JSONResponse(
            {"status": "not_ready", "reason": "dependencies_unhealthy"}, 
            status_code=503
        )
```

### 2. Prometheus 指标
```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# 定义指标
REQUEST_COUNT = Counter(
    'mcp_requests_total', 
    'Total MCP requests', 
    ['method', 'endpoint', 'status']
)

REQUEST_DURATION = Histogram(
    'mcp_request_duration_seconds',
    'MCP request duration',
    ['method', 'endpoint']
)

ACTIVE_CONNECTIONS = Gauge(
    'mcp_active_connections',
    'Number of active MCP connections'
)

# 中间件收集指标
@app.middleware("http")
async def metrics_middleware(request: Request, call_next):
    start_time = time.time()
    
    response = await call_next(request)
    
    # 记录指标
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.url.path,
        status=response.status_code
    ).inc()
    
    REQUEST_DURATION.labels(
        method=request.method,
        endpoint=request.url.path
    ).observe(time.time() - start_time)
    
    return response

# 启动 Prometheus 指标服务器
start_http_server(9090)
```

### 3. 结构化日志
```python
import structlog

# 配置结构化日志
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WW-AI-Lab/Awesome-MCP-Scaffold](https://github.com/WW-AI-Lab/Awesome-MCP-Scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
