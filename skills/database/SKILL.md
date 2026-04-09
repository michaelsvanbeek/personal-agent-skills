---
name: database
description: >-
  Database design conventions for DynamoDB single-table design and SQL/PostgreSQL.
  Use when: modeling access patterns, designing DynamoDB tables and GSIs, writing SQL
  migrations, configuring connection pools, optimizing queries, choosing between SQL and
  NoSQL, designing data models for serverless applications, auditing existing database
  schemas for performance or design issues, or improving query patterns and indexing.
tags:
    - developer
    - operations
---

# Database Design Conventions

## When to Use

- Designing a new DynamoDB table or SQL schema
- Modeling access patterns for single-table design
- Writing SQL migrations
- Choosing between SQL and NoSQL for a use case
- Configuring connection pools for serverless
- Auditing existing database schemas for missing indexes, inefficient access patterns, or N+1 queries
- Optimizing query performance in an existing database
- Reviewing GSI design for cost and read/write efficiency

---

## Choosing SQL vs DynamoDB

| Factor | Choose DynamoDB | Choose PostgreSQL/SQL |
|--------|----------------|----------------------|
| Access patterns | Known, finite, high-volume | Ad-hoc, complex joins, evolving |
| Scaling | Horizontal (single-digit ms at any scale) | Vertical (RDS, Aurora Serverless) |
| Schema | Flexible, evolving attributes | Strict, relational integrity required |
| Cost model | Pay-per-request works well | Provisioned instance always running |
| Serverless fit | Native (no connection pool needed) | Needs RDS Proxy or connection pooling |
| Transactions | Simple (up to 100 items, 4MB) | Complex multi-table ACID |

**Default for homelab scripts**: Use simple file-based storage (JSON, SQLite) or InfluxDB for time-series. DynamoDB/Postgres for production services.

---

## DynamoDB Single-Table Design

### Access Pattern Modeling

Always start with access patterns — never start with an entity-relationship diagram:

1. **List all access patterns** before designing the table.
2. **Define the primary key** (PK + SK) to satisfy the most common pattern.
3. **Add GSIs** for additional access patterns (max 20 per table).
4. **Denormalize** — duplicate data to avoid joins.

### Key Design

```
PK                  SK                      Attributes
────────────────    ─────────────────────   ──────────────────
USER#<id>           PROFILE                 name, email, created_at
USER#<id>           ORDER#<id>              total, status, created_at
USER#<id>           ORDER#<id>#ITEM#<id>    product_id, qty, price
ORG#<id>            META                    name, plan, created_at
ORG#<id>            USER#<id>               role, joined_at
```

**Naming conventions**:
- PK/SK values: `ENTITY_TYPE#value` (uppercase type, `#` separator).
- Attribute names: `snake_case`.
- GSI names: `gsi1`, `gsi2` (generic) with `gsi1pk` / `gsi1sk` attributes.
- Table name: `{service}-{stage}` (e.g., `my-api-prod`).

### Patterns

**One-to-many** — parent entity PK, children share PK with different SK prefix:
```python
# Get user profile: PK=USER#123, SK=PROFILE
# Get user orders:  PK=USER#123, SK begins_with ORDER#
```

**Many-to-many** — use inverted GSI:
```python
# PK=USER#1,  SK=GROUP#A  → "User 1 belongs to Group A"
# GSI1: gsi1pk=GROUP#A, gsi1sk=USER#1  → "Group A contains User 1"
```

**Time-series queries** — SK contains ISO 8601 timestamp:
```python
# PK=DEVICE#abc, SK=READING#2024-01-15T10:30:00Z
# Query with SK between for time range
```

### DynamoDB Best Practices

- **Always use `condition_expression`** for writes to prevent overwrites.
- **Use `UpdateExpression`** to modify attributes — never read-modify-write.
- **Set TTL** for ephemeral data (sessions, caches) — TTL attribute must be epoch seconds.
- **Use sparse GSIs** — only items with the GSI key attributes appear in the index.
- **Avoid scans** — every access pattern should be a Query or GetItem.
- **Paginate** — always handle `LastEvaluatedKey` in responses.
- **Use batch operations** (`batch_get_item`, `batch_write_item`) for bulk access (max 25 writes, 100 reads per batch).

### Python (boto3) Patterns

```python
import boto3
from boto3.dynamodb.conditions import Key

table = boto3.resource("dynamodb").Table("my-api-prod")

# Put with condition (prevent overwrite)
table.put_item(
    Item={"pk": "USER#123", "sk": "PROFILE", "name": "Alice"},
    ConditionExpression="attribute_not_exists(pk)",
)

# Query one-to-many
response = table.query(
    KeyConditionExpression=Key("pk").eq("USER#123") & Key("sk").begins_with("ORDER#"),
    ScanIndexForward=False,  # newest first
    Limit=20,
)
items = response["Items"]

# Paginate
while "LastEvaluatedKey" in response:
    response = table.query(
        KeyConditionExpression=Key("pk").eq("USER#123") & Key("sk").begins_with("ORDER#"),
        ExclusiveStartKey=response["LastEvaluatedKey"],
    )
    items.extend(response["Items"])
```

---

## PostgreSQL / SQL Conventions

### Schema Design

- **Table names**: `snake_case`, plural (`users`, `orders`, `order_items`).
- **Column names**: `snake_case`, no table prefix (`id`, not `user_id` in the `users` table).
- **Primary keys**: `id` (prefer `bigint generated always as identity` or `uuid`).
- **Foreign keys**: `<singular_table>_id` (e.g., `user_id` in `orders`).
- **Timestamps**: Always include `created_at` and `updated_at` with `timestamptz`.
- **Soft delete**: Use `deleted_at timestamptz` column — never physically delete user-facing data.
- **Booleans**: Prefix with `is_` or `has_` (e.g., `is_active`, `has_verified_email`).
- **Enums**: Use PostgreSQL `enum` types or `text` with `CHECK` constraints. Never store magic integers.

### Example Table

```sql
CREATE TABLE users (
    id          bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email       text NOT NULL UNIQUE,
    name        text NOT NULL,
    is_active   boolean NOT NULL DEFAULT true,
    created_at  timestamptz NOT NULL DEFAULT now(),
    updated_at  timestamptz NOT NULL DEFAULT now(),
    deleted_at  timestamptz
);

CREATE INDEX idx_users_email ON users (email) WHERE deleted_at IS NULL;
```

### Migrations

- Use a migration tool: **Alembic** (Python) or **golang-migrate**.
- **One change per migration** — never mix schema changes.
- **Always write a down migration** (rollback).
- Migration filenames: `YYYYMMDDHHMMSS_description.sql` or Alembic auto-generated.
- **Never modify a deployed migration** — create a new one.
- Test migrations against a copy of production data before deploying.

### Indexing

- **Index foreign keys** — all `_id` columns used in JOINs or WHERE clauses.
- **Use partial indexes** for filtered queries (`WHERE deleted_at IS NULL`).
- **Use composite indexes** for multi-column queries — put equality columns first, range columns last.
- **Avoid over-indexing** — each index slows writes. Only index columns used in WHERE, JOIN, ORDER BY.
- **Use `EXPLAIN ANALYZE`** to verify query plans use indexes.

### Connection Pooling

For serverless (Lambda + RDS):
- **Use RDS Proxy** — handles connection pooling at the infrastructure level.
- **Set connection timeout** to a reasonable value (5-10 seconds).
- Never open more connections than `max_connections / expected_concurrency`.

For long-running processes:
- Use **SQLAlchemy** connection pool with `pool_size`, `max_overflow`, `pool_recycle`.
- Set `pool_pre_ping=True` to detect stale connections.

```python
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql://user:pass@host/db",
    pool_size=5,
    max_overflow=10,
    pool_recycle=3600,
    pool_pre_ping=True,
)
```

### Query Safety

- **Always use parameterized queries** — never string-format SQL.
- **Use transactions** for multi-statement operations.
- **Set statement timeout** to prevent runaway queries.
- **Use `SELECT ... FOR UPDATE`** when reading rows you intend to modify.

---

## InfluxDB (Time-Series)

For homelab monitoring (the primary time-series use case):

### Data Model

- **Measurement**: The metric name (`disk_health`, `cert_expiry`, `network_scan`).
- **Tags**: Indexed metadata for filtering (`host`, `device`, `script_name`). Low cardinality only.
- **Fields**: The actual values (`temperature`, `days_remaining`, `response_time_ms`). Not indexed.
- **Timestamp**: Always include — InfluxDB is optimized for time-range queries.

### Writing

```python
from influxdb_client import InfluxDBClient, Point

point = (
    Point("disk_health")
    .tag("host", "nas01")
    .tag("device", "/dev/sda")
    .field("temperature", 38)
    .field("reallocated_sectors", 0)
    .field("power_on_hours", 12345)
)
```

### Best Practices

- **Never use high-cardinality values as tags** (UUIDs, timestamps, user IDs).
- **Use retention policies** to auto-delete old data.
- **Batch writes** — write multiple points at once, not one-by-one.
- **Align field types** — don't mix int and float for the same field across writes.

---

## SQLite (Local / Embedded)

For homelab scripts that need local state persistence:

- Use `sqlite3` from stdlib — no extra dependencies.
- Enable WAL mode for concurrent reads: `PRAGMA journal_mode=WAL;`
- Set `PRAGMA foreign_keys=ON;` at connection time.
- Use `with conn:` context manager for automatic transaction commit/rollback.
- Store in a well-known path (e.g., `/data/script-name/state.db`).

```python
import sqlite3
from pathlib import Path

db_path = Path("/data/my-script/state.db")
db_path.parent.mkdir(parents=True, exist_ok=True)

conn = sqlite3.connect(db_path)
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA foreign_keys=ON")
conn.row_factory = sqlite3.Row

with conn:
    conn.execute(
        "CREATE TABLE IF NOT EXISTS runs (id INTEGER PRIMARY KEY, ran_at TEXT, status TEXT)"
    )
```

For database migration strategies and zero-downtime schema changes, see the **data-migrations skill**. For API design patterns that query these data models, see the **api-design skill**. For loading data into databases via dlt pipelines, see the **data-pipelines skill**.
