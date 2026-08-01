---
trigger: always_on
description: This document describes the high-level architecture of the Microsoft JDBC Driver for SQL Server.
---

# mssql-jdbc Architecture

This document describes the high-level architecture of the Microsoft JDBC Driver for SQL Server.

## Overview

The driver implements the JDBC 4.2/4.3 specification and communicates with SQL Server using the TDS (Tabular Data Stream) protocol. It supports SQL Server 2012 and later, as well as Azure SQL Database.

## Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Application Layer                            │
│                  (User Application Code)                         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      JDBC API Layer                              │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐    │
│  │ Connection  │  │  Statement   │  │     ResultSet       │    │
│  │   Pool      │  │   Cache      │  │     Handling        │    │
│  └─────────────┘  └──────────────┘  └─────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Protocol Layer (TDS)                          │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐    │
│  │  IOBuffer   │  │  TDSParser   │  │   Stream Handlers   │    │
│  │  (I/O)      │  │  (Tokens)    │  │   (Data Types)      │    │
│  └─────────────┘  └──────────────┘  └─────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Transport Layer                               │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐    │
│  │   Socket    │  │     SSL      │  │   Authentication    │    │
│  │   I/O       │  │   Handler    │  │   (Kerberos/NTLM)   │    │
│  └─────────────┘  └──────────────┘  └─────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      SQL Server                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Package Structure

```
com.microsoft.sqlserver.jdbc/
│
├── Connection Management
│   ├── SQLServerConnection.java         # Main connection implementation
│   ├── SQLServerConnection43.java       # JDBC 4.3 extensions
│   ├── SQLServerDataSource.java         # DataSource implementation
│   ├── SQLServerConnectionPoolDataSource.java
│   ├── SQLServerPooledConnection.java
│   ├── SQLServerConnectionPoolProxy.java
│   ├── SQLServerXAConnection.java       # XA transaction support
│   ├── SQLServerXADataSource.java
│   └── SQLServerXAResource.java
│
├── Statement Execution
│   ├── SQLServerStatement.java          # Basic statements
│   ├── SQLServerPreparedStatement.java  # Parameterized queries
│   ├── SQLServerCallableStatement.java  # Stored procedures
│   ├── Parameter.java                   # Parameter binding
│   ├── ParameterMetaDataCache.java      # Prepared statement cache
│   └── SQLServerParameterMetaData.java
│
├── Result Processing
│   ├── SQLServerResultSet.java          # Result set implementation
│   ├── SQLServerResultSetMetaData.java  # Column metadata
│   ├── SQLServerDatabaseMetaData.java   # Database metadata
│   ├── Column.java                      # Column data handling
│   └── ScrollWindow.java                # Scrollable cursors
│
├── TDS Protocol
│   ├── IOBuffer.java                    # TDS packet I/O
│   ├── tdsparser.java                   # Token stream parsing
│   ├── StreamPacket.java                # Base packet handler
│   ├── StreamColInfo.java               # Column info token
│   ├── StreamColumns.java              # Column metadata token
│   ├── StreamDone.java                  # Done token
│   ├── StreamLoginAck.java             # Login acknowledgment
│   ├── StreamRetStatus.java             # Return status
│   ├── StreamRetValue.java              # Return value
│   └── StreamTabName.java               # Table name token
│
├── Data Types
│   ├── DataTypes.java                   # Type mappings
│   ├── DDC.java                         # Data type conversion
│   ├── dtv.java                         # Data type value handling
│   ├── SQLServerBlob.java              # BLOB support
│   ├── SQLServerClob.java              # CLOB support
│   ├── SQLServerNClob.java             # NCLOB support
│   ├── SQLServerSQLXML.java             # XML support
│   ├── Geography.java                   # Spatial - Geography
│   ├── Geometry.java                    # Spatial - Geometry
│   ├── SQLServerSpatialDatatype.java    # Spatial base class
│   └── SqlVariant.java                  # sql_variant support
│
├── Bulk Operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/mssql-jdbc](https://github.com/microsoft/mssql-jdbc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
