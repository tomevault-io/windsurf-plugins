---
trigger: always_on
description: > OpenCode 会话专属的精简指南。单一文件即可完成 ramp-up。
---

# SPMS-Server OpenCode 指南

> OpenCode 会话专属的精简指南。单一文件即可完成 ramp-up。

## 1. 项目一句话

基于 **Spring Boot 3 + JPA + MySQL + Redis + MQTT + InfluxDB** 的智能生产管理后端（MES/WMS/ERP/IoT），强依赖 [
`cn.hamm.airpower`](https://github.com/AirPowerTeam/AirPower4J) 6.5.0 父 POM 提供的 CURD / WebSocket / MCP / 字典 /
异常体系。单 Maven 模块，无 monorepo、无 submodule、无 CI。

## 2. 技术栈

| 技术                    | 版本            | 说明                                             |
|-------------------------|-----------------|--------------------------------------------------|
| Java                    | 17              | 已通过 `pom.xml` 父 POM 锁定                     |
| Spring Boot             | 3.x             | 由 airpower 父 POM 带入                          |
| Maven                   | 3.9+            | 使用 `./mvnw`（已含 wrapper），勿依赖系统 `mvn`  |
| AirPower                | 6.5.0           | 父 POM，几乎所有能力都从 `cn.hamm.airpower.*` 取 |
| JPA/Hibernate           | Spring Data JPA | `ddl-auto` 见 §4 "环境"                          |
| MySQL                   | 8.x             | 库名 `spms`                                      |
| Redis / MQTT / InfluxDB | -               | 缓存 / 物联网上行 / 时序数据                     |

## 3. 常用命令

```bash
./mvnw -s settings.xml clean package -DskipTests   # 编译 / 打包（与 Dockerfile 完全一致）
./mvnw test                                         # 注意：依赖 local-hamm profile，见 §5
./mvnw spring-boot:run -Dspring-boot.run.profiles=local-hamm   # 本地启动
docker build -t spms-server .                       # Dockerfile 已含 COPY settings.xml
./deploy.sh                                          # 仅在生产环境（/home/server/），注意 JAR 名见 §13
```

## 4. 环境与 Profile（启动前必读）

- **本地 profile 配置**：复制 `src/main/resources/application-template.yml` 为 `application-local-<your-name>.yml`，按需填入
  DB/Redis/MQTT/InfluxDB/OSS/邮箱密钥。`application-local-*.yml` 已在 `.gitignore`， **不要**提交个人密钥。
    - 例外：`application-local-hamm.yml` 是仓库自带的 hamm 本地配置，被误提交——保留即可，但其中含明文邮箱密钥、AI
      Key、企业微信密钥， **不要复制粘贴其中密钥**。
- **默认 profile**：`application.yml` 中 `spring.profiles.active=production`；开发时务必显式切换到 `local-*`，否则会用
  `production` profile 去连线上 MySQL/Redis。
- **ddl-auto 行为差异**：
    - `application.yml`（基线）→ `validate`
    - `application-production.yml` → `update`（profile 覆盖基线，生产实际是 update）
    - `application-local-hamm.yml` → `create-drop`（每次重启清表）
    - `application-template.yml` → `update`
- **依赖服务**：`localhost:3306` MySQL（库名 `spms`）、`localhost:6379` Redis、MQTT broker、InfluxDB。MQTT/InfluxDB 不在
  docker-compose 默认栈里，需自行准备。
- **端口**：`8080`（见 `application.yml`）。

## 5. 测试与数据初始化

- **测试目录目前只有一个**：`src/test/java/cn/hamm/spms/ApplicationTest.java`。它使用 `@ActiveProfiles("local-hamm")` +
  `RedisHelper`，所以：
    - 跑 `./mvnw test` **必须** 保证 `application-local-hamm.yml` 在 classpath（默认就在 `src/main/resources/` 下）。
    - 跑测试需要本地 Redis 在线。
- **`DevDataInitRunner`**：实现 `CommandLineRunner`，仅当 `app.is-dev-mode: true` 且（无 `init.lock` 或
  `ddl-auto=create-drop`）时执行种子数据（用户、权限、菜单、demo 物料/仓库/BOM 等）。`init.lock` 写在 **仓库根目录**，已被
  `.gitignore`；首次启动后即存在，再次启动不会重跑种子。删 `init.lock` + 重启可强制重跑。

## 6. 仓库目录结构

```
src/main/java/cn/hamm/spms/
├── Application.java              # 启动类（cn.hamm.spms.Application），含 MQTT report listener 初始化
├── DevDataInitRunner.java        # 种子数据
├── WebConfig.java                # WebSocket / 拦截器 / 过滤器注册
├── base/                         # BaseEntity / BaseService / BaseRepository / BaseController + bill/
├── common/                       # Configs / AppConfig / AppWebSocketHandler / aliyun / influx / cron / interceptor ...
└── module/                       # 业务模块根
    ├── asset/  channel/  chat/   factory/  iot/  mcp/
    ├── mes/    open/    personnel/  system/  wechat/  wms/

src/main/resources/
├── application.yml               # 基线配置（active=production，ddl-auto=validate）
├── application-template.yml      # 本地 profile 模板（gitignore 之外、用于复制）
├── application-production.yml    # 生产 profile（覆盖 ddl-auto=update）
├── application-local-hamm.yml    # hamm 本地配置（已误提交，含明文密钥）
├── logback-spring.xml
└── templates/
```

业务模块下的 **四件套**（如 `module/wms/inventory/`）：

```
function/
├── enums/                # 枚举（可选），命名 XXXType / XXXStatus
├── XxxEntity.java        # 大驼峰 + Entity 后缀
├── XxxRepository.java    # 大驼峰 + Repository 后缀
├── XxxService.java       # 大驼峰 + Service 后缀
└── XxxController.java    # 大驼峰 + Controller 后缀
```

命名规范：

| 层次        | 命名           | 示例                                        |
|-------------|----------------|---------------------------------------------|
| 包名        | 全小写英文单词 | `inventory`、`input`、`output`              |
| 类          | 大驼峰 + 后缀  | `InventoryEntity`                           |
| 方法 / 变量 | 小驼峰         | `getByMaterialIdAndStorageId`、`materialId` |
| 常量        | 全大写下划线   | `DEFAULT_PAGE_SIZE`                         |

模板示例见 `cn.hamm.spms.module.wms.inventory`、`cn.hamm.spms.module.factory.storage`。

## 7. 继承体系

所有业务类继承项目提供的基类（位于 `cn.hamm.spms.base`）：

| 层次       | 基类                                                                |
|------------|---------------------------------------------------------------------|
| Entity     | `BaseEntity<E extends BaseEntity<E>>`                               |
| Repository | `BaseRepository<E extends BaseEntity<E>>`                           |
| Service    | `BaseService<E extends BaseEntity<E>, R extends BaseRepository<E>>` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s-pms/SPMS-Server](https://github.com/s-pms/SPMS-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
