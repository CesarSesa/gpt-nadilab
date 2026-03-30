---
name: vertical-factory
description: Generates a new vertical (rubro) from scratch: DB tables, bot webhook, Kimi prompt, admin pages, and public storefront. Based on Vertical Lego Template v2.1.
trigger: "add new vertical, create new rubro, expand to X industry, new business type, restaurant, automotive, lab, vertical factory"
origin: Nadistudio
---

# Vertical Factory

Generates a new vertical (rubro) from scratch: DB tables, bot webhook, Kimi prompt, admin pages, and public storefront.

---

## Part 1: When to Activate

Activate this skill when:
- User says "add new vertical", "create new rubro", or "expand to X industry"
- Adding restaurant, automotive, lab, or any new business type

---

## Part 2: Information Gaps — Catastro

Before generating a new vertical, collect or confirm:

| Gap | Question | Why It Matters |
|-----|----------|----------------|
| **Rubro Type** | What is the slug/key for this vertical? (e.g., `restaurant`, `lab`, `automotive`) | Used throughout: table names, routes, bot handlers |
| **Entity Name** | What do we call the main item? (e.g., `menu_item`, `test`, `vehicle`) | Names the core table and UI labels |
| **Detection Fields** | What can Kimi detect from photos vs. manual input? | Defines the `detected_*` vs `final_*` schema |
| **Bot Handle** | What is the Telegram bot username? (e.g., `@restobot`, `@labbot`) | Required for webhook routing |
| **IRCE Enabled** | Does this vertical need conversational commands? | Determines if IRCE pipeline is wired |
| **Existing Features** | Can we reuse features from `feature-expert` catalog? | Avoids reinventing already-built components |
| **Storefront Public** | Is there a public-facing catalog? | Determines if storefront pages are needed |
| **Batch Review** | Does the vertical need draft/batch review flow? | Most verticals do; confirms M1-M9 pipeline |

---

## Part 3: The Recipe (7 Steps)

### Step 0: Check Feature Catalog
Before creating a new vertical, check [`feature-expert`](../feature-expert/SKILL.md) to see what features already exist and can be reused.

### Step 1: Database
```sql
-- Create tenant with new rubro_type
INSERT INTO tenants (name, slug, rubro_type, activation_code, plan, settings)
VALUES ('Business Name', 'slug', 'new-rubro', 'CODE-' || upper(substring(gen_random_uuid()::text, 1, 6)), 'starter', '{}'::jsonb);

-- Rubro-specific tables follow standard pattern
CREATE TABLE IF NOT EXISTS new_entity (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  -- detected_* fields from Kimi
  -- final_* fields for user overrides
  status TEXT CHECK (status IN ('auto_detected','batch_reviewed','ready_to_publish','published','archived')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Standard indexes
CREATE INDEX IF NOT EXISTS idx_new_entity_tenant ON new_entity(tenant_id);
CREATE INDEX IF NOT EXISTS idx_new_entity_status ON new_entity(tenant_id, status);

-- RLS
ALTER TABLE new_entity ENABLE ROW LEVEL SECURITY;
CREATE POLICY service_role_access_new_entity ON new_entity FOR ALL TO service_role USING (true) WITH CHECK (true);
```

### Step 2: Bot Webhook
```typescript
// app/api/bot/webhook/{new}/route.ts
export async function POST(req: NextRequest) {
  // Copy from ropero template
  // Change rubro filter: .eq('tenants.rubro_type', 'new-rubro')
  // Add vertical-specific commands
}
```

### Step 3: Kimi Prompt
```typescript
// lib/bot/kimi.ts
export async function analyzeNewEntityImages(images: Buffer[], caption?: string) {
  // Copy from analyzeProductImages
  // Change prompt for new domain
  // Define {Entity}Item interface with detected_* fields
}
```

### Step 4: Process Route (add branch)
```typescript
// app/api/bot/process/route.ts
if (rubroType === 'new-rubro') {
  analysis = await analyzeNewEntityImages(imageBuffers, caption);
  // Create drafts in new_entity_drafts
}
```

### Step 5: Admin Pages
```
app/admin/[vertical]/new/
  dashboard/page.tsx      → stats
  catalog/page.tsx        → CRUD
  drafts/page.tsx         → batch review
  drafts/[id]/page.tsx   → individual review
```

### Step 6: Public Storefront
```
app/[vertical]/page.tsx        → catalog
app/[vertical]/[slug]/page.tsx → detail
```

### Step 7: Middleware (add case)
```typescript
// middleware.ts
case 'new-rubro': path = `/admin/new${path}`; break;
```

---

## Part 4: Reference & Constraints

### Rubro Profile Template

| Field | Value |
|-------|-------|
| rubro_type | `'new-rubro'` |
| vertical directory | `app/admin/new/` |
| bot | `@new_bot` |
| irce_enabled | `true/false` |
| Kimi detects | list fields |

### Kimi Prompt Variables

List what Kimi extracts for this vertical:
- Fields Kimi CAN detect from photos
- Fields that require manual input

### Constraints

- Follow LEGO GRANDE rules: no separate CSS, inline Tailwind
- Use only shadcn/ui components
- Mobile-first responsive
