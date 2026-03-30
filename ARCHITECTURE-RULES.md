# ARCHITECTURE RULES — Nadistudio Scaffold

> 18 rules + 1 golden rule. Read BEFORE touching code. No exceptions.
> Version: 1.2 | Date: 2026-03-13 | Authors: Opus (auditor) + Nadi (architect)

---

## GOLDEN RULE: DESTRUCTION PROTECTION

**No LLM or agent may delete files, drop tables/columns, remove migrations, clean git history, force push, or remove auth logic without explicit written confirmation from Nadi.**

Reason: Lost context = duplicated work. Better to ask twice.

---

## ROUTING (R1-R5)

### R1: Route groups `(x)` do NOT create URL segments
Next.js route groups with parentheses are INVISIBLE in the URL.
`app/(admin)/[vertical]/` maps to `/{vertical}/`, NOT `/admin/{vertical}/`.
If you need `/admin` in the URL, use `app/admin/` (literal directory).

**Real case:** Created `app/(admin)/[vertical]/` expecting it to map to `/admin/{vertical}/dashboard`. The `[vertical]` captured "admin" instead of "retail". Login loop for hours.

### R2: Middleware is the ONLY owner of URL rewrites
`/admin/dashboard` -> `/admin/retail/dashboard` is done by middleware and NOBODY ELSE.
Do not create static pages that do `redirect()` to the same destination.
Two mechanisms for the same job = invisible bugs.

**Real case:** Created `app/admin/dashboard/page.tsx` with `redirect()` duplicating the middleware rewrite. Result: 404s.

### R3: One vertical = one directory under `app/admin/`
```
app/admin/
  retail/          <- retail-clothing
  real-estate/     <- real-estate
  restaurant/      <- future: just add directory + case in middleware
```
Adding a new vertical = 1 directory + 1 case in middleware. Nothing more.

**Note**: The structure uses static directories, not dynamic `[vertical]`. Middleware rewrites `/admin/dashboard` → `/admin/retail/dashboard` based on the `tenant_rubro` cookie.

### R4: Static routes shadow dynamic routes
If both `app/admin/dashboard/` and `app/admin/[vertical]/` exist at the same level, Next.js prioritizes `dashboard/` (static) over `[vertical]` (dynamic) when the URL is `/admin/dashboard`. This breaks rewrites. Never create static directories that collide with the `[vertical]` segment.

### R5: Clean external URLs, explicit internal URLs
```
User sees:      /admin/dashboard          (clean, no vertical)
System serves:  /admin/retail/dashboard   (explicit, with vertical)
```
Middleware handles the translation. The user NEVER sees the vertical in the URL.

---

## AUTH (A1-A3)

### A1: `[vertical]/layout.tsx` is the ONLY auth gate
Cookie verification `admin_auth_{slug}` happens in ONE place:
`app/admin/[vertical]/layout.tsx`. No other layout, page, or component
checks admin auth. If something needs protection, it lives under `[vertical]/`.

### A2: Cookies have fixed rules
```typescript
{
  httpOnly: true,                        // always
  secure: NODE_ENV === 'production',     // always
  sameSite: 'lax',                       // NEVER 'strict' (breaks post-login redirects)
  path: '/',                             // always root
}
```
`sameSite: 'strict'` blocks cookies after redirects. Already happened (LX-002). Don't repeat.

### A3: Cookie names are namespaced
```
admin_auth_{slug}    -> tenant auth (7 days)
tenant_rubro         -> vertical type for routing (1 day)
nadi_ops_key         -> /ops access (7 days)
```
Do not invent new cookies without documenting them here.

---

## DATA (D1-D3)

### D1: ALWAYS filter by tenant_id
Every query to any table that has `tenant_id` MUST filter by it.
No exceptions. RLS is the backup, not the first line of defense.

### D2: getUserTenant filters by rubro_type
Each bot filters by its own `rubro_type`:
```typescript
.eq('tenants.rubro_type', 'retail-clothing')  // ropero bot
.eq('tenants.rubro_type', 'real-estate')      // inmobiliario bot
```
A user can have tenants across multiple verticals. Each bot only sees its own.

### D3: Storage paths use slug, NEVER UUID
```
CORRECT:  {tenantSlug}/products/{productSlug}/1.jpg
WRONG:    {tenantId}/products/{productSlug}/1.jpg    // BUG-005, already happened
```

---

## CODE (C1-C5)

### C1: Code in English, UI in Spanish, Docs in English
Variables, functions, types, tables: English.
Labels, buttons, user-facing messages: Spanish.
Documentation and comments: English.
Commits: English type and scope, Spanish allowed in body.

### C2: Max 2 abstraction levels
If you need to understand 3+ files to follow a flow, it's too much.
`page.tsx -> lib/query.ts -> supabase` = 2 levels. OK.
`page.tsx -> hook.ts -> service.ts -> adapter.ts -> supabase` = 4 levels. NO.

### C3: Don't build features on broken foundations
Before building admin UI for a vertical, verify:
1. Routing reaches the correct page (middleware rewrite works)
2. Auth gate passes with valid cookie
3. API returns real data

If any fails, fix it FIRST. Don't build on top.

**Real case:** Built complete admin panels before verifying routing reached them. Everything functional but inaccessible.

### C4: Each layer has ONE job

| Layer | Job | Does NOT do |
|-------|-----|-------------|
| Middleware | Rewrite URLs, detect subdomain | Auth, render |
| `[vertical]/layout.tsx` | Auth gate | Routing, UI |
| Per-vertical layout | Sidebar, theme | Auth, routing |
| Page | Render content | Auth (already past the gate) |
| API route | CRUD, set cookies | Routing |

### C5: Slug-to-UUID resolution in all APIs
Any API receiving `tenantId` from the frontend MUST handle both slug and UUID:
```typescript
const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
if (!uuidRegex.test(tenantId)) {
  const { data: tenant } = await supabase
    .from('tenants').select('id').eq('slug', tenantId).single();
  if (!tenant) return NextResponse.json({ error: 'Tenant not found' }, { status: 404 });
  tenantUuid = tenant.id;
}
```
**Real case:** Frontend sent slug "demo-moda" as tenantId. PostgreSQL threw `invalid input syntax for type uuid`. Hours of debugging.

---

## COMMITS (CM1-CM3)

> **Philosophy:** Commits are the only documentation that never goes stale.  
> **Goal:** Maximum clarity for future reference. Disambiguate everything.

### CM1: Format is `type(scope): summary`

```
<type>(<scope>): <technical summary>

+++CONTEXT
Why. The problem before this commit.

+++CHANGES
- Specific change 1
- Specific change 2

+++DECISIONS
Explicit choices made and why. The "why" gets lost first.

+++LIMITS
Debt, edge cases, "this is mockup". Prevents false assumptions.

+++REFS
DOC: path/to/doc.md
ERROR: LX-XXX-XXX
```

| Section | Required | Purpose |
|---------|----------|---------|
| `CONTEXT` | Always | Problem being solved |
| `CHANGES` | Always | Bullet list of technical modifications |
| `DECISIONS` | Always | Design choices and rationale |
| `LIMITS` | Always | What's incomplete, mocked, or risky |
| `REFS` | If exists | Links to docs, errors, incidents |

### CM2: Types and scopes

| Type | Use | Example scope |
|------|-----|---------------|
| `feat` | New functionality | `api`, `ui`, `bot` |
| `fix` | Bug correction | `auth`, `db`, `routing` |
| `refactor` | Internal change, same behavior | `db`, `api` |
| `docs` | Documentation only | `api`, `soul`, `readme` |
| `test` | Tests | `api`, `e2e` |
| `chore` | Maintenance | `deps`, `config` |
| `hotfix` | Production urgent | `rls`, `auth` |

**Language:** English type/scope, Spanish allowed in body.

### CM3: Commit body rules

- **Max 5 lines per section** — if you need more, the commit is too big
- **Bullets for changes** — lists, not narratives
- **Never omit DECISIONS** — the "why" is critical
- **Never omit LIMITS** — honesty about what's missing
- **Sign collaborative work:** `Co-Authored-By: Name <email>`

**Anti-patterns:**
```
❌ "Hoy desperté con inspiración..." (poetry)
❌ "fix: Arregla bug en admin" (vague)
❌ "feat: Agrega cosas" (no context)
```

---

## PROCESS (P1-P2)

### P1: Validate the bridge before building the castle
For any structural or routing change:
1. Verify the URL reaches the correct file (`console.log` in the page)
2. Verify the auth gate passes with valid cookie
3. THEN build features

Don't build a castle on a bridge you haven't tested.

### P2: Three-states before coding
Every task passes through:
```
HOW WE WANT IT TO WORK      -> Target vision
HOW IT ACTUALLY WORKS        -> Real diagnosis
HOW WE MAKE IT WORK          -> Technical plan
```
Code only after all three are clear. This prevents partial solutions.

---

## DEBUG EPISTEMOLOGY

When facing a bug, always ask first:
1. **"Did this EVER work, or has it NEVER worked?"** — Debug path vs build path.
2. **"Is this deployed code or local code?"** — Stale deployments cause ghost bugs.
3. **"Is there a trigger or function doing this silently?"** — Check `information_schema.triggers`.
4. **"Are two mechanisms doing the same job?"** — Double-decrement, double-redirect, double-auth.

---

*19 rules. If you break one, document it in `00.CONFLICTOS ABIERTOS.md` with the reason.*
