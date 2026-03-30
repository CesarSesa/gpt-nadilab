# AGENTS.md — Nadistudio Scaffold

> **Operational Constitution for LLMs/Agents**
> **Project:** Castle Nadistudio v6.3.1
> **Version:** 1.2 | Updated: 2026-03-28
> **Language rule:** Code in English, UI/prompts in Spanish, docs in English

---

## THE PROJECT IN 30 SECONDS

**Nadistudio Scaffold** is a multi-tenant, multi-vertical SaaS platform that lets small business owners (property brokers, clothing store owners) manage their inventory via Telegram + automatic web catalog.

**The promise:** "Talk to your business. Send photos. The system does the rest."

**Two active verticals:**
- `retail-clothing` — 1 photo -> N products (expansion)
- `real-estate` — N photos -> 1 property (collapse)

**Stack:** Next.js 15 + Supabase (cloud) + Vercel + Kimi Vision API

---

## GOLDEN PRINCIPLES

1. **"Keep it real"** — Always distinguish EXISTS vs DESIGNED. Never present planned features as working.
2. **"No other dev"** — Everything must be executable by 1 person + LLMs. No hidden dependencies.
3. **"3 AM Rule"** — Every file, every flow, every error must be debuggable without prior context.
4. **"Max 2 abstraction levels"** — `page.tsx -> lib/query.ts -> supabase` = OK. Four hops = NO.
5. **"Miche Test"** — Can a non-technical shop owner do this without help? If not, simplify.
6. **"Debug epistemology"** — Always ask: "Did this EVER work, or has it NEVER worked?" before debugging.

---

## DESTRUCTION PROTECTION RULE

**NO LLM may:** Delete files, drop tables/columns, delete migrations, force push, remove auth logic.

**Without explicit 👍 from Nadi.** Lost context = duplicated work.

### The 5 NO-GOs (Automatic Escalation)

| # | Action | Ask Nadi |
|---|--------|----------|
| 1 | Delete any file/folder | Always |
| 2 | `DROP` table/column | Always |
| 3 | Force push / rewrite history | Always |
| 4 | Change middleware/auth logic | Always |
| 5 | Multi-table migration | Always |

---

## PAUSE POINTS (Stop & Ask)

Pause when:
- Multiple valid approaches (A vs B vs C)
- Breaking change risk (affects >3 files)
- Time exceeds appetite (2x original estimate)
- Context uncertainty (<80% confidence)

**The 30-Second Rule:** Can't articulate next 3 steps → STOP, ask Nadi.

---

## SUPABASE MCP: QUERIES & MIGRATIONS

The project has access to the Supabase MCP server, allowing direct database interaction. **Use it wisely. Queries are free, migrations cost trust.**

### Available Commands

| Command | Purpose | Safe to use? |
|---------|---------|--------------|
| `execute_sql` | SELECT queries on any table | ✅ **YES** — For investigation |
| `apply_migration` | DDL changes (CREATE, ALTER, DROP) | ⚠️ **REQUIRES NADI APPROVAL** |
| `list_tables` | Explore schema structure | ✅ **YES** |
| `list_migrations` | Check migration history | ✅ **YES** |
| `get_logs` | Debug API/auth/postgres issues | ✅ **YES** |
| `get_advisors` | Security/performance recommendations | ✅ **YES** |

### RULE Q1: Query Freely, Migrate Never Without Approval

**✅ GREEN ZONE — Queries (READ-ONLY)**
- You MAY query any table to understand the data structure
- You MAY use `execute_sql` with `SELECT` statements anytime
- Use this to disambiguate: "What columns does this table have?" "What rubro_type values exist?"
- **No approval needed.** Go wild.

**🔴 RED ZONE — Migrations (WRITE/DDL)**
- You MAY NOT execute `apply_migration` without explicit, written Nadi approval
- **Procedure for migrations:**
  1. STOP. Do not run the command.
  2. Explain the proposal to Nadi in natural language:
     - "I need to add a `captadora_config` JSONB column to `tenants`"
     - "Reason: Store Telegram bot webhook settings per tenant"
     - "SQL: `ALTER TABLE tenants ADD COLUMN captadora_config JSONB DEFAULT '{}'`"
  3. Wait for **explicit written approval** (e.g., "Approved, go ahead" or 👍)
  4. Only then execute `apply_migration`
  5. Document the migration name and purpose

### RULE Q2: No Blind Migrations

**Never** generate migrations from assumptions. If you think a column should exist:
1. Query first: `SELECT * FROM table LIMIT 1`
2. Verify the current schema
3. Explain the gap to Nadi
4. Get approval
5. Then migrate

### RULE Q3: Migration Naming

If approved, use descriptive snake_case names:
- ✅ `add_captadora_config_to_tenants`
- ✅ `create_property_views_table`
- ❌ `fix_stuff`
- ❌ `migration_001`

---

## DATABASE SCHEMA REFERENCE

> **⚠️ CRITICAL: This is the ONLY valid database URL**
> 
> **Supabase URL:** `https://wpstagyqnmqdlfjmlzoo.supabase.co`
> 
> **Project ID:** `jtxxmppzlyaswqcjbojh`
> 
> **Rule:** Any query to a different Supabase URL is WRONG. Stop and ask Nadi if unsure.

**Full documentation:** `.final-state-Scaffold/Our-updated-unified-DB.md`

When investigating data structures, query these tables (SELECT only):

### Critical Tables (Know these by heart)

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `tenants` | Multi-tenancy core — everything filters by `tenant_id` | `id`, `slug`, `rubro_type`, `plan`, `is_active` |
| `products` | Retail catalog items | `id`, `tenant_id`, `name`, `stock_quantity`, `status`, `deleted_at` |
| `properties` | Real-estate listings | `id`, `tenant_id`, `title`, `commune`, `price`, `status`, `deleted_at` |
| `product_drafts` | Retail photo-to-product pipeline | `id`, `tenant_id`, `status`, `batch_id`, `published_product_id` |
| `property_drafts` | Real-estate photo-to-property pipeline | `id`, `tenant_id`, `status`, `published_property_id` |
| `media_queue` | Universal intake queue — ALL content enters here | `id`, `tenant_id`, `status`, `rubro_context`, `source_type` |
| `entity_images` | Published images with full traceability | `id`, `tenant_id`, `entity_type`, `entity_id`, `source_queue_id` |
| `sales` / `sale_items` | Retail POS transactions | `id`, `tenant_id`, `customer_id`, `total_amount`, `status` |
| `clients` / `property_inquiries` | Real-estate CRM | `id`, `tenant_id`, `status`, `assigned_to` |

### Status Enums (Common Values)

```
product_drafts:    pending → pending_analysis → auto_detected → in_review → approved → published
property_drafts:   pending_analysis → auto_detected → in_review → ready_for_approval → approved → published
media_queue:       pending → accumulating → processing → completed | failed | expired
products:          active | archived
properties:        draft | published | archived | reserved
sales:             completed
clients:           new | contacted | visit_scheduled | negotiating | closed_won | closed_lost
```

### Quick Query Patterns

```sql
-- Check tenant exists by slug
SELECT * FROM tenants WHERE slug = 'demo-moda';

-- Get product with images
SELECT p.*, ei.public_url as main_image 
FROM products p 
LEFT JOIN entity_images ei ON ei.entity_id = p.id AND ei.sort_order = 0
WHERE p.tenant_id = 'uuid' AND p.status = 'active';

-- Check draft pipeline status
SELECT status, COUNT(*) FROM product_drafts 
WHERE tenant_id = 'uuid' GROUP BY status;

-- Find stuck media_queue items
SELECT * FROM media_queue 
WHERE status = 'processing' AND expires_at < now();
```

---

## ARCHITECTURE: THE 6 LAYERS (L0-L5)

```
L0: OBSERVABILITY    -> Tower (YAMATO CONTROL /ops/*)
L1: COMMUNICATION    -> PBX Switchboard (webhooks), Natural Agent (Kimi)
L2: TENANT OPS       -> Marketplace (storefronts), Tailor Robot (6 commands)
L3: PROTECTION       -> The Wall (middleware/auth), The Moat (RLS)
L4: DATA             -> The Gallery (Storage/entity_images), The Vaults (PostgreSQL)
L5: EXPERIMENTATION  -> The Basement (lab)
```

**External:** Alchemist Workshop (Kimi Vision API)

---

## ROUTE STRUCTURE

> **READ FIRST:** `ARCHITECTURE-RULES.md` — 15 rules before touching code.

### Multi-tenant (Clients)
```
{slug}.nadistudio.cl/                    -> Public catalog
{slug}.nadistudio.cl/admin               -> Redirect to /auth/login
{slug}.nadistudio.cl/admin/{feature}     -> Middleware rewrite to /admin/{vertical}/{feature}

Internal structure:
app/admin/[vertical]/                    -> [vertical] = "retail" | "real-estate"
  retail/dashboard/                      -> Retail dashboard
  retail/catalogo/                       -> Products
  retail/ventas/                         -> Retail only
  real-estate/dashboard/                 -> Real-estate dashboard
  real-estate/catalogo/                  -> Properties
  real-estate/pipeline/                  -> Real-estate only
```

### Operations Center (Nadi only)
```
nadistudio.cl/ops           -> COMMAND (War Room)
nadistudio.cl/ops/health    -> RAID FRAMES (health dashboard)
nadistudio.cl/ops/tenants   -> Tenant management
nadistudio.cl/ops/system    -> ENGINE (bots, DB status)
```

### Auth Flow
```
1. Login -> API sets cookies: admin_auth_{slug} + tenant_rubro
2. /admin/dashboard -> Middleware reads tenant_rubro -> rewrite to /admin/retail/dashboard
3. app/admin/[vertical]/layout.tsx verifies auth cookie -> [vertical]="retail"
4. app/admin/[vertical]/retail/dashboard/page.tsx renders
5. User sees: /admin/dashboard (clean URL, no vertical exposed)
```

---

## KEY DIRECTORIES

| Path | Contents | Touch with caution |
|------|----------|--------------------|
| `app/admin/page.tsx` | Unified /admin router | CRITICAL — detects rubro |
| `app/admin/[vertical]/layout.tsx` | Auth gate (cookie check) | ONLY auth checkpoint |
| `middleware.ts` | URL rewriting, subdomain detection | ONLY routing owner |
| `app/ops/health/` | RAID FRAMES dashboard | Consult before changing metrics |
| `lib/bot/kimi.ts` | Kimi API config | Temperature 0.6, thinking disabled |
| `lib/supabase/client.ts` | Bot client + query functions | Tenant isolation here |

---

## CODE CONVENTIONS

### TypeScript
- Always `strict: true`
- Named exports (no default exports)
- Prefer async/await over .then()
- Code and variable names: English
- UI labels, buttons, user messages: Spanish

### Database
- **ALWAYS** filter by `tenant_id` in queries (D1)
- **ALWAYS** filter by `rubro_type` in `getUserTenant()` (D2)
- Use `service_role` key in webhooks, never `anon`
- Soft delete pattern: `status: 'archived'` + `deleted_at` + `is_active: false`

### Storage Paths
```typescript
// CORRECT:
`${tenantSlug}/products/${productSlug}/1.jpg`

// WRONG (BUG-005):
`${tenantId}/products/${productSlug}/1.jpg`  // UUID = unreadable
```

### React Components
- Server Components for initial fetch
- Client Components for interactivity (hooks)
- No Server Actions yet (API route compatibility)

### API Response Format
```
Success: { success: true, data: {...} }
Error:   { error: 'Human readable message' }
```

### Slug-to-UUID Resolution
Any API that receives `tenantId` from the frontend MUST resolve slug to UUID:
```typescript
const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
if (!uuidRegex.test(tenantId)) {
  // resolve via tenants table
}
```

---

## TWO-LAYER AGENT ARCHITECTURE

The system uses **two orthogonal layers** for agent capabilities:

```
┌─────────────────────────────────────────┐
│  LAYER 1: PERSONAS (Who you are)        │
│  Location: .final-state-Scaffold/Profiles/ │
│  Activation: Manual by name or trigger  │
│  Purpose: Define MODE OF OPERATION      │
└──────────────┬──────────────────────────┘
               │ uses
┌──────────────▼──────────────────────────┐
│  LAYER 2: SKILLS (What you do)          │
│  Location: .kimi/skills/                │
│  Activation: Auto-trigger or reference  │
│  Purpose: Define TECHNICAL PROCEDURES   │
└─────────────────────────────────────────┘
```

### Layer 1: Personas (Profiles)

**Purpose:** Define WHO you are when acting — your voice, priorities, and decision framework.

**Activation triggers:**

| Trigger Words | Activate Persona | Location |
|--------------|------------------|----------|
| "audit", "review quality", "check risks", "validate" | **Auditor** | `auditor.md` |
| "implement", "code this", "build UI", "fix bug" | **Coder** | `coder.md` |
| "design API", "schema decision", "architect" | **Designer-Architect** | `designer-architect.md` |
| "organize", "track tasks", "coordinate", "secretario" | **Secretary** | `secretary.md` |
| "Nadi decides", "ask director", "product decision" | **Director** | Nadi (human) |

**Key principle:** Personas provide **context and voice**. Auditor sees risks first; Coder sees implementation first; Secretary sees coordination first. Same technical problem, different lens.

**Fluidity:** Personas are "your alignment, not your prison." Any agent can override with explicit marker: `// OVERRIDE: Acting as Coder for emergency fix`.

---

### Layer 2: Skills (.kimi/skills/)

**Purpose:** Define HOW to do X correctly — templates, checklists, validation rules.

**Activation triggers:**

| Trigger Words | Activate Skill | Key Constraints |
|--------------|----------------|-----------------|
| "create table", "migration", "schema change" | **db-guardian** | `tenant_id` required, RLS mandatory |
| "bot command", "natural language", "intent" | **irce-engineer** | Regex first, LLM fallback |
| "whatsapp", "new channel", "bridge" | **channel-adapter** | 3-change rule only |
| "bug", "error", "not working", "debug" | **debug-epistemologist** | 5 questions before touching code |
| "error handling", "API route", "guard" | **api-error-handler** | `guardSupabase()`, `withErrorHandler()` |
| "new product", "PRD", "4 docs" | **4-step-docs** | Must complete all 4 before coding |
| "photo pipeline", "M1-M9", "batch" | **m1-m9-pipeline** | 9 machines, strict order |
| "migration SQL", "CREATE TABLE" | **migration-generator** | `IF NOT EXISTS`, idempotent |

**Key principle:** Skills are **procedural knowledge**. A Coder uses `db-guardian` when writing queries; an Auditor uses `db-guardian` when validating them.

---

### Cross-Reference Matrix

| Task | Primary Persona | Relevant Skills |
|------|-----------------|-----------------|
| Implement WhatsApp bot | Coder | channel-adapter, irce-engineer, api-error-handler |
| Review DB schema change | Auditor | db-guardian, migration-generator |
| Debug production issue | Any | debug-epistemologist, db-guardian |
| Design new vertical API | Designer-Architect | 4-step-docs, vertical-factory |
| Track sprint progress | Secretary | (organization, no technical skills) |
| Add bot natural language | Coder | irce-engineer, kimi-prompt-engineer |

---

### Manual Activation (File Injection)

When explicit context is needed:

```bash
# Activate specific persona
cat .final-state-Scaffold/Profiles/auditor.md | kimi

# Reference specific skill  
cat .kimi/skills/channel-adapter/SKILL.md | kimi

# Combined: Auditor reviewing channel bridge
(cat .final-state-Scaffold/Profiles/auditor.md && echo "---" && cat .kimi/skills/channel-adapter/SKILL.md) | kimi
```

---

## TEAM ROLES

| Role | Primary Focus | Can Also Do |
|------|---------------|-------------|
| **Director (Nadi)** | Priorities, vision, product decisions, destructive action approval | Everything |
| **Auditor** | Architecture review, quality control, code audit | Implement fixes |
| **Coder** | UI implementation, frontend components | Light backend fixes |
| **Designer-Architect** | Backend architecture, APIs, DB schemas | UI when necessary |
| **Secretary** | Organization, tracking, documentation, coordination | Audit, report |

**Interchangeability rule:** Roles are fluid. If the Secretary has free context and an audit is needed, the Secretary audits. If Coder is at 75% and a fix is needed, another agent can investigate and Coder implements. Nadi directs the orchestra.

**Communication protocol:** Agents can communicate for speed, but architectural decisions flow through: Agent -> Auditor -> Director.

**Persona vs Skill:** A Coder (persona) can use db-guardian (skill) for implementation. An Auditor (persona) can use db-guardian (skill) for validation. The persona defines WHO; the skill defines HOW.

---

## THREE-STATES FRAMEWORK

Every task must pass through 3 states + 1 constraint:

```
HOW WE WANT IT TO WORK     -> Target vision
        |
HOW IT ACTUALLY WORKS       -> Real diagnosis (verify first!)
        |
HOW WE MAKE IT WORK         -> Technical plan
        |
APPETITE: X hours/days      -> Time box (exceeded? replan or descope)
```

**Time box prevents scope creep.** If estimate doubles, stop and reassess with Nadi.

---

## PLANNING & DOCUMENTATION SYSTEM

| Resource | Path | How to use |
|----------|------|------------|
| **Planner DB** | `00.general-secretary/nadistudio-planner.db` | SQLite — open with DB Browser or query via Python. Source of truth for tasks, features, conflicts, env vars, integrations, skills, playbooks, architecture rules |
| **Institutional Book** | `python book.py` | Compiles AGENTS + Castle + Playbooks + Profiles into one versioned .md. Archives old versions automatically |
| **PLANIFICACIONES** | `00.general-secretary/00.PLANIFICACIONES POR IMPLEMENTAR.md` | Human-readable mirror of planner DB tasks |
| **CONFLICTOS** | `00.general-secretary/00.CONFLICTOS ABIERTOS.md` | Open issues log (being migrated to planner DB `conflicts` table) |

### Planner DB tables (12)
`projects` `categories` `tasks` `conflicts` `decisions` `env_vars` `features` `documents` `arch_rules` `integrations` `playbooks` `skills`

### Quick planner queries
```sql
-- P0/P1 tasks right now
SELECT priority, title, status, assigned_to FROM tasks WHERE priority IN ('P0','P1') AND status != 'done';

-- Feature health (expectation vs reality)
SELECT name, status, expectation, reality FROM features ORDER BY status;
```

---

## LIVE CONTEXT (Check Before Acting)

- [ ] Last commit: `git log origin/main -1 --oneline`
- [ ] Uncommitted: `git status --short`
- [ ] Active agents: Check `00.NEWS-FOR-THE-TEAM/LIVE-*.md`
- [ ] Blockers: `SELECT * FROM conflicts WHERE status != 'resolved'` (planner DB)

**If someone else works on your target files → Coordinate first.**

---

## FOR LLMs WORKING WITH NADI

### Before starting:
1. Read this AGENTS.md + `ARCHITECTURE-RULES.md`
2. Check Live Context above
3. Ask: "Priority mode today? Deep work, coordination, or crisis?"

### Before committing:
1. Generate message following commit conventions
2. Reference documentation if relevant (DOC: path/to/doc.md)
3. Mention bugs if applicable (ERROR: BUG-XXX)

### Before ANY destructive action:
1. **STOP.** Re-read the Destruction Protection Rule above.
2. Ask Nadi for explicit written confirmation.
3. Document what was destroyed and why.

---

## ADDING A NEW VERTICAL (e.g., restaurant)

1. Create `app/admin/[vertical]/restaurant/` with layout + pages
2. Add case in `middleware.ts` (map rubro to vertical)
3. Public URL stays `/admin/dashboard` (clean, no vertical exposed)
4. Fill out a Vertical Lego Template (see `.docs/Castle-gets-real/`)

---

## CURRENT STATE (2026-03-29)

- **Build:** Passing (both repos)
- **Deploy:** Scaffold on Vercel (nadistudio.cl wildcards + nadi.cafe ops)
- **Multi-vertical routing:** Option C working (cookies + rewrite)
- **Critical bugs:** Captadora auth (mi-catalogo) — table name + auth method mismatch
- **Active tenants:** 5 (redproperty, tu-stilo, demo-inmo, demo-moda, yamato-lab)
- **E2E retail sales:** Validated (stock decrements automatically)
- **CRUD PUT/DELETE:** Implemented for products + properties
- **Dual deploy:** `DEPLOY_TARGET=public` (nadistudio.cl) / `DEPLOY_TARGET=ops` (nadi.cafe)

---

## E2E TESTING (Playwright)

**Config:** `playwright.config.ts`
**Test dir:** `tests/e2e/` (auth, retail, real-estate projects)
**Report:** `playwright-report/` (gitignored, local-only HTML artifact)
**Reporter:** `['list', ['html', { open: 'never' }]]`

### Running tests
```bash
npx playwright test              # all projects
npx playwright test --project=retail  # single vertical
npx playwright show-report       # view last HTML report
```

### ⚠️ STALE URLs (needs update)
The config currently points to **deleted tenants**:
- `RETAIL_URL = 'https://tiendaderopa-try2.nadistudio.cl'` → should be `demo-moda.nadistudio.cl`
- `REALESTATE_URL = 'https://inmotry2.nadistudio.cl'` → should be `demo-inmo.nadistudio.cl`

**Rule:** After tenant cleanup, always update `playwright.config.ts` URLs.

---

## EXTERNAL DOCUMENTATION

```
C:\Users\nadil\Repos\.docs\Castle-gets-real\
  .diagrams-json-manifests-and-pretty-things/
    .planos-generales-castillo/     -> FLUJO-*.md
    .for-specific-problems/         -> MAQUINA-*.md, BUG-*.md
    .by-vertical/                   -> Vertical Lego Templates
    INDEX-MAESTRO-DE-NAVEGACION.md  -> Castle GPS
  .ethos-and-work-guidelines/
    COMMIT_CONVENTIONS.md           -> Commit formats
```

---

*AGENTS.md v1.1 — The constitution. Read it, follow it, build on it.*
