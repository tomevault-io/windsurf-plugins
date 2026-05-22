---
trigger: always_on
description: This document outlines the database usage patterns and best practices for the OpenFrame project.
---

# Database Patterns

This document outlines the database usage patterns and best practices for the OpenFrame project.

## MongoDB Usage Patterns

### Document Design

- Design documents around specific use cases
- Embed related data when it's always accessed together
- Use references for large or frequently changing data
- Avoid deeply nested documents (limit to 2-3 levels)
- Use descriptive field names
- Follow consistent naming conventions (camelCase)

Example document:
```json
{
  "_id": "device123",
  "hostname": "server-01",
  "operatingSystem": "Linux",
  "status": "online",
  "lastSeen": "2023-04-01T12:00:00Z",
  "metadata": {
    "location": "Data Center 1",
    "rack": "A4",
    "model": "Dell PowerEdge R740"
  },
  "tags": ["production", "database", "critical"],
  "siteId": "site456"  // Reference to a site document
}
```

### Indexing Strategy

- Create indexes for frequently queried fields
- Use compound indexes for queries with multiple conditions
- Add indexes for sorting operations
- Consider partial indexes for filtered queries
- Use text indexes for full-text search
- Monitor and optimize index usage

Example indexes:
```java
@Document(collection = "devices")
public class Device {
    @Id
    private String id;
    
    @Indexed
    private String hostname;
    
    @Indexed
    private String status;
    
    @CompoundIndex(def = "{'siteId': 1, 'status': 1}")
    private String siteId;
    
    // Other fields
}
```

### Query Optimization

- Use projection to limit returned fields
- Filter documents as early as possible
- Use appropriate operators ($in, $gt, $lt, etc.)
- Leverage aggregation pipeline for complex queries
- Avoid large skip values for pagination
- Use cursor-based pagination for large collections

Example optimized query:
```java
List<Device> findActiveDevicesBySite(String siteId, int limit, String lastId) {
    Query query = new Query();
    
    if (lastId != null) {
        query.addCriteria(Criteria.where("_id").gt(lastId));
    }
    
    query.addCriteria(Criteria.where("siteId").is(siteId)
        .and("status").is("online"));
    
    query.fields()
        .include("hostname")
        .include("status")
        .include("lastSeen");
    
    query.limit(limit);
    query.with(Sort.by(Sort.Direction.ASC, "_id"));
    
    return mongoTemplate.find(query, Device.class);
}
```

### Aggregation Patterns

- Use aggregation pipeline for complex data transformations
- Break down complex pipelines into smaller stages
- Use $lookup for joining collections
- Leverage $group for data summarization
- Use $project to reshape documents
- Consider performance implications of each stage

Example aggregation:
```java
AggregationResults<DeviceStatusSummary> getDeviceStatusSummaryBySite(String siteId) {
    Aggregation aggregation = Aggregation.newAggregation(
        Aggregation.match(Criteria.where("siteId").is(siteId)),
        Aggregation.group("status").count().as("count"),
        Aggregation.project("count").and("status").previousOperation(),
        Aggregation.sort(Sort.Direction.DESC, "count")
    );
    
    return mongoTemplate.aggregate(
        aggregation, "devices", DeviceStatusSummary.class);
}
```

## Cassandra Data Modeling

### Table Design

- Design tables based on query patterns
- Denormalize data for efficient reads
- Use wide rows for related data
- Choose appropriate partition keys
- Use clustering columns for sorting
- Avoid large partitions

Example table definition:
```cql
CREATE TABLE events (
    device_id text,
    event_time timestamp,
    event_type text,
    severity text,
    message text,
    details map<text, text>,
    PRIMARY KEY (device_id, event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```

### Partition Strategy

- Choose partition keys that distribute data evenly
- Avoid hotspots by using composite partition keys if needed
- Keep partition sizes manageable (< 100MB)
- Consider time-based partitioning for time-series data
- Use bucketing for high-cardinality partition keys

Example composite partition key:
```cql
CREATE TABLE events_by_day (
    device_id text,
    day date,
    event_time timestamp,
    event_type text,
    severity text,
    message text,
    PRIMARY KEY ((device_id, day), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```

### Query Patterns

- Design queries around specific use cases
- Always include partition key in WHERE clause
- Use clustering columns for range queries
- Avoid using ALLOW FILTERING
- Use secondary indexes sparingly
- Consider materialized views for alternative access patterns

Example query patterns:
```java
// Efficient query - includes partition key
session.execute(
    "SELECT * FROM events WHERE device_id = ? AND event_time > ? AND event_time < ?",
    deviceId, startTime, endTime
);

// Materialized view for querying by event type
session.execute(
    "SELECT * FROM events_by_type WHERE event_type = ? AND severity = ?",
    eventType, severity
);
```

### Time Series Data

- Use time-based partitioning
- Consider TTL for automatic data expiration
- Use compaction strategies optimized for time series
- Implement data retention policies
- Consider downsampling for historical data

Example time series table:
```cql

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
