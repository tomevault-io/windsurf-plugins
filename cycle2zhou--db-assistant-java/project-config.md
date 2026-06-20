---
trigger: always_on
description: 通过上下文解析数据库连接信息，使用 JDBC 执行 SQL 查询。覆盖 MySQL/Oracle/PostgreSQL/Greenplum/GaussDB/ClickHouse/DM/Kingbase/HANA/Sybase/Doris/TDSQL/OceanBase/StarRocks/TDengine/Hive/Impala/DB2/Inceptor/ArgoDB/GBase/NebulaGraph 等类型。
---


# DB Assistant

通过对话上下文解析数据库连接信息，使用项目中已有的 JDBC 驱动执行 SQL 查询。

## 触发条件

当用户提问涉及以下内容时，激活此 Skill：

- 数据库表结构、字段、索引
- 查询数据、查看数据模型
- 需要了解数据库内容的场景
- 明确提到"数据库""SQL""表""查询"等关键词

## 连接信息解析规则

按以下优先级收集数据库连接信息，缓存到当前会话上下文：

### 优先级 1：对话上下文中用户直接提供

识别自然语言中的数据库信息：

- "连接 MySQL localhost:3306/myapp，用户 root，密码 xxx"
- "数据库是 PostgreSQL，地址 db.example.com:5432，用户 admin"
- "这个项目用 SQLite，文件在 ./data/dev.db"

解析模板：

```
<数据库类型> <主机>:<端口>/<数据库名> 用户 <用户名> 密码 <密码>
```

### 优先级 2：Spring Boot 配置文件自动发现

在项目目录中按顺序查找并解析：

1. `application.yml` / `application.yaml`
2. `application.properties`
3. `application-*.yml`（多环境配置，如 `application-dev.yml`）

提取字段：

```yaml
spring.datasource.url: jdbc:<type>://<host>:<port>/<dbname>
spring.datasource.username: <user>
spring.datasource.password: <password>
spring.datasource.driver-class-name: <driver>
```

`${ENV_VAR}` 占位符解析：

- 从项目 `.env` 文件中查找
- 从系统环境变量中读取

### 优先级 3：项目 .env 文件

查找 `DB_*` 或 `DATABASE_*` 开头的变量：

```
DB_TYPE=mysql
DB_HOST=localhost
DB_PORT=3306
DB_NAME=myapp
DB_USER=root
DB_PASSWORD=xxx
```

### 优先级 4：项目 AGENTS.md 数据库段落

查找 `AGENTS.md` 中包含"数据库""Database""DataSource"等关键词的段落。

## JDBC URL 构建规则

支持的 URL 模板：

| 类型 | JDBC URL 格式 | 参数追加规则 |
| --- | --- | --- |
| `MySQL` | `jdbc:mysql://<host>:<port>/<db>`；无库：`jdbc:mysql://<host>:<port>` | `?k=v&k2=v2` |
| `TDSQL` | `jdbc:mysql://<url>/<db>`；无库：`jdbc:mysql://<url>` | `?k=v&k2=v2` |
| `Doris` | `jdbc:mysql://<host>:<port>/<db>`；无库：`jdbc:mysql://<host>:<port>` | `?k=v&k2=v2` |
| `StarRocks` | `jdbc:mysql://<host>:<port>/<db>`；无库：`jdbc:mysql://<host>:<port>` | `?k=v&k2=v2` |
| `OceanBase` + `driverType=MySQL` | `jdbc:mysql://<host>:<port>/<db>`；无库：`jdbc:mysql://<host>:<port>` | `?k=v&k2=v2` |
| `OceanBase` + `driverType=Oracle` | `jdbc:oceanbase://<host>:<port>/<db>`；无库：`jdbc:oceanbase://<host>:<port>` | `?k=v&k2=v2` |
| `Oracle` + `connect_pattern=SERVICE_NAME` | `jdbc:oracle:thin:@<host>:<port>/<serviceName>` | `?k=v&k2=v2` |
| `Oracle` + `connect_pattern=SID` | `jdbc:oracle:thin:@<host>:<port>:<sid>` | `?k=v&k2=v2` |
| `PostgreSQL` | `jdbc:postgresql://<host>:<port>/<db>` | `?k=v&k2=v2` |
| `Greenplum` | `jdbc:postgresql://<host>:<port>/<db>` | `?k=v&k2=v2` |
| `GaussDB` | `jdbc:gaussdb://<host>:<port>/<db>` | `?k=v&k2=v2` |
| `SQLServer` | `jdbc:sqlserver://<host>:<port>;DatabaseName=<db>`；无库：`jdbc:sqlserver://<host>:<port>` | `;k=v;k2=v2` |
| `DB2` | `jdbc:db2://<host>:<port>/<db>` | `:k=v;k2=v2` |
| `HANA` | `jdbc:sap://<host>:<port>/<db>`；无库：`jdbc:sap://<host>:<port>` | `?k=v&k2=v2` |
| `DM` | `jdbc:dm://<host>:<port>?schema=<schemaOrDbName>` | `&k=v&k2=v2` |
| `ClickHouse` | `jdbc:clickhouse://<host>:<port>/<db>`；无库：`jdbc:clickhouse://<host>:<port>` | `?k=v&k2=v2` |
| `Kingbase` | `jdbc:kingbase8://<host>:<port>/<db>` | `?k=v&k2=v2` |
| `SyBase` | `jdbc:sybase:Tds:<host>:<port>/<db>`；无库：`jdbc:sybase:Tds:<host>:<port>` | `?k=v&k2=v2` |
| `TDengine` | `jdbc:TAOS-RS://<host>:<port>/<db>`；无库：`jdbc:TAOS-RS://<host>:<port>` | `?k=v&k2=v2` |
| `Hive` | `jdbc:hive2://<host>:<port>/<db>` | `;k=v;k2=v2` |
| `Hive` HA | `jdbc:hive2://<zkQuorum>/<db>;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=<namespace>` | `;k=v;k2=v2` |
| `Inceptor` | `jdbc:transwarp2://<host>:<port>/<db>` | `;k=v;k2=v2` |
| `Inceptor` HA | `jdbc:transwarp2://<zkQuorum>/<db>;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=<namespace>` | `;k=v;k2=v2` |
| `ArgoDB` | `jdbc:transwarp2://<host>:<port>/<db>`；空库默认 `default` | `;k=v;k2=v2` |
| `Kudu/Impala` + `extend1=No` | `jdbc:impala://<host>:<port>/<db>;AuthMech=0` | `;k=v;k2=v2` |
| `Kudu/Impala` + `extend1=SASL/PLAIN` | `jdbc:impala://<host>:<port>/<db>;AuthMech=3;UID=<user>;PWD=<password>` | `;k=v;k2=v2` |
| `Kudu/Impala` + `extend1=Kerberos` | `jdbc:impala://<host>:<port>/<db>;AuthMech=1;KrbHostFQDN=<fqdn>;krbRealm=<realm>;KrbServiceName=<service>;principal=<principal>` | `;k=v;k2=v2` |
| `GBase` 8a | `jdbc:gbase://<host>:<port>/<db>`；无库：`jdbc:gbase://<host>:<port>` | `?k=v&k2=v2` |
| `GBase` 8s | `jdbc:gbasedbt-sqli://<host>:<port>/<db>:GBASEDBTSERVER=<instanceName>` | `;k=v;k2=v2`；`driverType=Oracle` 时追加 `;SQLMODE=oracle` |
| `NebulaGraph` | `jdbc:nebula://<host>:<port>/<db>`；无库：`jdbc:nebula://<host>:<port>` | `?k=v&k2=v2` |
| `SQLite` | `jdbc:sqlite:<path>` | 本地文件数据库，无额外参数 |

构建约定：

- 若用户已经提供完整 `jdbc:*` URL，直接使用，不再重拼。
- 若用户提供 `typeName/url/port/dbName` 等离散字段，按上表拼接。
- `otherParams` 追加时遵循项目策略的连接符和分隔符，不混用 `?`、`&`、`;`、`:`。
- 需要 URL 转义时，对 `dbName` 和参数值做 UTF-8 URL encode，并把 `+` 替换为 `%20`。
- 实际能否执行查询取决于本机 Maven/Gradle 缓存或项目依赖中是否存在对应 JDBC driver。

## JDK 版本检测与驱动定位

### 操作系统检测

```bash
# 所有平台通用，输出: Linux / Darwin / Windows_NT / MINGW* / CYGWIN*
os_name=$(uname -s 2>/dev/null || echo "Windows")
```

根据 `uname -s` 结果判断平台：

| `uname -s` 值 | 平台 | 用户主目录 | 路径分隔符 | classpath 分隔符 |
|---|---|---|---|---|
| `Darwin` | macOS | `$HOME` | `/` | `:` |
| `Linux` | Linux | `$HOME` | `/` | `:` |
| `MINGW*` / `MSYS*` / `CYGWIN*` | Windows (Git Bash) | `$USERPROFILE` | `/` 或 `\` | `;` |
| （`uname` 不存在） | Windows (CMD/PowerShell) | `%USERPROFILE%` | `\` | `;` |

### Maven 与 Gradle 默认目录

按以下优先级定位构建工具缓存目录：

**优先级 1**：显式环境变量覆盖

```bash
# Maven 本地仓库（如已设置）
M2_REPO="${M2_REPO:-$HOME/.m2/repository}"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cycle2zhou/db-assistant-java](https://github.com/cycle2zhou/db-assistant-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
