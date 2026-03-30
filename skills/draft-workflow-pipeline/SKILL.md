---
name: draft-workflow-pipeline
description: Complete photo batch processing pipeline combining backend async accumulation, Kimi vision extraction, and frontend draft review UI with invoice-style grouping and bulk approval workflows.
origin: Nadistudio
trigger: When implementing a photo batch processing pipeline with Telegram bot integration, Kimi Vision extraction, and draft review UI. Use when user mentions "draft workflow", "photo batch processing", "borradores", "lote de fotos", "/listo command", "invoice-style review", "bulk photo upload", or "product draft approval".
---

# Draft Workflow Pipeline

---

## Part 1: Activation Triggers

### When to Use This Skill

When a user uploads product photos via Telegram bot, the pipeline:
1. **Accumulates** photos in `media_queue` (backend)
2. **Processes** via Kimi Vision when user sends `/listo` (backend)
3. **Extracts** ~140 product variables into `product_drafts` (backend)
4. **Displays** batches in invoice-style UI with grouping by `product_group_key` (frontend)
5. **Approves** drafts atomically, publishing products (frontend)

This skill integrates **batch-processor** (photo accumulation, async Kimi processing) + **draft-reviewer** (UI review, bulk approval).

---

## Part 2: Information

### Status Flow Diagram

```
accumulating → pending_analysis → auto_detected → batch_reviewed → ready_to_publish → published
                                        ↓                ↓                  ↓
                                     archived        archived          archived
```

**Flow Notes:**
- `accumulating` → `pending_analysis`: user sends `/listo`, webhook triggers `/api/bot/process`
- `pending_analysis` → `auto_detected`: Kimi Vision extracts product variables
- `auto_detected` → `batch_reviewed`: user reviews & edits in batch detail UI
- `batch_reviewed` → `ready_to_publish`: inline edits complete
- `ready_to_publish` → `published`: bulk approval API inserts to `products` table
- Any status can → `archived`: user discards batch

---

### Backend Pipeline

#### Step 1: Tables

```sql
CREATE TABLE IF NOT EXISTS media_queue (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id BIGINT NOT NULL,
  chat_id BIGINT NOT NULL,
  status TEXT DEFAULT 'accumulating',
  rubro_context TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS media_queue_photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  queue_id UUID NOT NULL REFERENCES media_queue(id) ON DELETE CASCADE,
  file_id TEXT NOT NULL,
  file_size INTEGER,
  sort_order INTEGER
);

CREATE INDEX IF NOT EXISTS idx_media_queue_tenant ON media_queue(tenant_id, user_id, status);
CREATE INDEX IF NOT EXISTS idx_media_queue_photos ON media_queue_photos(queue_id);
```

**Notes:**
- `status: 'accumulating'` until `/listo` fires
- `file_id` stored first, downloaded later during processing
- 10MB per-batch limit enforced at photo handler

#### Step 2: Telegram Webhook (Photo Handler)

```typescript
if (message.photo || message.document?.mime_type?.startsWith('image/')) {
  let queue = await getActiveQueue(userId, tenantId);
  if (!queue) queue = await createQueue(tenantId, userId, chatId, rubroType);

  const currentSize = await getQueueTotalSize(queue.id);
  const newSize = message.photo?.[0]?.file_size || message.document.file_size;

  // 10MB limit
  if (currentSize + newSize > 10 * 1024 * 1024) {
    await sendMessage(chatId, '⚠️ El lote supera 10MB.');
    return;
  }

  await addPhotoToQueue(queue.id, fileId, newSize);
  await sendMessage(chatId, `📸 Foto ${sortOrder+1} recibida. Envía /listo cuando termines.`);
  return;
}
```

**Key behaviors:**
- Creates queue on first photo if none exists
- Validates file size before storing
- User sends `/listo` to trigger async processing

#### Step 3: /listo Handler (Fire-and-Forget)

```typescript
// Trigger when user sends /listo
const { error } = await after(async () => {
  await fetch(`${process.env.NEXTAUTH_URL}/api/bot/process?queueId=${queueId}`, {
    method: 'POST',
    headers: { 'x-tenant-slug': tenantSlug }
  });
});
```

**Notes:**
- Uses `after()` from `next/server` for fire-and-forget
- Responds to user immediately; processing happens in background

#### Step 4: Process Route (maxDuration: 300)

```typescript
// app/api/bot/process/route.ts
export const maxDuration = 300;

export async function POST(req: NextRequest) {
  const queueId = req.nextUrl.searchParams.get('queueId');
  const tenantSlug = req.headers.get('x-tenant-slug');

  const supabase = createBotClient();
  const tenantId = await getTenantId(tenantSlug);

  // 1. Fetch queue + photos
  const { data: queue } = await supabase
    .from('media_queue').select('*').eq('id', queueId).single();

  const { data: photos } = await supabase
    .from('media_queue_photos')
    .select('*')
    .eq('queue_id', queueId)
    .order('sort_order', { ascending: true });

  // 2. Download photos
  const photoBuffers = await Promise.all(photos.map(p => downloadFile(p.file_id)));

  // 3. Call Kimi Vision
  const { data: visionResult } = await fetch('https://api.kimi.ai/vision', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${KIMI_API_KEY}` },
    body: JSON.stringify({
      images: photoBuffers,
      prompt: `Extract ~140 product variables: name, price, stock, dimensions, colors, etc. Return as JSON array per image.`
    })
  }).then(r => r.json());

  // 4. Create product_drafts
  let batchId = generateBatchId();
  for (const vision of visionResult) {
    const variables = safeParse(vision.extraction);

    await supabase.from('product_drafts').insert({
      tenant_id: tenantId,
      batch_id: batchId,
      photo_path: `s3://photos/${batchId}/...`,
      status: 'auto_detected',
      detected_name: variables.name,
      detected_price: safeInt(variables.price),
      detected_quantity: safeInt(variables.stock),
      detected_sku: variables.sku,
      product_group_key: variables.group_key || null,
      // ... ~130 more columns
      metadata: JSON.stringify(variables)
    });
  }

  // 5. Update media_queue status
  await supabase.from('media_queue')
    .update({ status: 'pending_analysis' })
    .eq('id', queueId);

  return NextResponse.json({ batchId, count: visionResult.length });
}
```

**Key behaviors:**
- 300s timeout for Kimi API calls + large batch processing
- Stores `file_id` references, downloads on-demand
- Creates drafts with `status: 'auto_detected'`
- Atomic batch creation (all drafts share `batch_id`)
- Uses `safeInt()` on all extracted numbers

---

### API Layer

#### Batch List Endpoint

```typescript
// GET /api/batches?tenantId=...
export async function GET(req: NextRequest) {
  const tenantId = await getTenantId(req);
  const supabase = createBotClient();

  const { data: batches } = await supabase
    .from('product_drafts')
    .select('batch_id, status, created_at, count(*)')
    .eq('tenant_id', tenantId)
    .not('batch_id', 'is', null)
    .group('batch_id, status, created_at')
    .order('created_at', { ascending: false });

  return NextResponse.json(batches);
}
```

#### Batch Detail Endpoint

```typescript
// GET /api/batches/[id]?tenantId=...
export async function GET(req: NextRequest, { params }: { params: { id: string } }) {
  const tenantId = await getTenantId(req);
  const supabase = createBotClient();

  const { data: drafts } = await supabase
    .from('product_drafts')
    .select('*')
    .eq('batch_id', params.id)
    .eq('tenant_id', tenantId)
    .order('product_group_key, sort_order');

  return NextResponse.json(drafts);
}
```

#### Bulk Approval Endpoint

```typescript
// POST /api/batches/[id]/complete
export async function POST(req: NextRequest, { params }: { params: { id: string } }) {
  const tenantId = await getTenantId(req);
  const supabase = createBotClient();
  const { drafts: draftIds } = await req.json();

  const { data: drafts } = await supabase
    .from('product_drafts')
    .select('*')
    .eq('batch_id', params.id)
    .eq('tenant_id', tenantId)
    .in('id', draftIds);

  // Atomic insert to products + update drafts to 'published'
  for (const draft of drafts) {
    await supabase.from('products').insert({
      tenant_id: tenantId,
      name: draft.final_name || draft.detected_name,
      sale_price: safeInt(draft.final_price || draft.detected_price),
      stock_quantity: safeInt(draft.final_stock || draft.detected_quantity),
      sku: draft.final_sku || draft.detected_sku,
      // ... map other fields
    });

    await supabase.from('product_drafts')
      .update({ status: 'published' })
      .eq('id', draft.id);
  }

  return NextResponse.json({ success: true, count: drafts.length });
}
```

---

### UI Layer

#### Step 1: Batch List Page

```tsx
// app/admin/[vertical]/borradores/page.tsx
import { Card, CardContent } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import Link from 'next/link';

export default async function DraftsListPage({ params }: { params: { slug: string } }) {
  const tenantId = await getTenantId(params.slug);
  const batches = await fetch(`/api/batches?tenantId=${tenantId}`).then(r => r.json());

  return (
    <div>
      <h1>Borradores por Lote</h1>
      <div className="grid gap-4">
        {batches?.map(batch => (
          <Link href={`/admin/retail/borradores/${batch.batch_id}`} key={batch.batch_id}>
            <Card className="hover:shadow-md">
              <CardContent className="p-4 flex justify-between">
                <div>
                  <p>Lote #{batch.batch_id.slice(-6)}</p>
                  <p className="text-sm text-muted">{batch.count} productos</p>
                </div>
                <Badge>{batch.status}</Badge>
              </CardContent>
            </Card>
          </Link>
        ))}
      </div>
    </div>
  );
}
```

**Features:**
- Rows grouped by `batch_id`
- Status badge shows current phase
- Click to enter batch detail

#### Step 2: Batch Detail (Invoice-Style Grouping)

```tsx
// app/admin/[vertical]/borradores/[batchId]/page.tsx
export default async function BatchDetailPage({ params }: { params: { batchId: string } }) {
  const tenantId = await getTenantId(params.slug);
  const drafts = await fetch(`/api/batches/${params.batchId}?tenantId=${tenantId}`).then(r => r.json());

  // Group by product_group_key
  const groups = drafts.reduce((acc, draft) => {
    const key = draft.product_group_key || 'other';
    if (!acc[key]) acc[key] = [];
    acc[key].push(draft);
    return acc;
  }, {});

  return (
    <div>
      <h1>Lote #{params.batchId.slice(-6)}</h1>
      {Object.entries(groups).map(([groupKey, items]) => (
        <section key={groupKey} className="mb-8">
          <h2>{groupKey}</h2>
          <table className="w-full border-collapse">
            <thead>
              <tr className="border-b">
                <th>Foto</th>
                <th>Nombre</th>
                <th>Precio</th>
                <th>Stock</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              {items.map(draft => (
                <tr key={draft.id} className="border-b hover:bg-gray-50">
                  <td><img src={draft.photo_path} alt="" className="w-12 h-12" /></td>
                  <td>
                    <input
                      defaultValue={draft.final_name || draft.detected_name}
                      onBlur={(e) => updateDraft(draft.id, { final_name: e.target.value })}
                    />
                  </td>
                  <td>
                    <input
                      type="number"
                      defaultValue={draft.final_price || draft.detected_price}
                      onBlur={(e) => updateDraft(draft.id, { final_price: safeInt(e.target.value) })}
                    />
                  </td>
                  <td>
                    <input
                      type="number"
                      defaultValue={draft.final_stock || draft.detected_quantity}
                      onBlur={(e) => updateDraft(draft.id, { final_stock: safeInt(e.target.value) })}
                    />
                  </td>
                  <td>
                    <button onClick={() => archiveDraft(draft.id)}>Archivar</button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </section>
      ))}
      <button onClick={() => approveBatch(params.batchId)} className="bg-green-600 text-white px-4 py-2">
        ✓ Publicar Todo
      </button>
    </div>
  );
}

async function updateDraft(draftId: string, updates: Record<string, any>) {
  await supabase.from('product_drafts').update(updates).eq('id', draftId);
}

async function archiveDraft(draftId: string) {
  await supabase.from('product_drafts').update({ status: 'archived' }).eq('id', draftId);
}

async function approveBatch(batchId: string) {
  const { data: drafts } = await supabase
    .from('product_drafts')
    .select('id')
    .eq('batch_id', batchId)
    .neq('status', 'archived');

  await fetch(`/api/batches/${batchId}/complete`, {
    method: 'POST',
    body: JSON.stringify({ drafts: drafts.map(d => d.id) })
  });
}
```

**Features:**
- Invoice-style table with grouping by `product_group_key`
- Inline edits on `final_name`, `final_price`, `final_stock`
- Archive individual drafts without publishing
- Bulk "Publicar Todo" button to publish all (except archived)

---

## Part 3: Instructions

### Backend Constraints
- **Use `after()`** from `next/server` for fire-and-forget processing
- **10MB limit** per batch enforced at photo handler
- **Store `file_id` first**, download later during `/api/bot/process`
- **maxDuration: 300** on processor route for Kimi API timeout
- **safeInt()** on all integer fields from Kimi extraction
- **Atomic transactions** for batch creation (all drafts or none)

### Frontend Constraints
- **Group by `product_group_key`** if present; null values → "other"
- **Bulk approval must be atomic** (transaction or manual rollback)
- **safeInt()** validation on edited numbers before API call
- **Status enum** enforced: `accumulating`, `pending_analysis`, `auto_detected`, `batch_reviewed`, `ready_to_publish`, `published`, `archived`

---

## Part 4: Output

### Related Skills

- **[m1-m9-pipeline](../m1-m9-pipeline/SKILL.md)** — upstream photo ingestion from M1–M9 (Mariana workflow)
- **[irce-engineer](../irce-engineer/SKILL.md)** — IRCE compilation & inter-LLM task dispatch for batch processing
- **[vertical-factory](../vertical-factory/SKILL.md)** — multi-vertical tenant isolation & rubro context
- **[feature-expert](../feature-expert/SKILL.md)** — AI-driven feature detection for product attributes

---

## Information Gaps — Catastro

### What This Skill Does NOT Cover (Yet)

The following areas require clarification or are intentionally left undefined. When implementing this pipeline, verify with the user if these aspects apply to their use case:

#### Database Schema Gaps
- **Exact column names** for the ~140 product variables extracted by Kimi Vision — the skill references them but does not enumerate all columns
- **`product_drafts` table full schema** — only partial columns shown (detected_name, detected_price, etc.)
- **`products` table schema** — assumed to exist but structure not defined
- **Foreign key relationships** beyond tenant_id references
- **Indexing strategy** for `product_drafts` table queries (batch_id, status, tenant_id)

#### Kimi Integration Gaps
- **Exact API endpoint** for Kimi Vision — placeholder URL used (`https://api.kimi.ai/vision`)
- **Authentication mechanism** — assumes Bearer token but actual implementation may vary
- **Rate limiting** handling and retry strategies not documented
- **Vision prompt optimization** — the extraction prompt is generic and may need vertical-specific tuning

#### Storage Gaps
- **Photo storage destination** — references `s3://photos/` but actual S3 configuration not specified
- **CDN configuration** for serving images in the UI
- **Photo lifecycle management** — when/how are original Telegram files cleaned up

#### State Management Gaps
- **Error recovery** — what happens if Kimi Vision fails mid-batch
- **Partial failure handling** — some drafts succeed, others fail
- **Retry mechanism** for failed photo processing
- **Queue cleanup** — when are old `media_queue` entries purged

#### UI/UX Gaps
- **Real-time updates** — no WebSocket or polling mechanism defined for status changes
- **Mobile responsiveness** of the invoice-style table
- **Bulk edit operations** — editing multiple drafts at once not covered
- **Search/filter** within batch detail view not implemented

#### Security Gaps
- **RLS policies** for `media_queue`, `media_queue_photos`, and `product_drafts` tables not specified
- **Rate limiting** on `/api/bot/process` endpoint to prevent abuse
- **File type validation** beyond basic mime-type check
- **Virus/malware scanning** of uploaded photos

#### Multi-tenancy Gaps
- **Tenant isolation verification** — ensure no cross-tenant data leakage in batch queries
- **Resource quotas** per tenant (max batches, max photos per batch beyond 10MB)

### Resolution Protocol

When encountering any of these gaps during implementation:

1. **Check related skills** — [m1-m9-pipeline](../m1-m9-pipeline/SKILL.md) may cover photo ingestion details
2. **Consult feature-expert** — for vertical-specific product attribute requirements
3. **Ask the user** — for business-specific decisions on error handling, quotas, and UX preferences
4. **Default to conservative** — implement strict validation and clear error messages when uncertain
