---
name: database-migrations
description: Database migration best practices for schema changes, data migrations, rollbacks, and zero-downtime deployments. Use when creating or altering tables, columns, indexes, or planning schema changes.
---

# Database Migration Patterns

Safe, reversible database schema changes for production systems.

## When to Activate

- Creating or altering database tables, columns, or indexes
- Running data migrations (backfill, transform)
- Planning zero-downtime schema changes
- Setting up migration tooling for a new project
- User asks for "migration", "schema change", or "database change"

## Core Principles

1. **Every change is a migration** — Never alter production databases manually; always use versioned migration files.
2. **Migrations are forward-only in production** — Rollbacks use new forward migrations, not editing old ones.
3. **Schema and data migrations are separate** — Do not mix DDL and large DML in one migration; split so rollback and testing are clear.
4. **Test against production-sized data** — A migration that works on 100 rows may lock or time out on 10M.
5. **Migrations are immutable once deployed** — Never edit a migration that has already run in production; add a new migration instead.

## Migration Safety Checklist

Before applying any migration:

- [ ] Migration has both UP and DOWN (or is explicitly documented as irreversible)
- [ ] No full table locks on large tables (use concurrent operations where supported)
- [ ] New columns have defaults or are nullable (do not add NOT NULL without default on existing tables)
- [ ] Indexes on existing large tables created concurrently (e.g. PostgreSQL `CREATE INDEX CONCURRENTLY`)
- [ ] Data backfill is in a separate migration from the schema change
- [ ] Tested against a copy of production data or staging
- [ ] Rollback plan documented

## PostgreSQL Patterns

### Adding a column safely

```sql
-- GOOD: Nullable column, no lock
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- GOOD: Column with default (Postgres 11+ is instant)
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- BAD: NOT NULL without default on existing table (full rewrite, long lock)
ALTER TABLE users ADD COLUMN role TEXT NOT NULL;
```

### Adding an index without downtime

```sql
-- BAD: Blocks writes on large tables
CREATE INDEX idx_users_email ON users (email);

-- GOOD: Non-blocking (CONCURRENTLY cannot run inside a transaction in some tools)
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);
```

### Renaming a column (zero-downtime expand-contract)

Do not rename directly in production. Use expand → migrate → contract:

```sql
-- Step 1: Add new column (migration 001)
ALTER TABLE users ADD COLUMN display_name TEXT;

-- Step 2: Backfill (migration 002, data only)
UPDATE users SET display_name = username WHERE display_name IS NULL;

-- Step 3: Deploy app that reads/writes both columns, then only new column

-- Step 4: Drop old column (migration 003)
ALTER TABLE users DROP COLUMN username;
```

### Removing a column safely

1. Remove all application references to the column and deploy.
2. In a later migration, drop the column.

### Large data migrations

```sql
-- BAD: Single transaction, locks table
UPDATE users SET normalized_email = LOWER(email);

-- GOOD: Batch with SKIP LOCKED (PostgreSQL)
DO $$
DECLARE batch_size INT := 10000; rows_updated INT;
BEGIN
  LOOP
    UPDATE users SET normalized_email = LOWER(email)
    WHERE id IN (
      SELECT id FROM users WHERE normalized_email IS NULL
      LIMIT batch_size FOR UPDATE SKIP LOCKED
    );
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    RAISE NOTICE 'Updated % rows', rows_updated;
    EXIT WHEN rows_updated = 0;
    COMMIT;
  END LOOP;
END $$;
```

## Tooling Workflows

### Prisma (Node/TypeScript)

```bash
npx prisma migrate dev --name add_user_avatar   # Create and apply (dev)
npx prisma migrate deploy                        # Apply pending (prod)
npx prisma migrate reset                         # Reset (dev only)
npx prisma generate                              # Regenerate client
```

For operations Prisma cannot generate (e.g. concurrent index):

```bash
npx prisma migrate dev --create-only --name add_email_index
# Edit the generated SQL file to use CREATE INDEX CONCURRENTLY
```

### Drizzle (Node/TypeScript)

```bash
npx drizzle-kit generate   # Generate migration from schema
npx drizzle-kit migrate    # Apply migrations
npx drizzle-kit push       # Push schema (dev only, no migration file)
```

### Django (Python)

```bash
python manage.py makemigrations                  # Generate from models
python manage.py migrate                         # Apply
python manage.py showmigrations                  # Status
python manage.py makemigrations --empty app_name -n description  # Empty migration
```

Data migration example:

```python
def backfill_display_names(apps, schema_editor):
    User = apps.get_model("accounts", "User")
    batch_size = 5000
    users = User.objects.filter(display_name="")
    while users.exists():
        batch = list(users[:batch_size])
        for u in batch:
            u.display_name = u.username
        User.objects.bulk_update(batch, ["display_name"], batch_size=batch_size)

class Migration(migrations.Migration):
    dependencies = [("accounts", "0015_add_display_name")]
    operations = [migrations.RunPython(backfill_display_names, migrations.RunPython.noop)]
```

### golang-migrate (Go)

```bash
migrate create -ext sql -dir migrations -seq add_user_avatar
migrate -path migrations -database "$DATABASE_URL" up
migrate -path migrations -database "$DATABASE_URL" down 1
```

## Zero-downtime strategy (expand-contract)

```
Phase 1 — EXPAND: Add new column/table (nullable or with default). Deploy app that writes to BOTH old and new. Backfill data.
Phase 2 — MIGRATE: Deploy app that reads from NEW, still writes to BOTH. Verify.
Phase 3 — CONTRACT: Deploy app that uses only NEW. Then run migration to drop old column/table.
```

## Anti-patterns

| Anti-pattern | Why it fails | Better approach |
|--------------|--------------|-----------------|
| Manual SQL in production | No audit, unrepeatable | Always use migration files |
| Editing deployed migrations | Drift between environments | Create new migration |
| NOT NULL without default on existing table | Long lock, full rewrite | Add nullable, backfill, then add constraint |
| Inline index on large table | Blocks writes | CREATE INDEX CONCURRENTLY |
| Schema + large data in one migration | Hard rollback, long transaction | Separate schema and data migrations |
| Drop column before removing code | App errors on missing column | Remove code first, drop column next release |

## Work Process

1. **Choose approach** — Additive change vs expand-contract for renames/removals. Check table size and locking behavior.
2. **Create migration(s)** — Use project's migration tool; keep one logical change per file. Write DOWN/rollback where supported.
3. **Test** — Run against staging or copy of prod data. Measure duration and lock time.
4. **Document** — In PR or runbook: what changed, rollback steps, and any manual steps (e.g. backfill script).
5. **Deploy** — Follow team runbook (order, backup, feature flags). Do not run destructive migrations without backup and rollback plan.

## Output

- Migration file(s) matching project structure and naming.
- Short summary: what changed, rollback steps, manual steps (if any).
- Note if migration is irreversible (e.g. data loss) so ops can plan.
