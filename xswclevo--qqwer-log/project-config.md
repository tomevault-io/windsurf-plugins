---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

企业级日志分析系统，采用单体架构（已从微服务架构简化），支持大规模日志数据的实时收集、存储、查询和统计分析。

**技术栈**:
- 后端: Java 21 + Spring Boot 3.2 + MyBatis Plus + Spring Security (JWT)
- 数据库: PostgreSQL 15 (元数据) + ClickHouse 23 (日志数据，使用 MWLOGDB_ANALYSIS 库)
- 前端: Vue 3 + TypeScript + Vite + Element Plus + ECharts

## 架构设计

### 双数据库架构

使用 MyBatis Plus 动态数据源管理两个数据库：

1. **PostgreSQL (postgres)** - 元数据库

2. **ClickHouse (clickhouse)** - 日志数据库

### 数据源切换

使用 `@DS` 注解指定数据源：

```java
// Mapper 级别 - PostgreSQL
@Mapper
@DS("postgres")
public interface UserMapper extends BaseMapper<User> {}

// 方法级别 - ClickHouse
@DS("clickhouse")
public Map<String, Object> queryLogs(LogQueryRequest request) {
    // 查询 syslog 表
}
```

### 模块结构

```
log-analysis-backend/
└── log-analysis-app/              # 单体应用
    ├── auth/                       # 认证授权模块 (JWT + Spring Security)
    │   ├── config/SecurityConfig  # Security配置，白名单在此配置
    │   ├── security/              # JWT过滤器、TokenProvider
    │   └── mapper/UserMapper      # @DS("postgres")
    ├── extraction/                 # 日志提取规则模块
    │   └── mapper/                # @DS("postgres")
    ├── alert/                      # 告警规则和事件模块
    │   └── mapper/                # @DS("postgres")
    ├── config/                     # 系统配置模块
    │   └── mapper/                # @DS("postgres")
    ├── stats/                      # 统计查询模块 (核心)
    │   └── service/StatsService   # @DS("clickhouse") 查询 syslog 表
    └── common/                     # 公共模块
        ├── response/Result         # 统一响应封装
        └── exception/              # 全局异常处理
```


## 关键开发规范

### Lombok 使用

- `@Data`: 所有实体对象
- `@RequiredArgsConstructor`: Service 构造器注入
- `@Slf4j`: 日志记录
- **禁止**: 字段注入 `@Autowired`，必须使用构造器注入

### 对象转换规范

**必须使用 MapStruct 进行对象转换**，禁止手动 set/get 赋值。

#### 使用场景

1. **Request DTO → Entity**: 接收前端请求，转换为实体对象
2. **Entity → Response DTO**: 查询数据库后，转换为响应对象
3. **Entity → Entity**: 不同层次的实体对象转换
4. **复杂对象映射**: 包含嵌套对象、集合等复杂结构

#### 正确示例

```java
/**
 * Datasource 对象转换器
 */
@Mapper(componentModel = "spring")
public interface DatasourceConverter {

    /**
     * Request → Entity
     */
    Datasource toEntity(CreateDatasourceRequest request);

    /**
     * Entity → Response DTO
     */
    DatasourceDTO toDTO(Datasource entity);

    /**
     * Entity List → DTO List
     */
    List<DatasourceDTO> toDTOList(List<Datasource> entities);

    /**
     * 更新 Entity（忽略 null 字段）
     */
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateEntity(UpdateDatasourceRequest request, @MappingTarget Datasource entity);
}
```

#### Service 层使用

```java
@Service
@RequiredArgsConstructor
public class DatasourceService {
    private final DatasourceMapper datasourceMapper;
    private final DatasourceConverter datasourceConverter;

    public Datasource createDatasource(CreateDatasourceRequest request) {
        // ✅ 使用 MapStruct 转换
        Datasource datasource = datasourceConverter.toEntity(request);
        datasourceMapper.insert(datasource);
        return datasource;
    }

    public DatasourceDTO getDatasource(String id) {
        Datasource entity = datasourceMapper.selectById(id);
        // ✅ 使用 MapStruct 转换
        return datasourceConverter.toDTO(entity);
    }
}
```

#### 错误示例（禁止）

```java
// ❌ 禁止手动赋值
public Datasource createDatasource(CreateDatasourceRequest request) {
    Datasource datasource = new Datasource();
    datasource.setName(request.getName());
    datasource.setType(request.getType());
    datasource.setHost(request.getHost());
    // ... 大量重复代码
    return datasource;
}
```

#### MapStruct 依赖配置

确保 `pom.xml` 中已配置 MapStruct：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.5.Final</version>
    <scope>provided</scope>
</dependency>
```

#### 规范说明

1. **类型安全**: 编译时检查，避免运行时错误
2. **代码简洁**: 自动生成转换代码，无需手动维护
3. **性能优越**: 生成的代码与手写代码性能相当
4. **易于维护**: 字段变更时只需修改 Converter 接口

### HTTP 接口规范

**所有后端接口统一使用 POST 方法**，不再使用 GET/PUT/DELETE/PATCH。

#### 规范说明

1. **统一性**: 前后端交互统一使用 POST，降低理解成本
2. **安全性**: POST 请求参数在 Body 中，避免 URL 暴露敏感信息
3. **灵活性**: POST 支持复杂请求体，不受 URL 长度限制
4. **简化**: 不需要区分 RESTful 语义，降低开发复杂度

#### 正确示例

```java
@RestController
@RequestMapping("/api/datasource")
@RequiredArgsConstructor
public class DatasourceController {

    private final DatasourceService datasourceService;

    /**
     * ✅ 创建数据源 - 使用 POST
     */
    @PostMapping("/create")
    public Result<Datasource> createDatasource(@Valid @RequestBody CreateDatasourceRequest request) {
        return Result.success(datasourceService.createDatasource(request));
    }

    /**
     * ✅ 查询数据源详情 - 使用 POST（而不是 GET）
     */
    @PostMapping("/get")
    public Result<DatasourceDTO> getDatasource(@RequestBody GetDatasourceRequest request) {
        return Result.success(datasourceService.getDatasource(request.getId()));
    }

    /**
     * ✅ 更新数据源 - 使用 POST（而不是 PUT）
     */
    @PostMapping("/update")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XSWClevo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
