---
name: db-guardian
description: Database safety protocol for multi-tenant Supabase applications. Ensures tenant isolation, RLS policies, proper indexing, and migration safety.
trigger: "create table", "migration", "schema", "ALTER TABLE", "tenant_id", "RLS policy", "database query", "supabase", "index", "column", "table"
origin: Nadistudio
cost_aware: false
validation:
  - scaffold-validator db-guardian --check tenant-filter
  - scaffold-validator db-guardian --check rls-enabled
  - scaffold-validator db-guardian --check indexes
---

# Part 1: Goals

## When to Activate

Load this skill when you are about to:
- Create a new database table
- Write a migration SQL file
- Add or modify a `.from()` Supabase query
- Modify database schema (ALTER TABLE)
- Debug "data not showing up" issues
- Review database architecture
- Add indexes for performance
- Configure RLS policies

## When NOT to Use

- **Static file operations** → Use standard file tools
- **Frontend-only changes** (no DB interaction) → No skill needed
- **Read-only analytics** on small datasets (< 1000 rows) → Standard queries OK
- **External API integrations** → Use `api-error-handler` instead

## Core Principles

1. **Tenant Isolation is Non-Negotiable** — Every business query MUST filter by `tenant_id`
2. **Defense in Depth** — RLS policies catch what application code misses
3. **Idempotent Migrations** — All DDL must use `IF NOT EXISTS` / `IF EXISTS`
4. **Explicit over Implicit** — Never rely on defaults; specify types, constraints, indexes
5. **Validate Before Deploy** — Ghost columns and type mismatches fail silently

---

# Part 2: Execution

## Quick Start: New Table Template

```sql
-- 1. Create table with tenant_id (unless global/system table)
CREATE TABLE IF NOT EXISTS new_table (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  -- ... your columns ...
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 2. Required indexes
CREATE INDEX IF NOT EXISTS idx_new_table_tenant_id ON new_table(tenant_id);
CREATE INDEX IF NOT EXISTS idx_new_table_tenant_created ON new_table(tenant_id, created_at DESC);

-- 3. RLS protection
ALTER TABLE new_table ENABLE ROW LEVEL SECURITY;

-- 4. Service role policy (bot + admin API access)
CREATE POLICY service_role_access ON new_table
  FOR ALL TO service_role
  USING (true) WITH CHECK (true);

-- 5. Tenant isolation policy
CREATE POLICY tenant_isolation ON new_table
  FOR ALL TO authenticated
  USING (tenant_id = current_setting('app.tenant_id')::uuid);
```

## Implementation Patterns

### Pattern 1: Mandatory tenant_id Filter

```typescript
// CORRECT ✅
const { data } = await supabase
  .from('products')
  .select('*')
  .eq('tenant_id', tenantId)  // MANDATORY
  .eq('status', 'active');

// WRONG ❌ — Data leakage across tenants
const { data } = await supabase
  .from('products')
  .select('*')
  .eq('status', 'active');
```

### Pattern 2: Use maybeSingle() not single()

```typescript
// CORRECT ✅ — Returns null if not found
const { data } = await supabase
  .from('products')
  .select('*')
  .eq('id', productId)
  .eq('tenant_id', tenantId)
  .maybeSingle();

// WRONG ❌ — Throws PGRST116 if not found
const { data } = await supabase
  .from('products')
  .select('*')
  .eq('id', productId)
  .single();
```

### Pattern 3: Handle Both Visibility Flags (Products)

```typescript
// Products have TWO flags: is_active AND is_visible_in_store
.eq('is_active', true)
.eq('is_visible_in_store', true)  // Don't forget this one
```

### Pattern 4: Soft Delete Query Pattern

```typescript
// Active records only
.is('deleted_at', null)

// Include soft-deleted (omit filter)
```

## Step-by-Step Workflows

### Workflow 1: Creating a New Table

1. **Define columns** following naming conventions
2. **Add tenant_id** (unless global table)
3. **Create indexes**: tenant_id, tenant+created, tenant+status
4. **Enable RLS**
5. **Add policies**: service_role + tenant_isolation
6. **Test queries** with and without tenant filter
7. **Run validation**: `scaffold-validator db-guardian --check tenant-filter`

### Workflow 2: Adding an Index

1. **Identify query pattern** using Index Decision Tree
2. **Check table size** — skip if < 1000 rows
3. **Write idempotent SQL**: `CREATE INDEX IF NOT EXISTS`
4. **Test in staging** first
5. **Deploy to production**

### Workflow 3: Debugging "Data Not Showing Up"

1. **Check ghost columns**: Query `information_schema.columns`
2. **Check error object**: `console.error(error)` not just `!data`
3. **Verify tenant_id filter**: Is it applied?
4. **Check RLS policies**: Are they blocking?
5. **Check soft delete**: Is `deleted_at` filtered?

---

# Part 3: Quality Standards

## Pre-Delivery Checklist

Before deploying any database change:

- [ ] All `.from()` calls have `.eq('tenant_id', tenantId)` (except global tables)
- [ ] New tables have `tenant_id UUID NOT NULL REFERENCES tenants(id)`
- [ ] RLS is enabled on new tables
- [ ] Both `service_role` and `tenant_isolation` policies exist
- [ ] Indexes created with `IF NOT EXISTS`
- [ ] Migrations use idempotent syntax (`CREATE TABLE IF NOT EXISTS`)
- [ ] Timestamp columns use `TIMESTAMPTZ`, not `TIMESTAMP`
- [ ] Foreign keys follow `{entity}_id` naming convention
- [ ] Soft delete columns use `deleted_at TIMESTAMPTZ`
- [ ] Tested queries return expected data

## Validation Commands

```bash
# Check all queries have tenant filter
scaffold-validator db-guardian --check tenant-filter --path lib/

# Verify RLS enabled on all business tables
scaffold-validator db-guardian --check rls-enabled

# Audit indexes on high-traffic tables
scaffold-validator db-guardian --check indexes --table products

# Manual audit (fallback until CLI ready)
grep -rn '\.from(' app/ lib/ --include='*.ts' --include='*.tsx' | grep -v 'Array.from' | grep -v 'tenants'
```

## Common Errors

| Error | Symptom | Cause | Fix |
|-------|---------|-------|-----|
| **Ghost Column (LX-GHOST-001)** | Query returns `null`, no error | `.select()` references non-existent column | Audit schema: `SELECT column_name FROM information_schema.columns` |
| **Tenant Leak** | User sees other tenant's data | Missing `.eq('tenant_id', ...)` | Add tenant filter to query |
| **PGRST116** | "Results contain 0 rows" error | Using `.single()` when row might not exist | Use `.maybeSingle()` instead |
| **RLS Block** | "new row violates row-level security policy" | RLS enabled but no matching policy | Add service_role policy or fix tenant_id |
| **Type Mismatch** | "invalid input syntax" on timestamp | Mixing `timestamp` and `timestamptz` | Convert to `timestamptz` |
| **Orphan Records** | Data without tenant_id | Missing `NOT NULL` constraint or bad insert | Add constraint, fix insert logic |

---

# Part 4: Technical Reference

## Column Naming Conventions

| Convention | Do ✅ | Don't ❌ |
|-----------|-------|----------|
| **Timestamps** | `created_at`, `updated_at`, `deleted_at` | `createdAt`, `date_created` |
| **Type** | Always `TIMESTAMPTZ` | Never bare `TIMESTAMP` |
| **Foreign keys** | `{entity}_id` (e.g., `property_id`) | `propertyId`, `prop_id` |
| **Booleans** | `is_active`, `is_visible` | `active`, `visible` |
| **Status** | `status TEXT` with string values | `status INTEGER` with magic numbers |
| **Money** | `NUMERIC` | Never `FLOAT` or `REAL` |
| **IDs** | `UUID` with `gen_random_uuid()` | Serial integers |
| **Slugs** | `slug TEXT` + unique per tenant | Slug derived at runtime |

## Index Decision Tree

```
New query pattern?
  │
  ├─ Filters by tenant_id only → idx_{table}_tenant_id
  │
  ├─ Filters by tenant_id + status → idx_{table}_tenant_status
  │
  ├─ Filters by tenant_id + date range → idx_{table}_tenant_{date_col} DESC
  │
  ├─ ILIKE '%search%' on text → GIN index with gin_trgm_ops
  │     (requires: CREATE EXTENSION IF NOT EXISTS pg_trgm)
  │
  ├─ Exact match on foreign key → idx_{table}_{fk_col}
  │
  ├─ Only active/published rows → Partial index: WHERE status = 'active'
  │
  ├─ Unique business rule → UNIQUE on (tenant_id, {column})
  │
  └─ Vector similarity → ivfflat or hnsw on embedding column
```

**When NOT to add an index:**
- Table has < 1000 rows
- Column has very low cardinality (e.g., boolean 50/50 split)
- Write-heavy table with rare reads

## Integration with Other Skills

| Skill | When to Use Together | Pattern |
|-------|---------------------|---------|
| `migration-generator` | Creating SQL migrations | Write migration → Validate with db-guardian |
| `api-error-handler` | Handling DB errors in API routes | Use `guardSupabase()` wrapper |
| `debug-epistemologist` | Debugging complex DB issues | Apply 5 questions to schema problems |
| `security-auditor` | Security audit before deploy | Check RLS policies after schema changes |

## Diagnostic Queries

```sql
-- All indexes on a table
SELECT indexname, indexdef FROM pg_indexes
WHERE tablename = 'YOUR_TABLE' AND schemaname = 'public';

-- RLS status across all tables
SELECT tablename, rowsecurity FROM pg_tables
WHERE schemaname = 'public' ORDER BY tablename;

-- All policies on a table
SELECT policyname, permissive, roles, cmd, qual
FROM pg_policies WHERE tablename = 'YOUR_TABLE';

-- Column names (ghost column debugging)
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'YOUR_TABLE' ORDER BY ordinal_position;

-- Orphaned records (no tenant)
SELECT count(*) FROM YOUR_TABLE WHERE tenant_id IS NULL;

-- Timestamp type audit
SELECT table_name, column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public' AND data_type LIKE '%timestamp%'
ORDER BY table_name;
```

## Current Schema Health

### ✅ Solid (No Action Needed)
- 100% RLS coverage across all 35 tables
- Composite indexes on high-traffic patterns
- Trigram search on products, properties, clients
- Vector search infrastructure (pgvector + ivfflat)
- Audit trail (operations_history, admin_login_attempts, system_logs)

### ⚠️ Known Debt (Fix When Convenient)
| Item | Impact | Fix |
|------|--------|-----|
| `clients.deleted_at` type | Low | `ALTER TABLE clients ALTER COLUMN deleted_at TYPE timestamptz;` |
| `products.deleted_at` type | Low | `ALTER TABLE products ALTER COLUMN deleted_at TYPE timestamptz;` |
| Queries dispersed in API routes | Medium | Centralize into Repository Pattern |
| Duplicate policies | None | Cosmetic — both `service_role_access` and `service_role_all_{table}` exist |

---

## Information Gaps — Catastro

| Sección | Estado | Qué falta definir |
|---------|--------|-------------------|
| Validation Commands | ⚠️ | Crear `scaffold-validator` CLI primero. Los comandos están especificados pero no implementados. |
| Cross-references | ✅ | Completas |
| Known Debt fixes | ⚠️ | Priorizar y asignar fecha para fixes de `deleted_at` type |

---

*Skill: db-guardian — v2.0*  
*Template: 4-Part Structure*  
*Validation: Pending CLI implementation*
