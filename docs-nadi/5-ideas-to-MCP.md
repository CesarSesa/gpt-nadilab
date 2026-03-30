# 5 Ideas de MCP para el Nadistudio Scaffold

> **Documento de arquitectura de herramientas** — Propuestas de Model Context Protocol para automatización y operación del castillo.  
> **Fecha:** 2026-03-26  
> **Contexto:** Stack Yamato (Next.js + Supabase + Vercel), MCP de Supabase ya operativo  
> **Nota:** Incluye ajuste post-corrección — separación clara entre debugging técnico (features/verticales) y gestión de clientes (tenants como negocio).

---

## 🎯 MCP #1: Yamato-Ops — Diagnóstico General por Features & Verticales

**Tipo:** MCP de Observabilidad Técnica  
**Prioridad:** Alta  
**Dependencia de Supabase MCP:** Usa el MCP existente para queries directas

### Propósito
Debuggear el scaffold por dimensiones técnicas (features, verticales, infraestructura), NO por tenant individual. Te da la vista del arquitecto, no del soporte al cliente.

### Tools Propuestas

#### `check_feature_health(feature_name)`
Diagnóstico de una feature completa (ej: "foto-pipeline", "irce", "draft-workflow").
```typescript
// Ejemplo: check_feature_health("m1-m9-pipeline")
{
  "feature": "m1-m9-pipeline",
  "status": "degraded",
  "machines": {
    "M1-M4": "healthy",
    "M5": "warning", // 3 timeouts en última hora
    "M6": "healthy",
    "M7-M9": "healthy"
  },
  "recommendation": "Revisar M5 (Tubo Neumático) — posible problema con after() en Vercel"
}
```

#### `check_vertical_status(rubro_type)`
Estado de un vertical completo (fashion, real-estate, etc.).
```typescript
// Ejemplo: check_vertical_status("real-estate")
{
  "rubro": "real-estate",
  "active_tenants": 5,
  "queues_processing": 12,
  "queues_stuck": 1,
  "avg_processing_time": "45s",
  "error_rate": "2%",
  "alerts": ["1 tenant sin webhooks configurados"]
}
```

#### `get_system_overview()`
Dashboard operacional del castillo.
```typescript
{
  "layers": {
    "L0": "healthy", // Governance
    "L1": "healthy", // Communication  
    "L2": "warning", // Tenant Ops — 2 tenants con drafts stuck
    "L3": "healthy", // Protection
    "L4": "healthy", // Data
    "L5": "standby"  // Lab — no experiments running
  },
  "critical_alerts": 1,
  "warnings": 3,
  "recommendations": [
    "Revisar L2: property_drafts con status 'processing' > 30min"
  ]
}
```

#### `simulate_load(tenant_id, scenario)`
Simula carga para testear escalabilidad.
```typescript
// scenario: "photo_burst" | "concurrent_sales" | "nlp_spam"
{
  "scenario": "photo_burst",
  "requests": 50,
  "success_rate": "96%",
  "avg_latency": "1.2s",
  "bottleneck": "M6 (Kimi API rate limit)"
}
```

### Casos de Uso
- "¿Cómo está el vertical de real estate en general?" → `check_vertical_status`
- "¿El pipeline de fotos está sano?" → `check_feature_health("m1-m9-pipeline")`
- "¿Qué tan saludable está el castillo ahora?" → `get_system_overview`

### Implementación Técnica
```javascript
// Servidor: mcp-yamato-ops/server.ts
// Conexión: Reusa Supabase MCP para queries
// Lógica: Agrega análisis y correlación entre tablas

const tools = [
  {
    name: "check_feature_health",
    handler: async ({ feature_name }) => {
      // Query via Supabase MCP
      const queues = await supabaseMcp.query("media_queue", {...});
      const drafts = await supabaseMcp.query("product_drafts", {...});
      
      // Análisis de correlación
      return analyzeHealth(queues, drafts, feature_name);
    }
  }
];
```

---

## 🎯 MCP #2: Schema-Guardian — Protocolo de Validación de Esquema

**Tipo:** MCP de Calidad de Datos / Guardián de Integridad  
**Prioridad:** Alta  
**Dependencia de Supabase MCP:** Cliente pesado — extiende el MCP con validaciones complejas

### Propósito
Prevenir el "schema drift" que ya te mordió (ghost columns, status constraints, columnas renombradas). Funciona como CI/CD para tu base de datos.

### Tools Propuestas

#### `validate_schema_compatibility(table_name)`
Compara código vs. DB real.
```typescript
// Ejemplo: validate_schema_compatibility("product_drafts")
{
  "table": "product_drafts",
  "in_code": ["detected_name", "final_price", "status"],
  "in_database": ["detected_name", "detected_price", "status", "final_price"],
  "ghost_columns_in_code": [], // ✅ Ninguno
  "missing_in_code": ["detected_price"], // ⚠️ Usas detected_price en DB pero no en schema TS
  "risk_level": "medium",
  "fix_suggestion": "Agregar detected_price a lib/schemas/index.ts"
}
```

#### `check_constraint_consistency(table_name)`
Valida que los CHECK constraints incluyan todos los valores usados en código.
```typescript
// Ejemplo: check_constraint_consistency("product_drafts")
{
  "table": "product_drafts",
  "constraint": "product_drafts_status_check",
  "allowed_in_db": ["pending", "auto_detected", "batch_reviewed", "published"],
  "used_in_code": ["pending", "auto_detected", "batch_reviewed", "ready_to_publish", "published"],
  "missing_in_constraint": ["ready_to_publish"], // 🔴 Error crítico
  "fix_sql": "ALTER TABLE product_drafts DROP CONSTRAINT... ADD CONSTRAINT..."
}
```

#### `detect_rls_gaps()`
Encuentra tablas sin RLS o con políticas débiles.
```typescript
{
  "tables_without_rls": ["old_logs"],
  "tables_without_tenant_isolation": [],
  "vulnerable_tables": [],
  "compliance_score": "95%"
}
```

#### `generate_migration_plan(target_state)`
Genera SQL de migración seguro.
```typescript
// target_state: "add_variant_grouping" o ruta a schema objetivo
{
  "steps": [
    { "order": 1, "sql": "ALTER TABLE products ADD COLUMN group_key UUID;", "risk": "low" },
    { "order": 2, "sql": "CREATE INDEX idx_products_group ON products(group_key);", "risk": "medium" }
  ],
  "rollback_plan": "Migration rollback SQL here",
  "estimated_downtime": "0s (online migration)"
}
```

### Casos de Uso
- Pre-commit hook: "¿Este código rompe el schema?" → `validate_schema_compatibility`
- Pre-deploy: "¿Los constraints están al día?" → `check_constraint_consistency`
- Seguridad: "¿Todas las tablas tienen RLS?" → `detect_rls_gaps`
- Evolución: "Cómo migro de v6.3 a v7.0" → `generate_migration_plan`

### Implementación Técnica
```javascript
// Necesita acceso a:
// 1. information_schema (vía Supabase MCP)
// 2. Código fuente (parsear lib/schemas/, app/api/)
// 3. Historial de migraciones (list_migrations vía Supabase MCP)

const tools = [
  {
    name: "validate_schema_compatibility",
    handler: async ({ table_name }) => {
      // 1. Leer schema real de DB
      const dbSchema = await supabaseMcp.query(
        "information_schema.columns", 
        { table_name }
      );
      
      // 2. Parsear código TypeScript
      const codeSchema = parseTsInterfaces(`lib/schemas/${table_name}.ts`);
      
      // 3. Comparar
      return diffSchemas(codeSchema, dbSchema);
    }
  }
];
```

---

## 🎯 MCP #3: Telegram-Simulator — Entorno de Pruebas Conversacionales

**Tipo:** MCP de Testing / QA Automation  
**Prioridad:** Media  
**Dependencia de Supabase MCP:** Ligera — usa para verificar estado post-simulación

### Propósito
Testear el flujo M1-M9 sin depender de Telegram ni de Miche. Un "target dummy" para tus bots.

### Tools Propuestas

#### `simulate_conversation(tenant_slug, persona, scenario)`
Simula una conversación completa.
```typescript
// Ejemplo: simulate_conversation("demo-moda", "miche", "batch_upload")
{
  "tenant": "demo-moda",
  "persona": "miche", // Carga comportamiento típico de Miche
  "scenario": "batch_upload", // Script predefinido
  "steps": [
    { "actor": "user", "action": "/fotos Lote de verano", "timestamp": "T+0s" },
    { "actor": "bot", "response": "Modo fotos activado", "timestamp": "T+0.5s" },
    { "actor": "user", "action": "upload_photo(6_pants.jpg)", "timestamp": "T+5s" },
    { "actor": "user", "action": "/listo", "timestamp": "T+30s" },
    { "actor": "bot", "response": "⏳ Procesando...", "timestamp": "T+31s" },
    { "actor": "system", "event": "M5 triggered", "timestamp": "T+31.5s" },
    { "actor": "system", "event": "M6 completed (3 items detected)", "timestamp": "T+45s" },
    { "actor": "bot", "response": "✅ 3 productos detectados...", "timestamp": "T+46s" }
  ],
  "result": "success",
  "drafts_created": 3,
  "errors": []
}
```

#### `inject_webhook_event(bot_type, payload)`
Envía evento directo al webhook.
```typescript
// Simula update de Telegram
{
  "bot_type": "ropero",
  "payload": {
    "update_id": 123456789,
    "message": {
      "message_id": 1,
      "from": { "id": 2051086661, "first_name": "Nadi" },
      "chat": { "id": 2051086661, "type": "private" },
      "date": 1711459200,
      "text": "/fotos"
    }
  },
  "result": {
    "status_code": 200,
    "response_time": "120ms",
    "queue_created": true
  }
}
```

#### `replay_session(session_id, speed)`
Reproduce una sesión real para debug.
```typescript
// Obtiene logs de session_id y reproduce paso a paso
{
  "session_id": "sess_abc123",
  "original_tenant": "tu-stilo",
  "original_date": "2026-03-20",
  "replay_steps": [...],
  "deviation_from_original": null // o diff si hay diferencias
}
```

#### `test_intent_classification(rubro, test_phrases)`
Valida el IRCE con frases de prueba.
```typescript
// test_phrases: ["vendí 2 jeans", "llegaron poleras", "cuántos quedan?"]
{
  "rubro": "retail-clothing",
  "results": [
    { "phrase": "vendí 2 jeans", "detected_intent": "SELL", "confidence": 0.94, "expected": "SELL", "match": true },
    { "phrase": "llegaron poleras", "detected_intent": "ADD_STOCK", "confidence": 0.89, "expected": "ADD_STOCK", "match": true }
  ],
  "accuracy": "98%"
}
```

### Casos de Uso
- "Quiero testear el flujo de fotos sin molestar a Miche" → `simulate_conversation`
- "Reproduce lo que pasó ayer con demo-moda" → `replay_session`
- "¿El bot detecta bien 'vendí' vs 'vendi'?" → `test_intent_classification`

### Implementación Técnica
```javascript
// Servidor local que expone endpoints de test
// No requiere credenciales de Telegram (modo sandbox)

const tools = [
  {
    name: "simulate_conversation",
    handler: async ({ tenant_slug, scenario }) => {
      // 1. Cargar scenario (JSON con pasos)
      const steps = loadScenario(scenario);
      
      // 2. Ejecutar cada paso contra webhook local
      const results = [];
      for (const step of steps) {
        const result = await fetch(`http://localhost:3000/api/bot/webhook/ropero`, {
          method: 'POST',
          body: JSON.stringify(step.payload)
        });
        results.push(await result.json());
      }
      
      // 3. Verificar estado en DB vía Supabase MCP
      const finalState = await supabaseMcp.query("media_queue", { tenant_slug });
      
      return { steps: results, final_state: finalState };
    }
  }
];
```

---

## 🎯 MCP #4: Image-Pipeline-Inspector — Debugger Visual de las 9 Máquinas

**Tipo:** MCP de Debugging Especializado / Observabilidad de Media  
**Prioridad:** Media-Alta  
**Dependencia de Supabase MCP:** Alta — necesita unir datos de DB + Storage

### Propósito
Visibilidad quirúrgica del flujo de fotos (M1-M9). Cuando algo falla, saber EXACTAMENTE en qué máquina se atascó.

### Tools Propuestas

#### `trace_photo(file_id)`
Rastrea una foto específica por todo el pipeline.
```typescript
// Ejemplo: trace_photo("AgACAgEAAxkBAAI9aWf...")
{
  "file_id": "AgACAgEAAxkBAAI9aWf...",
  "journey": [
    { "machine": "M1", "status": "completed", "timestamp": "2026-03-26T12:00:00Z", "data": { "caption": "Lote de verano" } },
    { "machine": "M2", "status": "completed", "timestamp": "2026-03-26T12:00:01Z", "data": { "queue_id": "q_abc123", "tenant": "demo-moda" } },
    { "machine": "M3", "status": "completed", "timestamp": "2026-03-26T12:00:05Z", "data": { "sort_order": 1, "file_size": 2048000 } },
    { "machine": "M4", "status": "completed", "timestamp": "2026-03-26T12:00:30Z", "data": { "trigger": "/listo" } },
    { "machine": "M5", "status": "completed", "timestamp": "2026-03-26T12:00:31Z", "data": { "process_invoked": true } },
    { "machine": "M6", "status": "failed", "timestamp": "2026-03-26T12:01:45Z", "error": "Kimi API timeout", "retry_count": 1 },
    { "machine": "M7-M9", "status": "blocked", "reason": "M6 failed" }
  ],
  "current_location": "M6 (failed)",
  "recommended_action": "Reintentar M6 o verificar KIMI_API_KEY"
}
```

#### `diagnose_machine(machine_name)`
Estado de salud de una máquina específica.
```typescript
// Ejemplo: diagnose_machine("M6")
{
  "machine": "M6 (Taller del Alquimista)",
  "status": "degraded",
  "metrics": {
    "avg_response_time": "45s",
    "error_rate": "5%",
    "timeout_rate": "3%",
    "queue_depth": 2 // Cuántos batches esperando
  },
  "recent_errors": [
    { "time": "2026-03-26T11:55:00Z", "error": "429 Rate Limit", "tenant": "demo-inmo" }
  ],
  "recommendation": "Considerar implementar circuit breaker"
}
```

#### `find_orphan_media()`
Detecta fotos perdidas (en Storage pero no en DB, o viceversa).
```typescript
{
  "orphan_in_storage": [
    { "path": "demo-moda/temp/drafts/q_old123/1.jpg", "size": "2MB", "age": "7d" }
  ],
  "orphan_in_db": [
    { "queue_id": "q_new456", "file_id": "xyz", "missing_in_storage": true }
  ],
  "wasted_storage_mb": 45,
  "cleanup_sql": "DELETE FROM media_queue_photos WHERE queue_id IN (...)"
}
```

#### `force_machine_retry(queue_id, machine)`
Reintenta una máquina específica para un batch.
```typescript
// Ejemplo: force_machine_retry("q_abc123", "M6")
{
  "queue_id": "q_abc123",
  "machine": "M6",
  "action": "reprocessing",
  "result": "success",
  "drafts_created": 3,
  "processing_time": "32s"
}
```

### Casos de Uso
- "¿Dónde se quedó esta foto?" → `trace_photo`
- "¿Por qué M6 está lento?" → `diagnose_machine`
- "¿Tenemos fotos huérfanas ocupando espacio?" → `find_orphan_media`
- "Reprocesar este batch que falló" → `force_machine_retry`

### Implementación Técnica
```javascript
// Necesita acceso a:
// 1. Supabase MCP (media_queue, media_queue_photos, drafts)
// 2. Storage MCP (listar archivos, verificar existencia)
// 3. Vercel Logs (para traces de M5-M6)

const tools = [
  {
    name: "trace_photo",
    handler: async ({ file_id }) => {
      // 1. Buscar en media_queue_photos
      const photo = await supabaseMcp.query("media_queue_photos", { file_id });
      
      // 2. Buscar queue asociada
      const queue = await supabaseMcp.query("media_queue", { id: photo.queue_id });
      
      // 3. Buscar drafts resultantes
      const drafts = await supabaseMcp.query("product_drafts", { batch_id: queue.id });
      
      // 4. Verificar Storage
      const inStorage = await storageMcp.exists(photo.storage_path);
      
      // 5. Reconstruir journey
      return reconstructJourney(queue, photo, drafts, inStorage);
    }
  }
];
```

---

## 🎯 MCP #5: Customer-360 — Panorámica de Clientes (Tenants como Negocio)

**Tipo:** MCP de CRM / Gestión Comercial  
**Prioridad:** Media (creciente a alta conforme escales)  
**Dependencia de Supabase MCP:** Media — queries agregadas por tenant  
**Nota:** ESTE es el que mencionaste explícitamente. Foco en "mis clientes a la mano", no debugging.

### Propósito
Ver tus tenants como **clientes de negocio**, no como entidades técnicas. Onboarding, engagement, churn risk, expansión. El CRM que Supabase no te da.

### Tools Propuestas

#### `get_customer_dashboard(view)`
Vista panorámica de tu base de clientes.
```typescript
// view: "all" | "active" | "at_risk" | "new"
{
  "view": "active",
  "total_customers": 23,
  "summary": {
    "by_plan": { "starter": 15, "pro": 7, "enterprise": 1 },
    "by_rubro": { "retail-clothing": 12, "real-estate": 8, "lab": 3 },
    "mrr": "$1,840 USD",
    "churn_risk_count": 2
  },
  "customers": [
    {
      "slug": "tu-stilo",
      "name": "Tu Stilo (Miche)",
      "plan": "pro",
      "rubro": "retail-clothing",
      "mrr": "$59",
      "health_score": "95/100",
      "last_activity": "2h ago",
      "engagement": "high", // fotos/semana, login frecuencia
      "expansion_opportunity": "whatsapp_integration"
    },
    {
      "slug": "demo-inmo-norte",
      "name": "Inmobiliaria Norte",
      "plan": "starter",
      "rubro": "real-estate",
      "mrr": "$29",
      "health_score": "45/100",
      "last_activity": "14d ago",
      "engagement": "low",
      "churn_risk": "high",
      "recommended_action": " outreach_call"
    }
  ]
}
```

#### `analyze_customer_journey(tenant_slug)`
Funnel de onboarding y engagement.
```typescript
// Ejemplo: analyze_customer_journey("tu-stilo")
{
  "tenant": "tu-stilo",
  "onboarding_status": "completed",
  "journey": {
    "signup": "2026-02-15",
    "first_photo": "2026-02-15", // mismo día = buen onboarding
    "first_publish": "2026-02-16",
    "first_sale": "2026-02-20"
  },
  "engagement_trend": "increasing", // flat | increasing | decreasing
  "usage_stats": {
    "photos_last_30d": 127,
    "drafts_published": 89,
    "sales_recorded": 45,
    "avg_session_duration": "12min"
  },
  "milestones": [
    { "date": "2026-02-20", "event": "First sale recorded", "celebrated": true }
  ],
  "next_milestone": "100_products_published"
}
```

#### `identify_expansion_opportunities()`
Detecta oportunidades de upsell/cross-sell.
```typescript
{
  "opportunities": [
    {
      "tenant": "tu-stilo",
      "opportunity": "upgrade_to_enterprise",
      "reason": "150 products (limit: 70), high engagement",
      "potential_mrr": "$199",
      "current_mrr": "$59",
      "confidence": "high"
    },
    {
      "tenant": "demo-inmo-norte",
      "opportunity": "whatsapp_addon",
      "reason": "Real estate vertical, no WhatsApp integration yet",
      "potential_mrr": "+$20",
      "confidence": "medium"
    },
    {
      "tenant": "laboratorio-salud",
      "opportunity": "referral_program",
      "reason": "Lab vertical, likely knows other labs",
      "potential": "3-5 new customers",
      "confidence": "medium"
    }
  ]
}
```

#### `get_churn_risk_report()`
Quiénes se están enfriando.
```typescript
{
  "at_risk_customers": [
    {
      "tenant": "demo-inmo-norte",
      "risk_score": 0.85,
      "indicators": [
        "No login in 14 days",
        "0 photos uploaded in 30 days",
        "Previous complaint about 'processing slow'"
      ],
      "recommended_action": "Personal call from Nadi",
      "save_probability": "60%"
    }
  ],
  "churned_this_month": [
    { "tenant": "old-demo-moda", "churn_date": "2026-03-15", "reason_inferred": "never_activated" }
  ]
}
```

#### `generate_customer_report(tenant_slug, format)`
Reporte para el cliente (white-label).
```typescript
// Genera PDF o Markdown con métricas de su negocio
{
  "tenant": "tu-stilo",
  "report_period": "2026-02-01 to 2026-02-29",
  "format": "pdf",
  "content": {
    "inventory_growth": "+45 products",
    "sales_summary": "$2,340 total, 67 transactions",
    "top_products": ["Jeans Azul M", "Polera Negra S"],
    "photo_processing": "127 batches, 99.2% success rate"
  },
  "usage_tip": "Considera usar /ventas mes para ver tus mejores productos"
}
```

### Casos de Uso
- "Muéstrame mis clientes" → `get_customer_dashboard`
- "¿Quién está en riesgo de irse?" → `get_churn_risk_report`
- "¿A quién le puedo vender más?" → `identify_expansion_opportunities`
- "¿Cómo está Miche este mes?" → `analyze_customer_journey("tu-stilo")`
- "Generar reporte mensual para el cliente" → `generate_customer_report`

### Implementación Técnica
```javascript
// Agrega lógica de negocio sobre datos de Supabase
// NO es debugging — es analytics y CRM

const tools = [
  {
    name: "get_customer_dashboard",
    handler: async ({ view }) => {
      // 1. Obtener tenants
      const tenants = await supabaseMcp.query("tenants", {});
      
      // 2. Para cada tenant, calcular métricas de engagement
      const enriched = await Promise.all(tenants.map(async t => {
        const activity = await calculateActivityMetrics(t.id);
        const health = calculateHealthScore(activity);
        return { ...t, ...activity, health_score: health };
      }));
      
      // 3. Clasificar por riesgo y oportunidad
      return classifyCustomers(enriched, view);
    }
  },
  
  {
    name: "analyze_customer_journey",
    handler: async ({ tenant_slug }) => {
      // Timeline de hitos del cliente
      const events = [
        await getSignupDate(tenant_slug),
        await getFirstPhoto(tenant_slug),
        await getFirstPublish(tenant_slug),
        await getFirstSale(tenant_slug),
        // ... etc
      ];
      
      return { journey: events, recommendations: generateTips(events) };
    }
  }
];

// Fórmulas de negocio (ejemplos)
function calculateHealthScore(metrics) {
  const recency = daysSince(metrics.last_activity); // 0-40 puntos
  const frequency = metrics.sessions_last_30d; // 0-30 puntos
  const monetary = metrics.mrr / 10; // 0-30 puntos
  return Math.min(100, recency + frequency + monetary);
}
```

---

## 🔄 Relación con MCP de Supabase Existente

| MCP Supabase | Nuestros MCPs | Relación |
|-------------|---------------|----------|
| `execute_sql` | Todos | Base de datos — nuestros MCPs usan Supabase MCP para queries |
| `apply_migration` | Schema-Guardian | Extiende con validación pre-migración |
| `get_advisors` | Yamato-Ops | Complementa — advisors ven seguridad, nosotros vemos features |
| `list_tables` | Todos | Metadata — usamos para descubrir schema |

**Arquitectura de capas:**
```
┌─────────────────────────────────────────┐
│  LLM (Kimi)                             │
│  "¿Cómo está el vertical de inmo?"      │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  MCP Yamato-Ops                         │
│  translate: "vertical inmo" → queries   │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  MCP Supabase (ya existente)            │
│  execute: queries reales                │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Supabase PostgreSQL                    │
└─────────────────────────────────────────┘
```

---

## 📊 Comparativa y Roadmap de Implementación

| MCP | Complejidad | Valor Inmediato | Orden Sugerido | Dependencias |
|-----|-------------|-----------------|----------------|--------------|
| **Yamato-Ops** | Media | Alto (debugging diario) | 1° | Supabase MCP ✅ |
| **Schema-Guardian** | Alta | Alto (previene bugs) | 2° | Supabase MCP + parser TS |
| **Customer-360** | Media | Medio (escala con clientes) | 3° | Supabase MCP + lógica de negocio |
| **Image-Pipeline-Inspector** | Media-Alta | Media (específico de fotos) | 4° | Supabase MCP + Storage API |
| **Telegram-Simulator** | Baja | Media (testing) | 5° | Webhook local + fixtures |

---

## 💡 Consideraciones de Diseño

### MCP vs. Protocolo Manual

Para **Schema-Guardian**, podría ser un **protocolo** (como sugeriste):
- Pre-commit hook que corre `validate_schema_compatibility`
- GitHub Action que bloquea PR si hay drift
- No necesita ser MCP, puede ser CLI tool

**Pero como MCP gana:**
- Kimi puede invocarlo *durante* la conversación: "Espera, déjame validar ese schema antes de que escribas el código"
- Feedback inmediato sin cambiar de contexto

### Estado Persistente

Los MCPs son stateless por diseño. Si necesitas "recordar" cosas entre invocaciones:
- Usar tablas en Supabase (ej: `mcp_execution_logs`)
- O simplemente pasar el contexto en cada llamada

### Seguridad

Todos estos MCPs deben correr:
- **Local** (desarrollo): `localhost:3001`
- **Protegidos** (producción): Solo accesibles desde Kimi CLI autenticado
- **Con service_role** de Supabase (bypass RLS para operaciones admin)

---

## 🚀 Próximo Paso Recomendado

**MVP: MCP Yamato-Ops con 3 tools**

1. `get_system_overview` — Dashboard de salud del castillo
2. `check_vertical_status` — Estado por rubro
3. `check_feature_health` — Diagnóstico por feature

**Tiempo estimado:** 2-3 horas  
**Stack:** Node.js + MCP SDK + Supabase MCP como dependency  
**Valor:** Inmediato — reemplaza ir a Vercel dashboard + Supabase + logs

---

*Documento guardado para referencia de arquitectura.*  
*Prioridad: Implementar Yamato-Ops primero, luego iterar.*
