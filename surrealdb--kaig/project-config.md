---
trigger: always_on
description: This rule provides information for writing queries for SurrealDB, using its query language SurrealQL
---


## Key concepts

### Record IDs
SurrealDB uses the format `table:id` for record identifiers:
- `person:john` - Record with ID "john" in "person" table
- `article:surreal_intro` - Record with ID "surreal_intro" in "article" table

### Update modes
The `update` tool supports three modes:
- **replace** (default): Replaces entire record content
- **merge**: Merges new data with existing data
- **patch**: Applies JSON patch operations

### Relationships
Use `relate` to create graph relationships:
- `person:john -> wrote -> article:surreal_intro`
- `person:john -> knows -> person:jane`

## Best practices

1. **Use specific record IDs** when you know them for better performance
2. **Use parameterized queries** to prevent SQL injection when dealing with user input
3. **Use the enhanced select tool** which safely parameterizes table names and supports user parameters
4. **Use the raw query tool** for complex operations not covered by convenience functions
5. **Use merge/patch modes** when updating records to preserve existing data
6. **Create relationships** to model graph data and enable complex queries
7. **Use table names** in select/delete when you want to operate on all records

## Example workflows

## SurrealQL reference

**IMPORTANT**: SurrealQL is NOT similar to ANSI-SQL. **Never** assume ANSI-SQL or SQL knowledge from other databases applies to SurrealQL. Always refer to the examples below, or the documentation at https://surrealdb.com/docs for accurate syntax and behavior.

### Key differences from SQL

- SurrealQL uses different syntax for many operations
- Every row in a table is called a Record
- Each Record in a table has a unique `id` field which can not be changed once specified
- A Record is similar to a MongoDB document, with the ability to store nested objects and arrays
- Record IDs use `table:id` format (e.g., `person:john`)
- Relationships are expressed with the following syntax:
    - `->` arrows for traversing outbound graph connections
    - `<-` arrows for traversing inbound graph connections
    - `<->` arros for traversing a graph connection in any direction
- Many operations work differently than in traditional SQL

### Best practices for SurrealQL

1. **Always use specific Record IDs** when you know them for better performance
2. **Use relationships** to model graph data instead of foreign keys
3. **Use MERGE and PATCH** for updates to preserve existing data
4. **Define schemas** for data validation and consistency
5. **Use indexes** on frequently queried fields
6. **Use LIVE queries** for real-time updates when needed

### Basic SurrealQL statements

#### SELECT statement

```surql
-- Select all records from a table (secure with type::table)
SELECT * FROM type::table($table);

-- Select specific fields
SELECT name, age FROM type::table($table);

-- Select by record ID (secure with type::table)
SELECT * FROM type::table($table):john;

-- Select with WHERE conditions
SELECT * FROM type::table($table) WHERE age > 25;

-- Select with WHERE, ORDER BY, and LIMIT
SELECT * FROM type::table($table) WHERE age > 25 ORDER BY name LIMIT 10;

-- Select with SPLIT ON (split records on specific fields)
SELECT * FROM type::table($table) SPLIT ON age, city;

-- Select with GROUP BY
SELECT count(), age FROM type::table($table) GROUP BY age;

-- Select with pagination (LIMIT and START AT)
SELECT * FROM type::table($table) ORDER BY name LIMIT 10 START AT 20;

-- Select with relationships (graph queries)
SELECT * FROM type::table($table) WHERE ->knows->person.age > 30;

-- Select with nested relationships
SELECT * FROM type::table($table) WHERE ->wrote->article->has->category.name = 'Technology';

-- Select with aggregation
SELECT count() FROM type::table($table);
SELECT count() FROM type::table($table) GROUP BY author;

-- Select with subqueries
SELECT * FROM type::table($table) WHERE id IN (SELECT author FROM article);

-- Complex query with multiple clauses
SELECT * FROM type::table($table)
WHERE age > 25 AND city = 'NYC'
SPLIT ON age
GROUP BY age, city
ORDER BY age DESC
LIMIT 10
START AT 5;
```

#### CREATE statement

```surql
-- Create a single record
CREATE person:john CONTENT {
    name: 'John Doe',
    age: 30,
    email: 'john@example.com'
};

-- Create multiple records
CREATE person CONTENT [
    { name: 'Alice', age: 25 },
    { name: 'Bob', age: 35 }
];

-- Create with specific ID
CREATE person:alice CONTENT { name: 'Alice', age: 25 };
```

#### UPDATE statement

```surql
-- Replace entire record content
UPDATE person:john CONTENT {
    name: 'John Doe',
    age: 31,
    city: 'New York'
};

-- Merge new data with existing data
UPDATE person:john MERGE {
    age: 31,
    city: 'New York'
};

-- Apply JSON patch operations
UPDATE person:john PATCH [
    { op: 'replace', path: '/age', value: 31 },
    { op: 'add', path: '/city', value: 'New York' }
];

-- Update multiple records
UPDATE person SET age += 1 WHERE age < 30;
```

#### DELETE statement

```surql
-- Delete a specific record
DELETE person:john;

-- Delete all records from a table
DELETE person;

-- Delete with conditions
DELETE person WHERE age < 18;
```

#### RELATE statement

```surql
-- Create a simple relationship
RELATE person:john->knows->person:jane;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surrealdb/kaig](https://github.com/surrealdb/kaig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
