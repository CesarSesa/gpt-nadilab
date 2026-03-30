---
name: feature-expert
description: Catálogo de features disponibles por vertical. Sabe qué páginas, APIs y DB integrations ya existen en el scaffold para evitar reinventar y favorecer reuso. Incluye cost awareness por feature.
trigger: "what features do we have, can we add X to vertical, cross-vertical feature, reusable components, feature catalog, vertical features, port feature, feature exists, feature cost, how much does X cost"
origin: Nadistudio
---

> **Note:** Before building a new vertical, check [`vertical-factory`](../vertical-factory/SKILL.md) skill for the creation recipe.  
> **Cost Context:** See `OPERATIONS-COSTS.md` for full cost model and incident history.

---

## Part 1: Mission — When to Activate

Activate this skill when:

- User asks "what features do we have?" or "can we add X to vertical Y?"
- Need to check if a feature exists before building something new
- Cross-vertical feature porting (e.g., "can CRM from Real Estate work in Retail?")
- Planning new vertical and need to know reusable components
- **Evaluating cost impact of a feature** ("how much does X cost per use?")

---

## Part 2: Context — Verticals Overview & Cost Awareness

### Verticals Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                      NADISTUDIO SCAFFOLD                            │
├──────────────────────┬──────────────────────┬───────────────────────┤
│   REAL ESTATE        │   RETAIL (Ropero)    │   TRANSVERSAL         │
│   (propiedades)      │   (tienda)           │   (shared)            │
├──────────────────────┼──────────────────────┼───────────────────────┤
│ • Property Catalog   │ • Product Catalog    │ • Draft System        │
│ • Property Detail    │ • Product Detail     │ • Batch Processing    │
│ • CRM/Clients        │ • Cart → WhatsApp    │ • Theme System        │
│ • Inquiries/Leads    │ • POS/Sales          │ • Image Systems       │
│ • Pipeline           │ • Inventory/Stock    │ • Bot Framework       │
│ • Search NLP         │ • Expenses           │ • Auth/Admin          │
│ • EasyProp Bridge    │ • IRCE Bot (voice)   │ • Tenant Mgmt         │
│                      │ • Clients            │ • Wizard Setup        │
└──────────────────────┴──────────────────────┴───────────────────────┘
```

### Cost Awareness by Feature

Each feature has a cost profile. When adding or scaling features, consider:

| Cost Driver | Description | Risk Level |
|-------------|-------------|------------|
| **Vercel** | Function invocations, CPU time, egress | 🟢 Low (generous limits) |
| **Supabase** | Egress (bandwidth out), DB size, storage | 🔴 **High** (egress is silent killer) |
| **Kimi** | Tokens per API call | 🟡 Medium (pay-as-you-go) |

### Cost by Feature Category

| Feature | Primary Cost | Avg Cost/Use | Risk | Notes |
|---------|--------------|--------------|------|-------|
| **EasyProp Bridge** | Kimi Vision | ~$0.025 | 🟡 Medium | 30 photos = one call |
| **IRCE Bot (regex)** | Vercel invocations | ~$0.00001 | 🟢 Low | Regex is cheap |
| **IRCE Bot (LLM)** | Kimi tokens | ~$0.001 | 🟡 Medium | 20% of cases |
| **Property Search NLP** | Kimi tokens | ~$0.002 | 🟡 Medium | Fallback only |
| **Image Upload** | Supabase Storage + Egress | Variable | 🔴 **High** | INC-001: egress bomb |
| **Cart → WhatsApp** | None (client-side) | $0 | 🟢 Low | No backend cost |
| **POS/Sales** | Supabase DB | ~$0.0001 | 🟢 Low | Simple writes |
| **Draft System** | Supabase DB | ~$0.0001 | 🟢 Low | Metadata only |
| **Image Transformations** | Supabase Pro feature | Per 1000 origins | 🟡 Medium | Resize on-the-fly |

**Incident History:**
- **INC-001 (2026-03-28):** Recursive logging caused 12GB egress (244% of free quota). Feature: Image Upload (indirect). Lesson: Any data egress can explode.

---

## Part 3: Execution — Detailed Feature Matrix

### 🔷 REAL ESTATE (`propiedades/`)

#### Public Pages
| Feature | Path | DB Table | Cost | Notes |
|---------|------|----------|------|-------|
| Property Listing | `/propiedades/page.tsx` | `properties` | 🟢 Low | Grid with filters |
| Property Detail | `/propiedades/[slug]/page.tsx` | `properties` | 🟡 Medium | Full card + images (egress) |
| Search NLP | Bot Telegram | `properties` | 🟡 Medium | Regex + Kimi fallback |

#### Admin Pages (`/admin/real-estate/`)
| Feature | Path | DB Tables | Cost | Special |
|---------|------|-----------|------|---------|
| Dashboard | `/dashboard/page.tsx` | stats, drafts | 🟢 Low | Pending drafts widget |
| Catalog | `/catalogo/page.tsx` | `properties` | 🟢 Low | CRUD operations |
| New Property | `/catalogo/nueva/page.tsx` | `property_drafts` | 🟢 Low | Creates draft first |
| Edit Property | `/catalogo/[id]/editar/page.tsx` | `properties` | 🟢 Low | Draft on edit |
| Clients/CRM | `/clientes/page.tsx` | `clients` | 🟢 Low | Contact management |
| Client Detail | `/clientes/[id]/page.tsx` | `clients` + history | 🟢 Low | Activity log |
| Inquiries | `/inquiries/page.tsx` | `property_inquiries` | 🟢 Low | Lead capture |
| Pipeline | `/pipeline/page.tsx` | `property_inquiries` | 🟢 Low | Kanban board |
| Drafts | `/borradores/page.tsx` | `property_drafts` | 🟢 Low | Approval workflow |
| Archived | `/archivados/page.tsx` | `properties` | 🟢 Low | Soft deleted |
| Settings | `/configuracion/page.tsx` | `tenants` | 🟢 Low | Tenant config |
| **EasyProp Bridge** | `/admin/easyprop-bridge/` | N/A | 🟡 **Medium** | AI extraction tool |

#### API Routes (`/api/`)
| Endpoint | Method | Purpose | Cost |
|----------|--------|---------|------|
| `/admin/properties` | GET/POST | List, create properties | 🟢 Low |
| `/admin/properties/[id]` | GET/PATCH/DELETE | CRUD single property | 🟢 Low |
| `/admin/property-drafts` | GET/POST | Draft management | 🟢 Low |
| `/admin/properties/draft-from-upload` | POST | AI extraction from image | 🟡 Medium |
| `/admin/properties/process-draft` | POST | Kimi process draft | 🟡 Medium |
| `/property-inquiries` | GET/POST | Leads from public | 🟢 Low |
| `/api/easyprop/analyze` | POST | Kimi Vision analysis | 🟡 **Medium** |
| `/api/easyprop/extract-text` | POST | PDF/DOC text extraction | 🟢 Low |

#### Bot Features (`lib/bot/`)
| Handler | File | Purpose | Cost |
|---------|------|---------|------|
| Search NLP | `search-properties.ts` | Regex extraction + Kimi fallback | 🟡 Medium |

#### EasyProp Bridge (Internal Tool)
**Purpose:** Extract property data from photos + documents → JSON for EasyProp SaaS

| Aspect | Details |
|--------|---------|
| **Path** | `/admin/easyprop-bridge/` |
| **Cost driver** | Kimi Vision API (~$0.025 per analysis) |
| **Input** | 1-30 photos + optional PDF/DOC/TXT |
| **Output** | JSON estructurado (operación, ubicación, características, textos) |
| **Limits** | 30 photos max, 35s timeout per document |
| **Steps** | 1) Upload photos → 2) Attach docs → 3) Review text → 4) Analyze |
| **Stack** | pdf2json (PDF), mammoth (DOCX), Kimi K2.5 Vision |
| **Added** | 2026-03-28 (Coder/Nadi collaboration) |

---

### 🔶 RETAIL (`tienda/`)

#### Public Pages
| Feature | Path | Storage | Cost | Notes |
|---------|------|---------|------|-------|
| Store Grid | `/tienda/page.tsx` | `products` | 🟢 Low | Product listing |
| Product Detail | `/tienda/[slug]/page.tsx` | `products` | 🟡 Medium | With Add to Cart |
| **Cart → WhatsApp** | `/tienda/carrito/page.tsx` | `localStorage` | 🟢 **Zero** | No backend, checkout WA |

#### Admin Pages (`/admin/retail/`)
| Feature | Path | DB Tables | Cost | Special |
|---------|------|-----------|------|---------|
| Dashboard | `/dashboard/page.tsx` | stats | 🟢 Low | Low stock alerts |
| Catalog | `/catalogo/page.tsx` | `products` | 🟢 Low | CRUD |
| New Product | `/catalogo/nuevo/page.tsx` | `products` | 🟢 Low | Direct or draft |
| Edit Product | `/catalogo/[id]/editar/page.tsx` | `products` | 🟢 Low | Draft on edit |
| Clients | `/clientes/page.tsx` | `clients` | 🟢 Low | Basic CRM |
| **Inventory** | `/inventario/page.tsx` | `stock_movements` | 🟢 Low | Stock levels |
| **Sales/POS** | `/ventas/page.tsx` | `sales` | 🟢 Low | Point of sale |
| Sales History | `/ventas/historial/page.tsx` | `sales` | 🟢 Low | Transactions |
| **Expenses** | `/gastos/page.tsx` | `expenses` | 🟢 Low | Expense tracking |
| Expense History | `/gastos/historial/page.tsx` | `expenses` | 🟢 Low | Past expenses |
| Drafts | `/borradores/page.tsx` | `product_drafts` | 🟢 Low | Approval flow |
| Archived | `/archivados/page.tsx` | `products` | 🟢 Low | Soft deleted |
| Settings | `/configuracion/page.tsx` | `tenants` | 🟢 Low | Tenant config |

#### API Routes
| Endpoint | Method | Purpose | Cost |
|----------|--------|---------|------|
| `/admin/products` | GET/POST | Product CRUD | 🟢 Low |
| `/admin/products/[id]` | GET/PATCH/DELETE | Single product | 🟢 Low |
| `/admin/products/[id]/stock` | POST | Stock adjustments | 🟢 Low |
| `/admin/sales` | GET/POST | Record sales | 🟢 Low |
| `/admin/expenses` | GET/POST | Record expenses | 🟢 Low |
| `/admin/dashboard-retail` | GET | Dashboard stats | 🟢 Low |

#### Bot Features (`lib/bot/` - IRCE Engine)
| Intent | Handler | File | Purpose | Cost |
|--------|---------|------|---------|------|
| SELL | Ventas | `sell-handler.ts` | Record sale by voice/text | 🟢 Low |
| QUERY | Consulta stock | `stock-query-handler.ts` | Check inventory | 🟢 Low |
| ADD_STOCK | Reposición | `add-stock-handler.ts` | Add stock | 🟢 Low |
| RECTIFY | Rectificación | `rectify-handler.ts` | Fix last sale | 🟢 Low |
| DELETE | Eliminación | `delete-handler.ts` | Delete item | 🟢 Low |
| **Engine** | Clasificación | `intent-engine.ts` | IRCE regex classifier | 🟢 Low |

#### Cart System (`lib/cart/`)
| Component | File | Purpose | Cost |
|-----------|------|---------|------|
| Context | `context.tsx` | React context for cart | 🟢 Zero |
| Icon | `cart-icon.tsx` | Cart icon with badge | 🟢 Zero |
| Hook | `useCart()` | Add, remove, update items | 🟢 Zero |

---

### ⚪ TRANSVERSAL Features

#### Draft System
**Purpose:** Aprobar cambios antes de publicar  
**Cost:** 🟢 Low (metadata only)

| Aspect | Details |
|--------|---------|
| Tables | `drafts` (generic), `property_drafts`, `product_drafts` |
| APIs | `/api/drafts/*`, `/api/drafts/[id]/approve`, `/api/drafts/[id]/discard` |
| Flow | Edit → Create Draft → Review → Approve (apply) / Discard |
| UI | Draft review page con comparación antes/después |

#### Batch Processing
**Purpose:** Procesar múltiples items (ej: imágenes masivas)  
**Cost:** 🟡 Medium (scales with volume)

| Aspect | Details |
|--------|---------|
| APIs | `/api/batches/[id]/*` (complete, discard, items) |
| Tables | `batch_operations`, `batch_items` |
| States | pending → processing → completed / failed |

#### Theme System (`lib/themes/`)
**Purpose:** Design tokens por vertical  
**Cost:** 🟢 Low (static assets)

| Theme | ID | Rubro | Colors |
|-------|-----|-------|--------|
| Real Estate Luxury | `real-estate-luxury` | real-estate | Gold accents |
| Real Estate Modern | `real-estate-modern` | real-estate | Blue/clean |
| Fashion Boho | `fashion-boho` | retail-clothing | Warm tones |
| Fashion Minimal | `fashion-minimal` | retail-clothing | B&W |
| Fashion Vibrant | `fashion-vibrant` | retail-clothing | Bold colors |

#### Image Systems (`lib/image-systems/`)
**Purpose:** Upload, optimización, variantes  
**Cost:** 🔴 **High** (egress risk — see INC-001)

| Feature | Description | Cost Risk |
|---------|-------------|-----------|
| Upload | Supabase Storage | 🟢 Low |
| Optimization | Resize, WebP conversion | 🟡 Medium |
| Variants | Thumbnail, medium, full | 🔴 High (multiple downloads) |

**Circuit Breakers:**
- Compress before upload (client-side)
- Use caching headers
- Consider R2 migration at 100+ tenants

#### Bot Framework (`lib/bot/`)
**Purpose:** Base para bots conversacionales  
**Cost:** 🟢 Low (stateless, cheap operations)

| Component | File | Purpose |
|-----------|------|---------|
| Config | `config.ts` | Tenant bot settings |
| Database | `database.ts` | Bot DB operations |
| Markdown | `markdown.ts` | Telegram formatting |
| Processor | `processor.ts` | Message processor |
| Session | `session.ts` | User session management |
| Telegram | `telegram.ts` | Telegram API wrapper |
| Voice | `voice.ts` | Voice message handling |

#### Auth/Admin
**Purpose:** Authentication and admin access  
**Cost:** 🟢 Low

| Feature | Path | Purpose |
|---------|------|---------|
| Admin Login | `/api/admin/auth` | Password auth (bcrypt) |
| Logout | `/api/admin/logout` | Clear cookies |
| Layout Guard | `layout.tsx` | Auth check per vertical |

#### Tenant Management
**Purpose:** Multi-tenancy core  
**Cost:** 🟢 Low

| Feature | Path | Purpose |
|---------|------|---------|
| Tenant CRUD | `/api/tenants/*` | Create/manage tenants |
| Check Slug | `/api/tenants/check-slug` | Slify availability |
| Password Gen | `/api/tenants/generate-passwords` | Initial passwords |

---

## Part 4: Validation — Cross-Vertical Portability & Decision Tree

### Cross-Vertical Feature Portability

#### ✅ CAN be ported (with adaptations)

| From | Feature | To | Notes |
|------|---------|-----|-------|
| Real Estate | CRM/Clients | Retail | Already exists in both |
| Real Estate | Pipeline | Other verticals | Good for lead tracking |
| Real Estate | Inquiries | Retail | Contact form → WhatsApp |
| Retail | Cart → WhatsApp | Real Estate | "Consultar" button instead |
| Retail | POS/Sales | Real Estate | For deposits/admin fees |
| Retail | Expenses | Real Estate | Operating costs tracking |
| Retail | Inventory | Real Estate | N/A (props are unique) |
| Both | Draft System | New verticals | Universal pattern |
| Both | Bot Framework | New verticals | Adapt intents |

#### ❌ CANNOT be ported directly

| Feature | Reason |
|---------|--------|
| Property Search NLP | Real Estate specific filters (dorms, baths, commune) |
| IRCE Sell Intent | Retail specific (stock management) |
| Property Pipeline | Real Estate sales cycle specific |
| EasyProp Bridge | Real Estate specific (property data schema) |

### Feature Decision Tree

```
Need to add feature X to vertical Y?
│
├─ Is it in this vertical already?
│  └─ Check matrix above
│
├─ Does another vertical have something similar?
│  └─ Check "Cross-Vertical Portability" table
│
├─ Is it a transversal feature?
│  └─ Use Draft System, Themes, Auth, etc.
│
├─ What's the cost impact?
│  └─ Check "Cost Awareness" table
│  └─ Will it scale with users (🟢) or with data (🔴)?
│
└─ Build new → Follow scaffold patterns
   ├─ Create page in app/[vertical]/
   ├─ Create API in app/api/[vertical]/
   ├─ Use existing DB patterns (tenant_id, RLS)
   ├─ Estimate cost per use
   └─ Add to bot if conversational needed
```

### Quick Reference: "Do We Have...?"

| Question | Answer | Location | Cost |
|----------|--------|----------|------|
| Cart with payment? | No | Only Cart → WhatsApp | N/A |
| CRM for clients? | Yes | Both verticals have it | 🟢 Low |
| POS system? | Yes | Retail only | 🟢 Low |
| Expense tracking? | Yes | Retail only | 🟢 Low |
| Property search? | Yes | Real Estate + Bot NLP | 🟡 Medium |
| Voice commands? | Yes | Retail bot (IRCE) | 🟢 Low |
| Draft approval? | Yes | Both verticals | 🟢 Low |
| Theme customization? | Yes | Theme system | 🟢 Low |
| Multi-language? | No | Spanish only | N/A |
| Mobile app? | No | Web only | N/A |
| WhatsApp checkout? | Yes | Retail cart | 🟢 Zero |
| Email notifications? | No | Not implemented | N/A |
| Analytics dashboard? | Basic | Stats widgets only | 🟢 Low |
| User roles/permissions? | No | Single admin per tenant | N/A |
| API for mobile? | Partial | Same endpoints | 🟢 Low |
| AI property extraction? | Yes | EasyProp Bridge | 🟡 Medium |

### Constraints & Limitations

1. **Cart is WhatsApp-only**: No payment gateway integration
2. **Bot only Telegram**: No WhatsApp Business API
3. **Single admin**: No multi-user per tenant
4. **No email**: No SMTP integration
5. **Spanish only**: No i18n framework
6. **Web only**: No React Native/PWA
7. **Chile focus**: Currency (CLP/UF), comunas hardcoded

### References

- `OPERATIONS-COSTS.md` — Full cost model, incident history, scaling scenarios
- `AGENTS.md` — Project constitution, destruction protection rules
- `ARCHITECTURE-RULES.md` — 19 rules before touching code

---

## Information Gaps — Catastro

**What this skill does NOT know (and must ask or infer):**

1. **New vertical requirements**: When user asks for a feature in a NEW vertical not listed here, must consult `vertical-factory` skill for creation recipe first.

2. **Exact code implementations**: This skill catalogs WHAT exists and WHERE, but not HOW it's implemented. For implementation details, must read the actual source files.

3. **Real-time cost data**: Cost estimates are approximate averages. Actual costs vary by usage patterns, data volume, and Supabase/Vercel pricing changes.

4. **Performance benchmarks**: Does not include latency metrics, query performance, or load testing results.

5. **Security audit status**: While features exist, their security posture (beyond RLS basics) is not cataloged here.

6. **User adoption metrics**: Does not track which features are actively used vs. deprecated.

7. **Third-party API limits**: Rate limits, quotas, and SLA terms for external services (Kimi Vision, Telegram, etc.) are not detailed here.

8. **Feature interdependencies**: Some features depend on others (e.g., Draft System requires proper RLS setup), but the full dependency graph is not mapped.

**When information is missing:**
- Ask user for clarification
- Read the actual source code in the referenced paths
- Check `OPERATIONS-COSTS.md` for cost details
- Consult `vertical-factory` for new vertical creation
- Use `migration-generator` skill for DB schema questions

---

*Skill: feature-expert v2.0 — 4-Part Format*  
*Updated: 2026-03-28 (migrated to 4-part structure)*
