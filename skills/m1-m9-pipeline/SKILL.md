---
name: m1-m9-pipeline
description: Las 9 Máquinas — Pipeline de procesamiento asíncrono para fotos y media. Use when building photo processing workflows, debugging stuck batches, or implementing new verticals with media upload. Covers the complete journey from Telegram photo to published product/property.
trigger:
  - "pipeline de fotos"
  - "m1-m9"
  - "las 9 máquinas"
  - "procesamiento de imágenes"
  - "telegram photo"
  - "media queue"
  - "batch processing"
  - "stuck batch"
  - "fotos no se procesan"
  - "/listo no funciona"
  - "debuggear pipeline"
  - "nuevo vertical con fotos"
  - "kimi vision"
  - "borradores no aparecen"
  - "fotos duplicadas"
---

# Las 9 Máquinas — El Viaje de la Foto

> *"Cada foto que envías viaja por 9 máquinas antes de convertirse en un producto publicado. Si algo falla, sabemos exactamente en qué máquina se atascó."*

Metáfora operativa del Nadistudio Scaffold para pipelines de procesamiento asíncrono. Convierte un flujo técnico complejo en una fábrica tangible que cualquier LLM (o humano) puede debuggear a las 3 AM.

---

## Part 1: Goals

### When to Activate

Activa este skill cuando:
- **Construyes** un nuevo vertical que procesa fotos desde Telegram
- **Debuggeas** batches atascados o fotos que "no pasan"
- **Implementas** el comando `/listo` o sus sinónimos
- **Conectas** Kimi Vision para análisis de imágenes
- **Migras** lógica de drafts de un vertical a otro
- **Explicas** el flujo async a un stakeholder no-técnico
- **Diagnostinas** por qué los borradores no aparecen en el admin
- **Extiendes** el pipeline a nuevos rubros (farmacia, automotriz, etc.)
- **Configuras** el timeout y fire-and-forget de procesamiento
- **Validas** que las fotos lleguen a Storage en el orden correcto

### When NOT to Use

NO uses este skill cuando:
- El procesamiento es **síncrono** (sin colas ni batches)
- No hay **fotos involucradas** (usa IRCE para texto puro)
- Estás en una **aplicación móvil nativa** sin Telegram
- El volumen es **masivo (>1000 fotos/hora)** — considera SQS
- Necesitas **procesamiento en tiempo real** (<1s latency)
- El cliente **no usa Telegram** como canal de entrada

### Core Principles

1. **Máquina = Checkpoint Debuggeable:** Cada máquina es un punto de inspección. Si algo falla, sabemos exactamente dónde.

2. **Acumular → Disparar:** Nunca procesamos fotos individuales. Siempre acumulamos en un batch y procesamos cuando el usuario da la señal (`/listo`).

3. **Fire-and-Forget Seguro:** El webhook debe responder rápido (<5s). El procesamiento pesado va en `after()` garantizado.

4. **Deduplicación por file_id:** Telegram envía el mismo file_id para la misma foto. Lo usamos para idempotencia.

5. **Sort Order Atómico:** Race-safe bajo webhooks concurrentes. Nunca `MAX() + 1` en cliente.

6. **Dual Write en Migraciones:** Durante transiciones, escribir a tablas legacy Y nuevas simultáneamente.

---

## Part 2: Execution

### Quick Start

Para implementar el pipeline completo en un nuevo vertical:

```bash
# 1. Crear tablas de cola
apply_migration: create_{vertical}_queue_tables

# 2. Implementar webhook handler con las 9 máquinas
# 3. Configurar Kimi Vision prompt específico
# 4. Crear tabla de drafts con estados
# 5. Testear flujo completo: /fotos → /listo → borradores
```

### Implementation Patterns

#### El Sistema Completo (ASCII Diagram)

```
Owner types a description note (optional)
        │
        ▼
🎙️ M1: El Micrófono — description captured with photos
        │  /fotos descripcion → media_queue.raw_text
        │  .txt file → media_queue.raw_text (inmobiliario only)
        │
Owner sends photo(s) via Telegram
        │
        ▼
📞 PBX receives the call (Layer 1)
        │ wrong secret → 401, call drops silently
        │ correct secret → proceed
        ▼
🗂️ M2: Mesa de Entrada — intake desk opens a batch envelope
        │ stamps: queue_id + received_at + tenant_id
        │ 30-minute clock starts
        │ getUserTenant filters by bot's rubro_type (1:1-per-rubro)
        ▼
📬 M3: Buzones Numerados — each photo gets a numbered slot
        │ sort_order via getNextSortOrder() — called PER PHOTO, atomic
        │ always take message.photo[last] — largest size only
        │ dedup by file_id (idempotency)
        ▼
Owner sends /listo (or synonyms: "terminé", "dale", "listos")
        │
        ▼
🔔 M4: El Disparador (/listo trigger) — batch is sealed and dispatched
        │ "⏳ Procesando..." sent immediately — clerk does NOT wait
        │ after() guarantees execution survives lambda exit
        ▼
🔩 M5: Tubo Neumático — sealed batch travels to external workshop
        │ fire-and-forget fetch → /api/bot/process (maxDuration: 300s)
        ▼
⚗️ M6: Taller del Alquimista [LAYER EXTERNAL — Kimi Vision API]
        │ Kimi K2.5, Allegro, api.moonshot.ai
        │ photos in → JSON report out: detected_*, confidence %, missing_critical
        │ has a 5-minute clock (maxDuration: 300)
        ▼
🖼️ M7: La Galería — photos uploaded to Supabase Storage
        │ path: /{tenant}/{entity}/{queue_id}/{sort_order}.ext
        │ public URL generated per photo
        │ DUAL WRITE: raw_image_urls[] + entity_images (migration phase)
        ▼
📋 M8: Bandeja de Borradores — draft tray filled with AI report
        │ Fashion: one draft per product (N photos → N drafts)
        │ Real Estate: one draft per property (N photos → 1 draft)
        │ awaiting human review at /admin/borradores
        ▼
📟 M9: Intercomunicador — buzzes the owner
        │ Telegram message with HTML parse mode
        │ Link: <a href="https://{slug}.nadistudio.cl/admin/borradores">Revisar</a>
        │ Rich feedback: per-product detail with emoji
        ▼
Owner reviews → approves (frame updates) or discards
```

### Step-by-Step

#### 🎙️ M1: El Micrófono

**Responsabilidad:** Capturar contexto textual junto con fotos.

**Input:**
- `/fotos [caption opcional]`
- Archivo `.txt` adjunto (inmobiliario)

**Output:** `media_queue.raw_text` (TEXT, nullable)

**Implementación:**
```typescript
// En webhook handler
const text = message.text || '';
const caption = text.startsWith('/fotos') 
  ? text.replace('/fotos', '').trim() 
  : '';

// Guardar en media_queue
await supabase.from('media_queue').insert({
  tenant_id: tenantId,
  user_id: userId,
  raw_text: caption || null,  // ← M1
  status: 'accumulating',
  rubro_context: rubroType,
});
```

**Lección Aprendida:** El caption puede ser crucial para Kimi. En inmobiliario, un `.txt` con la ficha técnica mejora drásticamente la detección.

---

#### 🗂️ M2: Mesa de Entrada

**Responsabilidad:** Crear el "sobre" (batch) y estamparlo con metadata.

**Input:** `tenantId`, `userId`, `chatId`, `rubro_context`

**Output:** Fila en `media_queue` con status `accumulating`

**Constraints:**
- 30-minute window (expires si no llega /listo)
- Un usuario = un queue activo por rubro
- Rubro filtering previene cross-contamination

**Implementación:**
```typescript
interface MediaQueueEntry {
  id: UUID;                    // queue_id
  tenant_id: UUID;
  user_id: BIGINT;
  chat_id: BIGINT;
  status: 'pending' | 'accumulating' | 'processing' | 'completed' | 'failed' | 'expired';
  rubro_context: 'fashion' | 'real-estate';
  raw_text: string | null;
  received_at: Timestamp;
  expires_at: Timestamp;       // received_at + 30 min
}

// Tenant resolution con rubro filter
const { data } = await supabase
  .from('telegram_users')
  .select('tenant_id, tenants!inner(id, slug, name, rubro_type)')
  .eq('telegram_user_id', userId)
  .eq('tenants.rubro_type', rubroType)  // CRÍTICO
  .order('activated_at', { ascending: false })
  .limit(1)
  .maybeSingle();
```

**Error Común (LX-TENANT-002):** Usuario con múltiples tenants del mismo rubro → fotos van a tienda equivocada. **Fix:** `order('activated_at', desc)` + filtro rubro.

---

#### 📬 M3: Buzones Numerados

**Responsabilidad:** Asignar número de orden atómico a cada foto, evitar duplicados.

**Input:** `file_id` de Telegram, `queue_id`

**Output:** Fila en `media_queue_photos` con `sort_order`

**Crítico:** Race-safe bajo webhooks concurrentes.

**Implementación:**
```typescript
// ATÓMICO: llamado DENTRO del loop por foto
async function getNextSortOrder(queueId: string): Promise<number> {
  const { data } = await supabase
    .from('media_queue_photos')
    .select('sort_order')
    .eq('queue_id', queueId)
    .order('sort_order', { ascending: false })
    .limit(1)
    .maybeSingle();
  
  return (data?.sort_order ?? 0) + 1;
}

// Procesamiento de foto
for (const photo of message.photo || []) {
  const fileId = photo.file_id;
  
  // Deduplicación por file_id
  const existing = await supabase
    .from('media_queue_photos')
    .select('id')
    .eq('file_id', fileId)
    .maybeSingle();
  
  if (existing) continue;  // Skip duplicado
  
  // Tomar SOLO la más grande (último elemento)
  const largestPhoto = message.photo[message.photo.length - 1];
  
  const sortOrder = await getNextSortOrder(queueId);
  
  await supabase.from('media_queue_photos').insert({
    queue_id: queueId,
    file_id: largestPhoto.file_id,
    sort_order: sortOrder,
    file_size: largestPhoto.file_size,
  });
}
```

**Anti-pattern:** `MAX(sort_order) + 1` en código cliente → race condition. **Siempre** consultar DB.

---

#### 🔔 M4: El Disparador

**Responsabilidad:** Detectar señal de inicio y validar antes de disparar.

**Triggers:** `/listo`, "terminé", "dale", "listos", "procesar", "analizar"

**Validaciones:**
- Queue existe y no está vacío
- Queue no expired (>30 min)
- User tiene tenant activo

**Implementación:**
```typescript
const LISTO_SYNONYMS = [
  'listo', 'listos', 'ready', 'done', 
  'terminé', 'termine', 'termino',
  'procesar', 'analizar', 'dale', 'enviar'
];

// En webhook
if (LISTO_SYNONYMS.includes(normalizedText)) {
  // Validar queue
  const { data: queue } = await supabase
    .from('media_queue')
    .select('id, status, created_at')
    .eq('user_id', userId)
    .eq('tenant_id', tenantId)
    .in('status', ['accumulating', 'pending'])
    .order('created_at', { ascending: false })
    .limit(1)
    .maybeSingle();
  
  if (!queue) {
    await sendMessage(chatId, '❌ No hay fotos pendientes. Usa /fotos primero.');
    return;
  }
  
  // Verificar fotos existen
  const { count } = await supabase
    .from('media_queue_photos')
    .select('*', { count: 'exact', head: true })
    .eq('queue_id', queue.id);
  
  if (count === 0) {
    await sendMessage(chatId, '❌ No hay fotos en este lote.');
    return;
  }
  
  // Responder inmediatamente
  await sendMessage(chatId, '⏳ Procesando tu lote... Esto puede tomar unos segundos. 🔄');
  
  // Disparar M5 (fire-and-forget)
  await triggerProcessing(queue.id, tenantId, userId, chatId);
}
```

---

#### 🔩 M5: Tubo Neumático

**Responsabilidad:** Transportar el batch al procesador sin bloquear el webhook.

**Problema que resuelve:** Vercel mata la función después de 60s (webhook) pero Kimi necesita 20-120s.

**Solución:** `after()` de Next.js 15+ (fire-and-forget garantizado).

**Implementación:**
```typescript
import { NextResponse, after } from 'next/server';

async function triggerProcessing(queueId, tenantId, userId, chatId) {
  const payload = { queueId, tenantId, userId, chatId };
  const processUrl = `${process.env.NEXT_PUBLIC_SITE_URL}/api/bot/process`;
  
  // ✅ CORRECTO: after() garantiza ejecución post-response
  after(async () => {
    try {
      const res = await fetch(processUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      });
      
      if (!res.ok) {
        console.error('[M5] Process failed:', res.status);
        // Notificar error
        await sendMessage(chatId, '❌ Error procesando. Intenta con /listo nuevamente.');
      }
    } catch (err) {
      console.error('[M5] after() error:', err);
    }
  });
  
  // Retornar 200 a Telegram INMEDIATAMENTE
  return NextResponse.json({ ok: true });
}

// ❌ ANTIGUO (bug): Bare fetch sin await
// fetch(processUrl, { ... }); // Se muere con el lambda
```

**Error Histórico:** Sin `after()`, el fetch moría al responder 200 a Telegram. Resultado: "Procesando..." eterno.

---

#### ⚗️ M6: Taller del Alquimista

**Responsabilidad:** Análisis inteligente de fotos con Kimi Vision.

**Input:** Array de buffers de imágenes, caption opcional

**Output:** JSON estructurado con `detected_*` fields

**Configuración Kimi (CRÍTICA):**
```typescript
const KIMI_CONFIG = {
  model: 'kimi-k2.5',
  temperature: 0.6,              // ÚNICO valor válido con thinking disabled
  response_format: { type: 'json_object' },
  thinking: { type: 'disabled' }, // CRITICAL para latencia
  max_tokens: 4000,
};
```

**Implementación (Fashion):**
```typescript
export async function analyzeProductImages(
  images: Buffer[],
  caption?: string
): Promise<ProductAnalysis> {
  
  const base64Images = images.map(buf => buf.toString('base64'));
  
  const response = await fetch('https://api.moonshot.ai/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.KIMI_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      ...KIMI_CONFIG,
      messages: [{
        role: 'user',
        content: [
          { 
            type: 'text', 
            text: `Analiza estas fotos de productos de retail. ${caption || ''}
            
REGLA DE AGRUPACION: Si ves varias prendas del MISMO tipo base pero en distintos colores o tallas, todas deben tener el MISMO "suggested_base_name".

Responde JSON con estructura:
{
  "items": [
    {
      "suggested_name": "Polera Azul Marino Talla M",
      "suggested_base_name": "Polera",  // Para agrupar variantes
      "detected_category": "poleras",
      "detected_brand": "Zara",
      "detected_color": "azul marino",
      "detected_size": "M",
      "detected_quantity": 1,
      "confidence": 0.92,
      "missing_critical": []
    }
  ]
}`
          },
          ...base64Images.map((b64, i) => ({
            type: 'image_url',
            image_url: { url: `data:image/jpeg;base64,${b64}` }
          })),
        ],
      }],
    }),
  });
  
  const data = await response.json();
  return JSON.parse(data.choices[0].message.content);
}
```

**safeInt Wrapper (IMPORTANTE):**
```typescript
// Kimi a veces retorna floats para campos integer
const safeInt = (v: any): number | null => 
  v != null ? Math.round(Number(v)) : null;

// Uso:
detected_quantity: safeInt(item.detected_quantity),
detected_price: safeInt(item.detected_price),
```

---

#### 🖼️ M7: La Galería

**Responsabilidad:** Persistir fotos en Storage con paths estructurados.

**Path Structure:**
```
{tenantId}/
  temp/
    drafts/{queueId}/{sortOrder}.jpg     ← Temporales
  entity/
    {entityType}/
      {entityId}/
        {displayOrder}.jpg               ← Finales
        thumbnails/
          {displayOrder}_thumb.jpg
```

**Implementación:**
```typescript
// Subida temporal (durante procesamiento)
async function uploadToStorage(
  buffer: Buffer,
  tenantId: string,
  queueId: string,
  sortOrder: number,
  bucket: string = 'product-images'
): Promise<string> {
  
  const path = `${tenantId}/temp/drafts/${queueId}/${sortOrder}.jpg`;
  
  const { error } = await supabase.storage
    .from(bucket)
    .upload(path, buffer, {
      contentType: 'image/jpeg',
      upsert: false,
    });
  
  if (error) throw error;
  
  const { data: { publicUrl } } = supabase.storage
    .from(bucket)
    .getPublicUrl(path);
  
  return publicUrl;
}

// DUAL WRITE (fase de migración)
await Promise.all([
  // Legacy
  supabase.from('product_drafts').update({
    raw_image_urls: publicUrls,
  }),
  // Future
  insertEntityImages({
    entity_type: 'draft',
    entity_id: draftId,
    storage_path: path,
    public_url: publicUrl,
    sort_order: sortOrder,
  }),
]);
```

---

#### 📋 M8: Bandeja de Borradores

**Responsabilidad:** Crear drafts con datos detectados, esperar aprobación humana.

**Flujo Fashion (1 foto → N productos):**
```typescript
// Kimi detectó 3 items en 1 foto
for (const item of analysis.items) {
  await supabase.from('product_drafts').insert({
    tenant_id: tenantId,
    batch_id: queueId,
    status: 'auto_detected',
    
    // Detected por Kimi (read-only)
    detected_name: item.suggested_name,
    detected_base_name: item.suggested_base_name,  // Para agrupar
    detected_category: item.detected_category,
    detected_brand: item.detected_brand,
    detected_color: item.detected_color,
    detected_size: item.detected_size,
    detected_quantity: safeInt(item.detected_quantity),
    detected_price: safeInt(item.detected_price),
    confidence: item.confidence,
    
    // Final (editable por usuario)
    final_name: null,
    final_price: null,
    final_stock: null,
    
    // Fotos
    raw_image_urls: photoUrls,
  });
}
```

**Flujo Real Estate (N fotos → 1 property):**
```typescript
// Una sola propiedad por batch
await supabase.from('property_drafts').insert({
  tenant_id: tenantId,
  batch_id: queueId,
  status: 'auto_detected',
  
  detected_title: analysis.suggested_title,
  detected_commune: analysis.detected_commune,
  detected_bedrooms: safeInt(analysis.bedrooms),
  detected_bathrooms: safeInt(analysis.bathrooms),
  // ... 20+ campos
  
  raw_image_urls: photoUrls,
});
```

**Estados del Draft:**
```
pending_analysis → auto_detected → batch_reviewed → ready_to_publish → published
                          ↓              ↓               ↓
                      archived      archived        archived
```

**Error Histórico (LX-DRAFT-001):** Admin panel legacy filtraba `status = 'active'` pero scaffold usa `'auto_detected'` → drafts invisibles.

---

#### 📟 M9: Intercomunicador

**Responsabilidad:** Notificar al usuario con mensaje rico y link a revisión.

**Mensaje Fashion:**
```typescript
const message = `✅ <b>Análisis completado</b>

📦 <b>${items.length} productos detectados:</b>
${items.map(i => 
  `• ${i.detected_name} (${i.detected_color}, ${i.detected_size})`
).join('\n')}

💰 <b>Precios sugeridos:</b> ${items.map(i => `$${i.detected_price}`).join(', ')}

⚠️ <b>Faltan:</b> ${missingFields.join(', ')}

<a href="https://${tenantSlug}.nadistudio.cl/admin/borradores">🔗 Revisar y publicar</a>`;

await sendMessage(chatId, message, { parse_mode: 'HTML' });
```

**Mensaje Real Estate:**
```typescript
const message = `✅ <b>Propiedad analizada</b>

🏠 <b>${draft.detected_title}</b>
📍 ${draft.detected_commune}
🛏 ${draft.detected_bedrooms}D / ${draft.detected_bathrooms}B
💰 ${draft.detected_price_uf} UF

📊 <b>Confianza:</b> ${draft.confidence}%

<a href="https://${tenantSlug}.nadistudio.cl/admin/borradores">🔗 Revisar ficha</a>`;
```

---

## Part 3: Quality Standards

### Pre-Delivery Checklist

Para agregar un nuevo vertical con las 9 Máquinas:

- [ ] M1: Definir cómo captura contexto (caption, .txt, voice?)
- [ ] M2: Crear tabla `{vertical}_queue` con rubro_context
- [ ] M3: Implementar `getNextSortOrder()` atómico
- [ ] M4: Definir sinónimos de "listo" para el vertical
- [ ] M5: Configurar `after()` con URL correcta
- [ ] M6: Escribir prompt Kimi específico del vertical
- [ ] M7: Definir path structure en Storage
- [ ] M8: Crear tabla `{vertical}_drafts` con estados
- [ ] M9: Diseñar mensaje de confirmación rico

### Validation Commands

**Debuggear por máquina:**

```sql
-- M2: Verificar queue activo
SELECT * FROM media_queue 
WHERE user_id = {user_id} 
AND status IN ('accumulating', 'pending')
ORDER BY created_at DESC;

-- M3: Verificar fotos en queue
SELECT * FROM media_queue_photos 
WHERE queue_id = '{queue_id}' 
ORDER BY sort_order;

-- M8: Verificar drafts creados
SELECT * FROM product_drafts 
WHERE batch_id = '{queue_id}';

-- Verificar duplicados potenciales
SELECT file_id, COUNT(*) 
FROM media_queue_photos 
GROUP BY file_id 
HAVING COUNT(*) > 1;
```

**Checklist por Máquina:**

| Máquina | Verificación | Query/Comando |
|---------|-------------|---------------|
| M1 | ¿Caption guardado? | `SELECT raw_text FROM media_queue WHERE id = X` |
| M2 | ¿Queue existe? | `SELECT * FROM media_queue WHERE user_id = X` |
| M3 | ¿Sort order único? | `SELECT sort_order FROM media_queue_photos WHERE queue_id = X` |
| M4 | ¿Sinónimo reconocido? | Revisar logs de webhook |
| M5 | ¿after() ejecutado? | Logs de Vercel /api/bot/process |
| M6 | ¿Kimi respondió? | Revisar respuesta en logs |
| M7 | ¿Fotos en Storage? | Bucket `product-images` o `properties` |
| M8 | ¿Drafts creados? | `SELECT * FROM product_drafts WHERE batch_id = X` |
| M9 | ¿Mensaje enviado? | Verificar en chat de Telegram |

### Common Errors

| Error | Máquina | Causa | Solución |
|-------|---------|-------|----------|
| LX-TENANT-002 | M2 | Usuario con múltiples tenants del mismo rubro | `order('activated_at', desc)` + filtro rubro |
| Race condition | M3 | `MAX(sort_order) + 1` en cliente | `getNextSortOrder()` atómico en DB |
| "Procesando..." eterno | M5 | Bare `fetch()` sin `after()` | Usar `after()` de Next.js 15+ |
| Error 22P02 | M6 | Sin `safeInt()` en campos Kimi | `Math.round(Number(v))` en todos los ints |
| Fotos mezcladas | M7 | Listar carpeta Storage en vez de array | Iterar `raw_image_urls[]` siempre |
| Ghost columns | M8 | Schema desactualizado | Validar schema antes de deploy |
| Parse error | M9 | Markdown v1 sin escape | Usar HTML mode con `escapeHtml()` |
| Drafts invisibles | M8 | Filtro `status = 'active'` legacy | Usar `'auto_detected'` |

---

## Part 4: Technical Reference

### API Reference

#### Estructura de Tablas

**media_queue:**
```sql
CREATE TABLE media_queue (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  user_id BIGINT NOT NULL,
  chat_id BIGINT NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  rubro_context VARCHAR(20),
  raw_text TEXT,
  received_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**media_queue_photos:**
```sql
CREATE TABLE media_queue_photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  queue_id UUID REFERENCES media_queue(id),
  file_id TEXT UNIQUE NOT NULL,
  sort_order INTEGER NOT NULL,
  file_size INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**product_drafts / property_drafts:**
```sql
CREATE TABLE product_drafts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  batch_id UUID REFERENCES media_queue(id),
  status VARCHAR(30) DEFAULT 'pending_analysis',
  detected_name TEXT,
  detected_base_name TEXT,
  detected_category TEXT,
  detected_brand TEXT,
  detected_color TEXT,
  detected_size TEXT,
  detected_quantity INTEGER,
  detected_price INTEGER,
  confidence DECIMAL(3,2),
  raw_image_urls TEXT[],
  final_name TEXT,
  final_price INTEGER,
  final_stock INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Funciones Clave

```typescript
// Obtener siguiente sort order (atómico)
async function getNextSortOrder(queueId: string): Promise<number>

// Disparar procesamiento (fire-and-forget)
async function triggerProcessing(
  queueId: string, 
  tenantId: string, 
  userId: bigint, 
  chatId: bigint
): Promise<void>

// Analizar imágenes con Kimi
async function analyzeProductImages(
  images: Buffer[], 
  caption?: string
): Promise<ProductAnalysis>

// Subir a Storage
async function uploadToStorage(
  buffer: Buffer,
  tenantId: string,
  queueId: string,
  sortOrder: number,
  bucket?: string
): Promise<string>

// Safe int wrapper
const safeInt = (v: any): number | null
```

### Integration with Other Skills

| Skill | Integración | Punto de Contacto |
|-------|-------------|-------------------|
| **irce-engineer** | M1 usa IRCE para parsear comandos de voz | `/fotos [descripción por voz]` |
| **kimi-prompt-engineer** | M6 usa prompts validados por este skill | Prompt de análisis de productos |
| **draft-workflow-pipeline** | M8 crea drafts que este skill consume | Tablas `*_drafts` |
| **api-error-handler** | M5-M7 usan este skill para errores | `withErrorHandler` en `/api/bot/process` |
| **db-guardian** | M2-M3 validan RLS y constraints | Políticas en `media_queue` |
| **migration-generator** | Usado para crear tablas M2, M3, M8 | `apply_migration` para nuevos verticales |

### Constraints

1. **Timeout Webhook:** 60s máximo (Vercel). Usar `after()` para procesamiento pesado.

2. **Kimi Timeout:** 300s máximo (`maxDuration` en route handler).

3. **Ventana de Acumulación:** 30 minutos desde primera foto.

4. **Tamaño de Fotos:** Telegram envía múltiples tamaños. Siempre usar `photo[-1]` (la más grande).

5. **Rate Limit Kimi:** Monitorear uso en dashboard de Moonshot.

6. **RLS Storage:** Service role necesario para escritura en buckets.

7. **Idempotencia:** Basada en `file_id` de Telegram. No re-procesa fotos duplicadas.

8. **Rubro Isolation:** Un bot = un rubro. Previene que fotos de moda vayan a inmobiliaria.

---

## Debugging por Máquina (Decision Tree)

**Síntoma: "Envié fotos y no pasó nada"**

```
M1: ¿Llegó el mensaje al webhook?
   └─ NO → Verificar webhook en Telegram, secret token, Vercel logs
   └─ SÍ → Ir a M2

M2: ¿Se creó el queue?
   └─ NO → Error en tenant resolution (¿rubro_type correcto?)
   └─ SÍ → Ir a M3

M3: ¿Hay fotos en media_queue_photos?
   └─ NO → Error en deduplicación o photo parsing
   └─ SÍ → Ir a M4

M4: ¿El usuario envió /listo (o sinónimo)?
   └─ NO → Instruir usuario: "Escribe /listo cuando termines"
   └─ SÍ → ¿Llegó "Procesando..."?
      └─ NO → Error en validación de queue
      └─ SÍ → Ir a M5

M5: ¿Hay logs de /api/bot/process?
   └─ NO → after() no disparó, revisar NEXT_PUBLIC_SITE_URL
   └─ SÍ → Ir a M6

M6: ¿Kimi respondió?
   └─ NO → API key inválida, rate limit, o timeout
   └─ SÍ → Ir a M7

M7: ¿Fotos en Storage?
   └─ NO → RLS error o bucket inexistente
   └─ SÍ → Ir a M8

M8: ¿Drafts creados?
   └─ NO → Ghost columns o constraint violation
   └─ SÍ → Ir a M9

M9: ¿Llegó mensaje a Telegram?
   └─ NO → Error en sendMessage o parse_mode
   └─ SÍ → Pipeline COMPLETO, revisar /admin/borradores
```

---

## Extensión a Nuevos Rubros

**Laboratorio Farmacéutico:**
- M1: Foto de factura + voz "llegó lote de paracetamol"
- M6: OCR de factura + validación de caducidad
- M8: Draft de entrada de inventario con alerta si vence en <6 meses

**Agenda/Reservas:**
- M1: "Quiero hora con Juan para el jueves"
- M6: NLP de intención + disponibilidad
- M8: Draft de cita pendiente de confirmación

**Automotriz:**
- M1: Fotos de vehículo + ficha técnica
- M6: Detección de daños + estimación de valor
- M8: Draft de ficha técnica para publicación

---

## Anti-Patterns & Lecciones

| Anti-pattern | Consecuencia | Solución |
|-------------|--------------|----------|
| M3: `MAX(sort_order) + 1` en cliente | Race condition → fotos sobreescritas | `getNextSortOrder()` atómico en DB |
| M5: Bare `fetch()` sin `after()` | Lambda muere, procesamiento perdido | Usar `after()` de Next.js 15+ |
| M6: Sin `safeInt()` en campos Kimi | Error 22P02 (invalid integer) | `Math.round(Number(v))` en todos los ints |
| M7: Listar carpeta Storage en vez de array | Procesa fotos de otros batches | Iterar `raw_image_urls[]` siempre |
| M8: Ghost columns en INSERT | Registro no creado, sin error visible | Validar schema antes de deploy |
| M9: Markdown v1 sin escape | Parse error, mensaje no llega | Usar HTML mode con `escapeHtml()` |

---

## Information Gaps — Catastro

Esta sección documenta lo que **NO** sabemos y necesitaríamos averiguar para completar el skill. Si encuentras respuestas, actualiza esta sección.

### Preguntas Abiertas

1. **M6 - Kimi Rate Limits:** ¿Cuál es el rate limit exacto de la API de Moonshot? ¿Tenemos retry logic implementado?

2. **M7 - Storage Cleanup:** ¿Hay un job automático que limpie fotos temporales de `temp/drafts/`? ¿Cuál es la política de retención?

3. **M8 - Batch Size Limits:** ¿Hay un límite máximo de fotos por batch? ¿Qué pasa si el usuario envía 100 fotos?

4. **M5 - Error Handling:** ¿Qué pasa si `after()` falla silenciosamente? ¿Tenemos dead letter queue?

5. **M2 - Expired Queues:** ¿Hay un job que marque automáticamente los queues expirados? ¿Notifica al usuario?

6. **M9 - Fallback:** ¿Qué pasa si el mensaje de Telegram no se puede enviar? ¿Se reintenta?

### Datos que Faltan

- Métricas de latencia por máquina (p95, p99)
- Tasa de error histórica por máquina
- Límites de tamaño de imagen por bucket
- Configuración de thumbnails (¿se generan automáticamente?)
- Estrategia de rollback si M6-M8 fallan parcialmente

### Contactos Útiles

| Tema | Quién lo sabe | Dónde encontrarlo |
|------|---------------|-------------------|
| Kimi Vision | ML Team | #ml-ops Slack |
| Storage RLS | Backend Lead | README de security-auditor |
| Telegram API | Bot Specialist | skill irce-engineer |
| Drafts UI | Frontend Team | skill draft-workflow-pipeline |

---

> *"Una imagen vale más que mil palabras, pero solo si puedes encontrarla en el Storage."* 🖼️

*Las 9 Máquinas: Metáfora operativa del Nadistudio Scaffold v7.0 — Formato 4-partes v2.0*
