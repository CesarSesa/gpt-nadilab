---
name: migration-generator
description: Generates idempotent Supabase migrations with RLS policies, indexes, and CHECK constraints following Nadistudio patterns.
trigger: "create migration", "add table", "add column", "update schema", "ALTER TABLE", "CREATE TABLE", "migration SQL", "IF NOT EXISTS"
origin: Nadistudio
cost_aware: false
validation:
  - scaffold-validator migration-generator --check idempotent
  - scaffold-validator migration-generator --check rls-present
  - scaffold-validator migration-generator --check indexes
---

# Part 1: Goals

## When to Activate

- User says "create migration", "add table", "add column", "update schema"
- You need to modify the database safely
- Creating a new table with proper RLS and indexes
- Adding constraints to existing tables
- Modifying column types safely

## When NOT to Use

- **Data migrations** (moving data between tables) → Use custom scripts
- **Complex schema refactoring** → Requires manual migration plan
- **Production hotfixes** → Use emergency procedures

## Core Principles

1. **Idempotent by Default** — Always use `IF NOT EXISTS` / `IF EXISTS`
2. **Defense in Depth** — RLS policies are mandatory, never skip
3. **Performance First** — tenant_id index is mandatory for all queries
4. **Type Safety** — `timestamptz` not `timestamp`, `NUMERIC` not `FLOAT`
5. **Test Before Deploy** — Always run on local/dev before production

---

# Part 2: Execution

## Quick Start: Standard Migration Template

```sql
-- Migration: YYYYMMDD_description.sql
-- Created by: Claude Coder

-- Enable extension if needed
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- 1. CREATE TABLE (if new)
CREATE TABLE IF NOT EXISTS {table_name} (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  -- your columns here --
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  deleted_at TIMESTAMPTZ, -- soft delete (optional)
  UNIQUE(tenant_id, {unique_column}) -- if unique per tenant
);

-- 2. Base indexes (MANDATORY)
CREATE INDEX IF NOT EXISTS idx_{table}_tenant_id ON {table}(tenant_id);
CREATE INDEX IF NOT EXISTS idx_{table}_tenant_created ON {table}(tenant_id, created_at DESC);

-- 3. Composite status index (if table has status)
CREATE INDEX IF NOT EXISTS idx_{table}_tenant_status ON {table}(tenant_id, status);

-- 4. Trigram index (if searchable text)
CREATE INDEX IF NOT EXISTS idx_{table}_{column}_trgm ON {table} USING gin ({column} gin_trgm_ops);

-- 5. Enable RLS
ALTER TABLE {table} ENABLE ROW LEVEL SECURITY;

-- 6. service_role policy (bot + admin API access)
CREATE POLICY service_role_access_{table} ON {table}
  FOR ALL TO service_role
  USING (true) WITH CHECK (true);

-- 7. Authenticated policy (optional, for web users)
CREATE POLICY tenant_isolation_{table} ON {table}
  FOR ALL TO authenticated
  USING (tenant_id IN (SELECT get_user_tenant_ids()));
```

## Implementation Patterns

### Pattern 1: Add Column

```sql
ALTER TABLE {table} ADD COLUMN IF NOT EXISTS {column_name} {type};
ALTER TABLE {table} ADD COLUMN IF NOT EXISTS {column_name} {type} DEFAULT {default};
```

### Pattern 2: Update Column Type

```sql
ALTER TABLE {table} ALTER COLUMN {column} TYPE {new_type};
```

### Pattern 3: Drop Column (Safe)

```sql
-- First check what references it
SELECT table_name, column_name
FROM information_schema.key_column_usage
WHERE referenced_table_name = '{table}';

-- Then drop if safe
ALTER TABLE {table} DROP COLUMN IF EXISTS {column_name};
```

### Pattern 4: Add CHECK Constraint

```sql
ALTER TABLE {table} DROP CONSTRAINT IF EXISTS {table}_status_check;
ALTER TABLE {table} ADD CONSTRAINT {table}_status_check
  CHECK (status IN ('pending', 'active', 'archived'));
```

## Step-by-Step Workflow

### Step 1: Create Migration File
```
supabase/migrations/YYYYMMDD_description.sql
```

### Step 2: Write Idempotent SQL
- Use `IF NOT EXISTS` for CREATE
- Use `IF EXISTS` for DROP
- Test syntax in Supabase SQL Editor

### Step 3: Validate Checklist
- [ ] Uses `IF NOT EXISTS` / `IF EXISTS` everywhere
- [ ] Has tenant_id column (unless global table)
- [ ] Has RLS enabled
- [ ] Has service_role policy
- [ ] Has base indexes (tenant_id, tenant_created)
- [ ] Uses TIMESTAMPTZ not TIMESTAMP
- [ ] Named correctly (migration, indexes, policies)

### Step 4: Test on Local/Dev
Run migration on local database first.

### Step 5: Deploy to Production
Apply via Supabase Dashboard SQL Editor.

---

# Part 3: Quality Standards

## Pre-Delivery Checklist

- [ ] Migration file named `YYYYMMDD_description.sql`
- [ ] All CREATE statements use `IF NOT EXISTS`
- [ ] All DROP statements use `IF EXISTS`
- [ ] Table has `tenant_id UUID NOT NULL REFERENCES tenants(id)`
- [ ] RLS enabled via `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`
- [ ] service_role policy created
- [ ] Base indexes created (tenant_id, tenant_created)
- [ ] Uses `TIMESTAMPTZ` not `TIMESTAMP`
- [ ] Foreign keys follow `{entity}_id` naming
- [ ] Booleans use `is_` prefix
- [ ] Money uses `NUMERIC` not FLOAT/REAL
- [ ] Tested on local/dev before production

## Validation Commands

```bash
# Check idempotency (IF NOT EXISTS usage)
scaffold-validator migration-generator --check idempotent --file migration.sql

# Verify RLS policies present
scaffold-validator migration-generator --check rls-present --file migration.sql

# Verify indexes defined
scaffold-validator migration-generator --check indexes --file migration.sql
```

## Common Errors

| Error | Symptom | Cause | Fix |
|-------|---------|-------|-----|
| **Migration fails on re-run** | "relation already exists" | Missing `IF NOT EXISTS` | Add `IF NOT EXISTS` to all CREATE |
| **RLS blocks queries** | Empty results | No service_role policy | Add service_role policy |
| **Slow queries** | Timeout on tenant data | Missing tenant_id index | Add `idx_{table}_tenant_id` |
| **Timezone bug** | Wrong hour in timestamps | Used `timestamp` not `timestamptz` | Change to `TIMESTAMPTZ` |
| **Constraint violation** | Check constraint fails | Missing status value in CHECK | Update CHECK constraint values |

---

# Part 4: Technical Reference

## Column Naming Conventions

| Type | Use ✅ | Never Use ❌ |
|------|--------|--------------|
| Timestamps | `created_at`, `updated_at`, `deleted_at` | `createdAt`, `date_created` |
| Foreign keys | `{entity}_id` | `entityId`, `{entity}Id` |
| Booleans | `is_active`, `is_visible` | `active`, `visible` |
| Status | `status TEXT` with values | `status INTEGER` |
| Money | `NUMERIC` | `FLOAT`, `REAL` |
| IDs | `UUID` | Serial integers |

## Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Migration file | `YYYYMMDD_description.sql` | `20260325_add_products.sql` |
| Index | `idx_{table}_{columns}` | `idx_products_tenant_status` |
| Policy | `{policy_type}_{table}` | `service_role_access_products` |
| Constraint | `{table}_{column}_check` | `products_status_check` |

## Golden Rules

1. **ALWAYS use `IF NOT EXISTS` / `IF EXISTS`** — idempotent, can re-run
2. **ALWAYS add RLS policies** — never skip step 5-6
3. **ALWAYS add indexes** — tenant_id index is mandatory
4. **ALWAYS use `timestamptz`** — never bare `timestamp`

## Integration with Other Skills

| Skill | When to Use Together | Pattern |
|-------|---------------------|---------|
| `db-guardian` | Validate schema changes | Migration → Validation |
| `security-auditor` | Security review | Check RLS policies |
| `4-step-docs` | New feature DB design | Design → Migration → Implementation |

## Constraints

- **NEVER use bare `CREATE TABLE`** — always `IF NOT EXISTS`
- **NEVER use `CONCURRENTLY`** in Supabase SQL Editor — wrap in transaction
- **NEVER drop columns without checking references first**
- **ALWAYS test on local/dev before production**

---

*Skill: migration-generator — v2.0*  
*Template: 4-Part Structure*  
*Core Principle: Idempotent by Default*
