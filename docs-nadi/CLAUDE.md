# CLAUDE.md — Nadistudio Scaffold
**Rubro:** multi-vertical (retail-clothing, real-estate)
**Stack:** Next.js 15 + Supabase (shared DB) + Vercel + Kimi K2.5
**Bots:** Telegram webhooks per vertical at `/api/bot/webhook/*`

---

## This is the platform scaffold

All verticals share this codebase. mi-catalogo is the external real-estate lab.
When in doubt about multi-tenant patterns, look here. When in doubt about real-estate specifics, look at mi-catalogo.

---

## Key file map

```
middleware.ts                              <- Domain routing, auth, DEPLOY_TARGET blocking
app/api/admin/                             <- 25+ admin API routes (all check admin_auth_{slug} cookie)
app/api/bot/                               <- Telegram bot webhook + processing
app/api/batches/                           <- Photo batch processing pipeline
app/api/cron/                              <- Scheduled jobs (cleanup-queues)
app/api/tenant/                            <- Public tenant data API
app/ops/                                   <- Ops center (nadi.cafe only)
app/wizard/                                <- Tenant creation wizard (nadi.cafe only)
app/admin/                                 <- Tenant admin panels (retail + real-estate)
app/(marketing)/                           <- Landing page (scaffold root)
lib/bot/                                   <- Bot logic, handlers, photo sessions
lib/themes/                                <- 5-theme system (ThemeProvider + CSS vars)
lib/supabase/                              <- Server/client Supabase helpers
lib/auth/                                  <- Auth utilities
lib/schemas/                               <- Zod validation schemas
lib/image-systems/                         <- Image processing utilities
lib/cart/                                  <- Shopping cart logic (retail)
.final-state-Scaffold/                     <- Documentation hub (Castle, SOUL, playbooks, profiles, templates)
00.general-secretary/                      <- Planning (SQLite DB, planificaciones, conflictos)
00.general-secretary/nadistudio-planner.db <- Source of truth for tasks, features, conflicts
book.py                                    <- Institutional book compiler (run: python book.py)
```

---

## Dual deploy architecture

| Target | Domain | `DEPLOY_TARGET` | Blocks |
|--------|--------|-----------------|--------|
| Public | nadistudio.cl (wildcards) | `public` | /ops, /wizard, /api/admin/vercel-logs |
| Ops | nadi.cafe | `ops` | Nothing (full access) |

Same repo, two Vercel projects, different env var.

---

## Multi-tenant routing

Middleware resolves tenant from subdomain: `{slug}.nadistudio.cl` -> reads `tenants` table -> sets cookie -> rewrites routes.

```
Request: demo-moda.nadistudio.cl/admin/dashboard
  -> middleware extracts slug "demo-moda"
  -> looks up tenant (rubro: retail-clothing)
  -> sets admin_auth_{slug} cookie
  -> rewrites to /admin/dashboard with tenant context
```

---

## Auth pattern

**Ops center:** `NADI_ADMIN_KEY` env var (query param `?key=` sets 7-day cookie)
**Tenant admin:** `ADMIN_PASSWORD` per tenant (stored in `tenants.admin_password`)
**Per-tenant cookie:** `admin_auth_{slug}` (httpOnly, sameSite: lax, 7-day expiry)
**API routes:** All `/api/admin/*` verify the cookie server-side

---

## Critical error patterns

| Pattern | Risk | Fix |
|---|---|---|
| `.single()` on optional row | PGRST116 crash | Use `.maybeSingle()` |
| Supabase `.catch()` | Silent swallow (PromiseLike) | Always destructure `{ data, error }` |
| Ghost column in `.select()` | PostgREST 400 swallowed | Verify column exists in DB |
| Kimi temp != 0.6 or thinking enabled | Garbage output | ONLY `temperature: 0.6` + `thinking: { type: "disabled" }` |
| `await` missing on `params` (Next.js 15) | Runtime crash | `const { id } = await params` |
| Fire-and-forget fetch | Killed by Vercel | Use `after()` or `waitUntil()` |

---

## Planning & documentation system

| Resource | Path | Purpose |
|----------|------|---------|
| **Planner DB** | `00.general-secretary/nadistudio-planner.db` | SQLite source of truth (tasks, features, conflicts, env vars, skills, docs) |
| **Institutional Book** | `python book.py` | Compiles all docs into versioned book |
| **AGENTS.md** | Root | Operational constitution |
| **ARCHITECTURE-RULES.md** | Root | Technical rules |
| **Castle v8** | `.final-state-Scaffold/` | Architecture map |
| **Playbooks** | `.final-state-Scaffold/playbooks/` | 9 operational guides (6 done) |
| **Templates** | `.final-state-Scaffold/templates/` | Copy-paste code recipes for agents |
| **Profiles** | `.final-state-Scaffold/Profiles/` | Agent personas |

---

## Active tenants (5)

| Slug | Name | Rubro | Notes |
|------|------|-------|-------|
| redproperty | RedPropertyChile | real-estate | Production (also in mi-catalogo) |
| tu-stilo | Tu Stilo | retail-clothing | Miche's future store |
| demo-inmo | Demo Inmobiliaria | real-estate | E2E test target |
| demo-moda | Demo Moda | retail-clothing | E2E test target |
| yamato-lab | Yamato Lab | retail-clothing | Vector/AI playground |

---

## Env vars required

```
NEXT_PUBLIC_SUPABASE_URL          # Shared Supabase project URL
NEXT_PUBLIC_SUPABASE_ANON_KEY     # Public anon key
SUPABASE_SERVICE_ROLE_KEY         # Secret service role key
TELEGRAM_BOT_TOKEN                # Bot token from @BotFather
KIMI_API_KEY                      # Moonshot API key
NADI_ADMIN_KEY                    # Ops center access key
ADMIN_PASSWORD                    # Default tenant admin password
DEPLOY_TARGET                     # "ops" or "public"
DEV_TENANT_SLUG                   # Local dev default tenant (e.g. demo-moda)
CRON_SECRET                       # Bearer token for /api/cron/* routes
NEXT_PUBLIC_SITE_URL              # Base URL for the deployment
```

---

## Anti-patterns — do not introduce

| Pattern | Correct approach |
|---|---|
| Hardcoded tenant UUID | Read from cookie/middleware context |
| Import from `proyecto-miche` | Use scaffold templates in `.final-state-Scaffold/templates/` |
| New CSS file per component | Tailwind only, inline styles, LEGO GRANDE rules |
| `console.log` in API routes | Use structured logging or remove |
| Skip `x-tenant-slug` header | Always pass tenant context in API calls |
| Destroy without asking Nadi | Read AGENTS.md Destruction Protection Rule |
