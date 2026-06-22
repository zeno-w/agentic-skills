# MySQL Database Conventions

## Table Design Rules

### 1. Table and Column Naming

- Table names and column names use lowercase letters or numbers separated by underscores.
  - Positive example: `user_info` / `order_detail`
  - Counter-example: `UserInfo` / `userInfo` / `ORDER_DETAIL`
- Table names must not start with a number.
  - Counter-example: `1_user_info`
- Disable reserved words as table/column names (e.g., `order`, `group`, `desc`). Use backticks if unavoidable.

### 2. Primary Key

- Every table must have a primary key.
- Use auto-increment `BIGINT UNSIGNED` for primary key.
- Primary key name should be `id` or `table_name_id`.

### 3. Column Types

- Use `DECIMAL` for monetary amounts. Do not use `FLOAT` or `DOUBLE`.
- Use `VARCHAR` for variable-length strings. Specify a reasonable length.
- Use `DATETIME` or `TIMESTAMP` for time fields.
  - `TIMESTAMP` range: 1970-01-01 to 2038-01-19
  - `DATETIME` range: 1000-01-01 to 9999-12-31
- Use `UNSIGNED` for non-negative integers.

### 4. Logical Deletion

- Use logical deletion (is_deleted / deleted) instead of physical deletion.
  - Positive example: `ALTER TABLE user ADD COLUMN is_deleted TINYINT DEFAULT 0;`

### 5. Common Fields

- Every table should include: `id`, `create_time`, `update_time`, `is_deleted`.
- `create_time` / `update_time` should use `DATETIME` type with `DEFAULT CURRENT_TIMESTAMP`.

### 6. Character Set

- Use `utf8mb4` character set (not `utf8`, which only supports 3-byte characters and cannot store emoji).
- Use `utf8mb4_general_ci` or `utf8mb4_unicode_ci` collation.

### 7. Table Design

- Single table column count should not exceed 30.
- Single table row size should not exceed 8KB.
- Use appropriate data types to minimize storage: `TINYINT` vs `INT` vs `BIGINT`.

## Index Rules

### 1. Index Naming

- Primary key index: `pk_` prefix
- Unique index: `uk_` prefix
- Normal index: `idx_` prefix
  - Positive example: `uk_user_email` / `idx_create_time`

### 2. Index Design

- Avoid duplicate or redundant indexes.
- Composite index: follow the leftmost prefix rule. Put the column with highest selectivity first.
- Limit the number of indexes per table (recommend ≤ 5).
- Index length for `VARCHAR` columns: specify prefix length to reduce index size.

### 3. Index Usage

- Avoid operations that invalidate indexes:
  - Functions on indexed columns: `WHERE YEAR(create_time) = 2024` → use `WHERE create_time >= '2024-01-01' AND create_time < '2025-01-01'`
  - Implicit type conversion: `WHERE varchar_col = 123` → use `WHERE varchar_col = '123'`
  - `LIKE` with leading wildcard: `WHERE name LIKE '%john'` → use full-text search
  - `OR` conditions across different indexes
  - `NOT IN`, `NOT EXISTS`, `!=` on indexed columns

### 4. Covering Index

- Use covering indexes to avoid table lookups.
  - Positive example: `SELECT id, name FROM user WHERE name = 'test'` with index on `(name, id)`

### 5. ORDER BY and Index

- `ORDER BY` field should be part of a composite index and placed last in the index order to avoid `filesort`.

## SQL Rules

### 1. Count

- Use `COUNT(*)` instead of `COUNT(column)` or `COUNT(1)`.
  - Reason: `COUNT(*)` counts all rows including NULL; `COUNT(column)` does not count NULL values.
- `COUNT(*)` is optimized by the engine and does not fetch all columns.

### 2. Avoid SELECT *

- Never use `SELECT *`. Specify the exact columns needed.
  - Reason: wastes network bandwidth, prevents covering index optimization, breaks when columns are added/removed.

### 3. INSERT Column List

- Always specify column list in INSERT statements.
  - Positive example: `INSERT INTO user (name, age) VALUES ('test', 20)`
  - Counter-example: `INSERT INTO user VALUES ('test', 20)` (breaks if columns are reordered)

### 4. Transaction Size

- Keep transactions small. Avoid long-running transactions that hold locks.
- Do not include RPC calls inside transactions.

### 5. IN Clause

- Limit the number of values in `IN` clause (recommend ≤ 500). Use batch queries or temporary tables for large sets.

### 6. Pagination

- Avoid deep pagination with large offsets.
  - Counter-example: `SELECT * FROM user LIMIT 1000000, 10`
  - Positive example: Use deferred join or seek method:
    ```sql
    SELECT u.* FROM user u
    INNER JOIN (SELECT id FROM user ORDER BY id LIMIT 1000000, 10) t
    ON u.id = t.id
    ```

### 7. UPDATE/DELETE with WHERE

- Always use `WHERE` clause in UPDATE and DELETE statements.
- Use `LIMIT 1` for single-row updates to prevent accidental mass updates.

### 8. Avoid Multiple Column Updates in One SQL

- Do not update multiple columns that are not logically related in one SQL statement.

### 9. IS NULL vs = NULL

- Use `IS NULL` / `IS NOT NULL` to check for NULL. Never use `= NULL` or `!= NULL`.

### 10. Large Text Fields

- Store large text (articles, descriptions) in a separate table to avoid affecting main table query performance.

## ORM Rules

### 1. Query Result Mapping

- Do not use `SELECT *` in MyBatis mappings. Map only the columns needed.

### 2. Parameter Binding

- Always use `#{}` (parameterized) instead of `${}` (string substitution) to prevent SQL injection.
  - Exception: `ORDER BY` dynamic column names may use `${}` but must be validated against a whitelist.

### 3. Batch Operations

- Use batch insert/update for multiple records. Avoid looping single operations.

### 4. Transaction Annotation

- Use `@Transactional` carefully. Understand rollback rules:
  - Default rollback only on `RuntimeException`.
  - Specify `rollbackFor = Exception.class` for checked exceptions.