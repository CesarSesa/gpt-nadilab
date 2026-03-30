# Scaffold Master Playbook — Patterns, Quirks & Hard-Won Lessons
## Everything we learned so the next LLM (or human) doesn't repeat our mistakes

*Generated: 2026-03-15 by Claude O4.6*
*Updated: 2026-03-23 — §73 batch-complete ghost columns fix, §65 product_drafts→products column mapping added*
*Source: Full codebase audit + battle scars from debugging sessions + legacy docs from proyecto-miche, mi-catalogo & past-try-destilation*

---

### The Playbook Standard

This document isn't just a reference for us. It's built so that **someone with zero context about this project can debug and build with it**. Not "a developer at 3 AM" — that still assumes familiarity. We mean a fresh LLM, a hired contractor, or Nadi six months from now after not touching the code.

That means every section needs three things:
1. **What the pattern is** — the rule itself
2. **Why we chose it over the alternative** — because the alternative often looks reasonable until it bites you
3. **How to detect when it's broken** — grep commands, diagnostic queries, symptoms

When two approaches seem equivalent (e.g., `after()` vs `waitUntil()`, soft delete vs hard delete, JSONB vs separate table), we document WHY we went with one. The "losing" option isn't wrong in general — it's wrong *for this specific infra and operational context*. Future readers need to understand that distinction so they don't "improve" things back into patterns we already escaped from.

---

## INDEX

1. [Tenant Resolution — The Three Tiers](#1-tenant-resolution)
2. [getUserTenant() — Cross-Rubro Isolation](#2-getusertenant-cross-rubro-isolation)
3. [Image Systems — Legacy vs Future (Dual Architecture)](#3-image-systems)
4. [Two-Phase Async Processing — Fire-and-Forget](#4-two-phase-async-processing)
5. [Server/Client Component Split — Data Passing](#5-serverclient-component-split)
6. [Admin Auth — Cookie-Based](#6-admin-auth)
7. [Bot Webhook — Message Processing Order](#7-bot-webhook-message-processing-order)
8. [Bot Webhook — Media Accumulation (media_group_id)](#8-media-accumulation)
9. [Bot Webhook — Callback Queries (Inline Buttons)](#9-callback-queries)
10. [Bot Webhook — Idempotence](#10-idempotence)
11. [Theme System](#11-theme-system)
12. [Supabase Clients — Which One When](#12-supabase-clients)
13. [LX Tenant Resolver (API Routes)](#13-lx-tenant-resolver)
14. [Status Values — The Source of Many Bugs](#14-status-values)
15. [WhatsApp URL Construction](#15-whatsapp-url-construction)
16. [Entity Images — No FK, No Join](#16-entity-images-no-fk)
17. [Vercel Limits & maxDuration](#17-vercel-limits)
18. [Danger Zones — Known Weak Points](#18-danger-zones)
19. [IRCE Pipeline — What's Missing (Transplant Pending)](#19-irce-pipeline)
20. [Phase Boundaries — Migration Markers](#20-phase-boundaries)

21. [Kimi API Configuration](#21-kimi-api-configuration)
22. [Soft Delete vs Hard Delete](#22-soft-delete-vs-hard-delete)
23. [Draft Button Actions](#23-draft-button-actions)
24. [Session Routing](#24-session-routing)
25. [Entity Resolver — Fuzzy Matching](#25-entity-resolver-fuzzy-matching)
26. [Confirmation Builder — Permissive Parser](#26-confirmation-builder-permissive-parser)
27. [Operation Executor — Atomicity](#27-operation-executor-atomicity)
28. [Multi-Tenant Bot — Activation Handshake](#28-multi-tenant-bot-activation-handshake)
29. [Architecture Decision Records](#29-architecture-decision-records)
30. [Master Pre-Flight Checklist](#30-master-pre-flight-checklist)
31. [Serverless Architecture](#31-serverless-architecture)
32. [Storage Bucket Organization](#32-storage-bucket-organization)
33. [Draft State Transitions](#33-draft-state-transitions)
34. [RLS Policy Template](#34-rls-policy-template)
35. [Supabase Query Robustness](#35-supabase-query-robustness)
36. [Kimi Configuration — Validation Matrix](#36-kimi-configuration-validation-matrix)
37. [Bot Message Pipeline — Immutable Sequence](#37-bot-message-pipeline)
38. [Dirty State & Navigation Guards](#38-dirty-state-navigation-guards)
39. [MCP — Model Context Protocol](#39-mcp-model-context-protocol)
40. [FK Migration](#40-fk-migration)
41. [Circuit Breaker — Kimi API Protection](#41-circuit-breaker)
42. [Rate Limiting — Per-User Abuse Prevention](#42-rate-limiting)
43. [Memory Leak Prevention — Bounded Collections](#43-memory-leak-prevention)
44. [Voice Correction — Chilean Spanish](#44-voice-correction)
45. [Image Optimization — Sharp Before Kimi](#45-image-optimization)
46. [Cross-Browser Drag & Drop — The Windows MIME Trap](#46-cross-browser-drag-drop)
47. [Split-Brain Prevention — Verify Working Directory](#47-split-brain-prevention)
48. [SSR Cache Data Leak](#48-ssr-cache-data-leak)
49. [SQL Forensics — Diagnostic Queries](#49-sql-forensics)
50. [Race Condition Detection — Optimistic Locking](#50-race-condition-detection)
51. [Supabase Connection Architecture](#51-supabase-connection-architecture)
52. [Fuzzy Command Matching](#52-fuzzy-command-matching)
53. [Debugging Methodology — The Guard Analogy](#53-debugging-methodology)
54. [TypeScript Build — The Silent Killer](#54-typescript-build)
55. [PowerShell Gotchas — Windows Dev](#55-powershell-gotchas)
56. [Turbopack Cache Corruption](#56-turbopack-cache-corruption)
57. [Supabase Migration Pattern](#57-supabase-migration-pattern)
58. [Security Pre-Checklist](#58-security-pre-checklist)
59. [Rubro Expansion — 7-Step Recipe](#59-rubro-expansion)
60. [Client Onboarding — 5 Minutes](#60-client-onboarding)
61. [Intelligence Layer — pgvector RAG](#61-intelligence-layer)
62. [Image System Unification — The Lego Vision](#62-image-system-unification)
63. [New Bot Creation — Webhook Template](#63-new-bot-creation)
64. [Anti-Hardcode Comprehensive Reference](#64-anti-hardcode-reference)
65. [Ghost Columns — Silent PostgREST Failures](#65-ghost-columns)
66. [safeInt() — Kimi Returns Floats for Integer Columns](#66-safeint)
67. [CHECK Constraints Must Match All Code Paths](#67-check-constraints)
68. [Dual-Table Archives](#68-dual-table-archives)
69. [Empty Body → request.json() Crash](#69-empty-body-crash)
70. [Polling Must Match Writer Status](#70-polling-status-match)
71. [Session Size Limits — Photo Upload Budget](#71-session-size-limits)
72. [Cron Jobs — Vercel Scheduled Functions](#72-cron-jobs)
73. [Batch-Complete Ghost Columns — The Silent Product Black Hole](#73-batch-complete-ghost-columns)

---

## 1. Tenant Resolution

**The chain**: Middleware reads the hostname and sets `x-tenant-slug` header on every request.

```
Request: https://demo-moda.nadistudio.cl/tienda
  → middleware extracts "demo-moda" from subdomain
  → sets header: x-tenant-slug = "demo-moda"
  → page reads: headers().get('x-tenant-slug')
  → queries: tenants.slug = 'demo-moda'
```

**Three tiers**:
| Tier | Source | Header Set | Example |
|------|--------|-----------|---------|
| 1 | Subdomain `*.nadistudio.cl` | `x-tenant-slug` | `demo-moda.nadistudio.cl` |
| 2 | Custom domain | `x-tenant-domain` | `misitio.com` |
| 3 | Local dev | `DEV_TENANT_SLUG` env var | `localhost:3000` |

**System subdomains** excluded from tenant routing:
`scaffold`, `www`, `admin`, `api`, `mail`

**Quirk — DEV_TENANT_SLUG_INMO**: Real estate pages (`/propiedades/*`) check `DEV_TENANT_SLUG_INMO` before `DEV_TENANT_SLUG`. You need BOTH env vars set in local dev if you work on both verticals.

**Quirk — Vercel preview URLs**: `*.vercel.app` intentionally returns 500 — no tenant detection on preview deployments. Use the actual subdomain.

**Quirk — Rubro cookie routing**: Middleware rewrites `/admin/dashboard` → `/admin/{retail|real-estate}/dashboard` based on `tenant_rubro` cookie. If the cookie is missing or stale, admin routes break.

**File**: `middleware.ts`

---

## 2. getUserTenant() — Cross-Rubro Isolation

**The bug that taught us**: A user linked to both "Demo Moda" (retail) and "RedPropertyChile" (real-estate) was getting photos processed under the wrong tenant. The ropero bot returned "Foto recibida — RedPropertyChile".

**Root cause**: `getUserTenant()` queried `telegram_users` by `telegram_user_id` without filtering by rubro. It returned the first tenant found — which happened to be the wrong one.

**The fix (now mandatory in every bot)**:
```typescript
async function getUserTenant(userId: number) {
  const { data } = await supabase
    .from('telegram_users')
    .select('tenant_id, activated_at, tenants!inner(id, slug, name, rubro_type)')
    .eq('telegram_user_id', userId)
    .eq('tenants.rubro_type', 'retail-clothing')  // hardcoded per bot
    .order('activated_at', { ascending: false })
    .limit(1)
    .maybeSingle();

  if (!data?.tenants) return null;
  const tenant = data.tenants as any;
  return { id: tenant.id, slug: tenant.slug, name: tenant.name };
}
```

**Three critical details**:
1. `tenants!inner(...)` — forces INNER JOIN so PostgREST filters on rubro actually exclude rows (without `!inner`, LEFT JOIN returns rows with `tenants: null`)
2. `.order('activated_at', { ascending: false })` — picks the most recently `/start`-ed tenant (user may link to multiple)
3. `.eq('tenants.rubro_type', ...)` — hardcoded per bot file, never dynamic

**Rule**: Every bot MUST filter by its own `rubro_type`. Never trust that a user belongs to only one tenant.

**Files**: `app/api/bot/webhook/ropero/route.ts`, `app/api/bot/webhook/inmobiliario/route.ts`

---

## 3. Image Systems

The scaffold runs TWO image systems simultaneously (legacy + future). This is intentional — migration to entity_images is incremental.

### Legacy System (ACTIVE — production reads)
- **Where**: `raw_image_urls[]` array column on `product_drafts` and `property_drafts`
- **How**: Bot downloads Telegram photos → uploads to Supabase Storage → saves public URLs in array
- **Read by**: Draft detail pages, approval routes

### Future System (STAGED — being populated)
- **Where**: `entity_images` table with `entity_type`, `entity_id`, `public_url`, `sort_order`
- **How**: `insertEntityImages()` called AFTER legacy insert — both systems populated simultaneously
- **Functions**: `getEntityImages()`, `insertEntityImages()`, `publishEntityImages()`, `archiveEntityImages()`
- **Read by**: Public pages (`/propiedades`, `/propiedades/[slug]`), admin catalog

### The Approval Flow
When a draft is approved:
1. Images copied from `{tenant}/temp/drafts/{uuid}/` to `{tenant}/properties/{slug}/`
2. New `entity_images` rows inserted with `entity_type: 'property'` pointing to published property ID
3. Old draft images archived (`status: 'archived'`)
4. Temp files deleted from storage

**Critical quirk**: See [#16 Entity Images — No FK](#16-entity-images-no-fk).

**Files**: `lib/image-systems/future-nadistudio-entity-images/index.ts`, `lib/image-systems/yamato-system-inmobiliario/index.ts`

---

## 4. Two-Phase Async Processing

**The pattern**: Heavy work (Kimi Vision, Whisper, bulk operations) MUST be split into a fast response + async worker. Vercel kills functions at their `maxDuration`.

### Bot Flow
```
Telegram → webhook (maxDuration: 60s)
  ├── save photos to media_queue
  ├── return 200 to Telegram immediately
  └── fire-and-forget → /api/bot/process (maxDuration: 300s)
       ├── download photos as buffers
       ├── send "📸 Fotos descargadas. Consultando IA..." message
       ├── call Kimi Vision
       ├── create drafts
       └── send rich feedback message
```

### Web Upload Flow
```
Browser → /api/admin/properties/draft-from-upload (maxDuration: 60s)
  ├── upload images to Supabase Storage
  ├── create draft with status: 'processing'
  ├── return { draft_id, status: 'processing' }
  └── fire-and-forget → /api/admin/properties/process-draft (maxDuration: 300s)
       ├── download images from storage
       ├── call Kimi Vision
       └── update draft → status: 'auto_detected'
```

### Frontend Polling
```typescript
useEffect(() => {
  if (draft?.status !== 'processing') return;
  const interval = setInterval(() => loadDraft(), 3000);
  return () => clearInterval(interval);
}, [draft?.status]);
```

**Fire-and-forget pattern**:
```typescript
// ⚠️ OLD PATTERN (broken on Vercel — fetch killed after response sent):
// fetch(processUrl, { method: 'POST', body: JSON.stringify(payload) })
//   .catch(err => console.error('Fire-and-forget error:', err));

// ✅ CORRECT PATTERN — after() from next/server (Next.js 15+):
import { NextRequest, NextResponse, after } from 'next/server';

after(async () => {
  try {
    const res = await fetch(processUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    if (!res.ok) console.error('Process failed:', res.status);
  } catch (err) {
    console.error('after() error:', err);
  }
});

return NextResponse.json({ draft_id, status: 'processing' });
```

**CRITICAL UPDATE (2026-03-17)**: Bare `fetch()` without `await` gets killed by Vercel after the response is sent. The `after()` function from `next/server` (Next.js 15+) guarantees execution post-response. Both bot webhooks already used this — `draft-from-upload` was the last holdout. See `Playbook-two-phase-async-processing.md` for the full deep-dive.

**Rule**: The async worker MUST always write a final status (`auto_detected` or `error`). Never leave a record stuck in `processing`.

---

## 5. Server/Client Component Split

**When you need server data in a client component** (e.g., tenant phone for WhatsApp in the cart):

```typescript
// page.tsx (Server Component) — fetches data
export default async function CartPage() {
  const supabase = createBotClient();
  const { data: tenant } = await supabase
    .from('tenants')
    .select('contact_phone, name')
    .eq('slug', resolvedSlug)
    .single();

  return <CartContent storePhone={tenant?.contact_phone || ''} storeName={tenant?.name || ''} />;
}

// cart-content.tsx (Client Component) — receives as props
'use client';
export default function CartContent({ storePhone, storeName }: Props) {
  // Use storePhone to build WhatsApp URL
}
```

**Rule**: Server component = thin wrapper that fetches data. Client component = all the interactivity. Props are the bridge. No Redux, no Context for server data.

---

## 6. Admin Auth

**Mechanism**: Password-based, cookie-stored.

```
POST /api/admin/auth { password, slug }
  → look up tenant by slug
  → compare password === tenant.admin_password (plaintext!)
  → set cookie: admin_auth_{slug} = 'authenticated' (7 days)
  → set cookie: tenant_rubro = '{rubro_type}' (1 day)
```

**Gotchas**:
- Passwords stored in PLAINTEXT in `tenants.admin_password` column — hash with bcrypt before production clients
- Each tenant has its own cookie (`admin_auth_demo_moda`, `admin_auth_demo_inmo`)
- The `tenant_rubro` cookie is what middleware uses to route `/admin/` pages to the correct vertical
- No server-side protection on admin page rendering — only the API routes check the cookie

---

## 7. Bot Webhook — Message Processing Order

> **See §37 for the complete 12-step pipeline** with session state checks. This section is the quick-reference.

**IMMUTABLE. Do not rearrange.**

Both bots process messages in this exact order:

```
1. /start {activation_code} → link user to tenant
2. Get userTenant (error if not linked)
3. /fotos or /cargar → start photo session
4. Photos (compressed) → add to media_queue (take largest resolution)
5. Documents (image files sent as docs) → add to media_queue
6. Text documents .txt (inmobiliario ONLY) → parse and add to description
7. Natural language search (inmobiliario ONLY) → search properties
7.5 IRCE intent engine (retail: SELL/QUERY/ADD_STOCK) → already deployed
8. Listo synonyms → trigger processing
9. "🤔 No entendí" catch-all fallback
```

**Rule**: Listo synonyms check MUST come before "no entendí" fallback. If you add a new handler, insert it at the correct position.

**Why a fixed order instead of a flexible router?** Because Telegram sends the same message through the webhook once — there's no retry, no middleware chain. If the wrong handler grabs a message first, the user's intent is lost. A photo during an active session MUST go to the session handler (step 4), not to the intent engine (step 7.5). Sequential if/else with explicit priority is ugly but unambiguous. A "smart" router that matches patterns would be cleaner code but would produce invisible routing bugs that are hell to debug in production.

---

## 8. Media Accumulation

**Problem**: Telegram sends grouped photos as separate messages with the same `media_group_id`. They arrive within ~1-2 seconds of each other. You can't process them individually.

**Solution**: media_queue with accumulation window.

```
Photo 1 (media_group_id: "abc") → create queue, status: 'accumulating'
Photo 2 (media_group_id: "abc") → add to same queue
Photo 3 (media_group_id: "abc") → add to same queue
...
/listo → find queue in last 30 min → process all photos together
```

**Key details**:
- `media_queue` row: `tenant_id`, `chat_id`, `user_id`, `status`, `bot_type`
- `media_queue_photos` rows: `queue_id`, `file_id`, `sort_order`
- Sort order is auto-incremented per queue
- Queue considered "active" if created within last 30 minutes
- `/fotos` command creates a new queue (cleans up any existing one)

---

## 9. Callback Queries

**When a user taps an inline button**, Telegram sends a `callback_query` update, NOT a `message` update. Handle them separately.

```typescript
if (update.callback_query) {
  const { data, message, from } = update.callback_query;
  // ALWAYS answer the callback first (removes loading spinner on button)
  await answerCallbackQuery(update.callback_query.id);
  // Then handle the action
  await handleCallbackQuery(data, message.chat.id, from.id);
  return;
}
```

**Callback data format**: Short strings like `upload`, `help`, `cancel`, or structured like `confirm:SELL:uuid:2`.

**Gotcha**: Always call `answerCallbackQuery()` first, even if you don't need to show a toast. Otherwise the button stays in "loading" state for the user.

---

## 10. Idempotence

**Problem**: Telegram retries webhook delivery if it doesn't get a 200 within ~5 seconds. Without idempotence, the same photo gets processed multiple times.

**Solution**: In-memory Map tracking processed `update_id`s.

```typescript
const processedUpdates = new Map<number, number>();

// Before processing:
if (processedUpdates.has(update.update_id)) return;
processedUpdates.set(update.update_id, Date.now());

// Cleanup every 60 seconds (TTL: 5 min):
setInterval(() => {
  const fiveMinAgo = Date.now() - 5 * 60 * 1000;
  for (const [id, ts] of processedUpdates) {
    if (ts < fiveMinAgo) processedUpdates.delete(id);
  }
}, 60_000);
```

**Limitation**: In-memory Map is per-function-instance. On Vercel, cold starts create new instances. For truly bulletproof idempotence, use a DB check (but the 5-min window covers 99.9% of Telegram retry scenarios).

---

## 11. Theme System

**Registry**: Hardcoded themes in `lib/themes/index.ts`. Each theme has: colors, typography, card variants, button variants, spacing.

**Available themes**:
- `fashion-minimal`, `fashion-boho`, `fashion-vibrant` (retail)
- `real-estate-luxury`, `real-estate-modern` (real estate)

**How it works**:
1. Tenant has `theme_config: { theme_id: 'real-estate-luxury' }` in DB
2. Server components call `getPageThemeClasses(themeId)`, `getCardThemeClasses(themeId)`
3. Client components use `<ThemeProvider>` which injects CSS variables into `document.documentElement`

**Fallback**: If no theme set, defaults to `fashion-minimal`. No errors.

**Files**: `lib/themes/index.ts`, `lib/themes/types.ts`, `lib/themes/ThemeProvider.tsx`, `lib/themes/theme-classes.ts`

---

## 12. Supabase Clients

| Client | Key Used | RLS | Use In |
|--------|----------|-----|--------|
| `createBotClient()` | `SUPABASE_SERVICE_ROLE_KEY` | Bypassed | API routes, server components, processors |
| `createClientClient()` | `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Enforced | Client components only |

**Rule**: `createBotClient()` bypasses RLS. This means EVERY query MUST include `.eq('tenant_id', tenantId)` manually. There's no safety net — a missing filter leaks data across tenants.

**Query helpers** (exported from `lib/supabase/client.ts`):
- `getTenantBySlug(supabase, slug)` — single tenant lookup
- `getProductsByTenant(supabase, tenantId, filters?)` — active + visible products
- `getPropertiesByTenant(supabase, tenantId, filters?)` — published properties
- `getDraftsByBatch(supabase, batchId, rubroType)` — drafts by batch

---

## 13. LX Tenant Resolver

**For API routes** that need tenant_id from the request:

```typescript
import { getTenantId } from '@/src/lx/tenant-resolver';

export async function POST(request: NextRequest) {
  const tenantId = await getTenantId(request);
  // tenantId is guaranteed to be valid — throws if not
}
```

**How it works**:
1. Reads `x-tenant-slug` header (set by middleware)
2. Falls back to `x-tenant-domain` header
3. Looks up tenant in DB
4. Returns tenant UUID
5. Throws with helpful error message if anything fails

**File**: `src/lx/tenant-resolver.ts`

---

## 14. Status Values — The Source of Many Bugs

**The #1 cause of "data doesn't show up" bugs**: status field mismatches between what writes the data and what reads it.

### Property Drafts
| Status | Set by | Meaning |
|--------|--------|---------|
| `pending_analysis` | draft-from-upload (web) | Draft created, waiting for Kimi |
| `processing` | process-draft | Kimi analysis in progress (set AFTER pending_analysis) |
| `auto_detected` | process-draft / bot processor | Kimi done, ready for review |
| `approved` | approve route | Published to properties table |
| `rejected` | admin action | Discarded |
| `error` | process-draft on failure | Something broke |
| `archived` | discard route | Soft-deleted, visible in archivados |

### Properties
| Status | Set by | Meaning |
|--------|--------|---------|
| `published` | approve route | Live on public pages |
| `draft` | — | Not used currently |
| `archived` | admin action | Hidden from public |

### Products
| Status | Set by | Meaning |
|--------|--------|---------|
| `is_active: true` | approve-product route | Available for sale |
| `is_visible_in_store: true` | admin toggle | Shown in storefront |

### Product Drafts
| Status | Set by | Meaning |
|--------|--------|---------|
| `pending` | bot processor | Initial state |
| `pending_analysis` | draft-from-upload (web) | Waiting for AI |
| `auto_detected` | Kimi processor | AI done, ready for batch review |
| `batch_reviewed` | batch approval | Sent to individual product review |
| `approved` | publish-product route | Published to products table |
| `archived` | discard-product route | Soft-deleted |
| `awaiting_photo` | Kimi processor | Detected item but missing dedicated photo |

**THE BUG**: Admin catalog for properties was filtering `status = 'active'` but properties are created with `status = 'published'`. Result: empty catalog. **Fixed 2026-03-15.**

**Rule**: When adding a new page that reads data, CHECK what status value the writer sets. Don't assume.

---

## 15. WhatsApp URL Construction

```typescript
const phone = (tenant.contact_phone || '').replace(/[^0-9]/g, '');
const message = encodeURIComponent('Hola! Me interesa...');
const url = phone
  ? `https://wa.me/${phone}?text=${message}`
  : `https://wa.me/?text=${message}`;
```

**Rules**:
- Strip everything except digits from phone (removes +, spaces, dashes)
- Always have a fallback URL without phone number (opens WhatsApp with just the message)
- `contact_phone` field must be read from tenant table — it's not on the product/property

**The bug**: Cart page was a client component that couldn't read tenant data. Fixed with server/client split (pattern #5).

---

## 16. Entity Images — No FK

**The `entity_images` table has NO foreign key to `properties` or `products`**. It only has FKs to `tenants` and `media_queue`.

**Why this matters**: Supabase's PostgREST join syntax (`entity_images!left(...)`) requires a FK relationship to work. Without it, the join silently returns null.

```typescript
// THIS DOESN'T WORK (no FK):
const { data } = await supabase
  .from('properties')
  .select('*, entity_images!left(public_url)')
  .eq('id', propertyId);
// property.entity_images is always null/empty

// THIS WORKS (explicit query):
const images = await getEntityImages('property', propertyId);
```

**Rule**: Always use `getEntityImages(entityType, entityId)` from `lib/image-systems/future-nadistudio-entity-images/index.ts`. Never try to join via PostgREST.

---

## 17. Vercel Limits & maxDuration

| Tier | Default Timeout | Max with maxDuration | Middleware |
|------|----------------|---------------------|------------|
| Hobby | 10s | 60s | 30s |
| Pro | 15s | 300s | 30s |

**Where we set maxDuration**:
- Webhooks: `maxDuration = 60` (fast response, fire-and-forget the heavy work)
- Processors: `maxDuration = 300` (Kimi Vision can take 30-120s for multi-image)
- Approval routes: `maxDuration = 60` (image copying is fast)
- Drag-and-drop upload: `maxDuration = 60` (just storage uploads)

**Rule**: If an endpoint might take > 15s, set `maxDuration`. If it might take > 60s, split into two phases.

---

## 18. Danger Zones — Known Weak Points

| Zone | Issue | Risk Level | Mitigation |
|------|-------|-----------|------------|
| **Admin passwords** | Stored in plaintext | HIGH | Hash with bcrypt before production clients |
| **createBotClient() without tenant filter** | Bypasses RLS — all data accessible | HIGH | Always `.eq('tenant_id', tenantId)` |
| **entity_images orphans** | No FK cascade on property delete | MEDIUM | Use soft delete (`status: 'archived'`), never hard delete properties |
| **In-memory idempotence** | Lost on cold start | LOW | 5-min TTL covers 99.9% of Telegram retries |
| **media_queue 30-min window** | Old queues hang if user never says /listo | LOW | Consider a cleanup cron for queues > 1h old |
| **tenant_rubro cookie stale** | User switches tenant, cookie points to old rubro | LOW | Re-set on login, 1-day expiry |
| **DISABLE_LOGGING=true** | Logging kill switch is ON since the $7 incident | LOW | Turn back on with recursion guard post-launch |
| **Supabase security warnings** | 18 functions without search_path, 6 SECURITY DEFINER views | LOW | Fix post-launch (see security triage doc) |
| **Ghost columns in .select()** | PostgREST 400 swallowed silently — null data, no error | HIGH | Audit `lib/schemas/index.ts` against DB; never guess column names |
| **Kimi float→integer** | 22P02 on `detected_*` fields when Kimi returns decimals | MEDIUM | `safeInt()` wrapper: `Math.round(Number(v))` on all Kimi integer output |
| **Missing CHECK constraint values** | Status writes fail silently, records get stuck forever | HIGH | Query `pg_constraint` before adding new status values in code |
| **after() not used for fire-and-forget** | Bare `fetch()` killed by Vercel post-response | HIGH | Always use `after()` from `next/server` in Next.js 15+ |
| **Dual-table archives** | Discarded drafts vanish — not in borradores, not in archivados | MEDIUM | Archivados pages must query BOTH published table AND drafts table |
| **Empty body → request.json() crash** | POST routes crash with `SyntaxError: Unexpected end of JSON input` | MEDIUM | Use `request.text()` + conditional `JSON.parse()` for optional-body routes |
| **Polling status mismatch** | Frontend polls on `'processing'` but writer sets `'pending_analysis'` | HIGH | Grep for the status string in BOTH writer and reader when adding polling |

---

## 19. IRCE Pipeline — What's Missing

The bots currently work as **command executors** (`/fotos`, `/listo`). The legacy proyecto-miche had an **IRCE pipeline** (Intent → Resolution → Confirmation → Execution) that made the bot understand natural language ("vendí 2 jeans azules").

**What exists**: Photo upload, Kimi Vision analysis, listo synonyms, inline buttons
**What's missing**: Regex intent engine, entity resolver with fuzzy matching, confirmation flow for sales/stock, voice (Whisper)

**Transplant plan**: `00.claude/IRCE-transplant-plan-from-proyecto-miche.md`
**Vertical playbook**: `00.claude/Playbook-IRCE-pipeline-for-verticals.md`

**Integration point**: The intent engine slots into the webhook message processing order at position 7.5 — after listo synonyms, before "no entendí" fallback. ~10 lines of code per webhook.

---

## 20. Phase Boundaries — Migration Markers

### FASE 1 (Current — Data Unification)
- `entity_images` table created and being populated ✅
- Both legacy (`raw_image_urls[]`) and future (`entity_images`) populated simultaneously ✅
- Public pages read from `entity_images` via `getEntityImages()` ✅
- yamato-system still used for bot media accumulation ✅

### FASE 4 (Migration Window — Future)
- Switch `entity_images` as sole source of truth
- Stop populating `raw_image_urls[]`
- Drop legacy image columns from drafts tables
- Remove yamato-system-inmobiliario code
- Add FK from `entity_images` to properties/products (enables PostgREST joins)

### Code markers to search for
- `[LEGACY]` — Production code, will be removed in FASE 4
- `[FUTURE]` — Staged code, not yet primary
- `[PHASE-X]` — Scheduled change for that phase
- `// CRÍTICO` or `// CRITICAL` — Do not modify without understanding

---

## Phase 2 — Integrated from Legacy Docs

> Source: 9 documents from `Kimiverse-memory/for-SCRAPING/SCRAPED/`
> Covering: intent engine, entity resolver, confirmation builder, operation executor, buttons/drafts, multi-tenant bot, architecture decisions

---

## 21. Kimi API Configuration — The Only Combo That Works

Discovered via A/B testing in production, NOT documented by Moonshot:

```typescript
{
  model: 'kimi-k2.5',
  temperature: 0.6,                    // ONLY 0.6 works with thinking disabled
  thinking: { type: 'disabled' },      // Required for low latency
  response_format: { type: 'json_object' }  // Structured output
}
```

**Rules**:
- Temperature 1.0 DOES NOT work with thinking disabled — produces garbage
- Temperature 0.6 is the sweet spot between creativity and reliability
- Always use `json_object` response format for intent classification
- Inject business context (brands, sizes, colors as arrays) in the system prompt to prevent hallucination

**ADR-003**: This was validated empirically. If Kimi changes their API, re-test temperatures.

---

## 22. Soft Delete vs Hard Delete — The State Machine

**Universal rule**: Business entities use soft delete. Only archived items can be hard-deleted.

```
                    ARCHIVE
   pending ─────────────────────→ archived
      ↓                               ↓
   in_review ───────────────────→ archived
      ↓                               ↓
   ready_for_approval ──────────→ archived
      ↓                               ↓
   approved (published)              HARD DELETE
                                  (only from archived)
```

**The `archived_source` field**: When archiving, store WHERE the draft was archived from (`in_review`, `ready_for_approval`, etc.). Without this, the RESTORE button doesn't know where to put it back.

**Image deletion**: NEVER delete images immediately when user removes them from a draft. Mark for deletion, cleanup on save. If user cancels the edit, images are preserved.

**Gotcha**: Two admins editing the same draft = last write wins. No optimistic locking yet.

---

## 23. Draft Button Actions — Complete Map

| Button | From Status | To Status | Side Effects |
|--------|------------|-----------|-------------|
| GUARDAR | pending | in_review | Updates fields + `reviewed_at` |
| ENVIAR A APROBACIÓN | in_review | ready_for_approval | Validates completeness |
| APROBAR Y PUBLICAR | ready_for_approval | approved | Image reorganization + entity creation + slug validation |
| ARCHIVAR | ANY | archived | Stores `archived_source` |
| RESTAURAR | archived | {original status} | Uses `archived_source` to determine target |
| ELIMINAR | archived ONLY | HARD DELETE | Removes from DB + Storage |

**Image lifecycle on APPROVE**:
```
1. Iterate draft.raw_image_urls[] (NOT Storage folder listing)
2. Copy each: /uploads/{tempId}/{i}.jpg → /{slug}/{i}.jpg
3. Delete temp files
4. Create entity_images rows pointing to new paths
5. Archive draft entity_images (status: 'archived')
```

**Rule**: Always iterate the URL array from the DB record. Never `supabase.storage.list()` a folder — it accumulates old images from failed operations.

---

## 24. Session Routing — Check BEFORE Intent Classification

**Critical order**: Active session takes priority over new intent classification.

```typescript
// STEP 1: Check active session FIRST
const session = await getActiveSession(userId, tenantId);
if (session) {
  // User is mid-conversation (e.g., confirming a sale, adding photos)
  // Continue that flow, do NOT classify new intent
  return await continueSession(session, text, tenantId);
}

// STEP 2: Only if no session, classify new intent
const intent = await classifyIntent(text, rubroType);
```

**Why**: If user is confirming a sale and types "sí", the intent engine would classify it as... nothing useful. But the session knows it means "confirm the pending sale."

**Session storage**: Use `active_sessions` table with `step`, `data` (JSONB), `expires_at`. NOT in-memory (serverless = no persistence between invocations).

---

## 25. Entity Resolver — Fuzzy Matching Rules

**Pre-filter ALWAYS**: `WHERE is_active = true AND stock_quantity > 0 AND tenant_id = $tenantId`

Products with zero stock should NEVER appear in search results. The resolver's job is to show only viable options.

**Scoring weights (per rubro)**:
```typescript
// Retail
{ name: 1.0, color: 0.9, size: 0.8, category: 0.5 }

// Real estate
{ commune: 1.0, property_type: 0.9, price_range: 0.7, bedrooms: 0.6 }
```

**Normalization before matching**:
- Lowercase everything
- Remove accents: á→a, é→e, í→i, ó→o, ú→u, ñ→n
- Strip extra whitespace
- Map synonyms: "jeans" = "jean" = "pantalón jean"

**Result states**:
| Score | Count | Action |
|-------|-------|--------|
| > 0.9 | 1 match | Auto-select, show confirmation |
| > 0.8 | Multiple | Show top 3-5, ask user to pick by number |
| < 0.7 | Any | "No encontré X. Sugerencias: Y, Z" |
| Found but stock=0 | — | "Solo hay N disponibles" (shouldn't happen if pre-filtered) |

---

## 26. Confirmation Builder — Permissive Parser

**The cardinal rule**: Only cancel on EXPLICIT "no". Everything ambiguous = ask for clarification.

```typescript
const CANCEL = ['no', 'n', 'nop', 'nope', 'cancelar', 'abortar'];
const CONFIRM = ['sí', 'si', 's', 'yes', 'ok', 'confirmar', 'dale', 'listo', 'ya'];

function parseResponse(text: string): 'CONFIRM' | 'CANCEL' | 'CLARIFY' {
  const normalized = text.toLowerCase().trim();
  if (CANCEL.includes(normalized)) return 'CANCEL';
  if (CONFIRM.includes(normalized)) return 'CONFIRM';
  return 'CLARIFY';  // NOT cancel — ask again
}
```

**The bug that taught us**: User typed "Precio $15,000" while confirming → old parser saw it wasn't "sí" or "no" and cancelled the operation. Fix: return CLARIFY, not CANCEL.

**Confirmation message format**:
```
Voy a registrar VENTA:
📦 Jeans Azul Marino Talla M  x2  = $36.000
📦 Stock resultante: 3 unidades
💰 Total venta: $36.000

¿Es correcto? [✅ Sí] [❌ No]
```

**Double confirmation required for**:
- Deletions (stock → 0)
- RECTIFY operations that change quantity
- Sales > configurable threshold

**Stock must be validated BEFORE showing confirmation**. Never show "2x Jeans Azul" if only 1 is in stock.

---

## 27. Operation Executor — Atomicity Checklist

**Every destructive operation follows this pipeline**:

```
1. VALIDATE
   ├── tenantId matches? (CRITICAL — security)
   ├── Permissions? (user role allows this?)
   ├── Stock sufficient? (for SELL)
   └── Entity exists and is active?

2. CALCULATE IMPACT
   ├── What will change? (stock: 5 → 3)
   ├── Financial impact? (total: $36.000)
   └── Side effects? (low stock warning?)

3. EXECUTE (atomic transaction)
   ├── UPDATE products SET stock = stock - N
   ├── INSERT sales (...)
   ├── INSERT sale_items (...)
   └── INSERT audit_log (who, what, when, before/after)

4. RESPOND
   ├── Success message with operation ID (#V001)
   ├── Updated state (new stock count)
   └── Next action suggestions (/rectificar, /stock)
```

**Idempotence**: Each operation gets a unique ID (e.g., `update_id` from Telegram). Check if already executed before running. Prevents double-sales from webhook retries.

**Soft delete pattern for sessions**: Use SELECT + UPDATE/INSERT instead of upsert. Upsert with `onConflict` fails silently if the unique constraint doesn't exist.

---

## 28. Multi-Tenant Bot — Activation Handshake

**Two handshakes, every time**:

```
HANDSHAKE 1 — Activation (once per tenant):
  User sends: /start ACTIVATION_CODE
  Bot does:
    1. Look up code in tenants table
    2. UPSERT telegram_users (telegram_user_id, tenant_id, activated_at)
    3. Send welcome message with tenant name

HANDSHAKE 2 — Every message after:
  Bot does:
    1. getUserTenant(userId) → filters by rubro_type
    2. All subsequent queries use returned tenant_id
    3. If no tenant found → "Usa /start CÓDIGO para activar"
```

**The upsert**: `onConflict: 'telegram_user_id,tenant_id'` — a user can be linked to multiple tenants. The unique constraint is on the PAIR, not just the user ID.

---

## 29. Architecture Decision Records (ADRs)

From production debugging across proyecto-miche and mi-catalogo:

| ADR | Decision | Why | Reversible? |
|-----|----------|-----|-------------|
| 001 | Photos in separate table, not JSONB | Serverless race conditions lose data in JSONB arrays | No |
| 002 | `after()` for fire-and-forget | Vercel kills function after 200 response without it. **Why `after()` not `waitUntil()`**: `waitUntil()` is a Web API that Vercel supports but Next.js doesn't expose natively. `after()` from `next/server` (Next.js 15+) is the framework-native equivalent — works identically but doesn't require reaching into platform APIs. We migrated 100% to `after()` on 2026-03-17. | Yes (use queue) |
| 003 | Kimi temp 0.6 + thinking disabled | Only working combo, discovered empirically | Yes (re-test) |
| 004 | Business context in LLM prompts | Constraining outputs to valid values prevents hallucination | Yes (quality drops) |
| 005 | service_role for bot, RLS for web | Bot is backend agent, not end user | No |
| 006 | English code, Spanish UI | Prevents confusion between DB fields and user-facing text | Yes (high cost) |
| 007 | Hardcode production URL for processors | Preview deployment URLs change, break fire-and-forget | Yes |

---

## 30. Master Pre-Flight Checklist

Use this when adding a new feature, vertical, or bot:

### Multi-Tenancy
- [ ] All business functions have `tenantId` parameter (required, not optional)
- [ ] No hardcoded `DEFAULT_TENANT_ID` anywhere
- [ ] All queries filter by `.eq('tenant_id', tenantId)`
- [ ] getUserTenant() filters by `rubro_type`

### Bot Webhooks
- [ ] Every code branch returns `NextResponse.json({ ok: true })`
- [ ] Fire-and-forget uses `after()` from `next/server` (NOT bare fetch, NOT waitUntil)
- [ ] Processor URL hardcoded to production
- [ ] Idempotence check on `update_id`
- [ ] Session check BEFORE intent classification

### Images & Storage
- [ ] Never list Storage folders (use URL array from DB)
- [ ] Cleanup temp files after copy on approve
- [ ] Hard delete only from archived state
- [ ] `archived_source` stored for restore logic
- [ ] `revokeObjectURL()` on preview component unmount

### Intent & Confirmation
- [ ] Regex threshold configurable per rubro
- [ ] Parser is permissive (CLARIFY on ambiguous, not CANCEL)
- [ ] Stock validated BEFORE showing confirmation
- [ ] Kimi temperature 0.6 + thinking disabled
- [ ] Double confirmation for deletions

### Entity Resolution
- [ ] Normalize input (lowercase, remove accents)
- [ ] Pre-filter: `is_active = true AND stock_quantity > 0`
- [ ] Fuzzy scoring with weighted fields per rubro
- [ ] Disambiguation shows top 3-5 with stock counts

### Navigation (Next.js App Router)
- [ ] Dynamic routes `[id]` not query params
- [ ] `decodeURIComponent()` on params
- [ ] Status values match between writer and reader

---

## 31. Serverless Architecture — Cold Starts, Pools & Logs

**Cold start**: First request 500ms-2s, subsequent <50ms. Never store state in global variables — they reset on cold start.

**Connection pool exhaustion**: Supabase has 100 default connections. Each Vercel instance opens a new one. For >10 concurrent instances, use the pooler URL (port 6543 instead of 5432).

**Client singleton**: Create Supabase client OUTSIDE the handler to reuse across warm requests:
```typescript
// ✅ Top-level (reused across requests)
const supabase = createBotClient();

export async function POST(request: NextRequest) {
  // Use supabase here
}
```

**Strategic logging (required pattern)**:
```typescript
console.log('[FuncName] Entry:', { userId, tenantId });
console.log('[FuncName] Calling Kimi:', { photoCount });
console.log('[FuncName] Result:', { success, itemCount });
console.error('[FuncName] Error:', { message, stack, context });
```

**Common serverless errors**:
| Error | Cause | Fix |
|-------|-------|-----|
| "No response is returned" | Missing return in a branch | Every branch must return NextResponse |
| "Cannot read properties of undefined" | `process.env` in Edge runtime | Use Node.js runtime for env vars |
| "Too many connections" | No connection pooling | Use pooler URL or client singleton |
| "ENOENT" | Reading /tmp files across invocations | Use Supabase Storage, not /tmp |

---

## 32. Storage Bucket Organization

```
product-images/
├── {tenant-slug}/
│   ├── drafts/{batch-id}/0.jpg, 1.jpg...    (temporary)
│   └── products/{product-slug}/0.jpg, 1.jpg  (permanent)

property-images/
├── {tenant-slug}/
│   ├── temp/drafts/{draft-uuid}/0.jpg...     (temporary)
│   └── properties/{property-slug}/0.jpg...   (permanent)
```

**Image URL storage**: Store FULL public URLs in DB, not just paths. Frontend doesn't need to construct URLs.

**Telegram CDN expiry**: Telegram file URLs expire in ~1-2 hours. Bot MUST download → upload to Storage → save permanent URL. If download fails, ask user to re-send immediately.

**On-the-fly transformations**: Append query params to Supabase Storage URLs:
```
?width=200&height=200&resize=cover  → thumbnails
?width=600&height=450&resize=cover  → card images
?format=webp                        → optimization
```

**Copy-not-move on approval**: Always copy images to permanent path FIRST, then delete temps. If the approval INSERT fails after move, the images are lost. Copy allows rollback.

---

## 33. Draft State Transitions — Valid Paths Only

Not all transitions are valid. Enforce in both UI and API:

```typescript
const VALID_TRANSITIONS: Record<string, string[]> = {
  'processing':          ['auto_detected', 'error'],
  'auto_detected':       ['in_review', 'archived'],
  'in_review':           ['ready_for_approval', 'archived'],
  'ready_for_approval':  ['approved', 'archived'],
  'archived':            ['auto_detected', 'in_review'],  // restore
  'error':               ['archived'],
};

function canTransition(from: string, to: string): boolean {
  return VALID_TRANSITIONS[from]?.includes(to) ?? false;
}
```

**Archive with reason**: Always store WHY something was archived (`discarded`, `client_withdrew`, `duplicate`, `deal_closed`). Essential for audit and analytics.

**Race condition prevention**: When changing status, use the current status as a guard:
```typescript
const { error } = await supabase
  .from('property_drafts')
  .update({ status: 'ready_for_approval' })
  .eq('id', draftId)
  .eq('status', 'in_review');  // Only if still in_review
```

---

## 34. RLS Policy Template

**Every new table needs both policies**:

```sql
-- 1. Enable RLS
ALTER TABLE new_table ENABLE ROW LEVEL SECURITY;

-- 2. Web users: tenant isolation via auth
CREATE POLICY "tenant_isolation" ON new_table
  FOR ALL USING (tenant_id IN (SELECT get_user_tenant_ids()));

-- 3. Bot/API: service role bypass
CREATE POLICY "service_role_full_access" ON new_table
  FOR ALL TO service_role USING (true) WITH CHECK (true);
```

**The silent failure**: If `service_role_full_access` policy is missing, bot queries with `createBotClient()` get `auth.uid() = NULL`. All queries return empty arrays — NOT a permission error, just no results. This is the hardest bug to debug because there's no error message.

---

## 35. Supabase Query Robustness

**`.maybeSingle()` vs `.single()`**: Use `.maybeSingle()` when the row might not exist. `.single()` throws `PGRST116` error if no row found.

```typescript
// ❌ Throws if no session exists
const { data } = await supabase
  .from('active_sessions')
  .select('id')
  .eq('user_id', userId)
  .single();

// ✅ Returns null if no session
const { data } = await supabase
  .from('active_sessions')
  .select('id')
  .eq('user_id', userId)
  .maybeSingle();
```

**Robust upsert pattern** (when you can't trust `onConflict`):
```typescript
const existing = await supabase
  .from('table')
  .select('id')
  .eq('key', value)
  .maybeSingle();

if (existing?.data) {
  await supabase.from('table').update(data).eq('id', existing.data.id);
} else {
  await supabase.from('table').insert(data);
}
```

**Integer prices**: Supabase INTEGER columns can't store decimals. Always `Math.round()` before insert. "15,990" as float = 15989.999... = silent data corruption.

---

## 36. Kimi Configuration — Complete Validation Matrix

| Temperature | Thinking | Valid? | Use Case |
|------------|----------|--------|----------|
| 0.6 | disabled | ✅ | Intent classification, fast JSON responses |
| 1.0 | enabled | ✅ | Creative descriptions, property analysis |
| 1.0 | disabled | ❌ | Silent error |
| 0.6 | enabled | ❌ | Silent error |
| Any other | Any | ❌ | "invalid temperature" error |

**Invalid combos fail SILENTLY** — no error message, just garbage output. Always test with `/api/test-kimi` before deploying changes.

---

## 37. Bot Message Pipeline Order — The Immutable Sequence

> **This is the authoritative version.** §7 is the quick-reference. This section includes session state checks that §7 omits.
> **Why it's repeated**: §7 exists in the "core patterns" zone (§1-20) for fast lookup. This section adds the session-aware steps (3-5) that cause the subtlest bugs.

```
1. /start {code}          → activation
2. getUserTenant()         → tenant resolution (with rubro filter!)
3. Check pending lote      → if user has active photo session
4. Check product draft     → if user is mid-draft creation
5. Check pending operation → if user has confirmation pending
6. /fotos, /cargar         → start new photo session
7. Photos/documents        → add to media_queue
8. Text documents .txt     → parse description (inmobiliario only)
9. Natural language search → property search (inmobiliario only)
10. Listo synonyms         → trigger processing
11. [FUTURE] Intent engine → IRCE regex + Kimi classification
12. "No entendí" fallback  → catch-all
```

**THE BUG**: If step 3 (pending lote) comes AFTER step 5 (pending operation), user saying "listo" during a photo session gets routed to the operation confirmation handler instead of processing photos. Order is everything.

---

## 38. Dirty State & Navigation Guards

**Prevent data loss on unsaved edits**:
```typescript
useEffect(() => {
  const handleBeforeUnload = (e: BeforeUnloadEvent) => {
    if (isDirty) {
      e.preventDefault();
      e.returnValue = '';
    }
  };
  window.addEventListener('beforeunload', handleBeforeUnload);
  return () => window.removeEventListener('beforeunload', handleBeforeUnload);
}, [isDirty]);
```

**Next.js App Router navigation**: Use dynamic routes `[id]` not query params. Query params fail silently in some App Router scenarios. Always `decodeURIComponent()` on received params.

---

## 39. MCP — Model Context Protocol for Supabase

**Config on Windows**: Use `cmd /c npx` as wrapper. The CLI `claude mcp add` converts `/c` to `C:/` — edit `.claude.json` manually to fix.

**Critical rules**:
- Never commit `mcp.json` with tokens
- MCP loads at CLI startup — must restart CLI after config changes
- Use `execute_sql` for queries, `apply_migration` for DDL (CREATE/ALTER/DROP)
- `get_advisors` returns security + performance warnings (use periodically)

**When MCP breaks** (multiple terminals, cache corruption):
```powershell
# Kill orphan processes, clear cache, restart
taskkill /F /IM node.exe /T 2>$null
Remove-Item "$env:LOCALAPPDATA\npm-cache\_npx" -Recurse -Force -ErrorAction SilentlyContinue
# Open fresh terminal
```

---

## 40. FK Migration — Don't Orphan Historical Data

**Before dropping an old table**, check what references it:
```sql
SELECT table_name, column_name
FROM information_schema.key_column_usage
WHERE referenced_table_name = 'old_table_name';
```

Re-point FKs to the new table BEFORE dropping the old one. Views and functions that reference old column names will also break silently.

---

## Master Summary — The 15 Rules That Save Hours

1. **Always filter by `tenant_id`** — no exceptions, no defaults
2. **getUserTenant() filters by `rubro_type`** — prevents cross-rubro contamination
3. **Kimi: 0.6 + thinking disabled** — the only valid fast combo
4. **Fire-and-forget needs `after()` from `next/server`** — lambda dies after 200. Not `waitUntil()` (platform API), not bare `fetch()` (killed post-response).
5. **Photos in separate table, not JSONB** — serverless race conditions
6. **Copy images before deleting** — enables rollback on failure
7. **Never list Storage folders** — use the URL array from DB
8. **`.maybeSingle()` not `.single()`** — unless you WANT an error on no match
9. **Check session BEFORE classifying intent** — conversational continuity
10. **Permissive confirmation parser** — only cancel on explicit "no"
11. **Validate files by extension, not MIME type** — Windows leaves MIME empty
12. **Symptom ≠ cause** — "credentials invalid" might be middleware blocking
13. **`npm run build` before push** — TS errors block ALL deploys, not just the broken route
14. **In-memory collections need TTL** — unbounded Set/Map = memory leak in serverless
15. **Resize images before Kimi** — 1024x1024 @ 85% JPEG, not raw phone photos

---

## Phase 3 — Integrated from Legacy Docs (Batch 2)

> Source: 30+ documents from `Kimiverse-memory/for-SCRAPING/lessons-learned/`, `past-try-destilation/`, and root
> Covering: robustness patterns, debugging methodology, cross-browser quirks, voice correction, image optimization, SQL forensics

---

## 41. Circuit Breaker — Kimi API Protection

When Kimi API fails 3 times in a row, stop calling it for 60 seconds. Prevents cascading failures and wasted API spend.

```typescript
const circuitState = { failures: 0, state: 'CLOSED', lastFailure: 0 };

async function callWithCircuitBreaker<T>(fn: () => Promise<T>): Promise<T> {
  if (circuitState.state === 'OPEN') {
    if (Date.now() - circuitState.lastFailure > 60_000) {
      circuitState.state = 'HALF_OPEN'; // Try one request
    } else {
      throw new Error('Circuit breaker OPEN — Kimi unavailable');
    }
  }

  try {
    const result = await fn();
    circuitState.failures = 0;
    circuitState.state = 'CLOSED';
    return result;
  } catch (err) {
    circuitState.failures++;
    circuitState.lastFailure = Date.now();
    if (circuitState.failures >= 3) circuitState.state = 'OPEN';
    throw err;
  }
}
```

**States**: `CLOSED` (normal) → `OPEN` (fast-fail, 60s cooldown) → `HALF_OPEN` (test one request) → back to CLOSED or OPEN.

**Fallback when OPEN**: Return a user-friendly message ("IA temporalmente no disponible, intenta en un minuto") instead of timing out.

---

## 42. Rate Limiting — Per-User Abuse Prevention

In-memory Map per function instance. Prevents users from flooding the bot.

```typescript
const rateLimits = new Map<number, { count: number; windowStart: number }>();

function checkRateLimit(userId: number, limit = 10, windowMs = 60_000): boolean {
  const now = Date.now();
  const entry = rateLimits.get(userId);

  if (!entry || now - entry.windowStart > windowMs) {
    rateLimits.set(userId, { count: 1, windowStart: now });
    return true; // allowed
  }

  if (entry.count >= limit) return false; // blocked
  entry.count++;
  return true;
}
```

**Limits**: 10 req/min + 30 req/hour per user.

**Limitation**: Same as idempotence (§10) — in-memory, resets on cold start. Sufficient for preventing accidental floods, not for DDoS protection.

---

## 43. Memory Leak Prevention — Bounded Collections

**The bug**: Using `Set<number>` for processedUpdates grows unbounded. After days of uptime on a warm instance, memory usage climbs until the function crashes.

**Fix**: Always use `Map<key, timestamp>` with TTL cleanup, never unbounded `Set`:

```typescript
// ❌ MEMORY LEAK
const processed = new Set<number>();
processed.add(updateId); // never removed

// ✅ BOUNDED
const processed = new Map<number, number>();
processed.set(updateId, Date.now());
// Cleanup every 60s (see §10)
```

**Rule**: Any in-memory collection in a serverless function MUST have a TTL eviction strategy. If it grows forever, it leaks.

---

## 44. Voice Correction — Chilean Spanish via Whisper

Whisper transcription of Chilean Spanish introduces predictable errors. Apply corrections BEFORE intent classification:

```typescript
const VOICE_CORRECTIONS: [RegExp, string][] = [
  [/\bbendí\b/gi, 'vendí'],           // b/v confusion
  [/\bvendidos\b/gi, 'vendí dos'],     // Whisper merges "vendí dos"
  [/\bpoleron\b/gi, 'polerón'],        // Missing accent
  [/\bjins\b/gi, 'jeans'],             // Phonetic spelling
  [/\bbuzo\b/gi, 'polerón'],           // Regional synonym
  [/\bremera\b/gi, 'polera'],          // Argentine → Chilean
  [/\bplayera\b/gi, 'polera'],         // Mexican → Chilean
];

function correctVoiceInput(text: string): string {
  let corrected = text;
  for (const [pattern, replacement] of VOICE_CORRECTIONS) {
    corrected = corrected.replace(pattern, replacement);
  }
  return corrected;
}
```

**Rule**: Voice corrections are rubro-specific. Fashion corrections (polera, jeans) don't apply to real estate. Load the right correction set based on `rubro_type`.

---

## 45. Image Optimization — Sharp Before Kimi

Sending raw 4000x3000 phone photos to Kimi wastes tokens and time. Resize first:

```typescript
import sharp from 'sharp';

async function optimizeForKimi(buffer: Buffer): Promise<Buffer> {
  return sharp(buffer)
    .resize(1024, 1024, { fit: 'inside', withoutEnlargement: true })
    .jpeg({ quality: 85 })
    .toBuffer();
}
```

**Impact**: 4MB photo → ~150KB. 21 images go from ~84MB to ~3MB. Kimi processes faster, costs less.

**Rule**: Always optimize before sending to any vision API. The 1024x1024 cap matches Kimi's effective resolution — larger images don't improve detection.

**Where to apply**: In `process-draft/route.ts` and `bot/process/route.ts`, after downloading images and before calling `analyzePropertyImages()`.

---

## 46. Cross-Browser Drag & Drop — The Windows MIME Trap

**The bug**: Windows leaves `file.type` empty for many file types (including `.txt`). Code that validates `file.type.startsWith('image/')` silently rejects valid files on Windows.

```typescript
// ❌ BREAKS ON WINDOWS
if (!file.type.startsWith('image/')) reject(file);

// ✅ VALIDATE BY EXTENSION
const ext = file.name.split('.').pop()?.toLowerCase();
const IMAGE_EXTS = ['jpg', 'jpeg', 'png', 'gif', 'webp', 'heic'];
if (!ext || !IMAGE_EXTS.includes(ext)) reject(file);
```

**Brave browser quirk**: Privacy shields can block `dataTransfer.files` entirely. Always provide a click-to-upload fallback alongside drag & drop.

**Rule**: Never trust MIME types from the browser. Validate by file extension. Always provide click fallback.

---

## 47. Split-Brain Prevention — Verify Working Directory

**The incident**: Developer had two copies of the repo open (`proyecto-miche/` and `mi-catalogo/`). Made 2 hours of changes in the wrong directory. Dev server was running from the other copy — so changes never appeared.

**Prevention**:
```powershell
# Before ANY coding session, verify you're in the right repo
Get-Location  # PowerShell
pwd           # Bash
```

**For Claude/LLM agents**: Always check `process.cwd()` or the working directory at session start. If multiple repos exist with similar names, confirm before editing.

**Extended pattern**: If `npm run dev` output doesn't match your changes, check if the dev server is running from a different directory.

---

## 48. SSR Cache Data Leak — Module-Level Variables in Node.js

**The bug**: Module-level variables in Next.js Server Components persist across requests in the same Node.js process:

```typescript
// ❌ DATA LEAK — shared between ALL users
let cachedTenant: Tenant | null = null;

export default async function Page() {
  if (!cachedTenant) cachedTenant = await fetchTenant(); // User A's tenant...
  return <div>{cachedTenant.name}</div>; // ...shown to User B!
}
```

**Fix**: Use `React.cache()` or key by user/tenant:

```typescript
import { cache } from 'react';

// ✅ SCOPED TO REQUEST
const getTenant = cache(async (slug: string) => {
  return await fetchTenant(slug);
});
```

**Rule**: Never use module-level mutable variables in Server Components. They leak data between requests. Use `React.cache()`, or pass data as function parameters.

---

## 49. SQL Forensics — Diagnostic Queries for Production Debugging

When something "doesn't show up" or "processes twice", these queries find the root cause:

```sql
-- Orphaned photos (queue exists but no photos)
SELECT mq.id, mq.status, mq.created_at,
  (SELECT count(*) FROM media_queue_photos WHERE queue_id = mq.id) as photo_count
FROM media_queue mq
WHERE (SELECT count(*) FROM media_queue_photos WHERE queue_id = mq.id) = 0
  AND mq.created_at > now() - interval '24 hours';

-- Duplicate sessions (race condition detector)
SELECT user_id, tenant_id, count(*), array_agg(id)
FROM media_queue
WHERE status = 'accumulating'
  AND created_at > now() - interval '1 hour'
GROUP BY user_id, tenant_id
HAVING count(*) > 1;

-- Stuck processing (fire-and-forget that never completed)
SELECT id, tenant_id, status, created_at,
  extract(epoch from now() - created_at) / 60 as minutes_stuck
FROM property_drafts
WHERE status = 'processing'
  AND created_at < now() - interval '10 minutes';

-- Timestamp sync check (are writes happening in UTC?)
SELECT id, created_at, created_at AT TIME ZONE 'America/Santiago' as local_time
FROM property_drafts
ORDER BY created_at DESC
LIMIT 5;
```

**Rule**: When debugging, query the DB directly before reading code. The data tells you what happened; the code tells you what should have happened.

---

## 50. Race Condition Detection — Optimistic Locking

For entities that multiple users/processes can modify simultaneously:

```typescript
// 1. Read with version
const { data: draft } = await supabase
  .from('property_drafts')
  .select('id, version, status')
  .eq('id', draftId)
  .single();

// 2. Update with version guard
const { data: updated, error } = await supabase
  .from('property_drafts')
  .update({ status: 'approved', version: draft.version + 1 })
  .eq('id', draftId)
  .eq('version', draft.version) // Only if nobody else changed it
  .select()
  .single();

if (!updated) {
  // Another process modified this record — reload and retry or abort
  throw new Error('Concurrent modification detected');
}
```

**Where needed**: Draft approval (two admins click "Approve" simultaneously), stock updates (two sales at the same time), photo session processing.

**Not yet implemented**: This is a pattern for when concurrent access becomes a real problem. Currently mitigated by single-admin-per-tenant assumption.

---

## 51. Supabase Connection Architecture — REST vs Direct vs Pooler

| Connection Type | Port | Works on Free? | IPv6 Required? | Use When |
|----------------|------|---------------|----------------|----------|
| **REST API** (PostgREST) | 443 (HTTPS) | ✅ Yes | No | Default — all CRUD via `.from()` |
| **Direct** (PostgreSQL) | 5432 | ⚠️ Requires IPv6 | Yes | Raw SQL, migrations |
| **Pooler** (PgBouncer) | 6543 | ✅ Yes | No | High-concurrency, connection reuse |

**The gotcha on Supabase Free tier**: Direct connections require IPv6 support from your network. Many ISPs, corporate networks, and cloud providers don't have IPv6. If `psql` hangs or times out, switch to the pooler URL.

**For the scaffold**: We use REST API exclusively via `createBotClient()` and `createClientClient()`. No direct PostgreSQL connections. This works on Free tier from any network.

**Connection string switch**:
```
# Direct (may fail on some networks)
postgresql://postgres:xxx@db.xxx.supabase.co:5432/postgres

# Pooler (always works)
postgresql://postgres:xxx@db.xxx.supabase.co:6543/postgres
```

---

## 52. Fuzzy Command Matching — Typo Tolerance for Bot Commands

Users type `/fotoss`, `/lissto`, `/empezar` (synonym for `/fotos`). Handle gracefully:

```typescript
const COMMAND_ALIASES: Record<string, string[]> = {
  '/fotos':  ['/foto', '/fotoss', '/photos', '/cargar', '/empezar'],
  '/listo':  ['/lissto', '/done', '/ready', '/terminar', '/procesar'],
  '/stock':  ['/inventario', '/stok'],
  '/start':  [],  // Never fuzzy-match /start — it has activation codes
};

function resolveCommand(input: string): string | null {
  const normalized = input.toLowerCase().trim().split(' ')[0];
  for (const [canonical, aliases] of Object.entries(COMMAND_ALIASES)) {
    if (normalized === canonical || aliases.includes(normalized)) {
      return canonical;
    }
  }
  return null;
}
```

**Rule**: `/start` is NEVER fuzzy-matched — it carries activation codes. All other commands can have aliases.

**The "listo" synonyms** (already implemented in both bots):
```typescript
const LISTO_KEYWORDS = ['listo', 'listos', 'ready', 'done', 'procesar', 'analizar', 'enviar'];
```

---

## 53. Debugging Methodology — The Guard Analogy

**The incident**: Webhook returning 500. Logs showed "Supabase credentials invalid." Developer spent 3 hours regenerating keys, checking env vars, testing direct connections. All credentials were fine.

**Root cause**: `middleware.ts` was blocking the webhook path. It tried to extract a tenant slug from `/api/bot/webhook/ropero` (no subdomain), failed, and returned 500 before the handler ever ran. The "credentials invalid" message was from a different log line.

**The methodology**:
```
1. ENTRY POINT FIRST
   → Does the request even reach your handler?
   → Check middleware, proxy, WAF, DNS

2. LAYER BY LAYER
   → Request → Middleware → Route Handler → DB → External API → Response
   → Add a console.log at EACH layer, find where it stops

3. SYMPTOM ≠ CAUSE
   → "Empty response" might be RLS, not missing data
   → "Credentials invalid" might be middleware, not credentials
   → "Timeout" might be the wrong URL, not slow processing
```

**Middleware exclusion pattern** (already in place):
```typescript
// middleware.ts — skip tenant resolution for API/bot routes
if (pathname.startsWith('/api/bot/')) return NextResponse.next();
```

---

## 54. TypeScript Build — The Silent Killer

**The incident**: TypeScript build errors block ALL Vercel deploys — not just the broken route. The entire app goes down if you push a type error.

**Why it's worse than you think**: Vercel shows "Build failed" in the dashboard, but the OLD deployment stays live. If someone pushes another commit thinking "it'll fix itself," the deploy queue stacks up.

**Prevention**:
```bash
# ALWAYS run before push
npm run build  # or next build

# Common errors that block builds:
# - Missing interface fields (added a column but didn't update the type)
# - Import from deleted file
# - Unused imports (with strict mode)
# - `as` casts that don't match actual shapes
```

**The Supabase join gotcha**: `supabase.from('table').select('*, related(name)')` returns `related` as an ARRAY `{ name: string }[]`, not an object. Casting with `as { name: string }` compiles but crashes at runtime.

```typescript
// ❌ Compiles but crashes
const name = (data.tenants as { name: string }).name;

// ✅ Correct — it's an array from joins
const name = (data.tenants as { name: string }[])?.[0]?.name;

// ✅ Or use .single() to force single object
const { data } = await supabase.from('telegram_users')
  .select('tenant_id, tenants(id, slug, name)')
  .eq('telegram_user_id', userId)
  .single();  // Now data.tenants is an object
```

---

## 55. PowerShell Gotchas — Windows Dev Environment

| Bash | PowerShell | Notes |
|------|-----------|-------|
| `cmd1 && cmd2` | `cmd1 ; cmd2` | `&&` not supported in PS5 |
| `cmd1 \|\| cmd2` | Not supported | Use `if ($LASTEXITCODE -ne 0)` |
| `"path/file"` | `"path\file"` | But `/` works in most PS contexts |
| `export VAR=val` | `$env:VAR = "val"` | Session-scoped only |

**The `[id]` trap**: PowerShell interprets `[id]` in file paths as a wildcard pattern. Use `-LiteralPath` instead of `-Path`:
```powershell
# ❌ Fails silently — [id] is a glob pattern
Get-Content -Path "app/propiedades/[id]/page.tsx"

# ✅ Works
Get-Content -LiteralPath "app/propiedades/[id]/page.tsx"
```

**Exit code 1 ≠ error**: Many Node.js tools exit with 1 for warnings. PowerShell treats this as an error. Don't panic at red text from `next build` warnings.

**UTF-8**: Always verify file encoding before editing. If a file was created with UTF-16 (common on Windows), editing it with UTF-8 tools corrupts accented characters (á, é, ñ):
```powershell
[System.IO.File]::ReadAllBytes("file.ts")[0..3]  # BOM check
# FF FE = UTF-16 LE (problem), EF BB BF = UTF-8 BOM (ok), no BOM = UTF-8 (ok)
```

---

## 56. Turbopack Cache Corruption — The Nuclear Fix

**Symptom**: Dev server starts but pages 500 with cryptic SST/chunk errors. No code changes fix it.

**Cause**: Turbopack's incremental cache gets corrupted after branch switches, crashes, or `node_modules` changes.

**Fix**:
```bash
rm -rf .next/ && rm -rf node_modules/.cache/
npm run dev  # Fresh rebuild
```

**When to suspect cache corruption**:
- Error references a file you already deleted
- Error shows line numbers that don't match the actual file
- Works in production but not in dev (or vice versa)
- Started after `git checkout` to a different branch

---

## 57. Supabase Migration Pattern — Rename, Don't Delete

**When evolving DB schema across versions**:

```sql
-- v1: Original table
CREATE TABLE sessions (...);

-- v2: Don't DROP — rename the old one, create new
ALTER TABLE sessions RENAME TO sessions_v1_deprecated;
CREATE TABLE active_sessions (...);

-- v3: After confirming v2 works, drop the old
DROP TABLE sessions_v1_deprecated;
```

**Why**: Dropping and recreating in one migration can fail halfway, leaving you with no table at all. Renaming preserves the data as a safety net.

**Critical indexes for the scaffold**:
```sql
-- These indexes are performance-critical, always verify they exist
CREATE INDEX IF NOT EXISTS idx_products_tenant ON products(tenant_id);
CREATE INDEX IF NOT EXISTS idx_entity_images_entity ON entity_images(entity_type, entity_id);
CREATE INDEX IF NOT EXISTS idx_media_queue_user ON media_queue(user_id, tenant_id, status);
CREATE INDEX IF NOT EXISTS idx_property_drafts_tenant ON property_drafts(tenant_id, status);
```

---

## 58. Security Pre-Checklist — Before Going Live

From a security audit of the codebase (7 findings, 3 critical):

| ID | Severity | Issue | Status |
|----|----------|-------|--------|
| C1 | CRITICAL | Debug endpoints (`/api/test-*`) accessible without auth | Remove before production |
| C2 | CRITICAL | Webhook has no secret validation | Add `X-Telegram-Bot-Api-Secret-Token` header check |
| C3 | CRITICAL | `process-draft` has no auth — anyone can POST | Add internal secret or check origin |
| H1 | HIGH | Admin passwords in plaintext | Hash with bcrypt |
| M1 | MEDIUM | `middleware.ts` + `proxy.ts` coexistence causes I18 crash | Remove proxy.ts |
| M2 | MEDIUM | No CSRF protection on admin actions | Add token or SameSite cookie |
| I18 | LOW | `[locale]` route + middleware redirect loop | Fixed by removing unused i18n |

**Webhook secret validation** (not yet implemented):
```typescript
export async function POST(request: NextRequest) {
  const secret = request.headers.get('x-telegram-bot-api-secret-token');
  if (secret !== process.env.TELEGRAM_WEBHOOK_SECRET) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  // ... handle webhook
}
```

---

## Phase 4 — Integrated from Legacy Docs (Batch 3)

> Source: 10 documents from `.docs/HumanTry/` — rubro expansion, intelligence layer, onboarding, image unification, bot comparison
> These docs had heavy overlap with §1-58 but contained strategic/operational content not yet captured.

---

## 59. Rubro Expansion — 7-Step Recipe for Adding a Vertical

When adding a new business vertical (rubro) to the platform:

### Step 1 — Database
```sql
INSERT INTO tenants (name, slug, rubro_type, activation_code, plan, settings)
VALUES ('Business Name', 'business-slug', 'new-rubro',
  'CODE-' || upper(substring(gen_random_uuid()::text, 1, 6)),
  'starter', '{}'::jsonb);
```
Create rubro-specific tables (entities, drafts). Every table: `tenant_id UUID NOT NULL REFERENCES tenants(id)`, RLS enabled.

### Step 2 — Bot Entry Point
Add rubro branch in the webhook or create a new webhook route at `app/api/bot/webhook/{rubro}/route.ts`.

### Step 3 — Bot Commands
Create `lib/bot/commands/{rubro}/` with: `index.ts` (main router), `intents.ts` (NLP patterns), one file per major flow.

### Step 4 — Web API Routes
Create `app/api/{rubro}/` routes. Every handler starts with `getTenantId(request)` + `createBotClient()`.

### Step 5 — Web UI
Create `app/(rubro)/` directory. Reuse existing components (tables, cards, forms).

### Step 6 — Subdomain
Wildcard `*.nadistudio.cl` handles new subdomains automatically. Just add the tenant to DB.

### Step 7 — Activation
`/start CODE` flow is rubro-agnostic — nothing to change.

### Rubro Profiles (Validated + Pipeline)

| Rubro | Status | Core Value | Key Difference |
|-------|--------|-----------|----------------|
| `retail-clothing` | Production | Stock + sales via bot | N drafts per photo batch, sales confirmation |
| `real-estate` | Production | Property publishing + leads | 1 draft per batch, 20+ detected fields |
| `agenda` | Pipeline | Appointment booking | Needs cron for reminders, timezone-aware |
| `subscription` | Pipeline | Recurring delivery mgmt | Customer-facing bot (not admin-only) |
| `sales-assistant` | Pipeline | RAG-powered Q&A | Requires pgvector, context embedding |

### Naming Convention
```
rubro_type (DB)  → snake-case, stable forever: retail-clothing, real-estate
slug (subdomain) → kebab-case, URL-safe: tu-stilo, casa-norte, dr-smith
tenant_id (DB)   → UUID, always from DB query, never hardcoded
```

---

## 60. Client Onboarding — 5 Minutes Start to Finish

### The Manual Process (Until Deploy Wizard Exists)

| Step | Action | Time |
|------|--------|------|
| 1 | Create tenant in DB (SQL INSERT with auto-generated activation code) | 2 min |
| 2 | Test subdomain (`slug.nadistudio.cl`) — works instantly with wildcard DNS | 30s |
| 3 | Send client activation message (bot name + `/start CODE` + web URL) | 1 min |
| 4 | Smoke test: activate bot, send test message, check web loads | 2 min |
| **Total** | | **~5 min** |

### Activation Code Generation
```sql
SELECT id, name, slug, activation_code FROM tenants WHERE slug = 'new-client';
-- activation_code: 'ESTRELLA-A3F9BC' (auto-generated on INSERT)
```

### What Client Needs to Know
- Their activation code (one-time: `/start CODE`)
- Their web panel URL (`slug.nadistudio.cl`)
- Which bot to find in Telegram (`@ropero_v1_bot` or `@Bot_inmobiliario_v2_bot`)

### What Client Does NOT Need to Know
- Supabase, Vercel, middleware, tenantId
- That other businesses share the same infrastructure

### Offboarding
```sql
-- 1. Export their data first (products, sales, etc.)
-- 2. Soft deactivate
UPDATE tenants SET plan = 'inactive', slug = slug || '-inactive' WHERE id = 'uuid';
-- 3. Hard delete after 30 days if no disputes
```

### Troubleshooting

| Symptom | Fix |
|---------|-----|
| `/start CODE` does nothing | Wrong code or wrong bot for the rubro |
| Web shows "tenant not found" | `tenants.slug` doesn't match subdomain exactly |
| Bot responds in wrong context | User activated with different code — check `telegram_users` |
| Images not uploading | Storage bucket missing or wrong name |

---

## 61. Intelligence Layer — pgvector RAG Architecture (Future)

### Current Stack
```
Voice → Whisper → text
Text  → Regex (80%) / Kimi (20%) → intent
Photo → Kimi Vision → structured JSON
```

### What pgvector Adds
The bot knows the rubro's vocabulary but not THIS SPECIFIC BUSINESS's data. pgvector enables semantic search over business-specific context.

### Context Store Schema (Not Yet Built)
```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE business_context (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  content_type TEXT NOT NULL,  -- 'product_description' | 'policy' | 'faq'
  title TEXT,
  content TEXT NOT NULL,
  embedding vector(1536),     -- text-embedding-3-small
  tags TEXT[],
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX business_context_embedding_idx
ON business_context USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

### Embedding Pipeline
```typescript
// Write path: embed on product create/update
const embedding = await openai.embeddings.create({
  model: 'text-embedding-3-small', input: productDescription
});
await supabase.from('business_context').upsert({ tenant_id, content, embedding: embedding.data[0].embedding });

// Read path: semantic search on user message
const queryEmbedding = await openai.embeddings.create({ model: 'text-embedding-3-small', input: userMessage });
const { data: context } = await supabase.rpc('match_business_context', {
  query_embedding: queryEmbedding.data[0].embedding,
  match_tenant_id: tenantId,
  match_threshold: 0.75,
  match_count: 3,
});
```

### Implementation Priority
1. **Layer 0**: Product embedding only — makes entity resolver dramatically better ("jeans oscuro" finds "Jeans Azul Marino")
2. **Layer 1**: + policies and FAQs — "the bot knows this business"
3. **Layer 2**: + brand knowledge and seasonal context
4. **Layer 3**: Conversation history (persistent memory)

### Cost Estimate
| Operation | Model | Cost/call | At 100 clients, 1000 msgs/day |
|-----------|-------|-----------|-------------------------------|
| Voice transcription | whisper-1 | $0.006/min | ~$18/day |
| Text embedding | text-embedding-3-small | $0.00002 | $2/day |
| Intent classification | kimi-k2.5 (20% fallback) | ~$0.001 | ~$200/day |
| **Total** | | | **~$220/day (~$6,600/mo)** |

Revenue at $50/month × 100 clients = $5,000/month. Optimization levers: better regex (reduce Kimi calls), prompt caching, tiered plans.

---

## 62. Image System Unification — The Lego Vision

### The Problem (Two Legacy Systems)

| Aspect | Fashion (proyecto-miche) | Real Estate (mi-catalogo) |
|--------|--------------------------|---------------------------|
| Draft field | `image_urls[]` | `raw_image_urls[]` |
| Entity field | `image_urls[]` | **None** (constructed from slug!) |
| Bucket | `product-images` | `properties` |
| On approve | Stays in `/drafts/` | Copies to `/{slug}/` |
| Cleanup | No | Yes |

### The 4 Smells

1. **Divergent naming** — Code can't be portable: `draft.image_urls || draft.raw_image_urls`
2. **Phantom images** (Real Estate) — URLs built from slug at runtime. If slug changes, all images break
3. **Orphaned photos** (Fashion) — Approved drafts leave files in `/drafts/` forever. Storage grows unbounded
4. **Hardcoded URLs vs dynamic paths** — Fashion stores full URLs (easy to use, hard to migrate), Real Estate constructs dynamically (flexible, fragile)

### The Solution: `entity_images` (Already Built — §3, §16)

The `entity_images` table is the unified answer. Currently in dual-write mode:
- Both legacy (`raw_image_urls[]`) and future (`entity_images`) populated simultaneously
- Public pages read from `entity_images` via `getEntityImages()`
- FASE 4 migration will drop legacy fields

### Key Rules from the Legacy Image Docs
- **Storage first, DB second**: Only create DB record if upload succeeds
- **Iterate URL array, not Storage folder**: `for (const url of draft.raw_image_urls)`, never `storage.list(folder)`
- **Copy before delete on approval**: If the INSERT fails after a move, images are lost. Copy allows rollback
- **Cleanup always on archive/delete**: Get URLs BEFORE deleting record, then remove from Storage

---

## 63. New Bot Creation — Complete Webhook Template

The canonical webhook handler every new bot should start from. This combines all patterns from §7, §9, §10, §28, §37, §42:

```typescript
export const runtime = 'nodejs';
export const maxDuration = 60;

export async function POST(req: NextRequest) {
  const update = await req.json();

  // 1. IDEMPOTENCE (§10)
  if (isProcessed(update.update_id)) return NextResponse.json({ ok: true });

  // 2. PARSE
  const from = update.message?.from || update.callback_query?.from;
  const message = update.message || update.callback_query?.message;
  const chatId = message?.chat?.id;
  const userId = from?.id;
  if (!chatId || !userId) return NextResponse.json({ ok: true });

  // 3. CALLBACK QUERIES (§9)
  if (update.callback_query) {
    await answerCallbackQuery(update.callback_query.id);
    await handleCallback(update.callback_query.data, chatId, userId);
    return NextResponse.json({ ok: true });
  }

  // 4. RATE LIMIT (§42)
  const rl = checkRateLimit(userId);
  if (!rl.canProceed) {
    await sendMessage(chatId, `⏱️ Espera ${rl.retryAfter}s.`);
    return NextResponse.json({ ok: true });
  }

  // 5. ACTIVATION (§28)
  const text = message?.text || '';
  if (text.startsWith('/start ')) {
    await handleActivation(userId, from.username, text.replace('/start ', '').trim(), chatId);
    return NextResponse.json({ ok: true });
  }

  // 6. TENANT LOOKUP (§2) — filter by THIS bot's rubro
  const tenant = await getUserTenant(userId); // filters by rubro_type
  if (!tenant) {
    await sendMessage(chatId, '👋 Usa /start CÓDIGO para activarte.');
    return NextResponse.json({ ok: true });
  }
  const tenantId = tenant.id;

  // 7. MESSAGE ROUTING (§7, §37) — ORDER MATTERS
  // 7a. Check active photo session
  // 7b. Check pending operation/confirmation
  // 7c. Handle photos/documents
  // 7d. Handle voice
  // 7e. Handle text commands
  // 7f. NLP / intent engine
  // 7g. "No entendí" fallback

  return NextResponse.json({ ok: true });
}
```

### New Bot Checklist

```
□ DB: tenant row with activation_code, rubro_type
□ DB: rubro entity tables with tenant_id FK + RLS
□ ENV: TELEGRAM_BOT_TOKEN for new bot
□ Webhook: register with Telegram API
□ Code: webhook route with all 7 steps above
□ Code: processor route with maxDuration: 300
□ Test: /start CODE → welcome message
□ Test: photo → accumulate → /listo → process
□ Test: verify draft created with correct tenant_id
□ Test: web UI shows the draft
```

---

## 64. Anti-Hardcode Comprehensive Reference

The single most important security rule in the codebase. Extended from §1-2 with the full anti-pattern catalog:

### The Three Poisonous Forms

```typescript
// Form 1: Exported constant — infects everything that imports it
export const DEFAULT_TENANT_ID = '550e8400-...';

// Form 2: Default parameter — silent escape hatch
async function getInventory(tenantId?: string) {
  const id = tenantId || DEFAULT_TENANT_ID; // ← poison
}

// Form 3: Default function parameter — same thing, different syntax
async function handleStock(chatId: number, tenantId = DEFAULT_TENANT_ID) {}
```

### The Audit Script

```bash
grep -r "DEFAULT_TENANT_ID" app/ lib/
grep -r "550e8400" .  # or whatever the hardcoded UUID was
grep -rn "tenantId\?:" lib/bot/  # optional tenantId = bug waiting to happen
grep -rn "tenantId.*=.*DEFAULT" .
```

### Middleware Resilience

```typescript
// middleware.ts — MUST wrap updateSession in try/catch
// If NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY is missing, updateSession() throws
// synchronously → MIDDLEWARE_INVOCATION_FAILED on ALL requests
try {
  response = await updateSession(request);
} catch {
  response = NextResponse.next({ request }); // degrade gracefully
}
```

### The Complete Anti-Pattern Table

| Never | Always | What Breaks |
|-------|--------|-------------|
| `DEFAULT_TENANT_ID` as export | No exported default ID | Silent data leak |
| `tenantId?: string` (optional) | `tenantId: string` (required) | Skippable = bug |
| `.single()` on telegram_users | `.order().find()` by rubro | Crash for multi-bot users |
| Query without `.eq('tenant_id')` | Every query filtered | Cross-tenant data leak |
| `updateSession()` without try/catch | Wrap in try/catch | Middleware kills ALL requests |
| JSONB array for photo accumulation | `session_photos` table, atomic INSERT | Race condition → lost photos |
| Iterate Storage folder | Iterate `raw_image_urls[]` from DB | Processes ALL tenant images |
| Upload to Storage then fail DB insert | Storage first, DB only on success | Orphaned DB records |
| `rubro_context` not set in sessions | Always set explicitly | CHECK constraint violation |
| Session TTL < 30 min for photo uploads | 30 minutes minimum | Expires mid-upload |
| Admin whitelist in env var | `telegram_users.role = 'admin'` in DB | Requires redeploy |
| Hardcoded site URL | `process.env.NEXT_PUBLIC_SITE_URL` | Breaks other deployments |
| No `response_format: json_object` | Always set for Kimi NLP calls | JSON.parse crashes |

---

## 65. Ghost Columns — Silent PostgREST Failures

**The pattern**: When `.select('id, title, price_uf')` includes a column that doesn't exist in the DB, PostgREST returns HTTP 400. But the Supabase JS client often swallows this — no error thrown, just null/empty data.

**How it manifests**: "The page loads but shows no data." No console errors. No crash. Just emptiness.

**Why it's silent**: The Supabase JS client treats a 400 from PostgREST as `{ data: null, error: { message: '...', code: 'PGRST204' } }`. Most code checks `if (!data) return []` — it never reads the error object. The fix isn't to crash harder; it's to prevent the mismatch in the first place.

**How to detect** (run when "data is empty for no reason"):
```sql
-- List all columns for a table:
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'YOUR_TABLE' ORDER BY ordinal_position;
```

```bash
# Find all .select() calls that might reference ghost columns:
grep -rn "\.select(" app/ lib/ --include="*.ts" --include="*.tsx" | grep -v node_modules
```

**The fix**:
1. Always verify column names against the actual DB with the query above
2. Keep `lib/schemas/index.ts` (canonical TypeScript interfaces) in sync with the DB
3. When a field "doesn't work", check if it exists before debugging logic
4. **Check the `error` field**: `const { data, error } = await query; if (error) console.error(error);`

**Why not auto-generate types from Supabase?** We tried `supabase gen types`. The output is 2000+ lines and drifts whenever we add columns without re-running. For a small team where one person controls the schema, the manual mapping table below is faster to consult and update.

**Known ghost column mappings** (properties table):

| What code used | What DB has | Notes |
|---------------|-------------|-------|
| `price_uf` | `price` + `price_currency` | price_currency is 'UF', 'CLP', etc. |
| `built_m2` / `sqm` | `built_area` | |
| `total_m2` / `lot_size` | `total_area` | |
| `is_active` | `status` ('published'/'draft'/'archived') | products still use `is_active` |
| `is_featured` | `featured` | |
| `parking` | `parking_count` | |
| `city` | `region` | |
| `price` (products) | `sale_price` | |
| `expense_date` (expenses) | `date` | Caused the gastos registration bug 2026-03-17 |

**Files hit by this in production**: `app/page.tsx`, `app/admin/real-estate/dashboard/`, `app/propiedades/[slug]/page.tsx`, `components/ui/PropertyCard.tsx`, `app/admin/real-estate/catalogo/[id]/editar/page.tsx` — all at once on 2026-03-17.

**Known ghost column mappings** (product_drafts → products via `batches/[id]/complete`):

| What code used | What DB has | Notes |
|---------------|-------------|-------|
| `draft.final_category` | doesn't exist | Use `draft.detected_category` |
| `draft.final_brand` | doesn't exist | Use `draft.detected_brand` |
| `draft.final_size` | doesn't exist | Use `draft.detected_size` |
| `draft.final_color` | doesn't exist | Use `draft.detected_color` |
| `draft.final_quantity` | doesn't exist | Use `draft.final_stock` or `draft.detected_quantity` |
| `draft.final_purchase_price` | doesn't exist | No equivalent in product_drafts |
| `purchase_price` (products insert) | doesn't exist | Use `cost_price` |
| `status: 'published'` (products insert) | column exists but admin API queries `status: 'active'` | Use `'active'` to match admin catalog query |

**Fixed 2026-03-23**: `app/api/batches/[id]/complete/route.ts` — all 8 ghost references above were causing the entire product insert to fail silently. Products were never created via batch-complete flow. See §73.

---

## 66. safeInt() — Kimi Returns Floats for Integer Columns

**The bug**: Kimi Vision returned `128.4` for `detected_built_area`. The DB column is `integer`. PostgreSQL error: `22P02: invalid input syntax for type integer: "128.4"`.

**Why this happens**: LLMs don't know your DB schema. Kimi sees "128 m²" in a photo and sometimes returns `128`, sometimes `128.4`, sometimes `"128"` as a string. The output is probabilistic, not typed.

**Why `Math.round()` and not `parseInt()`**: `parseInt("128.4")` returns `128` (truncates). `Math.round(Number("128.4"))` returns `128` (rounds). Both work for this case, but `Math.round` is more correct for values like `2.7 bathrooms` → `3` (yes, Kimi has returned this). `parseInt` would give `2`. And `Number()` handles edge cases like `"  128 "` that `parseInt` silently accepts but `Math.round` coerces correctly.

**The fix** — define once, use everywhere:
```typescript
// Define at top of file or in a shared util:
const safeInt = (v: any): number | null => (v != null ? Math.round(Number(v)) : null);

// Apply to ALL integer fields from Kimi output:
detected_bedrooms: safeInt(analysis.bedrooms),
detected_bathrooms: safeInt(analysis.bathrooms),
detected_built_area: safeInt(analysis.built_area),
detected_total_area: safeInt(analysis.total_area),
detected_year_built: safeInt(analysis.year_built),
detected_parking_count: safeInt(analysis.parking_count),
detected_common_expenses: safeInt(analysis.common_expenses),
```

**How to detect**: grep for Kimi output being written to DB without coercion:
```bash
grep -rn "analysis\." app/api/ --include="*.ts" | grep -v safeInt | grep detected
```

**Rule**: Never trust AI output types. Always coerce before DB write. This applies to Kimi, GPT, any LLM — they all do it.

**File**: `app/api/admin/properties/process-draft/route.ts`

---

## 67. CHECK Constraints Must Match All Code Paths

**The cascade failure**: `process-draft` tried to set `status: 'error'` on failure, but `property_drafts_status_check` didn't include `'error'`. The error handler ITSELF failed. Draft stuck in `pending_analysis` forever with no way to recover.

**Why this is especially dangerous**: The constraint violation happens at DB level, AFTER your code runs. The Supabase client returns `{ data: null, error: { code: '23514' } }`. If your error handler tries to write a status that also violates the constraint, you get an infinite failure loop — the error handler can't report the error.

**Three constraint violations in one session**:
1. `property_drafts_status_check` missing `'error'` — error recovery broken
2. `property_drafts_status_check` missing `'processing'` — web upload flow broken
3. `property_inquiries_status_check` missing `'converted'` — inquiry conversion broken

**Why CHECK constraints instead of just validating in code?** Because multiple code paths write to the same table (bot, web upload, admin panel, cron). A CHECK constraint is the only guarantee that ALL paths respect the same set of values. Code-only validation means each new endpoint is a potential constraint drift. The trade-off: you must update the DB constraint when adding new statuses, which requires a migration.

**Diagnostic query** (run this BEFORE deploying code that uses a new status):
```sql
-- See all CHECK constraints on a table:
SELECT conname, pg_get_constraintdef(oid)
FROM pg_constraint
WHERE conrelid = 'TABLE_NAME'::regclass AND contype = 'c';

-- Fix: drop old constraint, add new one with all statuses:
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
ALTER TABLE table_name ADD CONSTRAINT constraint_name
  CHECK (status IN ('pending_analysis', 'processing', 'auto_detected', 'approved', 'archived', 'error'));
```

**How to detect** (find status strings in code that might not be in the DB):
```bash
# Find all status assignments across the codebase:
grep -rn "status.*=.*'" app/api/ lib/ --include="*.ts" | grep -v node_modules | grep -v '//'
```

**Rule**: When adding a new status value ANYWHERE in code, run the diagnostic query first. The constraint must include the value before the code deploys.

---

## 68. Dual-Table Archives

**The pattern**: When entities exist in both a "published" table (`properties`) and a "drafts" table (`property_drafts`), the archivados page must query BOTH.

```
Discarded from borradores → property_drafts.status = 'archived'
Archived from catálogo    → properties.status = 'archived'
```

Both land in `/admin/real-estate/archivados` — but they're different tables, different APIs, different restore routes.

**Implementation**:
```typescript
const [propsRes, draftsRes] = await Promise.all([
  fetch(`/api/admin/properties?slug=${slug}&status=archived`),
  fetch(`/api/admin/property-drafts?slug=${slug}&status=archived`),
]);
```

**Restore routes**:
- Properties: `PUT /api/admin/properties/[id]` with `{ status: 'published' }`
- Drafts (inmo): `POST /api/drafts/[id]/restore` → sets `auto_detected`
- Drafts (retail): `POST /api/drafts/[id]/restore-product` → sets `auto_detected`

**Files**: `app/admin/real-estate/archivados/page.tsx`, `app/admin/retail/archivados/page.tsx`

---

## 69. Empty Body → request.json() Crash

**The bug**: A POST route calls `await request.json()` but the frontend sends no body (just method + headers). Result: `SyntaxError: Unexpected end of JSON input`.

**The fix** (for routes where body is optional):
```typescript
// BEFORE (crashes on empty body):
const { reason, source = 'drafts' } = await request.json();

// AFTER (graceful fallback):
const body = await request.text();
const { reason, source = 'drafts' } = body ? JSON.parse(body) : {};
```

**Rule**: If a route's body fields are all optional, use the text-then-parse pattern. If body is required, `request.json()` is fine — the crash is the correct behavior.

**File**: `app/api/drafts/[id]/discard/route.ts`

---

## 70. Polling Must Match Writer Status

**The gotcha**: Draft detail page polled with `if (draft?.status !== 'processing') return;` but `draft-from-upload` was creating drafts with `status: 'pending_analysis'`. Polling never started — the page just sat there.

**Fix**: Poll on ALL "in-progress" statuses:
```typescript
if (draft?.status !== 'processing' && draft?.status !== 'pending_analysis') return;
```

**Rule**: When adding polling, grep for the status string in:
1. The writer (API route that creates/updates the record)
2. The reader (frontend useEffect that polls)

If they don't match, polling silently never triggers.

**File**: `app/admin/real-estate/borradores/[id]/page.tsx`

---

## 71. Session Size Limits — Photo Upload Budget

**The rule**: 10MB total per media_queue session, not per file. Enforced in both bots (ropero + inmobiliario).

**Why 10MB total, not per file?** Telegram compresses photos to ~200-500KB each. A normal session of 10-15 photos is 3-5MB. The limit protects against: (1) users sending photos as uncompressed documents, (2) burst uploads that exceed Vercel function memory, (3) Kimi Vision base64 inflation (~33% overhead on top of raw size).

**Why not compress server-side?** We don't process photos on receive — we just store the `file_id`. Compression would require downloading first, which defeats the purpose of the budget check. Future optimization: `sharp` resize before sending to Kimi (post-download, pre-analysis), but that's a processing optimization, not an upload gate.

**Implementation**: `getQueueTotalSize()` in `lib/image-systems/yamato-system-inmobiliario/index.ts` sums `file_size` from `media_queue_photos`. Both webhooks check before `addPhotoToQueue()`.

**Files**: `app/api/bot/webhook/ropero/route.ts`, `app/api/bot/webhook/inmobiliario/route.ts`

---

## 72. Cron Jobs — Vercel Scheduled Functions

**Pattern**: Vercel crons are GET requests to API routes, configured in `vercel.json`. They run on Vercel's schedule (Hobby: daily, Pro: configurable).

**Active crons**:
| Path | Schedule | What it does |
|------|----------|-------------|
| `/api/cron/cleanup-queues` | `0 * * * *` (hourly) | Marks media_queues >1h as 'abandoned' |

**Security**: All cron endpoints require `Authorization: Bearer {CRON_SECRET}`. Vercel automatically sends this header for configured crons. For manual testing: `curl -H "Authorization: Bearer $CRON_SECRET" https://your-domain/api/cron/cleanup-queues`.

**Why soft-abandon not hard-delete?** The photos in `media_queue_photos` might be useful for debugging or recovery. Setting status to 'abandoned' excludes them from active queries without losing data. A future cron can hard-delete abandoned queues older than 30 days.

**File**: `vercel.json`, `app/api/cron/cleanup-queues/route.ts`

---

## 73. Batch-Complete Ghost Columns — The Silent Product Black Hole

**Date**: 2026-03-23 | **Severity**: Critical | **Symptom**: "Approved drafts never appear in catalog"

**The bug**: `app/api/batches/[id]/complete/route.ts` tried to insert products using 6 non-existent columns from `product_drafts` (`final_category`, `final_brand`, `final_size`, `final_color`, `final_quantity`, `final_purchase_price`) and wrote to a non-existent `purchase_price` column in `products` (correct name: `cost_price`). PostgREST rejected the insert with PGRST204, but the catch block silently logged the error and continued — reporting `success: false` per draft but no user-visible error.

**Second bug in same file**: Even if the insert had worked, `status: 'published'` was set — but the admin catalog API at `/api/admin/products` queries `.eq('status', 'active')`. Products would have been invisible.

**Third bug found nearby**: `/api/admin/product-drafts` received `status=batch_reviewed,ready_to_publish` from the UI but didn't parse comma-separated values. It treated the truthy string as "not archived" and returned ALL drafts. Client-side `.filter()` masked this.

**Root cause**: Two endpoints for publishing products were written at different times with different column assumptions:
- `publish-product/route.ts` — correct, aligned with DB schema ✅
- `batches/[id]/complete/route.ts` — assumed columns that never existed ❌

**The lesson**: When you have **two code paths to the same DB table**, they MUST use the same column names. If one works and the other doesn't, diff them. The working endpoint is your source of truth.

**Detection recipe**:
```sql
-- Compare what code inserts vs what DB accepts:
SELECT column_name FROM information_schema.columns WHERE table_name = 'products';
-- Then grep the insert object in the endpoint:
-- grep -A 20 "\.insert(" app/api/batches/\[id\]/complete/route.ts
```

**Fix**: Aligned `batches/[id]/complete` with the proven `publish-product` pattern. Changed `status: 'published'` → `'active'`, removed ghost columns, added `is_visible_in_store: true`.

**Files**: `app/api/batches/[id]/complete/route.ts`, `app/api/admin/product-drafts/route.ts`

---

## Phase 5 — Still to Add

- [ ] Product variants and batch grouping logic (batch_id grouping exists, variant model pending)
- [x] Cart system — localStorage cart, stock validation on checkout, WhatsApp order. Tested E2E 2026-03-18.
- [ ] Deployment checklist (env vars, Vercel config, DNS, webhook registration)
- [ ] Webhook URL registration (how to set/update Telegram webhook — `setWebhook` API call)
- [ ] Contextual memory system (episodic memory table, anaphora resolution — Lab concept)
- [ ] Security protocol (bot hardening, anti-nuke, tenant limits — designed 2026-03-18, see horizonte doc)
- [ ] Image compression pipeline (sharp resize before Kimi, reduce base64 payload)

---

## References

| Document | What |
|----------|------|
| `Playbook-bot-architecture-and-vertical-expansion.md` | Bot capabilities, feature parity, vertical expansion recipe |
| `Playbook-IRCE-pipeline-for-verticals.md` | IRCE pattern for any vertical |
| `Playbook-two-phase-async-processing.md` | Fire-and-forget pattern in depth |
| `IRCE-transplant-plan-from-proyecto-miche.md` | Specific file-by-file transplant plan |
| `Bot-transplant-plan-from-legacy.md` | T1-T4 UX transplants (done) |
| `00.vertical-lego-template.v2.0 vigente auditar actualizar.md` | Feature composability model |
| `possibilities-about-generating-webs-for-the-future.md` | AI web generation architecture |

---

*This is the doc you hand to someone with zero context about this project and expect them to debug, build, and ship without calling the original author. Not a 3 AM developer — a fresh pair of eyes with no institutional knowledge. Everything that bit us is here. Everything that saved us is here. And for every pattern, the reason we chose it over the alternative that looked just as good.*
