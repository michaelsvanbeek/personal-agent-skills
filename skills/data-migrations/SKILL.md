---
name: data-migrations
description: >-
  Database schema and data migration patterns for zero-downtime production
  deployments. Use when: writing SQL migrations, changing DynamoDB table
  structures, running data backfills, planning breaking schema changes,
  designing rollback strategies, auditing an existing project for migration
  risks, or coordinating database changes with application deployments. Covers
  expand-contract migrations, backward-compatible changes, data backfills,
  rollback plans, and migration testing.
tags:
  - developer
  - operations
---

# Data Migrations

## When to Use

- Writing a SQL migration (ALTER TABLE, new columns, index changes)
- Changing DynamoDB table structure (new GSI, attribute changes)
- Running a data backfill or transformation on existing records
- Planning a breaking schema change (column rename, type change, table split)
- Designing rollback strategies for database changes
- Coordinating database migration with application deployment
- Auditing an existing project for unsafe migration patterns
- Migrating data between systems (database-to-database, legacy-to-new)

---

## Core Principle: Migrations Must Be Backward-Compatible

**Every migration must work with both the old and new application version running simultaneously.** During deployment, both versions coexist. If the new schema breaks the old code, the deploy fails or requires downtime.

---

## The Expand-Contract Pattern

The safest approach for breaking schema changes. Three phases, each deployed independently:

### Phase 1: Expand

Add the new structure alongside the old one. Both old and new code work.

```sql
-- Migration: Add new column (nullable, no default required)
ALTER TABLE users ADD COLUMN display_name TEXT;
```

Application writes to both old and new columns. Reads from old column.

### Phase 2: Migrate

Backfill data from old structure to new. Deploy application update to read from new column.

```sql
-- Backfill: Copy existing data
UPDATE users SET display_name = username WHERE display_name IS NULL;
```

Application reads from new column. Writes to both.

### Phase 3: Contract

Remove the old structure. Only after confirming all reads use the new column.

```sql
-- Migration: Drop old column (only after full rollout verified)
ALTER TABLE users DROP COLUMN username;
```

**Rule**: Each phase is a separate migration and a separate deployment. Never combine expand and contract in one step.

---

## Safe vs Unsafe Operations

### SQL (PostgreSQL)

| Operation | Safe? | Notes |
|-----------|-------|-------|
| `ADD COLUMN` (nullable) | Yes | No table rewrite, no lock |
| `ADD COLUMN` with `DEFAULT` (PG 11+) | Yes | Stored in catalog, no rewrite |
| `ADD COLUMN NOT NULL` without default | **No** | Fails on existing rows |
| `DROP COLUMN` | Caution | Safe if no code reads it; run contract phase |
| `RENAME COLUMN` | **No** | Breaks existing code instantly |
| `ALTER COLUMN TYPE` | **No** | May require table rewrite and exclusive lock |
| `CREATE INDEX` | **No** | Locks writes unless `CONCURRENTLY` |
| `CREATE INDEX CONCURRENTLY` | Yes | Non-blocking; slower but safe |
| `DROP INDEX` | Yes | Instant |
| `ADD CONSTRAINT` (CHECK, FK) | **No** | Scans table, holds lock |
| `ADD CONSTRAINT ... NOT VALID` + `VALIDATE` | Yes | Two-step: add without scanning, then validate without lock |

### DynamoDB

| Operation | Safe? | Notes |
|-----------|-------|-------|
| Add new attribute to items | Yes | Schema-free; old items unaffected |
| Add GSI | Yes | Backfills asynchronously; reads return partial results during build |
| Remove GSI | Yes | Instant, no data loss from base table |
| Change partition key | **No** | Requires new table + data migration |
| Change sort key | **No** | Requires new table + data migration |
| Change attribute type | **No** | Old items retain old type; must backfill |

---

## Migration File Standards

### Naming Convention

```
YYYYMMDDHHMMSS_description.sql
```

Examples:
```
20250115120000_add_display_name_to_users.sql
20250116090000_backfill_display_name.sql
20250120140000_drop_username_from_users.sql
```

### Migration File Structure

Each migration file must contain both `up` and `down` sections:

```sql
-- migrate:up
ALTER TABLE users ADD COLUMN display_name TEXT;

-- migrate:down
ALTER TABLE users DROP COLUMN display_name;
```

### Rules

- One logical change per migration file.
- Every migration has a rollback (`down` section).
- Migrations are immutable once applied — never edit a deployed migration.
- Test both `up` and `down` in CI before deploying.

---

## Data Backfills

### Batch Processing Pattern

Never update all rows in a single transaction — it locks the table and may time out:

```python
import time

BATCH_SIZE = 1000
SLEEP_BETWEEN_BATCHES = 0.1  # seconds — limit load on database


def backfill_display_name(db: Connection) -> int:
    """Backfill display_name from username in batches."""
    total_updated = 0
    while True:
        result = db.execute(
            """
            UPDATE users
            SET display_name = username
            WHERE display_name IS NULL
            AND id IN (
                SELECT id FROM users
                WHERE display_name IS NULL
                LIMIT %(batch_size)s
            )
            """,
            {"batch_size": BATCH_SIZE},
        )
        db.commit()
        updated = result.rowcount
        total_updated += updated

        if updated < BATCH_SIZE:
            break
        time.sleep(SLEEP_BETWEEN_BATCHES)

    return total_updated
```

### DynamoDB Backfill

```python
import boto3
from typing import Any

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("users")


def backfill_attribute(attribute: str, compute_fn: Any) -> int:
    """Scan and update items missing an attribute."""
    total = 0
    scan_kwargs: dict[str, Any] = {
        "FilterExpression": Attr(attribute).not_exists(),
    }

    while True:
        response = table.scan(**scan_kwargs)
        items = response.get("Items", [])

        with table.batch_writer() as batch:
            for item in items:
                item[attribute] = compute_fn(item)
                batch.put_item(Item=item)
                total += 1

        if "LastEvaluatedKey" not in response:
            break
        scan_kwargs["ExclusiveStartKey"] = response["LastEvaluatedKey"]

    return total
```

### Backfill Rules

- Run backfills as separate deployable scripts, not inside migrations.
- Backfills must be **idempotent** — safe to run multiple times.
- Log progress: "Updated 5000 of 50000 rows (10%)".
- Add a `--dry-run` flag to preview changes without applying.
- Throttle to avoid overwhelming the database.
- Monitor database CPU/IOPS during execution.

---

## Rollback Strategy

### Before Every Migration

Document the rollback plan:

```markdown
## Migration: add_display_name_to_users

**Forward**: `ALTER TABLE users ADD COLUMN display_name TEXT`
**Rollback**: `ALTER TABLE users DROP COLUMN display_name`
**Data loss on rollback**: None (column is nullable, no data depends on it yet)
**Rollback window**: Before backfill runs (Phase 2)
```

### Rollback Decision Matrix

| Scenario | Action |
|----------|--------|
| Migration failed partway | Run `down` migration; fix and retry |
| Migration succeeded but app broken | Run `down` migration; redeploy old app version |
| Backfill completed but data is wrong | Run corrective backfill; do NOT roll back schema |
| Contract phase removed old column | **Cannot roll back** — old data is gone. Restore from backup if needed |

### Point of No Return

Some migrations cannot be rolled back:

- Dropping a column or table (data gone)
- Changing data types with lossy conversion
- Deleting or overwriting rows in a backfill

**Rule**: Identify the point of no return before executing. Ensure backups exist before crossing it.

---

## Coordinating Migrations with Deployments

### Deploy Order for Expand Phase

1. Run migration (add new column/index).
2. Deploy new application code (writes to both old and new).
3. Verify: new column is populated for new writes.

### Deploy Order for Contract Phase

1. Deploy application code that only reads new column.
2. Verify: no code references old column.
3. Run migration (drop old column).

### CI/CD Integration

```yaml
# Example: migration step in deployment pipeline
steps:
  - name: run-migrations
    commands:
      - dbmate up
    when:
      event: deployment

  - name: deploy-application
    commands:
      - serverless deploy
    depends_on:
      - run-migrations
```

**Rule**: Migrations run before application deployment. Rollback migrations run after application rollback.

---

## Migration Testing

### In CI

```bash
# Test migrations against a clean database
dbmate up          # Apply all migrations
dbmate rollback    # Roll back last migration
dbmate up          # Re-apply — must succeed
```

### Schema Diff Validation

After migrations, compare the resulting schema against expected state:

```bash
pg_dump --schema-only production_db > expected_schema.sql
pg_dump --schema-only test_db > actual_schema.sql
diff expected_schema.sql actual_schema.sql
```

### Data Integrity Checks

After backfills, verify:

```sql
-- No NULLs remain after backfill
SELECT COUNT(*) FROM users WHERE display_name IS NULL;
-- Expected: 0

-- Row count unchanged
SELECT COUNT(*) FROM users;
-- Expected: same as before backfill
```

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Rename column in one step | Breaks running application instantly | Use expand-contract: add new → backfill → drop old |
| `ALTER TABLE` with `NOT NULL` on populated table | Fails or locks table for full scan | Add nullable, backfill, then add constraint |
| `CREATE INDEX` without `CONCURRENTLY` | Blocks writes for duration of index build | Always use `CREATE INDEX CONCURRENTLY` in production |
| Backfill inside migration transaction | Locks table, may time out, blocks other queries | Run backfills as separate idempotent scripts |
| No rollback plan | Stuck if migration causes issues | Write `down` migration and document rollback before running `up` |
| Editing deployed migrations | Diverges local/CI from production state | Migrations are append-only; create new migration to fix |
| Running untested migrations in production | Schema drift, data loss | Test `up` and `down` in CI against real schema |
| Skipping the contract phase | Dead columns and indexes accumulate | Schedule contract cleanup; track in backlog |

---

## Audit Checklist

When auditing an existing project for migration practices:

- [ ] Migration tool is configured and integrated into CI (dbmate, Alembic, Flyway, or equivalent)
- [ ] Every migration has a corresponding rollback (`down` step)
- [ ] Migrations are tested in CI (up, rollback, re-up cycle)
- [ ] No `ALTER TABLE` uses blocking operations without `CONCURRENTLY` or equivalent
- [ ] No column renames or type changes done in a single step
- [ ] Breaking schema changes use expand-contract pattern
- [ ] Backfills run as separate scripts with batching and throttling
- [ ] Backfill scripts are idempotent (safe to re-run)
- [ ] Rollback plan documented for each migration
- [ ] Point of no return identified for destructive migrations
- [ ] Database backups verified before destructive operations
- [ ] Migration order coordinates with application deployment order
- [ ] No dead columns or unused indexes left from incomplete contract phases
