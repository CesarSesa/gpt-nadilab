---
name: irce-engineer
description: Implements the IRCE pipeline (Intent → Resolution → Confirmation → Execution) for building conversational bots with natural language understanding. Transforms bots from command executors into conversational assistants.
trigger: "bot command", "natural language", "intent", "IRCE", "conversational", "understand text", "vendi", "vendí", "cuantos quedan", "agregar stock"
origin: Nadistudio
cost_aware: true
validation:
  - scaffold-validator irce-engineer --check session-first
  - scaffold-validator irce-engineer --check confirmation-flow
  - scaffold-validator irce-engineer --check intent-coverage
---

# Part 1: Goals

## When to Activate

- Implementing bot commands from natural language input
- Adding SELL/QUERY/ADD_STOCK intents to a vertical
- Building Telegram/WhatsApp bots with conversational interfaces
- The vertical has `irce_enabled: true`
- User says: "add natural language to the bot", "implement IRCE", "make the bot conversational"

## When NOT to Use

- **Static command bots** (only `/start`, `/help`) → No NLP needed
- **Simple notifications** (one-way messages) → Use standard bot webhook
- **Photo-only workflows** → Use `m1-m9-pipeline` instead
- **Complex multi-session flows** without confirmation → Requires custom session management

## Core Principles

1. **Understand First** — Regex (80%, $0) before LLM (20%, ~$0.001)
2. **Confirm Second** — Show what will happen before destructive operations
3. **Execute Third** — Atomic transactions with operation IDs
4. **Narrate Always** — Rich feedback + next steps
5. **Session Before Intent** — Check `active_sessions` before classifying
6. **Tenant Isolation** — Every query filters by `tenant_id`

---

# Part 2: Execution

## Quick Start: 6-Layer Architecture

```
┌──────────────────────────────────────────────────────────────┐
│  ① INPUT         Capture any input type → normalize to text  │
│  ② COMPREHENSION Classify intent + extract entities          │
│  ③ RESOLUTION    Match entities to real DB records            │
│  ④ CONFIRMATION  Show what will happen, ask permission        │
│  ⑤ EXECUTION     Atomic DB transaction                       │
│  ⑥ RESPONSE      Rich feedback + next steps                  │
└──────────────────────────────────────────────────────────────┘
```

## Implementation Patterns

### Pattern 1: Session Check Before Intent (CRITICAL)

```typescript
// CORRECT ✅
const session = await getActiveSession(userId, tenantId);
if (session) {
  await continueSession(session, text, tenantId);
  return;
}
// Only then classify intent
const { intent, confidence } = await classifyIntent(text, rubroType);

// WRONG ❌
const { intent, confidence } = await classifyIntent(text, rubroType); // May interrupt session!
```

### Pattern 2: Hybrid Intent Classification (Regex → LLM)

```typescript
// 1. Regex fast path (80%, <50ms, $0)
for (const intent of INTENT_ORDER) {
  const pattern = new RegExp(INTENTS[intent].keywords.join('|'), 'i');
  if (pattern.test(normalized)) {
    const entities = extractEntities(normalized, intent);
    if (entities.confidence > 0.7) {
      return { intent, confidence: 0.9, entities, method: 'regex' };
    }
  }
}

// 2. LLM fallback (20%, ~500ms, ~$0.001)
return await llmClassify(text, rubro);
```

### Pattern 3: Confirmation Parser (Explicit Only)

```typescript
// CORRECT ✅ — Only explicit "no" cancels
const CANCEL = ['no', 'n', 'nop', 'nope', 'cancelar', 'abortar'];
const CONFIRM = ['sí', 'si', 's', 'yes', 'ok', 'confirmar', 'dale', 'listo', 'ya'];

export function parseConfirmation(text: string): 'CONFIRM' | 'CANCEL' | 'CLARIFY' {
  const normalized = text.toLowerCase().trim();
  if (CANCEL.includes(normalized)) return 'CANCEL';
  if (CONFIRM.includes(normalized)) return 'CONFIRM';
  return 'CLARIFY'; // NOT CANCEL — ask again
}

// WRONG ❌ — Ambiguous becomes cancel (bad UX)
if (!CONFIRM.includes(text)) return 'CANCEL';
```

### Pattern 4: Fuzzy Resolution with Weights

```typescript
// Score products by weighted field matching
const scored = products.map(p => ({
  ...p,
  score: (p.name.includes(normalized) ? 50 : 0) +
         (p.color?.includes(normalized) ? 30 : 0) +
         (p.size?.includes(normalized) ? 20 : 0),
}));
```

### Pattern 5: safeInt() Wrapper for AI Outputs

```typescript
// CORRECT ✅
const safeInt = (v: any): number | null =>
  v != null ? Math.round(Number(v)) : null;

const insertData = {
  detected_quantity: safeInt(analysis.quantity),  // Prevents 22P02 errors
  detected_price: safeInt(analysis.price),
};

// WRONG ❌ — Direct AI output can be float → 22P02 error
const insertData = {
  detected_quantity: analysis.quantity,
};
```

## Step-by-Step Workflows

### Workflow 1: Adding IRCE to New Vertical

1. **Define intents** (30 min): List what users can DO
2. **Write regex patterns** (1 hour): Keywords for each intent
3. **Define entity weights** (15 min): What fields matter for resolution
4. **Write command handlers** (2-4 hours): One per destructive intent
5. **Wire into router** (15 min): Add to dispatch table
6. **Add to webhook** (10 min): Call `classifyIntent()` before fallback

**Total: ~4-6 hours** per vertical.

### Workflow 2: Webhook Handler Structure

```typescript
// 1. Check session FIRST
const session = await getActiveSession(userId, tenantId);
if (session) {
  await continueSession(session, text, tenantId);
  return;
}

// 2. Classify intent
const { intent, confidence, entities } = await classifyIntent(text, rubroType);
if (confidence > 0.6) {
  // 3. Check if confirmation needed
  if (requiresConfirmation(intent)) {
    await savePendingOperation(chatId, userId, intent, entities);
    await sendMessage(chatId, buildConfirmation(intent, entities));
    return;
  }
  // 4. Execute directly
  const result = await executeIntent(intent, entities, tenantId);
  await sendMessage(chatId, result);
  return;
}

// 5. Fall through to "No entendí"
```

---

# Part 3: Quality Standards

## Pre-Delivery Checklist

- [ ] Session check happens BEFORE intent classification
- [ ] Regex patterns cover 80% of common messages
- [ ] All destructive intents require confirmation
- [ ] Confirmation only cancels on EXPLICIT "no"
- [ ] Fuzzy resolution filters by `tenant_id` + `is_active`
- [ ] `safeInt()` wrapper used on ALL AI integer outputs
- [ ] Operations use atomic transactions with operation IDs
- [ ] Response includes status + next steps
- [ ] Uses `after()` from `next/server` for fire-and-forget
- [ ] Cost tracking documented (regex vs LLM split)

## Validation Commands

```bash
# Check session check order
scaffold-validator irce-engineer --check session-first --path app/api/webhook/

# Verify confirmation flow coverage
scaffold-validator irce-engineer --check confirmation-flow --path lib/bot/

# Audit intent coverage for vertical
scaffold-validator irce-engineer --check intent-coverage --vertical retail
```

## Common Errors

| Error | Symptom | Cause | Fix |
|-------|---------|-------|-----|
| **Session Interrupt** | User mid-flow gets "no entendí" | Intent classified before session check | Reorder: session check FIRST |
| **Phantom Cancel** | Operation cancelled on ambiguous reply | Confirmation parser too aggressive | Use CLARIFY, not CANCEL |
| **22P02 Error** | "invalid input syntax for integer" | AI returned float for integer field | Use `safeInt()` wrapper |
| **Cross-Tenant Match** | User sees wrong product | Resolution query missing tenant filter | Add `.eq('tenant_id', ...)` |
| **Double Execution** | Sale recorded twice | Telegram retry, no idempotency | Check `update_id` before processing |
| **Ghost Product** | Resolved product out of stock | No pre-filter in resolver | Add `.gt('stock_quantity', 0)` |

---

# Part 4: Technical Reference

## Intent Output Format

```typescript
interface IntentResult {
  intent: string;           // Rubro-specific intent name
  confidence: number;       // 0.0 - 1.0
  entities: {
    [key: string]: any;     // Rubro-specific extracted data
  };
  method: 'regex' | 'llm';  // Which engine resolved it
}
```

## Confirmation Rules Matrix

| Operation | Confirm? | Why |
|-----------|----------|-----|
| SELL (deduct stock) | Always | Destructive — can't un-sell |
| ADD_STOCK | Always | Changes inventory counts |
| DELETE | Always | Removes product |
| RECTIFY | Always | Modifies historical records |
| QUERY (read stock) | Never | Read-only, no side effects |
| SEARCH | Never | Read-only |
| ADD_BATCH > 10 items | Yes | Large operation |
| ADD_BATCH ≤ 10 items | No | Normal operation |

## Resolution Score Thresholds

| Score | Action |
|-------|--------|
| `> 90` with single match | Auto-confirm (high confidence) |
| `75 - 90` or multiple matches | Show options to user |
| `< 75` | "No encontré ese producto. Quisiste decir...?" |

## Entity Weights by Vertical

```typescript
// Retail
const RETAIL_WEIGHTS = { name: 50, color: 30, size: 20 };

// Real estate
const REALESTATE_WEIGHTS = { commune: 50, property_type: 30, price_range: 20 };

// Automotive (future)
const AUTO_WEIGHTS = { brand: 40, model: 30, year: 20, color: 10 };

// Services (future)
const SERVICE_WEIGHTS = { service_name: 70, professional: 30 };
```

## Operation ID Format

```
#V001 = Sale (Venta) #1
#V002 = Sale #2
#L001 = Batch (Lote) #1
#R001 = Rectification #1
```

Counter lives in `operation_counters` table per tenant per type.

## Integration with Other Skills

| Skill | When to Use Together | Pattern |
|-------|---------------------|---------|
| `m1-m9-pipeline` | Photo input → IRCE processing | Photo → Kimi Vision → Layer 1 Input |
| `draft-workflow-pipeline` | IRCE creates drafts → Batch processing | Intent → Draft → M1-M9 pipeline |
| `vertical-factory` | Creating new vertical with IRCE | Step 5: IRCE configuration |
| `feature-expert` | Check existing intents per vertical | Reference matrix before adding new |
| `db-guardian` | Database queries in resolution | Always `.eq('tenant_id', ...)` |
| `api-error-handler` | Handling errors in handlers | Use `guardSupabase()` wrapper |

## Cost Structure

| Component | Cost | Frequency |
|-----------|------|-----------|
| Regex classification | $0 | 80% of messages |
| LLM intent (text) | ~$0.001/msg | 20% of messages |
| LLM Vision (photos) | ~$0.01/batch | Photo analysis only |
| Whisper (voice) | $0.006/min | Voice notes only |

**Example: 100 messages/day**
- 80 regex ($0) + 20 LLM ($0.02) + 5 voice ($0.03) + 2 photo ($0.02) = **~$0.07/day**

---

## Information Gaps — Catastro

| Sección | Estado | Qué falta definir |
|---------|--------|-------------------|
| Validation Commands | ⚠️ | Crear `scaffold-validator` CLI |
| pgvector upgrade path | ⚠️ | Semantic matching (future Phase 2) |
| WhatsApp adapter | ⚠️ | Same IRCE, different input API |
| Multi-turn conversations | ⚠️ | Complex session state patterns |

---

*Skill: irce-engineer — v2.0*  
*Template: 4-Part Structure*  
*Core Pattern: Intent → Resolution → Confirmation → Execution*
