# NADISTUDIO CASTLE v8.0 — OPUS UPDATE EDITION (COMPLETE)
> 🏰 **La Constitución Operacional del Castillo**
> 📅 Versión: 8.0 Complete | Fecha: 2026-03-25
> 🎯 **Para:** Nadi-Etéreo, Colaboradores LLM, DevOps de Emergencia, La Hermana
> ⚡ **Principio:** "Documentación que funciona a las 3 AM sin contexto previo"
> 🆕 **v8.0:** v7.0 Fusion + Skills consolidation (17→13) + DB Hardening Sprint 1 + mi-catalogo Sprint 2 + EasyProp Bridge

---

## ÍNDICE DE NAVEGACIÓN DE EMERGENCIA

| Necesitas... | Ve a... | Tiempo |
|--------------|---------|--------|
| **Quién construye esto (LEEME PRIMERO)** | [Parte 0: Contexto Humano](#parte-0-contexto-humano--quién-construye-este-castillo) | 3 min |
| Especificaciones técnicas duras | [Parte I-B: Topología Operativa](#parte-i-b-topología-operativa) | 2 min |
| Bot no responde AHORA | [Runbook RB-001](#rb-001-bot-completely-silent) | 2 min |
| Fotos van a tienda equivocada | [M2: Mesa de Entrada](#m2-mesa-de-entrada--intake-desk) | 5 min |
| "Procesando..." eterno | [M5-M6: Tubo/Alquimista](#m5-m6-tubo--alquimista) | 5 min |
| **Crear nuevo tenant (WIZARD)** | [Parte VI-B: Wizard](#parte-vi-b-wizard--tenant-en-2-minutos) | 1 min |
| **Entender el routing interno** | [Parte VI-C: Los Planos del Castillo](#parte-vi-c-los-planos-del-castillo--routing-interno) | 2 min |
| Los 6 comandos universales | [Parte III: 6 Comandos](#parte-iii-los-6-comandos-universales--robot-de-sastre) | 10 min |
| Blueprints copy-paste | [Parte VI: Blueprints](#parte-vi-blueprints-técnicos-copy-paste-ready) | 5 min |
| Matriz Bots ↔ Tenants | [Topología de Bots](#topología-de-bots-1-bot--1-rubro) | 1 min |
| SQL de auditoría | [Audit SQL Queries](#audit-sql-queries) | 1 min |
| **Estado Marzo 2026 (NUEVO)** | [Parte XII: Estado Operacional](#parte-xii-estado-operacional-marzo-2026) | 5 min |

---

## PARTE 0: CONTEXTO HUMANO — Quién Construye Este Castillo

> **Para cualquier LLM que lea esto por primera vez:** Esta sección te ahorra preguntar "¿quién es La Hermana?" o "asígnale esto a otro dev". Léela antes de opinar.
>
> *Contexto compilado desde v6.2 Kimi + v6.3 Opus + Wizard Deploy 2026-03-04*

### El Arquitecto

**Nadi-Etéreo** (César en el mundo físico, GitHub: CesarSesa) es UNA persona. Sociólogo de profesión, con 3 años de ingeniería civil electrónica, corredor de propiedades de oficio. No programa — **vibecodea** con LLMs desde enero 2026. 5 semanas de desarrollo. Todo este sistema fue construido así: **prompts + arquitectura de sistemas + iteración brutal**.

**No hay equipo de desarrollo.** Nadi + LLMs = el equipo completo. El patrón Professor Manhattan (Layer 6) es literal: opera en múltiples layers simultáneamente porque es la única persona.

El dolor es real: Nadi llena miles de campos manualmente en corretaje de propiedades. Este sistema nació para matar ese dolor.

Fanático de sci-fi (Asimov, Bradbury, Futurama, anime). Piensa en sistemas, layers y analogías — el Castle nació como narrativa y DESPUÉS se volvió técnico. Descubrió que las metáforas mejoran el debugging de LLMs más que las descripciones directas de bugs. Trabaja de noche. Las mejores ideas salen a las 3 AM con café.

### Los Trabajadores (LLMs)

| Worker | Rol en el Castillo | Plan | Fortaleza |
|--------|-------------------|------|-----------|
| **Kimi** (Moonshot) | Alquimista en producción (Vision API) + Coder principal | Allegro ($40/mes) | Multimodal, obediente con buen prompting. api.moonshot.ai |
| **Claude** (Anthropic) | Arquitecto/Coder quirúrgico | Pro ($20/mes) | Precisión brutal. Opus 4.6 = excelencia operacional |
| **Antigravity** (Google) | Escriba, documentación, UI | — | Velocidad, compilación cross-doc, Markdown nativo |

### Los Humanos Reales

| Alias | Quién es | Nivel Técnico | Rol en el Castillo |
|-------|----------|---------------|---------------------|
| **Nadi-Etéreo** (César) | Fundador. Corredor de propiedades. Sociólogo | Vibecoder (LLM-assisted) | Arquitecto, operador, tester, PM — TODO |
| **La Hermana** (Mariana) | La hermana biológica de César. Corredora RedPropertyChile | CERO | Tester QA "normie" en Plaza Mayor + operadora de EasyProp Bridge. **Si ella puede usarlo, cualquiera puede** |
| **Miche** (la pechugona) | Primera clienta real. Dueña de tienda "tu-stilo" | CERO | Inspiración del vertical retail-clothing. Odia inventariar a mano |

### Estado del Negocio (marzo 2026)

- **Producto:** Plataforma SaaS multi-tenant, 2 rubros (retail-clothing, real-estate)
- **Clientes pagando:** Ninguno aún. Lanzamiento semana del 4 de marzo
- **Pricing Tier 1:** $18-25 USD/mes — subdominio `*.nadistudio.cl`, 70 productos ó 30 propiedades
- **Marca comercial:** Yamato Core (landing, marketing, cara al cliente)
- **Marca técnica:** Nadistudio (plataforma, scaffold, infraestructura)
- **Legacy:** Deprecado el 2026-03-04. Todo concentrado en scaffold
- **Presupuesto:** Supabase Nano + Vercel Pro + Kimi Allegro + Claude Pro
- **Repos activos:** `Nadistudio-Scaffold-byKimi-FRESH` (scaffold) + `mi-catalogo` (real-estate reference impl)

### 🎉 HITO: Wizard Funcionando (2026-03-04 ~02:00)

> **"Mi hermana puede crear un tenant sin ayuda"** — Nadi, después de 8 horas de debugging

El **WIZARD** en `/wizard` permite crear tenants en 2 minutos:
1. Nombre del negocio → auto-genera slug
2. Rubro + Tema + Plan
3. Confirmar → URL + Código de activación listos

**Primer tenant creado:** `test-demo.nadistudio.cl` → Código `MODA-4OSH7G`

### Calibración para LLMs

Cuando sugieras algo:
- **No hay "otro dev"** — las sugerencias deben ser ejecutables por 1 persona + LLMs
- **El tester real es La Hermana** — si requiere terminal, no sirve como test de usabilidad
- **El presupuesto es limitado** — no sugieras services adicionales sin justificar el costo
- **La velocidad importa más que la perfección** — MVP funcional > arquitectura perfecta
- **"Keep it real"** = siempre distinguir lo que EXISTE vs lo que está DISEÑADO/PLANEADO
- **Máximo 2 niveles de abstracción** — regla de código del proyecto

---

## PARTE I: ARQUITECTURA DE CAPAS (L0-L5)

### El Stack Vertical Completo (Conceptual)

```
EXTERNAL ──── ⚗️ TALLER DEL ALQUIMISTA (Kimi Vision API — api.moonshot.ai)
                     ↑ photos in         ↓ analysis out
                     Kimi K2.5, Allegro $40/mes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LAYER 0: OBSERVABILITY + GOVERNANCE
  ☁️  Torre de Vigía    (monitoring, logs, feature gate)
  🧙‍♂️ Torre del Mago    (Nadi opera solo — deployer, tenant creation via wizard)
  🏰  Plaza Mayor       (landing, marketing, La Hermana testea aquí)

LAYER 1: COMMUNICATION
  📞  PBX Switchboard    (webhooks — technical entry)
  🧵  Agente Natural     (Kimi API — NL → commands)
  🕊️  Palomas            (outbound notifications)

LAYER 2: TENANT OPERATIONS
  🏪  El Mercado          (storefront: {slug}.nadistudio.cl)
  🧵  El Robot de Sastre  (6 commands + vertical wardrobe)
  🔧  La Trastienda       (9 machines: photo pipeline)

LAYER 3: PROTECTION
  🌊  El Foso             (pgvector memory — DISEÑADO, no construido)
  🛡️  El Muro             (middleware, auth, rate limiting)
  🔑  La Caja de Llaves   (env vars — all secrets)

LAYER 4: DATA
  🖼️  La Galería          (Supabase Storage — sa-east-1)
  📦  Las Bodegas         (PostgreSQL + RLS — Nano tier)

LAYER 5: EXPERIMENTATION
  🔬  El Sótano           (lab, staging)
  🧪  El Bot Maestro      (DISEÑADO, no construido — placeholder @nadistudio_lab_bot)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Regla de debug:** Un bug es casi siempre *"el mensaje se perdió entre dos capas"*. Identifica quién envió y quién debió recibir.

---

## PARTE I-B: TOPOLOGÍA OPERATIVA

> Datos duros que necesitas a las 3 AM. Analogías = comprensión, Topología = acción.

### Infraestructura Crítica

| Componente | Identificador | Región | Especificación |
|------------|---------------|--------|----------------|
| **Supabase** | `wpstagyqnmqdlfjmlzoo` | `sa-east-1` (São Paulo) | t4g.nano (0.5GB RAM, 5GB storage) |
| **Vercel** | `prj_Ty6ivfY5I6mOFDbax63qxh80M5Bu` | `iad1` (US East) | Pro Plan (serverless) |
| **Repo** | `Nadilabs-scaff` | GitHub | `main` branch = producción |
| **Wildcard** | `*.nadistudio.cl` | Vercel DNS | Apunta a `76.76.21.21` |

```yaml
supabase:
  project_id: "wpstagyqnmqdlfjmlzoo"
  url: "https://wpstagyqnmqdlfjmlzoo.supabase.co"
  region: "sa-east-1"
  instance: "t4g.nano"
  tier: "Nano"

vercel:
  team: "nadilabs"
  project_id: "prj_Ty6ivfY5I6mOFDbax63qxh80M5Bu"
  primary_domain: "scaffold.nadistudio.cl"
  wildcard: "*.nadistudio.cl"
  edge_regions: ["iad1", "gru1", "scl1"]
```

### Topología de Bots (1 Bot = 1 Rubro)

**Principio de oro:** Cada rubro tiene SU bot. No hay mezcla. El bot rechaza activaciones de rubros incorrectos.

| Bot Username | Token Env Var | Webhook Endpoint | Rubro | Estado |
|--------------|---------------|------------------|-------|--------|
| `@ropero_v1_bot` | `TELEGRAM_BOT_TOKEN_ROPERO` | `/api/bot/webhook/ropero` | `retail-clothing` | ✅ Activo |
| `@Bot_inmobiliario_v2_bot` | `TELEGRAM_BOT_TOKEN_INMO` | `/api/bot/webhook/inmobiliario` | `real-estate` | ✅ Activo |
| `@nadistudio_lab_bot` | `TELEGRAM_BOT_TOKEN` | `/api/bot/webhook/lab` | `*` (testing) | ✅ Lab |

### Esquema de Dominios y Tiers

```
┌─────────────────────────────────────────────────────────────────┐
│                    TIER 1: STARTER (Wildcards)                   │
│                    {slug}.nadistudio.cl                         │
├─────────────────────────────────────────────────────────────────┤
│  DNS: *.nadistudio.cl → CNAME → cname.vercel-dns.com           │
│                                                                  │
│  Resolución por Middleware:                                      │
│  tu-stilo.nadistudio.cl → Vercel → x-tenant-slug: tu-stilo      │
│                                                                  │
│  Tenants activos:                                                │
│  • tu-stilo.nadistudio.cl → retail-clothing                     │
│  • demo-inmo.nadistudio.cl → real-estate                        │
│  • test-moda-primavera.nadistudio.cl → testing                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    TIER 2: PRO (Custom Domain)                   │
│                    dominio-propio.com                            │
│                    DISEÑADO, NO IMPLEMENTADO                     │
└─────────────────────────────────────────────────────────────────┘
```

### Modelo de Tenants: 1 Usuario = 1 Tenant per Rubro

**Regla de negocio:** Un usuario de Telegram tiene exactamente un tenant por rubro. `getUserTenant()` filtra por `tenants.rubro_type` del bot que recibe el mensaje.

```
Nadi testeando (telegram_user_id: 2051086661)
         │
         ├── @ropero_v1_bot → tenant: test-moda-primavera (retail-clothing)
         ├── @Bot_inmobiliario_v2_bot → tenant: demo-inmo (real-estate)
         └── Cada bot filtra por su rubro → no hay confusión
```

---

## PARTE II: LAYER 6 — LA CONCIENCIA

### El Patrón Professor Manhattan

**Es literal, no metafórico.** Nadi-Etéreo opera simultáneamente en múltiples layers porque es la única persona. No es multitasking — es multi-presencia forzada por ser equipo de uno.

```
Nadi-Etéreo (César, 3 AM, con café) ve:
  → L0: "¿Deberíamos permitir batches de 50 fotos?" (decisión de producto)
  → L2: "El timeout de 300s se alcanza con 15+ fotos" (constraint técnico)
  → L5: "Podríamos implementar chunked processing" (solución experimental)
    ↓
Síntesis: "Límite de 10 fotos por batch, con cola para procesamiento
          secuencial si hay más. Implementar en Lab primero."
```

---

## PARTE III: LOS 6 COMANDOS UNIVERSALES + ROBOT DE SASTRE

### Principio: Array Architecture

**Todos los comandos operan sobre arrays.** El Robot de Sastre sabe *cómo expandir o colapsar* esos arrays según el rubro.

| Comando | Operación | Fashion (1→N) | Real Estate (N→1) | Supermarket (Text→N) |
|---------|-----------|---------------|-------------------|---------------------|
| `añadir-lote` | Write (array) | 1 foto → N productos (detección múltiple) | N fotos → 1 propiedad (análisis conjunto) | Invoice OCR → N SKUs |
| `vender` | Transaction | Array de ítems en caja rápida | N/A (ciclo largo) | Venta masiva de productos |
| `rectificar` | Update/Delta | Cambiar talla/stock/precio en array de SKUs | Cambiar precio/estado de propiedad | Cambiar precio/caducidad de SKUs |
| `consultar` | Query/RAG | "¿Cuántos jeans talla M?" | "¿Disponible en Ñuñoa?" | "¿Qué vence esta semana?" |
| `publicar` | Activate | Array de SKUs → catálogo web | 1 propiedad → mapa público | Array → góndola digital |
| `añadir-producto` | Write (single) | 1 prenda, fotos múltiples → 1 SKU | 1 propiedad, N fotos → 1 ficha | 1 SKU con caducidad específica |

### Schema JSON Unificado

```json
{
  "comando": "string — uno de los 6 universales",
  "rubro": "fashion | real-estate | supermarket",
  "payload": {
    "entrada": {
      "tipo": "array",
      "items": "contexto-rubro-específico"
    },
    "transformacion": "expand | collapse | passthrough",
    "salida": "array | objeto | confirmacion"
  },
  "universo_variables": ["lista de atributos válidos para este rubro"]
}
```

**Ejemplo — `añadir-lote` transformaciones:**

```json
// FASHION — Expansión 1→N
{
  "comando": "añadir-lote",
  "rubro": "fashion",
  "payload": {
    "entrada": { "fotos": ["url1"], "contexto": "lote de verano" },
    "transformacion": "expand",
    "salida": [
      { "sku": "auto", "nombre": "Polera Azul", "talla": "M", "color": "azul" },
      { "sku": "auto", "nombre": "Jeans Negro", "talla": "32", "color": "negro" }
    ]
  },
  "universo_variables": ["sku", "nombre", "talla", "color", "marca", "stock", "precio"]
}

// REAL ESTATE — Colapso N→1
{
  "comando": "añadir-lote",
  "rubro": "real-estate",
  "payload": {
    "entrada": { "fotos": ["url1", "url2", "url3"], "titulo": "Depto Ñuñoa" },
    "transformacion": "collapse",
    "salida": {
      "propiedad": { "titulo": "Depto Ñuñoa", "dormitorios": 2, "baños": 1 },
      "fotos": ["url1", "url2", "url3"]
    }
  },
  "universo_variables": ["metros_cuadrados", "dormitorios", "baños", "precio_uf", "comuna"]
}
```

---

## PARTE IV: LAS 9 MÁQUINAS — EL VIAJE DE LA FOTO

### Diagrama Completo

```
Owner types a description note (optional)
        │
        ▼
🎙️ M1: El Micrófono — description captured with photos
        │
Owner sends photo(s) via Telegram
        │
        ▼
📞 PBX receives the call (Layer 1)
        │ wrong combination → 401, call drops silently
        │ correct combination → proceed
        ▼
🗂️ M2: Mesa de Entrada — intake desk opens a batch envelope
        │ stamps: queue_id + received_at + tenant_id
        │ 30-minute clock starts
        │ active queue's tenant wins over last-activated tenant
        ▼
📬 M3: Buzones Numerados — each photo gets a numbered slot
        │ sort_order via getNextSortOrder() — race-safe, atomic
        │ always take message.photo[last] — largest size only
        ▼
Owner sends /listo
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
        │ photos in → JSON report out: detected_*, confidence %, missing_critical
        │ has a 5-minute clock (maxDuration: 300)
        ▼
🖼️ M7: La Galería — photos uploaded to Supabase Storage
        │ path: /{tenant}/{entity}/{queue_id}/{sort_order}.ext
        │ public URL generated per photo
        ▼
📋 M8: Bandeja de Borradores — draft tray filled with AI report
        │ Fashion: one draft per product (N photos → N drafts)
        │ Real Estate: one draft per property (N photos → 1 draft)
        │ awaiting human review at /admin/borradores
        ▼
📟 M9: Intercomunicador — buzzes the owner
        │ Telegram message: "✅ Analysis complete. Review at /admin/borradores"
        │ Link format: https://{slug}.nadistudio.cl/admin/borradores
        ▼
Owner reviews → approves (frame updates) or discards
```

### Referencia Rápida de Máquinas

| # | Máquina | Técnico | Clock | Estado | Notas |
|---|---------|---------|-------|--------|-------|
| M1 | **El Micrófono** | `media_queue.raw_text` | — | ✅ FIXED 2026-03-03 | Caption extraído con `startsWith('/fotos')`, guardado en DB, pasado a Kimi |
| M2 | **Mesa de Entrada** | `media_queue` handler | 30 min | ✅ | Tenant resolution con prioridad de queue activo |
| M3 | **Buzones Numerados** | `media_queue_photos` + `getNextSortOrder()` | — | ✅ | Race-safe, atómico |
| M4 | **El Disparador** | `/listo` command handler | — | ✅ | Validaciones de queue no vacío |
| M5 | **Tubo Neumático** | `fire-and-forget` + `after()` | 60s/300s | ✅ | Garantía de ejecución post-lambda |
| M6 | **Taller del Alquimista** | Kimi Vision API | 300s max | ✅ | Prompts específicos por rubro |
| M7 | **La Galería** | Supabase Storage upload | — | ✅ | Path estructurado, compresión WebP pendiente |
| M8 | **Bandeja de Borradores** | `product_drafts` / `property_drafts` | — | ✅ | Diferencia N drafts vs 1 draft por rubro |
| M9 | **Intercomunicador** | `sendMessage()` outbound | — | ✅ | Markdown escaping corregido v6.2 |

---

## PARTE V: LOS EDIFICIOS — FASHION VS REAL ESTATE

### Comparativa Completa + Topología de Bots

| Dimensión | Fashion (retail-clothing) | Real Estate |
|-----------|--------------------------|-------------|
| **Ratio Foto:Entidad** | 1 foto → N productos | N fotos → 1 propiedad |
| **Unidad mínima** | Producto individual (SKU) | Propiedad completa |
| **Bot** | `@ropero_v1_bot` | `@Bot_inmobiliario_v2_bot` |
| **Token Env** | `TELEGRAM_BOT_TOKEN_ROPERO` | `TELEGRAM_BOT_TOKEN_INMO` |
| **Webhook** | `/api/bot/webhook/ropero` | `/api/bot/webhook/inmobiliario` |
| **Draft logic** | N drafts por batch | 1 draft por batch |
| **NLP principal** | Venta conversacional | Búsqueda por filtros |
| **Variables clave** | SKU, Talla, Color, Marca, Stock | m², Dormitorios, Baños, UF, Comuna |
| **Precio** | CLP fijo | UF + CLP + gastos comunes |
| **Stock** | Sí (cantidades) | No (disponible/no disponible) |
| **Geografía** | No aplica | Crítica (mapa, comuna) |
| **Web pública** | `/tienda/[slug]` | `/propiedades/[slug]` |
| **Admin borradores** | `/admin/borradores` (batches) | `/admin/borradores` (individual) |

---

## PARTE VI: BLUEPRINTS TÉCNICOS (Copy-Paste Ready)

### BP-001: Webhook Handler Bulletproof

```typescript
// app/api/bot/webhook/[bot]/route.ts
export const runtime = 'nodejs';
export const maxDuration = 60;

export async function POST(request: Request) {
  // 1. VALIDATE (fail fast)
  const secret = request.headers.get('x-telegram-bot-api-secret-token');
  if (secret !== process.env.TELEGRAM_WEBHOOK_SECRET_ROPERO) {
    return new Response('Unauthorized', { status: 401 });
  }

  // 2. PARSE (with safety)
  let update: TelegramUpdate;
  try {
    update = await request.json();
  } catch {
    return new Response('Bad JSON', { status: 400 });
  }

  const message = update.message;
  if (!message?.from?.id) {
    return new Response('No user', { status: 200 });
  }

  const userId = message.from.id;
  const chatId = message.chat.id;
  const text = message.text || '';

  // 3. RESOLVE TENANT (priority: active queue > last activated)
  const tenantId = await getUserTenant(userId);
  if (!tenantId) {
    await sendMessage(chatId, '👋 Usa /start CÓDIGO para activarte');
    return new Response('OK', { status: 200 });
  }

  // 4. ROUTE
  try {
    if (text.startsWith('/start')) {
      await handleStart(userId, chatId, text, tenantId);
    } else if (text === '/fotos' || text.startsWith('/fotos ')) {
      await handlePhotos(userId, chatId, message.photo, text, tenantId);
    } else if (text === '/listo') {
      await handleListo(userId, chatId, tenantId);
    }
    return new Response('OK', { status: 200 });
  } catch (error) {
    // 5. ERROR HANDLING (never crash webhook)
    console.error('[Webhook Error]', error);
    await sendMessage(chatId, '❌ Error temporal. Intenta de nuevo.');
    return new Response('Error handled', { status: 200 });
  }
}
```

### BP-002: getUserTenant con Prioridad de Queue (1 Usuario = 1 Tenant)

```typescript
async function getUserTenant(userId: number): Promise<string | null> {
  const supabase = createBotClient();

  // PRIORITY 1: Active queue (last 30 min)
  const thirtyMinutesAgo = new Date(Date.now() - 30 * 60 * 1000).toISOString();
  const { data: activeQueue } = await supabase
    .from('media_queue')
    .select('tenant_id')
    .eq('source_user_id', userId)
    .in('status', ['pending', 'accumulating'])
    .gte('received_at', thirtyMinutesAgo)
    .order('received_at', { ascending: false })
    .limit(1);

  if (activeQueue?.length > 0) {
    return activeQueue[0].tenant_id;
  }

  // PRIORITY 2: Last activated tenant (1 usuario = 1 tenant)
  const { data: mapping } = await supabase
    .from('telegram_users')
    .select('tenant_id, tenants!inner(id, slug, name)')
    .eq('telegram_user_id', userId)
    .order('activated_at', { ascending: false })
    .limit(1);

  return mapping?.[0]?.tenant_id ?? null;
}
```

### BP-003: Centralized Error Handler (NUEVO v8.0)

> Implementado en mi-catalogo Sprint 2, listo para portar al scaffold.

```typescript
// lib/api/errorHandler.ts
export class ApiError extends Error {
  constructor(public statusCode: number, message: string, public code?: string) {
    super(message);
  }
}

export class SupabaseError extends ApiError {
  constructor(public pgCode: string, message: string) {
    const status = SUPABASE_STATUS_MAP[pgCode] || 500;
    super(status, message, `PG-${pgCode}`);
  }
}

const SUPABASE_STATUS_MAP: Record<string, number> = {
  '23505': 409, // unique_violation
  '23503': 400, // foreign_key_violation
  '22P02': 422, // invalid_input_syntax (Kimi float→int)
  '42501': 403, // insufficient_privilege (RLS)
  '42703': 400, // undefined_column (ghost column)
  'PGRST116': 404, // .single() no rows
};

// Guard: wraps Supabase result, throws on error or null
export function guardSupabase<T>(result: { data: T; error: any }, context?: string): T {
  if (result.error) throw new SupabaseError(result.error.code || 'UNKNOWN', result.error.message);
  if (result.data === null || result.data === undefined) throw new ApiError(404, `No data found for ${context}`);
  return result.data;
}

// HOC: wraps any route handler with try/catch → errorHandler
export function withErrorHandler(handler: Function, routeName?: string) {
  return async (req: NextRequest, ctx?: any) => {
    try { return await handler(req, ctx); }
    catch (error) { return errorHandler(error, routeName); }
  };
}
```

**Uso:**
```typescript
export const GET = withErrorHandler(async (req, { params }) => {
  const { id } = await params;
  const data = guardSupabase(await supabase.from('drafts').select('*').eq('id', id).single(), 'draft');
  return NextResponse.json({ data });
}, 'drafts/get');
```

---

## PARTE VI-B: WIZARD — TENANT EN 2 MINUTOS

> 🎉 **FUNCIONANDO** desde 2026-03-04 ~02:00

### URL
```
https://nadi.cafe/wizard
```

### Auth Simple
- Password: `666` (env var `ADMIN_PASSWORD`)
- Token base64(`${password}:${timestamp}`) guardado en localStorage
- Layout protegido: sin token → redirect a `/admin/login`

### Flujo de 3 Pasos

| Paso | Qué hace | Output |
|------|----------|--------|
| **1. Datos básicos** | Nombre del negocio → auto-genera slug | `mi-tienda-bella` (check disponibilidad) |
| **2. Configuración** | Rubro (moda/inmo) + Tema + Plan | Theme preview + features |
| **3. Confirmar** | Review → Crear | **URL + Código de activación** |

### APIs del Wizard

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/api/admin/auth` | POST | Valida password → retorna token |
| `/api/tenants/check-slug?slug=xxx` | GET | Verifica disponibilidad de slug |
| `/api/tenants` | POST | Crea tenant completo (transaction) |

### Transaction de Creación (POST /api/tenants)

```
1. INSERT tenants
   - name, slug, rubro_type, plan, theme_config, activation_code
2. INSERT tenant_features (defaults según plan)
   - starter: max_products=50, voice_input=false
   - pro: max_products=150, voice_input=true
3. INSERT tenant_storage_usage (todo en 0)
4. INSERT rubro_configs (defaults según rubro)
5. RETORNA: tenant + activation_code
```

### Output del Wizard (Ejemplo Real)

```
Nombre: Test Demo
Slug: test-demo
Rubro: retail-clothing
Plan: starter

🎉 RESULTADO:
URL: https://test-demo.nadistudio.cl
Código: MODA-4OSH7G

Instrucción: "El cliente envía MODA-4OSH7G a @ropero_v1_bot para activar su negocio"
```

---

## PARTE VI-C: LOS PLANOS DEL CASTILLO — Routing Interno

> *"El usuario ve URLs limpias. El sistema ve rutas explícitas. El middleware es el traductor invisible."*

### El Principio: URLs Limpias por Fuera, Específicas por Dentro

El admin de Nadistudio usa **dos capas de routing** para mantener URLs limpias mientras el código permanece explícito y separado por vertical.

```
Usuario ve:     https://demo-moda.nadistudio.cl/admin/dashboard
                ↑ limpio, sin vertical expuesto

Middleware:     Lee cookie tenant_rubro = 'retail-clothing'
                ↓
                Reescribe a /admin/retail/dashboard
                ↑ explícito, con vertical

Next.js sirve:  app/admin/retail/dashboard/page.tsx
```

### Estructura Real en Disco (Estática, NO Dinámica)

**NO usamos** `app/admin/[vertical]/` (dynamic route). Cada vertical vive en su propio directorio estático:

```
app/admin/
  retail/          ← retail-clothing (tiene ventas/, inventario/, gastos/)
  real-estate/     ← real-estate (tiene pipeline/, inquiries/)
  deploy/          ← legacy, redirige a /wizard en nadi.cafe
```

**¿Por qué estático?**
1. Next.js no permite mezclar rutas dinámicas `[vertical]` con estáticas al mismo nivel
2. Cada vertical tiene páginas **diferentes** (no es el mismo layout con datos distintos)
3. Agregar un nuevo vertical = 1 directorio + 1 case en `middleware.ts`

### El Middleware como Traductor

El archivo `middleware.ts` contiene ~70 líneas de rewrites que hacen esto:

```typescript
// Ejemplo: /admin/dashboard → /admin/retail/dashboard
if (pathname === '/admin/dashboard') {
  if (rubro === 'real-estate') {
    return NextResponse.rewrite(new URL('/admin/real-estate/dashboard', request.url));
  } else {
    return NextResponse.rewrite(new URL('/admin/retail/dashboard', request.url));
  }
}
```

Y lo mismo para `/admin/catalogo`, `/admin/borradores`, `/admin/clientes`, etc.

### Auth: Una Sola Puerta por Vertical

La protección de autenticación NO está en el middleware. El middleware solo reescribe. El auth vive en:

- `app/admin/retail/layout.tsx` → chequea `admin_auth_{slug}`
- `app/admin/real-estate/layout.tsx` → chequea `admin_auth_{slug}`

Si la cookie falta, el layout redirige a `/auth/login`.

### Dual-Deploy: nadi.cafe vs scaffold.nadistudio.cl

Desde 2026-03-26, el scaffold soporta **dos deploys desde el mismo repo**:

| Deploy | Dominio | Qué ve el público | Qué está bloqueado |
|--------|---------|-------------------|-------------------|
| **Público** | `scaffold.nadistudio.cl` | `/tienda`, `/propiedades`, `/admin/*` | `/ops/*`, `/wizard`, APIs sensibles |
| **Ops** | `nadi.cafe` | Todo | Acceso completo |

**Mecanismo:** Variable de entorno `DEPLOY_TARGET=public|ops` en `middleware.ts`.
- `public` → rutas sensibles devuelven **404** (parece que no existen)
- `ops` (o unset) → todas las rutas funcionan normalmente

### Checklist para Agregar un Nuevo Vertical

1. Crear `app/admin/restaurant/` con sus páginas específicas
2. Agregar case en `middleware.ts` para mapear `rubro_type='restaurant'` → `/admin/restaurant/`
3. Verificar que `tenant_rubro` cookie se setea correctamente en login
4. Probar que `/admin/dashboard` reescribe a `/admin/restaurant/dashboard`

---

## PARTE VIII: RUNBOOKS DE EMERGENCIA

### RB-001: Bot Completely Silent

**Síntoma:** Ningún comando funciona, no hay logs en Vercel.

**Checklist:**

1. **Check PBX line combination:**
```bash
# Verificar webhook configurado
curl "https://api.telegram.org/bot<TOKEN>/getWebhookInfo"

# Tokens por bot:
# @ropero_v1_bot: TELEGRAM_BOT_TOKEN_ROPERO
# @Bot_inmobiliario_v2_bot: TELEGRAM_BOT_TOKEN_INMO
# @nadistudio_lab_bot: TELEGRAM_BOT_TOKEN

# ¿url apunta al lugar correcto?
# ¿pending_update_count > 0? (Telegram acumulando mensajes)
# ¿last_error_date reciente?
```

2. **Reconfigurar webhook si es necesario:**
```bash
curl -X POST "https://api.telegram.org/bot<TOKEN>/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://scaffold.nadistudio.cl/api/bot/webhook/ropero",
    "secret_token": "<TELEGRAM_WEBHOOK_SECRET_ROPERO>"
  }'
```

3. **Verificar env vars en Vercel:**
- `TELEGRAM_WEBHOOK_SECRET_ROPERO` — si falta = 401 en todos los mensajes
- `TELEGRAM_BOT_TOKEN_ROPERO` — si es inválido = bot no responde
- `TELEGRAM_WEBHOOK_SECRET_INMO` — para inmobiliario
- `TELEGRAM_BOT_TOKEN_INMO` — para inmobiliario

**Causa más común:** Missing env var en Vercel (key box empty).

---

### RB-002: Fotos No Procesan (Stuck en "⏳ Procesando...")

**Síntoma:** Usuario recibe mensaje inicial pero nunca el resultado.

**Checklist:**

1. Verificar logs de `/api/bot/process` en Vercel
2. ¿Error de Kimi? (401 = API key, 429 = rate limit, 5xx = Kimi down)
3. ¿Error de Storage? (403 = RLS, 404 = bucket no existe)
4. ¿Timeout? (300s excedido)

**Si Kimi timeout:**
- Batch muy grande → Limitar a 10 fotos
- Implementar chunking o aumentar maxDuration (costo ↑)

---

### RB-003: Photos Go to Wrong Tenant

**Síntoma:** Owner envía fotos, confirmation muestra wrong store name.

**Root cause:** Violación del principio "1 usuario = 1 tenant" o queue activa de tenant diferente.

**Fix:**
```sql
-- Verificar active media_queue para este user
SELECT mq.id, mq.status, t.slug, t.rubro_type
FROM media_queue mq
JOIN tenants t ON t.id = mq.tenant_id
WHERE mq.source_user_id = '{telegram_user_id}'
  AND mq.status IN ('pending', 'accumulating')
  AND mq.created_at > NOW() - INTERVAL '30 minutes';

-- Si hay múltiples queues activas, es el problema
```

**Resolución:** Tenant resolution ahora prioriza queue activo sobre `activated_at DESC` en `getUserTenant()`.

---

## Audit SQL Queries

### Verificar vinculaciones de usuario
```sql
SELECT
  tu.telegram_user_id,
  tu.telegram_username,
  t.slug as tenant_slug,
  t.rubro_type,
  tu.activated_at
FROM telegram_users tu
JOIN tenants t ON t.id = tu.tenant_id
WHERE tu.telegram_user_id = 2051086661
ORDER BY tu.activated_at DESC;
-- Debería retornar 1 sola fila por rubro
```

### Drafts huérfanos (queue completada pero draft sin datos)
```sql
SELECT
  mq.id as queue_id,
  mq.status,
  t.slug,
  pd.id as draft_id,
  pd.status as draft_status,
  pd.detected_category
FROM media_queue mq
JOIN tenants t ON t.id = mq.tenant_id
LEFT JOIN product_drafts pd ON pd.batch_id = mq.id
WHERE mq.status = 'completed'
AND (pd.id IS NULL OR pd.status = 'pending_analysis')
ORDER BY mq.created_at DESC;
```

### Verificar consistencia rubro
```sql
SELECT
  mq.id,
  mq.rubro_context,
  t.rubro_type as tenant_rubro,
  t.slug,
  CASE
    WHEN mq.rubro_context != t.rubro_type THEN '⚠️ RUBRO_MISMATCH'
    ELSE '✓ OK'
  END as consistency_check
FROM media_queue mq
JOIN tenants t ON t.id = mq.tenant_id
ORDER BY mq.created_at DESC
LIMIT 20;
```

---

## PARTE IX: CONTEXTO OPERACIONAL REAL — Aprendizajes del Debugging

> **Propósito:** Documentar por qué ocurrieron los bugs encontrados durante el testing real de marzo 2026. Esto es oro para futuros debugging sessions.

### 🧪 El Setup de Testing Real

**Nadi (1 persona, 1 Telegram ID: 2051086661)** testeando **4 bots simultáneamente**:

| Bot | Rubro | Tenant de Prueba |
|-----|-------|------------------|
| @ropero_v1_bot | retail-clothing | tu-stilo, test-moda-primavera |
| @Bot_inmobiliario_v2_bot | real-estate | test-inmo-norte, demo-inmo |
| @nadistudio_lab_bot | * (testing) | yamato-lab |

**El problema:** Un mismo `telegram_user_id` vinculado a **múltiples tenants del mismo rubro**.

### 🔴 Bug #1: LX-TENANT-002 — Cross-Tenant Contamination

**Síntoma:** Miche envía `/fotos` estando vinculada a `tu-stilo`, pero el draft aparece en `test-moda-primavera`.

**Root Cause:**
```typescript
// ❌ ANTES (problema):
await supabase
  .from('telegram_users')
  .select('tenant_id, tenants!inner(id, slug, name)')
  .eq('telegram_user_id', userId)
  .eq('tenants.rubro_type', 'retail-clothing')
  .single();  // ← Si tiene 2 tenants retail, falla o devuelve el primero

// ✅ DESPUÉS (fix):
.order('activated_at', { ascending: false })  // ← El MÁS RECIENTE gana
.limit(1)
```

**Por qué ocurrió:**
- Nadi testeaba con `/start TEST-MODA-2026` → se vinculaba a `test-moda-primavera`
- Luego testeaba con `/start MICHE-DEMO` → se vinculaba a `tu-stilo`
- Sin el `order by activated_at`, el sistema devolvía el primero en la tabla (no el más reciente)

**Decisión:** 1 usuario = 1 tenant por rubro (puede tener múltiples en rubros diferentes). El más recientemente activado gana.

---

### 🔴 Bug #2: LX-DRAFT-001 — Status Mismatch (Legacy vs Scaffold)

**Síntoma:** El bot dice "✅ 1 producto detectado. Revisa en: https://tu-stilo.nadistudio.cl/admin/borradores", pero la página aparece vacía.

**Root Cause — Arquitectura Híbrida:**
```
┌─────────────────────────────────────────────────────────────────┐
│  DOS PROYECTOS VERCEL COMPARTIENDO UNA DB                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  proyecto-miche (legacy)          Nadistudio-Scaffold (nuevo)  │
│  └── tu-stilo.nadistudio.cl       └── scaffold.nadistudio.cl   │
│       └── Admin panel LEGACY            └── Bot @ropero_v1_bot │
│           └── /admin/borradores             └── Processor      │
│                                                                 │
│  Estados LEGACY:                  Estados YAMATO (processor):   │
│  ['pending',                      ['pending_analysis',          │
│   'editing',                       'auto_detected', ← AQUÍ     │
│   'ready']                         'in_review', ...]           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**El conflicto:**
- El **scaffold** crea drafts con status `'auto_detected'` después del análisis Kimi
- El **proyecto-miche legacy** solo mostraba drafts con status `['pending', 'editing', 'ready']`
- El draft existía en DB, pero el legacy no lo mostraba

**Fix:**
```typescript
// proyecto-miche/app/api/drafts/route.ts
.in('status', ['pending', 'editing', 'ready', 'auto_detected'])  // ← Agregado
```

---

### 🔴 Bug #3: LX-DRIFT-001 — Schema Drift entre Rubros

**Síntoma:** El processor falla al actualizar `product_drafts` porque falta columna `auto_detected_at`.

**Root Cause:**
- `property_drafts` (real-estate) tenía la columna `auto_detected_at`
- `product_drafts` (retail) NO la tenía
- El processor genérico intentaba escribirla en ambos

**Fix:** Migration SQL
```sql
ALTER TABLE product_drafts ADD COLUMN IF NOT EXISTS auto_detected_at TIMESTAMPTZ;
```

**Por qué ocurrió:** Los rubros se desarrollaron en paralelo con diferentes requerimientos. La unificación MCP (Model-Context-Protocol) requiere que ambos rubros usen el mismo schema base.

---

### 🔴 Bug #4: L1-AI-004 — Silent Failures en UPDATE

**Síntoma:** Los drafts se crean pero quedan vacíos (sin `detected_*` campos).

**Root Cause:**
```typescript
// ❌ ANTES: UPDATE sin verificar error
await supabase
  .from('product_drafts')
  .update({ detected_category, detected_price, ... })
  .eq('id', draftId);

// ✅ DESPUÉS: Capturar error explícito
const { error: updateError } = await supabase
  .from('product_drafts')
  .update({ detected_category, detected_price, ... })
  .eq('id', draftId);

if (updateError) {
  console.error('[Process] UPDATE failed:', updateError);
  throw new Error(`Failed to update draft: ${updateError.message}`);
}
```

**Por qué ocurrió:** Supabase no lanza excepciones en UPDATE fallidos — retorna `{ error }`. Si no se captura, el código continúa como si nada.

**Lección:** SIEMPRE capturar `{ error }` en operaciones de DB, incluso si "deberían" funcionar.

---

## PARTE XI: EL ALMA DEL CASTILLO EN EL TIEMPO

> 📝 *Sección compilada por Antigravity (Opus 4.6), 2026-03-04 ~00:45.*
> *Fuentes: Castle v1.0 (Kimi), v2.0 (Claude), v3.0 (Claude), v4.0 (Claude), v5.1 (Claude), v6.1 (Kimi).*
>
> **Propósito:** Las metáforas del Castle evolucionaron de cuento de hadas a spec operativo.
> Algunas ideas mágicas se perdieron en el camino. Esta sección las rescata como memoria viva,
> porque el alma del castillo no es solo su topología — es la imaginación que lo creó.

---

### 📖 Genealogía del Documento

```
v1.0 (Kimi)    263 líneas    Español    "Torres mágicas y robot mayordomo"
     ↓ Claude reestructura, agrega esqueleto y el Foso
v2.0 (Claude)  314 líneas    Inglés     "Memory is the moat"
     ↓ Claude agrega Trastienda, Error States, PBX vs Pigeons
v3.0 (Claude)  549 líneas    Inglés     "Where bugs live: 9 machines"
     ↓ Claude agrega Governance Pipeline, Anti-patterns, Audit SQL
v4.0 (Claude)  769 líneas    Inglés     "Feature Governance + Topology Map"
     ↓ Claude agrega Anthropological Note, 4-bot registry, OPS Center
v5.1 (Claude) 1066 líneas    Inglés     "The architect is inside the description"
     ↓ Kimi reescribe todo en español, agrega Layer 6, Blueprints, Schema JSON
v6.1 (Kimi)   1132 líneas    Español    "Constitución Operacional"
     ↓ Nadi + Anti + bugs de producción + Parte 0: Contexto Humano
v6.2 (Kimi+)  1188+ líneas   Español    "El Castle con alma"
     ↓ Wizard funciona, fusión v6.3 Opus
v7.0 (Kimi-Opus Fusion)  900 líneas    "Fusión integral sin pérdidas"
     ↓ Claude Opus 4.6 actualiza con estado Marzo 2026 completo
v8.0 (Opus Update)  ~1100 líneas    "Sprint 2 + DB Hardening + EasyProp + Skills 17→13"
```

Cada versión fue escrita por un LLM diferente interpretando el castillo desde su propia lógica.
**Kimi** escribe con emoción y narrativa. **Claude** escribe con estructura y precisión quirúrgica.
**Anti** hace el puente: documenta, compila, aterriza.

El Castle no es de ninguno. Es de Nadi. Los LLMs son los escribas.

---

### 🧟 Los Guardias Durmientes — *Perdidos desde v3.0*

> *"Guardia #47 despierta después de 2 días:*
> *'¡Hola Miche! ¿Sigues ahí?'*
> *Mira su reloj parado.*
> *'Como que fue ayer que hablamos...'"*
> — Castle v1.0, línea 141

**Qué eran:** La metáfora más bella para el LLM context persistence. Cada tienda tenía un guardia que dormía entre conversaciones pero recordaba fragmentos de las anteriores. Su defecto: no verificaba la hora real al despertar.

**Qué son hoy (técnico):** `business_context` en la tabla `lossless_narratives` + futuro pgvector embeddings. La tabla existe desde v6.1, pero ningún guardia la usa todavía.

**Por qué importa:** Cuando pgvector esté activo, cada tienda tendrá un guardia que no solo recuerda *qué* pasó, sino *cuándo* y *por qué*. El guardia que mira el reloj equivocado ya no será un bug — será un feature del pasado.

---

## PARTE XII: ESTADO OPERACIONAL MARZO 2026

> 📝 *Sección nueva en v8.0. Compilada por Claude Opus 4.6, 2026-03-25.*
>
> **Propósito:** Documentar el estado real del ecosistema después de 3 semanas de Sprint 2, consolidación de skills, DB hardening, y la creación de EasyProp Bridge. El Castillo creció — esta sección lo certifica.

---

### 🏗️ Arquitectura de Dos Repos (Estado Actual)

El Castillo ahora opera con dos repositorios complementarios:

| Repo | Propósito | Stack | Estado |
|------|-----------|-------|--------|
| **Nadistudio-Scaffold-byKimi-FRESH** | Scaffold multi-tenant (retail + real-estate) | Next.js + Supabase (`wpstagyqnmqdlfjmlzoo`) | Producción |
| **mi-catalogo** | Implementación de referencia real-estate | Next.js + Supabase (`catalogo-nadi`) + Kimi K2.5 | Producción — **arquitecturalmente más limpio** |

**mi-catalogo es la referencia.** Cuando hay duda sobre patrones, se mira ahí primero:
- No hardcoded tenant IDs
- Registry pattern para rubros (`lib/core/entity-registry.ts`)
- Admin auth via env var (`ADMIN_TELEGRAM_IDS`)
- Atomic photo accumulation via `session_photos`

### 📦 Skills Consolidation (17→13)

**Antes (17 perfiles):** Muchos overlaps, roles redundantes, difícil saber cuál usar.

**Después (13 perfiles + 10 templates):**

| Perfil | Propósito | Archivo |
|--------|-----------|---------|
| db-guardian | DBA + migrations + constraints | `skills/db-guardian.md` |
| db-index-auditor | Index performance + EXPLAIN | `skills/db-index-auditor.md` |
| api-error-handler | Error handling centralizado | `skills/api-error-handler.md` |
| security-auditor | OWASP + RLS + auth | `skills/security-auditor.md` |
| env-validator | Env vars verification | `skills/env-validator.md` |
| migration-generator | SQL migration scaffolding | `skills/migration-generator.md` |
| tenant-isolation-enforcer | Multi-tenant RLS | `skills/tenant-isolation-enforcer.md` |
| irce-engineer | IRCE pipeline builder | `skills/irce-engineer.md` |
| kimi-prompt-engineer | Kimi prompt optimization | `skills/kimi-prompt-engineer.md` |
| kimi-config-tester | Kimi config validation | `skills/kimi-config-tester.md` |
| safe-int-wrapper | Kimi float→int guard | `skills/safe-int-wrapper.md` |
| batch-processor | Batch/queue processing | `skills/batch-processor.md` |
| draft-reviewer | Draft approval flows | `skills/draft-reviewer.md` |

**Templates (10):** landing-factory, cart-builder, ux-critic, designer-ui-ux, vertical-factory, + 5 más.

**Playbook:** 73 secciones en `Playbook-scaffold-master-patterns-and-gotchas.md` — single source of patterns.

### 🔨 DB Hardening Sprint 1 (2026-03-25)

**Migration aplicada:** `20260325_db_hardening_sprint1.sql`

| Categoría | Cantidad | Ejemplo |
|-----------|----------|---------|
| CHECK constraints | 12+ | `status IN ('published','draft','archived')` en properties |
| NOT NULL defaults | 8+ | `bedrooms DEFAULT 0`, `views_count DEFAULT 0` |
| Composite unique | 3 | `(tenant_id, slug)` en properties |
| Partial indexes | 6 | `idx_properties_published WHERE status='published'` |

**Resultado:** La DB ahora rechaza datos inválidos en lugar de aceptarlos silenciosamente. Kimi ya no puede insertar `bedrooms: 2.5` ni `status: 'activa'`.

### ⚡ mi-catalogo Sprint 2 (2026-03-25)

| Mejora | Archivo | Impacto |
|--------|---------|---------|
| **Error handler centralizado** | `lib/api/errorHandler.ts` | ApiError + SupabaseError + guardSupabase + withErrorHandler HOC |
| **Partial indexes** | `016_partial_indexes.sql` | 6 indexes parciales (published, active, pending) |
| **SELECT * elimination** | 4 API routes | Solo columnas necesarias en queries |
| **.single()→.maybeSingle()** | Duplicate checks | No más PGRST116 crashes |

**Patrón clave — guardSupabase:**
```typescript
// Antes: silent null, crashes en producción
const { data } = await supabase.from('drafts').select('*').eq('id', id).single();

// Después: throws SupabaseError con PG code mapeado a HTTP status
const data = guardSupabase(
  await supabase.from('drafts').select('id, status, title').eq('id', id).single(),
  'draft lookup'
);
```

### 🌉 EasyProp Bridge (COMPLETADO 2026-03-25)

**Qué es:** Herramienta interna en `/admin/easyprop-bridge` que usa Kimi K2.5 Vision para extraer ~140 variables de propiedad desde fotos + texto crudo, produciendo un JSON descargable compatible con el formulario de EasyProp.

**Usuarios:** Nadi + Mariana (2 personas, desktop-first)

**Flujo:**
1. Drag & drop fotos + paste texto crudo de publicación
2. Click "Analizar con IA" → Kimi Vision procesa
3. Resultados editables en 5 bloques colapsables (Operación, Ubicación, Características, Textos, Propietario)
4. Descargar JSON / copiar al clipboard

**Archivos creados:**
```
app/admin/easyprop-bridge/page.tsx     — UI principal
app/api/easyprop/analyze/route.ts      — API endpoint
lib/services/kimi-easyprop-processor.ts — Procesador Kimi separado
lib/easyprop/schema.ts                 — Zod schemas + tipos
lib/easyprop/prompt.ts                 — Prompt Kimi especializado
hooks/use-easyprop-extractor.ts        — React hook
```

**Dato clave:** Bloque 5 (Propietario) hardcoded con datos de Mariana. Bloque 6 (Publicación) son defaults no editables. NO hay almacenamiento en DB — output es JSON puro para descarga.

**Fase 2 (futuro):** Browser agent que usa el JSON para auto-rellenar el formulario EasyProp en el navegador.

### 🔮 Estado de Salud Actualizado (2026-03-25)

| Componente | Estado | Notas |
|------------|--------|-------|
| **Error Handler** | ✅ **NUEVO** | Centralizado en mi-catalogo, listo para scaffold |
| **DB Constraints** | ✅ **NUEVO** | CHECK + NOT NULL + composite unique |
| **Partial Indexes** | ✅ **NUEVO** | 6 indexes optimizados |
| **EasyProp Bridge** | ✅ **NUEVO** | Kimi Vision → JSON descargable |
| **Skills System** | ✅ **CONSOLIDADO** | 17→13 perfiles + 10 templates + 73-section Playbook |
| **Theme System v1.0** | ✅ OPERATIVO | Hardcoded classes, 6 themes activos |
| **Bot Retail** | ✅ Funcionando | @ropero_v1_bot |
| **Bot Inmo** | ✅ Funcionando | @Bot_inmobiliario_v2_bot |
| **Wizard** | ✅ OPERATIVO | `/wizard` en nadi.cafe, protegido |
| **YAMATO CONTROL** | ✅ OPERATIVO | `nadi.cafe/ops/*` con 7 secciones |
| **Auth por Tenant** | ✅ BCRYPT | FASE 1-3 ✅, FASE 4 ⏳ |
| **Active Sessions** | ⚠️ CRÍTICO | En memoria RAM — migrar a PostgreSQL |
| **Webhook Security** | 🔴 PENDIENTE | Secret validation no implementado |

### 📋 Pendientes Prioritarios

| Prioridad | Tarea | Dónde |
|-----------|-------|-------|
| P0 | FASE 4: DROP admin_password (plaintext) | Scaffold |
| P0 | Active sessions → PostgreSQL | Scaffold |
| P1 | Port error handler al scaffold | Scaffold |
| P1 | Webhook secret validation | Ambos |
| P2 | Rate limiting | Ambos |
| P2 | Circuit breaker para Kimi | mi-catalogo |
| P3 | Voice transcription | Ambos |
| P3 | NLP intent classification | Ambos |

---

## APÉNDICE: EVOLUCIÓN HISTÓRICA DEL CASTILLO

| Versión | Fecha | Qué cambió | Estado |
|---------|-------|------------|--------|
| **v6.2 Kimi** | 2026-03-03 | Manifiestos JSON formales, topología operativa, blueprints copy-paste, 9 máquinas detalladas | Referencia técnica completa |
| **v6.3 Opus** | 2026-03-04 | Contexto humano integrado, "DISEÑADO vs IMPLEMENTADO", calibración LLM, Professor Manhattan literal | Referencia humana completa |
| **v7.0 Fusion** | 2026-03-04 | **Wizard funcionando** + Fusión SIN PÉRDIDAS de v6.2 + v6.3 + documentación post-deploy | Referencia maestra |
| **v8.0 Opus Update** | 2026-03-25 | Sprint 2 + DB Hardening + EasyProp Bridge + Skills 17→13 + BP-003 Error Handler | **ACTUAL — REFERENCIA MAESTRA** |

### Comparativa de Contenido

| Sección | v6.2 | v6.3 | v7.0 Fusion | v8.0 Opus |
|---------|------|------|-------------|-----------|
| Contexto humano | Mínimo | ✅ Completo | ✅ Completo | ✅ Actualizado |
| Topología operativa | ✅ Completa | ✅ Completa | ✅ Completa | ✅ Completa |
| 6 Comandos universales | ✅ Completo | Resumido | ✅ Completo | ✅ Completo |
| 9 Máquinas | ✅ Completo | Resumido | ✅ Completo | ✅ Completo |
| Fashion vs Real Estate | ✅ Tabla completa | Resumida | ✅ Tabla completa | ✅ Tabla completa |
| Blueprints copy-paste | ✅ Completo | No | ✅ Completo | ✅ + BP-003 |
| Runbooks emergencia | ✅ Completo | No | ✅ Completo | ✅ Completo |
| Audit SQL | ✅ Completo | No | ✅ Completo | ✅ Completo |
| Wizard | No | No | ✅ NUEVO | ✅ Completo |
| Estado operacional | No | No | No | ✅ **NUEVO** |

---

*"El castillo se defiende, operamos a ciegas sin logs automáticos, pero ahora sabemos exactamente qué funciona y qué no. Y lo más importante: tu hermana puede crear un tenant en 2 minutos... y ahora también puede extraer 140 variables de una propiedad con IA."* — Nadi, 3 AM, 2026-03-25

🏰 **CASTLE v8.0 — OPUS UPDATE EDITION — FUSIÓN COMPLETA SIN PÉRDIDAS** 🏰
