# SOUL v2.0 - Nadistudio Ecosystem

> **Plataforma multi-rubro unificada: Retail-Clothing + Real Estate**
> Última actualización: 2026-03-25 ~03:00 (Claude Opus 4.6 + Nadi) — **SPRINT 2 + DB HARDENING + EASYPROP BRIDGE**
> Responsable: Nadi + Kimi + Claude
> **Status:** ✅ **SPRINT 2 EN CURSO** | Skills 17→13 | DB Hardened | EasyProp Bridge DONE | Code Review 67/100 → mejorando

---

## ✅ ACTUALIZACIÓN 2026-03-25 — SPRINT 2 + CONSOLIDACIÓN MASIVA

> **Semana de consolidación:** Skills reducidos de 17 a 13, DB hardening Sprint 1 aplicado, error handler centralizado en mi-catalogo, EasyProp Bridge completado, Playbook en 73 secciones.

### 📦 Skills Consolidation (17→13)

**Problema:** 17 perfiles con overlaps, difícil saber cuál usar para qué tarea.

**Solución:** Consolidación a 13 perfiles especializados + 10 templates + Playbook unificado.

| Categoría | Perfiles | Propósito |
|-----------|----------|-----------|
| **DB** | db-guardian, db-index-auditor, migration-generator | Todo lo PostgreSQL |
| **API** | api-error-handler, security-auditor, env-validator | Backend quality |
| **Tenant** | tenant-isolation-enforcer | RLS + multi-tenant |
| **AI** | kimi-prompt-engineer, kimi-config-tester, safe-int-wrapper | Kimi pipeline |
| **Pipeline** | irce-engineer, batch-processor, draft-reviewer | Data flows |

**Templates (10):** landing-factory, cart-builder, ux-critic, designer-ui-ux, vertical-factory, + 5 más.

**Playbook:** `Playbook-scaffold-master-patterns-and-gotchas.md` — 73 secciones, single source of truth para patrones.

### 🔨 DB Hardening Sprint 1

**Migration:** `20260325_db_hardening_sprint1.sql` — aplicada en Supabase scaffold.

**Qué se hizo:**
- 12+ CHECK constraints (status enums, positive numbers, valid formats)
- 8+ NOT NULL con defaults sensatos (bedrooms DEFAULT 0, views_count DEFAULT 0)
- 3 composite unique constraints (tenant_id + slug)
- 6 partial indexes (published, active, pending rows)

**Impacto:** La DB ahora es estricta — Kimi no puede insertar `bedrooms: 2.5` ni `status: 'activa'`. Los datos basura se rechazan en Layer 4 antes de llegar a Layer 2.

**Informe:** `25-March-26-DB-Hardening-Sprint1-Informe.md`

### ⚡ mi-catalogo Sprint 2

Mejoras implementadas en el repo de referencia:

| Mejora | Archivo | Antes → Después |
|--------|---------|-----------------|
| Error handler centralizado | `lib/api/errorHandler.ts` | try/catch ad-hoc → ApiError + SupabaseError + guardSupabase + withErrorHandler HOC |
| Partial indexes | `016_partial_indexes.sql` | Full table scans → 6 filtered indexes |
| SELECT * elimination | 4 API routes | `SELECT *` → columnas explícitas |
| .single()→.maybeSingle() | Duplicate checks | PGRST116 crashes → graceful nulls |

**Patrón estrella — withErrorHandler:**
```typescript
export const GET = withErrorHandler(async (req, { params }) => {
  const { id } = await params;
  const data = guardSupabase(
    await supabase.from('drafts').select('id, status, title').eq('id', id).single(),
    'draft lookup'
  );
  return NextResponse.json({ data });
}, 'drafts/get');
```

### 🌉 EasyProp Bridge (COMPLETADO)

**Qué es:** Herramienta interna en `/admin/easyprop-bridge` para Nadi y Mariana. Usa Kimi K2.5 Vision para extraer ~140 variables de propiedad desde fotos + texto crudo → JSON descargable compatible con formulario EasyProp.

**Stack:** Next.js App Router + shadcn/ui + Kimi Vision API (procesador separado de kimi-processor.ts)

**Archivos:**
```
app/admin/easyprop-bridge/page.tsx       — UI (drag-drop + 5 bloques editables)
app/api/easyprop/analyze/route.ts        — API endpoint
lib/services/kimi-easyprop-processor.ts  — Procesador Kimi separado
lib/easyprop/schema.ts                   — Zod schemas
lib/easyprop/prompt.ts                   — Prompt especializado
hooks/use-easyprop-extractor.ts          — React hook
```

**5 bloques activos:** Operación, Ubicación, Características (~80 campos), Textos, Propietario (hardcoded Mariana).

**Decisiones clave:**
- NO database storage — output es JSON descargable
- Procesador Kimi SEPARADO del existente (no contamina pipeline principal)
- Bloque Propietario hardcoded con datos de Mariana Vera
- Fase 2 futura: browser agent para auto-fill

---

## ✅ ACTUALIZACIÓN 2026-03-20 ~02:30 — AUTH SEGURO + DEUDA TÉCNICA IDENTIFICADA

> **Auditoría externa completa:** Password hashing implementado (FASE 1-3), FASE 4 pendiente. Active sessions en memoria identificado como crítico. proyecto-miche tiene la solución lista para transplantar.

### 🔐 Sistema de Auth — "Doble Cerradura" Implementada

**FASE 1-3 COMPLETADAS (2026-03-15):**
```sql
-- Tabla de superadmins (Nadi + equipo core)
system_admins
├── email: nad666.m2@gmail.com
├── password_hash: bcrypt $2b$10$...
└── last_login_at: TIMESTAMPTZ

-- Tenants con hashing
ALTER TABLE tenants
ADD COLUMN admin_password_hash TEXT,  -- ✅ bcrypt
ADD COLUMN password_version INTEGER DEFAULT 1;  -- 1=legacy, 2=hashed
```

**Migración masiva:** 16 tenants migrados exitosamente. Fallback automático: si login usa plaintext, se hashea automáticamente.

**FASE 4 PENDIENTE:**
```sql
-- Ejecutar después de validación de 7 días sin errores
ALTER TABLE tenants DROP COLUMN admin_password;  -- Eliminar plaintext
```

**Tabla de auditoría:**
```sql
admin_login_attempts
├── tenant_id | email
├── ip_address | user_agent
├── success BOOLEAN
└── attempted_at TIMESTAMPTZ
```

**Decisiones de Arquitectura:**
- ✅ **MANTENER AUTH CUSTOM** (no migrar a Supabase Auth)
- ✅ Apropiado para modelo "1 tenant = 1 dueño de negocio"
- ✅ Menos vendor lock-in, más simple operativamente
- ⚠️ Cambiar a Supabase Auth SOLO SI: múltiples usuarios por tenant con RBAC complejo

### 🧠 Active Sessions — Problema Crítico Identificado

**EL PROBLEMA:**
```typescript
// lib/bot/session.ts (ACTUAL)
const sessions = new Map<number, LabSession>();  // IN-MEMORY ⚠️
```
- Cold start en Vercel = sesiones perdidas
- No escala horizontalmente
- Race conditions potenciales

**LA SOLUCIÓN (desde proyecto-miche):**
```sql
-- Tabla active_sessions (lista para transplantar)
CREATE TABLE active_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id BIGINT NOT NULL,
  chat_id BIGINT NOT NULL,
  session_type TEXT NOT NULL,  -- 'venta_pendiente', 'add_product', 'rectify_pending'
  step TEXT NOT NULL,          -- 'product_resolved', 'confirming', 'collecting_data'
  data JSONB DEFAULT '{}',     -- Estado completo
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_active_sessions_lookup ON active_sessions(user_id, tenant_id, session_type);
CREATE INDEX idx_active_sessions_expires ON active_sessions(expires_at);
```

**Tipos de sesiones validadas en proyecto-miche:**
| session_type | step | Uso |
|--------------|------|-----|
| `venta_pendiente` | `product_resolved` | Esperando selección de producto |
| `venta_pendiente` | `confirming` | Esperando confirmación sí/no |
| `add_product_pending` | `confirming` | Confirmar creación de producto |
| `rectify_pending` | `selecting` | Seleccionar operación a rectificar |
| `rectify_pending` | `awaiting_new_value` | Esperando nueva cantidad |
| `product_draft` | `collecting_data` | Acumulando datos (precio, stock, marca) |

**Estado:** Planificado para implementación. Esfuerzo estimado: 2-3 días.

### 🎯 Hallazgos de Auditoría Externa (67/100)

| Hallazgo | Severidad | Estado |
|----------|-----------|--------|
| Passwords plaintext (FASE 4 pendiente) | 🔴 Alta | ⏳ Esperando validación |
| Código duplicado (`getSlugFromHostname` 10x) | 🟡 Media | 📋 Backlog técnico |
| Active sessions en memoria | 🔴 Crítica | 📋 Planificada |
| Webhook secret no validado | 🔴 Alta | 📋 Backlog seguridad |
| Caché .next/ (703 MB) | 🟢 Baja | 🗑️ Eliminación segura |

**Referencia completa:** Ver `00.NEWS-FOR-THE-TEAM/auth-and-active-sessions-for-scaffold.md`

---

## ✅ ACTUALIZACIÓN 2026-03-06 ~00:30 — WIZARD RECUPERADO + PANEL TENANTS REAL

> **"La Torre del Mago tiene su puerta de vuelta."** — Wizard recuperado desde el olvido del commit `e935dc6f`.

### 🎯 Lo que pasó (aka: El Mapa Sin Territorio)

Durante la reorganización de estructura admin (commit `3f5982b2`), eliminamos colisiones de rutas entre `(admin-real-estate)` y `(admin-retail-clothing)`. Pero **sin querer, una viga cayó sobre la Torre del Mago**: el wizard en `/admin/deploy` desapareció.

**No era un bug.** Era **ausencia**. El portal apuntaba a una habitación que el castillo ya no tenía construida.

### 🛠️ Fixes Aplicados

| Issue | Fix | Commit |
|-------|-----|--------|
| Wizard perdido | Recuperado desde git history + nuevo layout de protección | `e110fd7e` |
| Wizard expuesto en subdominios | Layout verifica hostname — solo `scaffold.nadistudio.cl` | `e3ad5647` |
| Tenants sin password | Panel `/ops/tenants` + endpoint `POST /api/tenants/[id]/password` | `5c6bead5` |
| Root de tenants 404 | `app/page.tsx` redirige según rubro | `e110fd7e` |

### 🆕 Panel /ops/tenants — De Dummy a Real

**Antes:** Tabla con datos inventados (demo-moda, demo-inmo hardcodeados)
**Ahora:** Server Component conectado a Supabase, datos reales, acciones funcionales

**Features:**
- Stats en tiempo real: total, activos, sin contraseña
- Columna Admin Password interactiva:
  - 🔴 Sin contraseña → botón "Generar"
  - 🟢 Con contraseña → máscara `DEMO-INMO-****`
  - 👁️ Revelar/ocultar
  - 📋 Copiar al clipboard
  - 🔄 Reset con modal de confirmación
- Links directos a tenant y su admin panel

### 🏢 Centro de Operaciones — El Espacio de la Hermana

**Ubicación:** `scaffold.nadistudio.cl/admin/*`

La Hermana (pitcher, gestora, legal, corredora de RedpropertyChile) tendrá su propio dashboard:

| Ruta | Propósito | Estado |
|------|-----------|--------|
| `/admin` | Dashboard general, leads, métricas | 🔵 Planificado |
| `/admin/deploy` | Wizard crear tenants (✅ funciona) | ✅ Listo |
| `/admin/leads` | Gestión leads + campañas Meta Ads/Instagram | 🔵 Diseño |
| `/admin/legal` | Documentos, contratos, gestión legal | 🔵 Diseño |
| `/admin/rubros` | Catálogo verticales, clientes por rubro, pagos, historial | 🔵 Diseño |
| `/admin/bot` | Configuración Bot Asistencia (pgvector + compresión lossless) | 🔵 R&D |

**Nota técnica del bot:** Entrenamiento con "compresión lossless" — contexto semántico denso (metáfora: "Nadi druida oso tank Karazhan" = estrategia, orden, wipe/retry correcto). Cada rubro con su calibración: inmobiliaria (2 comandos simples) vs retail (6 comandos + variables).

### 📊 Métricas de la Sesión

| Métrica | Valor |
|---------|-------|
| **Commits** | 3 (wizard + protección + panel real) |
| **Archivos creados** | 6 |
| **Líneas de código** | ~800 |
| **Bugs resueltos** | 4 (006, 007 + 404s de tenant root) |
| **Features nuevos** | 2 (panel tenants real, protección wizard) |
| **Horas** | ~4 |

---

## ✅ ACTUALIZACIÓN 2026-03-05 ~01:00 — INFRAESTRUCTURA DE ADMINS POR RUBRO

> **AVANCE NO PREVISTO PERO CRÍTICO:** Levantamos infraestructura completa de administración por vertical. Cada tenant tiene su propio panel operativo con "llave mágica" de acceso.

### 🏗️ Filosofía: "Llave de tu Casa"

Cada cliente recibe:
- **URL única:** `tutienda.nadistudio.cl/admin`
- **Password personal:** `TUTIENDA-X7K9M2P3` (formato `SLUG-AAA1111`)
- **Cookie aislada:** `admin_auth_tutienda` (7 días)
- **Panel adaptado:** Según rubro (retail vs real-estate)

### 👕 Admin Retail — Lo Bueno de Miche

**Rutas:** `{slug}.nadistudio.cl/admin-retail/*`

| Página | Función | Joya del Legacy |
|--------|---------|-----------------|
| `/` | Login con password | Nuevo sistema auth |
| `/dashboard` | "Qué hacer hoy" + stats | Dashboard visual proyecto-miche |
| `/productos` | CRUD simplificado | Sin tallas para MVP |
| `/borradores` | Aprobación Telegram → Web | **JOYA** — Flujo Miche probado |
| `/ventas` | Registro simple | Sin caja POS compleja |
| `/configuracion` | Tema, WhatsApp, datos | Personalización completa |

### 🏘️ Admin Real Estate — Lo Bueno de mi-catalogo

**Rutas:** `{slug}.nadistudio.cl/admin-real-estate/*`

| Página | Función | Joya del Legacy |
|--------|---------|-----------------|
| `/` | Login con password | Nuevo sistema auth |
| `/dashboard` | Stats + Kanban mini | Pipeline visual mi-catalogo |
| `/propiedades` | CRUD + ubicación | Gestión completa mi-catalogo |
| `/clientes` | CRM con tipificación | **JOYA** — Base de clientes |
| `/pipeline` | Kanban completo | **JOYA** — New → Visit → Neg → Closed |
| `/configuracion` | Tema, agente, oficina | Personalización completa |

### 🔐 Sistema de Autenticación

```
1. Usuario entra a: miche.nadistudio.cl/admin
2. Middleware detecta subdominio → slug = "miche"
3. Si no hay cookie admin_auth_miche → Muestra login
4. Usuario ingresa: MICHE-X7K9M2P3
5. API valida contra tenants.admin_password
6. Setea cookie admin_auth_miche = "authenticated"
7. Redirige a dashboard con datos del tenant
```

### 🗄️ Base de Datos

**Migración aplicada:**
```sql
ALTER TABLE tenants ADD COLUMN admin_password TEXT;
```

**Generación de password:**
- Wizard crea automáticamente: `SLUG-AAA1111`
- Formato: slug en mayúsculas + 3 letras + 4 números
- Ejemplo: `DEMO-MODA-X7K9M2P3`

### 📁 Archivos Creados (Estructura Coherente v3.2)

```
app/
├── admin/                              # Routers unificados (evitan colisiones)
│   ├── page.tsx                        # Router login: detecta rubro → form
│   ├── dashboard/page.tsx              # Router dashboard: detecta rubro → dashboard
│   └── layout.tsx                      # Layout base compartido
│
├── (admin-real-estate)/                # Route group inmobiliario
│   └── admin/
│       ├── AdminLoginForm.tsx          # Form azul/dorado
│       ├── clientes/page.tsx           # CRM
│       ├── pipeline/page.tsx           # Kanban pipeline
│       └── propiedades/page.tsx        # Gestión propiedades
│
└── (admin-retail-clothing)/            # Route group retail (renombrado)
    └── admin/
        ├── AdminLoginForm.tsx          # Form morado/rosa
        ├── borradores/page.tsx         # Flujo Telegram
        ├── productos/page.tsx          # CRUD productos
        └── ventas/page.tsx             # Registro ventas
```

**Cambio importante:** `(admin-tenant)` fue renombrado a `(admin-retail-clothing)` para coherencia con `(admin-real-estate)`. Los routers en `app/admin/` evitan colisiones de rutas.

### 🎯 Commits

- `e935dc6f` — feat(admin-tenant): Sistema de admin por tenant con password simple
- `f0f135e6` — feat(admin-real-estate): Admin completo para inmobiliarias

---

## ✅ ACTUALIZACIÓN 2026-03-04 ~23:00 — YAMATO CONTROL (Esqueleto de Titanio)

> **"Quería un esqueleto, creé un cuerpo bien vestido."** — El Centro de Operaciones ya está online.

### 🏛️ Las 7 Rutas del Imperio

| Ruta | Código | Propósito | Estado |
|------|--------|-----------|--------|
| `/ops` | **COMMAND** | War Room - Overview del sistema | ✅ Deployed |
| `/ops/tenants` | **DOMAINS** | Gestión de tenants (tabla densa) | ✅ Deployed |
| `/ops/pipeline` | **FORGE** | Flujo de trabajo fotos → publicaciones | ✅ Deployed |
| `/ops/system` | **ENGINE** | Health de bots, DB, storage | ✅ Deployed |
| `/ops/backroom` | **ARSENAL** | Recursos, APIs, verticales potenciales | ✅ Deployed |
| `/ops/alerts` | **SIGNALS** | Centro de notificaciones priorizadas | ✅ Deployed |
| `/ops/ledger` | **TREASURY** | MRR, costos, proyecciones | ✅ Deployed |

### 🎨 Diseño Imperial

- **Tema:** Dark mode slate-950 + amber-400 (del Imperial Dashboard legacy)
- **Layout:** Sidebar 256px fijo + main content
- **Densidad:** Estilo Bloomberg (más info, menos scroll)
- **Auth:** Cookie-based con `?key=666` (mismo que deployer)

### 📁 Archivos Creados

```
app/ops/
├── layout.tsx          # Layout Imperial + auth
├── page.tsx            # COMMAND - Stat cards + activity
├── tenants/page.tsx    # DOMAINS - Tabla con filtros
├── pipeline/page.tsx   # FORGE - Kanban workflow
├── system/page.tsx     # ENGINE - Bot health cards
├── backroom/page.tsx   # ARSENAL - APIs + verticales
├── alerts/page.tsx     # SIGNALS - Alertas priorizadas
└── ledger/page.tsx     # TREASURY - Métricas financieras

app/components/ops/
├── Sidebar.tsx         # Navegación 7 rutas
├── StatCard.tsx        # Cards de métricas
└── StatusBadge.tsx     # Indicadores 🟢🟡🔴
```

### 🔐 Acceso

```
https://nadistudio.cl/ops?key=666    → Setea cookie + redirige
https://nadistudio.cl/ops            → Dashboard (con cookie)
```

---

## ✅ ACTUALIZACIÓN 2026-03-04 ~21:30 — PRIMER TRAJE FUNCIONAL + FICHAS INDIVIDUALES

> **Mensaje para nosotros del futuro:** El Robot Sastre finalmente sabe coser. Las páginas tienen colores, las fichas funcionan, y el sistema está listo para que Miche comparta sus productos.

### 🎨 Logros de la Jornada (Maratón de Consolidación)

| Hora | Logro | Estado |
|------|-------|--------|
| 17:00 | **Theme System v1.0** implementado | ✅ Tailwind safelist + hardcoded classes |
| 18:00 | **Schema Drift Fix** completo | ✅ rubro_configs creada, themes asignados |
| 19:00 | **Home Redirect** `/` | ✅ Detecta rubro → /tienda o /propiedades |
| 20:00 | **Fichas Individuales** `[slug]` | ✅ /tienda/[slug] + /propiedades/[slug] |
| 21:00 | **Build estable** | ✅ TypeScript feliz, deploy funcionando |

### 🎯 Theme System v1.0 — "Primer Traje Funcional"

**Arquitectura:** Hardcoded Tailwind classes (no CSS variables dinámicas)

```typescript
// lib/themes/theme-classes.ts
const THEME_PAGE_CLASSES = {
  'fashion-vibrant':   'bg-rose-50 text-rose-900',
  'fashion-boho':      'bg-amber-50 text-amber-900',
  'fashion-minimal':   'bg-white text-slate-900',
  'real-estate-modern':'bg-blue-50 text-blue-900',
  'real-estate-luxury':'bg-slate-900 text-amber-100',
};
```

**Uso en páginas:**
```typescript
const themeId = tenant.theme_config?.theme_id;
const pageClasses = getPageThemeClasses(themeId);  // 'bg-rose-50 text-rose-900'
return <div className={'min-h-screen ' + pageClasses}>...</div>;
```

**Tenants con tema activo:**
| Tenant | Theme | Color |
|--------|-------|-------|
| 123 | fashion-vibrant | 🌸 Rosa |
| try1 | fashion-boho | 🟠 Ámbar |
| ggr | real-estate-modern | 🔵 Azul |
| demo-moda | fashion-minimal | ⚪ Blanco |
| demo-inmo | real-estate-modern | 🔵 Azul |
| redpropertycl | real-estate-luxury | 🖤 Dark |

### 🏠 Fichas Individuales — "El Showroom"

**Nuevas rutas creadas:**

```
app/
├── tienda/
│   ├── page.tsx              # Catálogo (ya existía)
│   └── [slug]/
│       └── page.tsx          # 🆕 Ficha producto individual
├── propiedades/
│   ├── page.tsx              # Listado (ya existía)
│   └── [slug]/
│       └── page.tsx          # 🆕 Ficha propiedad individual
```

**Componentes creados:**

| Componente | Props | Función |
|------------|-------|---------|
| `ImageGallery` | images[], alt, themeClasses | Hero image + thumbnails (no zoom) |
| `CTABar` | whatsappUrl, itemName, themeColor | WhatsApp sticky (mobile), Save, Share |

**Diseño mobile-first:**
- Sticky CTA bar en mobile (thumb-reachable)
- Precio visible inmediatamente (above the fold)
- WhatsApp CTA primario (un tap para contactar)
- Save en localStorage (corazón)
- Share con Web Share API

**Flujo de Miche:**
1. Sube fotos por Telegram
2. Aprueba en `/admin/borradores`
3. Sistema genera slug automáticamente
4. Miche comparte: `tu-stilo.nadistudio.cl/tienda/jean-vintage-azul`
5. Cliente ve ficha bonita → tap en WhatsApp → conversación

### 📊 Schema Final (Después de Drift Fix)

**Columnas corregidas:**
```typescript
// REAL ESTATE
properties.operation_type   // NOT operation (renamed 2026-03-03)
properties.commune          // NOT comuna
properties.images           // English

// RETAIL
products.images             // NOT imagenes (renamed 2026-03-03)

// DB tiene
rubro_configs               // Creada 2026-03-04
```

**Themes en DB:**
```sql
-- Todos los tenants tienen theme_config
SELECT slug, theme_config->>'theme_id'
FROM tenants
WHERE theme_config IS NOT NULL;
-- 6 tenants activos con tema
```

---

## ✅ ACTUALIZACIÓN 2026-03-03 ~17:15 — SCHEMA DRIFT PURGE (Holy Pally Completo)

> **PURGA DE SCHEMA DRIFT:** Zanjamos las incongruencias de columnas entre DB y código. Todo en inglés ahora.

### 🧹 Migraciones Aplicadas

```sql
-- properties: operation → operation_type (English)
ALTER TABLE properties RENAME COLUMN operation TO operation_type;

-- products: imagenes → images (English)
ALTER TABLE products RENAME COLUMN imagenes TO images;
ALTER TABLE products DROP COLUMN IF EXISTS imagenes; -- eliminar duplicado
```

### 📝 Código Fixeado

| Archivo | Cambio |
|---------|--------|
| `lib/schemas/index.ts` | `PropertyFilters.commune` (no `comuna`), `Product.images` (no `imagenes`) |
| `lib/supabase/client.ts` | Query usa `operation_type`, `commune` |
| `lib/bot/search-properties.ts` | `SearchFilters.operation_type` (no `operation`) |
| `app/propiedades/page.tsx` | Filtros usan `commune` |
| `components/ui/FilterBar.tsx` | `availableOptions.communes` (no `comunas`) |
| `components/ui/ProductCard.tsx` | `product.images` (no `imagenes`) |

### 🛡️ Holy Pally Trinity Completa

| Script | Comando | Propósito |
|--------|---------|-----------|
| **Aura of Devotion** | `npm run buff:quick` | Check rápido (5s) — ¿puedo codear? |
| **Blessing of Kings** | `npm run buff` | Check completo — estado del castillo |
| **Ready Check** | `npm run ready-check` | Pre-deploy validation |
| **Castle Menu** | `npm run castle` | Menú RPG interactivo |

---

## 🏰 ARQUITECTURA ACTUAL (Marzo 2026)

### Sistema de Themes v1.0

```
tenants.theme_config (JSONB)
├── theme_id: "fashion-vibrant" | "fashion-boho" | "real-estate-modern" | ...
├── name: "Fashion Vibrant"
└── colors: { primary, accent, background }

lib/themes/theme-classes.ts (HARDCODED)
├── THEME_PAGE_CLASSES: Record<theme_id, tailwind_string>
├── THEME_CARD_CLASSES: Record<theme_id, {base, hover}>
├── getPageThemeClasses(themeId): string
└── getCardThemeClasses(themeId): {base, hover}

Aplicación en páginas:
├── app/page.tsx              → redirect por rubro
├── app/tienda/page.tsx       → theme en catálogo
├── app/tienda/[slug]/page.tsx → theme en ficha
├── app/propiedades/page.tsx  → theme en listado
└── app/propiedades/[slug]/page.tsx → theme en ficha
```

### Páginas Públicas (Tenant-facing)

| Ruta | Rubro | Propósito | Theme |
|------|-------|----------|-------|
| `/` | Both | Redirect a /tienda o /propiedades | ✅ Heredado |
| `/tienda` | retail | Catálogo con filtros | ✅ Aplicado |
| `/tienda/[slug]` | retail | Ficha producto individual | ✅ Aplicado |
| `/propiedades` | real-estate | Listado con filtros | ✅ Aplicado |
| `/propiedades/[slug]` | real-estate | Ficha propiedad individual | ✅ Aplicado |

### Flujo Completo Validado

```
Usuario final (cliente de Miche):
1. Recibe link: 123.nadistudio.cl/tienda/jean-azul
2. Ve página con fondo rosa (theme fashion-vibrant)
3. Ve imagen grande del jean + precio
4. Tap en "Consultar por WhatsApp"
5. Abre WhatsApp con mensaje pre-llenado
6. Conversación directa con Miche

Miche (tenant admin):
1. Manda fotos a @ropero_v1_bot
2. Recibe notificación de borradores listos
3. Va a /admin/borradores
4. Aprueba → genera slug automático
5. Copia link y comparte en IG/WhatsApp
```

---

## 🏥 Estado de Salud (Actualizado: 2026-03-25 ~03:00)

| Componente | Estado | Notas |
|------------|--------|-------|
| **Error Handler** | ✅ **NUEVO** | Centralizado en mi-catalogo (ApiError + guardSupabase + withErrorHandler) |
| **DB Constraints** | ✅ **NUEVO** | CHECK + NOT NULL + composite unique (Sprint 1) |
| **Partial Indexes** | ✅ **NUEVO** | 6 filtered indexes en ambos repos |
| **EasyProp Bridge** | ✅ **NUEVO** | /admin/easyprop-bridge — Kimi Vision → JSON |
| **Skills System** | ✅ **CONSOLIDADO** | 17→13 perfiles + 10 templates + Playbook 73 secciones |
| **Theme System v1.0** | ✅ **OPERATIVO** | Hardcoded classes, 6 themes activos |
| **Home Redirect** | ✅ **FUNCIONANDO** | / → /tienda o /propiedades por rubro |
| **Fichas [slug]** | ✅ **DEPLOYADAS** | /tienda/[slug] + /propiedades/[slug] |
| **Schema** | ✅ **ESTABLE** | rubro_configs creada, columnas corregidas |
| **Build** | ✅ **PASANDO** | TypeScript sin errores |
| **ImageGallery** | ✅ **NUEVO** | Hero + thumbnails, theme-aware |
| **CTABar** | ✅ **NUEVO** | WhatsApp sticky, Save, Share |
| **Bot Retail** | ✅ Funcionando | @ropero_v1_bot |
| **Bot Inmo** | ✅ Funcionando | @Bot_inmobiliario_v2_bot |
| **Wizard** | ✅ **RECUPERADO** | /admin/deploy protegido, solo scaffold |
| **YAMATO CONTROL** | ✅ **OPERATIVO** | /ops/* dashboard con 7 secciones |
| **/ops/tenants** | ✅ **DATOS REALES** | Gestión de passwords, stats, links directos |
| **Admin Retail** | ✅ **ESQUELETO** | /admin-retail/* 5 páginas operativas |
| **Admin Real Estate** | ✅ **ESQUELETO** | /admin-real-estate/* 5 páginas operativas |
| **Auth por Tenant** | ✅ **BCRYPT IMPLEMENTADO** | FASE 1-3 ✅, FASE 4 ⏳ (drop columna plaintext) |
| **System Admin** | ✅ **OPERATIVO** | Login con bcrypt para Nadi + equipo core |
| **Active Sessions** | ⚠️ **CRÍTICO** | En memoria RAM — migrar a PostgreSQL (planificado) |
| **Holy Pally** | ⚠️ **DUEÑA TÉCNICA** | Código duplicado detectado (10 copias función slug) |
| **Webhook Security** | 🔴 **PENDIENTE** | Secret validation no implementado |
| **Centro Hermana** | 🔵 **DISEÑO** | scaffold.nadistudio.cl/admin/* planificado |
| **Sistema Imágenes** | ⚠️ **DUAL** | Legacy + entity_images coexisten (FASE 4 futura) |
| Vercel Logs | 🔴 PAUSADO | Log Drains desactivados |

---

## 🎯 ROADMAP ACTUALIZADO — Post Sprint 2

### 🔥 Prioridad P0 (Próxima semana)
| Tarea | Impacto | Repo |
|-------|---------|------|
| FASE 4: DROP admin_password | Eliminar riesgo plaintext | Scaffold |
| Active sessions → PostgreSQL | Persistencia cross-deploy | Scaffold |
| Webhook secret validation | Seguridad bot | Ambos |

### 🔥 Prioridad P1 (Sprint 3)
| Tarea | Impacto | Repo |
|-------|---------|------|
| Port error handler al scaffold | Consistencia errores | Scaffold |
| Port partial indexes al scaffold | Performance queries | Scaffold |
| Rate limiting per-user | Anti-abuso | Ambos |
| Repository pattern expansion | DRY data access | mi-catalogo |

### Prioridad P2 (Este trimestre)
| Tarea | Impacto | Repo |
|-------|---------|------|
| Circuit breaker para Kimi | Resiliencia API China | mi-catalogo |
| Voice transcription | UX bot | Ambos |
| NLP intent classification | Lenguaje natural | Ambos |
| UI/UX polish pass | Calidad visual | Ambos |
| useQuery custom hook | Data fetching estándar | mi-catalogo |
| Retry con exponential backoff | Kimi stability | mi-catalogo |

### Futuro
| Feature | Esfuerzo | Valor |
|---------|----------|-------|
| EasyProp Bridge Fase 2 (browser agent) | 1 semana | Auto-fill formulario |
| pgvector memory (Guardias Durmientes) | 2 semanas | Contexto persistente |
| Cursor pagination | 2 días | Large datasets |
| Virtualization (tanstack-virtual) | 1 día | Large product lists |
| Seed data con Faker | 1 día | Dev/test consistency |

---

## 🎊 MÉTRICAS DE ÉXITO ACUMULADAS

### Marzo 3-6, 2026 (Scaffold)
| Antes | Después |
|-------|---------|
| Páginas sin color (bg-gray-50) | ✅ Themes aplicados por tenant |
| Schema drift (tablas faltantes) | ✅ rubro_configs creada |
| Productos sin URL única | ✅ Fichas [slug] funcionando |
| Build roto (TypeScript errors) | ✅ Build pasando |
| Cliente no puede compartir links directos | ✅ /tienda/[slug] operativo |
| **Sin admin por tenant** | ✅ **Infraestructura completa: Retail + Real Estate** |
| **Sin auth específica** | ✅ **Password por tenant: `SLUG-AAA1111`** |
| **Sin pipeline de ventas** | ✅ **Kanban CRM para inmobiliarias** |

### Marzo 15-20, 2026 (Auth)
| Antes | Después |
|-------|---------|
| Passwords plaintext en DB | ✅ bcrypt FASE 1-3 implementado |
| Sin auditoría de login | ✅ admin_login_attempts table |
| Sin system_admins | ✅ Tabla con bcrypt + last_login |

### Marzo 23-25, 2026 (Sprint 2 + Consolidación)
| Antes | Después |
|-------|---------|
| 17 skills con overlaps | ✅ 13 perfiles + 10 templates + 73-section Playbook |
| DB acepta datos basura | ✅ CHECK constraints + NOT NULL defaults |
| try/catch ad-hoc en APIs | ✅ Error handler centralizado (mi-catalogo) |
| SELECT * en queries | ✅ Columnas explícitas en 4 routes |
| .single() en checks opcionales | ✅ .maybeSingle() en duplicate checks |
| 20min manual para llenar EasyProp | ✅ EasyProp Bridge — Kimi Vision → JSON en 30s |
| Sin indexes parciales | ✅ 6 partial indexes (published, active, pending) |

---

## 📚 Documentación Relacionada

| Documento | Ubicación | Propósito |
|-----------|-----------|-----------|
| **Castle v8.0** | `.final-state-Scaffold/nadistudio-castle-v8.0-opus-update.md` | Constitución operacional completa |
| **Auth & Active Sessions** | `00.NEWS-FOR-THE-TEAM/auth-and-active-sessions-for-scaffold.md` | Roadmap técnico detallado |
| **Code Review Externo** | `00.NEWS-FOR-THE-TEAM/Project-external-review-comments-and-suggestions.md` | Auditoría completa 67/100 |
| **Password Hash P0-001** | `00.NEWS-FOR-THE-TEAM/2026-03-15-P0-001-PASSWORD-HASH-IMPLEMENTATION.md` | Implementación FASE 1-3 |
| **DB Hardening Informe** | `.final-state-Scaffold/25-March-26-DB-Hardening-Sprint1-Informe.md` | Sprint 1 constraints |
| **Unified DB State** | `.final-state-Scaffold/final-state-unified-db.md` | Estado actual de todas las tablas |
| **Playbook** | `.final-state-Scaffold/Playbook-scaffold-master-patterns-and-gotchas.md` | 73 secciones de patrones |
| **AGENTS.md** | `.final-state-Scaffold/AGENTS.md` | Perfiles de skills/agents |
| **Analogías v2.0** | `ERRORS/ANALOGIES.md` | "Portero sin Validación", "Pizarrón Borrable" |

---

*Documento v1.0 creado por Nadi + Kimi Coder (Mar 3-20, 2026). v2.0 actualizado por Claude Opus 4.6 (Mar 25, 2026). Para actualizaciones, buscar "ACTUALIZACIÓN" en el documento.*

**"El Castillo tiene 2 repos, 13 skills, 140 variables extraídas por IA, y una hermana que puede crear tenants y extraer propiedades sin tocar una línea de código."** 🏰🔑✨
