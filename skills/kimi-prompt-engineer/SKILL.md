---
name: kimi-prompt-engineer
description: Writes and validates Kimi prompts for each vertical with JSON schemas, few-shot examples, the ONLY valid configuration (temp 0.6 + thinking disabled), config validation matrix, and A/B test scripts.
trigger: "Kimi prompt", "prompt engineering", "AI extraction", "optimize prompt", "Kimi config", "temperature", "thinking disabled", "json_object"
origin: Nadistudio
cost_aware: true
validation:
  - scaffold-validator kimi-prompt-engineer --check config
  - scaffold-validator kimi-prompt-engineer --check json-schema
  - scaffold-validator kimi-prompt-engineer --check safeint-usage
---

# Part 1: Goals

## When to Activate

- User says "write a Kimi prompt for [vertical]", "optimize AI extraction", or "improve detection"
- Adding a new vertical with Kimi Vision or text classification
- Kimi returns garbage or wrong fields
- Configuring AI temperature and thinking mode
- A/B testing prompt variations

## When NOT to Use

- **Non-AI text processing** → Use regex or standard parsing
- **Simple classification** → Use regex in `irce-engineer`
- **Image processing without extraction** → Use storage only
- **Chat/conversation** → Use different pattern (not extraction)

## Core Principles

1. **0.6 + Disabled is Law** — Only valid config for structured extraction
2. **JSON Schema First** — Define output format before writing prompt
3. **Few-Shot Examples** — Show, don't just tell
4. **SafeInt Everything** — Kimi returns floats for integers
5. **Validate Before DB** — JSON parse before insert

---

# Part 2: Execution

## Quick Start: The ONLY Valid Config

```typescript
{
  model: 'kimi-k2.5',
  temperature: 0.6,                      // ONLY 0.6 works
  thinking: { type: 'disabled' },         // REQUIRED
  response_format: { type: 'json_object' } // Structured output
}
```

**Invalid combos fail SILENTLY.** No error, just garbage.

## Implementation Patterns

### Pattern 1: Configuration Matrix

| Temp | Thinking | Valid? | Use Case |
|------|----------|--------|----------|
| **0.6** | **disabled** | ✅ | Structured extraction |
| 1.0 | enabled | ✅ | Creative descriptions |
| 0.6 | enabled | ❌ | Silent garbage |
| 1.0 | disabled | ❌ | Silent garbage |
| Any other | Any | ❌ | Error |

### Pattern 2: Complete Prompt Template

```typescript
export const PRODUCT_PROMPT = `
Eres un asistente que analiza fotos de ropa para inventario.

TAREA:
Analiza la(s) foto(s) y extrae los productos detectados.

REGLA DE AGRUPACION:
Si ves varias prendas del MISMO tipo base pero en distintos colores o tallas, todas deben tener el MISMO "suggested_base_name". El color y talla van en campos separados.

OUTPUT FORMAT (JSON):
{
  "items": [
    {
      "suggested_name": "Pantalón de Pana Azul Marino Talla M",
      "suggested_base_name": "Pantalón de Pana",
      "suggested_category": "pantalones",
      "suggested_brand": "Zara",
      "suggested_color": "azul marino",
      "suggested_size": "M",
      "suggested_quantity": 1,
      "suggested_condition": "nuevo",
      "suggested_price": 19990,
      "confidence": 0.95
    }
  ]
}

EJEMPLOS:
Input: [foto de tres jeans apilados, colores negro, azul, beige]
Output: {
  "items": [
    { "suggested_name": "Jeans Negro Talla 32", "suggested_base_name": "Jeans", "suggested_color": "negro", "suggested_size": "32", "suggested_quantity": 1 },
    { "suggested_name": "Jeans Azul Talla 32", "suggested_base_name": "Jeans", "suggested_color": "azul", "suggested_size": "32", "suggested_quantity": 1 },
    { "suggested_name": "Jeans Beige Talla 32", "suggested_base_name": "Jeans", "suggested_color": "beige", "suggested_size": "32", "suggested_quantity": 1 }
  ]
}

IMPORTANTE: Responde ÚNICAMENTE con JSON válido.
`;
```

### Pattern 3: safeInt() Wrapper (CRITICAL)

```typescript
// CRITICAL: Kimi returns floats for integers
const safeInt = (v: any): number | null =>
  v != null ? Math.round(Number(v)) : null;

// Wrap ALL integer fields
const insertData = {
  detected_name: analysis.name,
  detected_price: safeInt(analysis.price),
  detected_quantity: safeInt(analysis.quantity),
};
```

### Pattern 4: Integration Function

```typescript
export async function analyzeProductImages(images: Buffer[], caption?: string) {
  const messages = [
    { role: 'system', content: PRODUCT_PROMPT },
    { role: 'user', content: buildUserMessage(images, caption) }
  ];
  
  const response = await callKimi(messages, {
    temperature: 0.6,
    thinking: { type: 'disabled' },
    response_format: { type: 'json_object' }
  });
  
  return parseProductResponse(response);
}
```

### Pattern 5: A/B Test Script

```typescript
async function testTemperatureCombo(temp: number, thinking: boolean) {
  const result = await callKimi({
    temperature: temp,
    thinking: thinking ? { type: 'enabled' } : { type: 'disabled' },
    prompt: 'Responde JSON: {"value": 42}'
  });

  try {
    const parsed = JSON.parse(result);
    return { temp, thinking, success: parsed.value === 42, parsed };
  } catch {
    return { temp, thinking, success: false, raw: result };
  }
}

// Run tests
const tests = await Promise.all([
  testTemperatureCombo(0.6, false),   // Expected: ✅
  testTemperatureCombo(1.0, true),    // Expected: ✅
  testTemperatureCombo(0.6, true),    // Expected: ❌
  testTemperatureCombo(1.0, false),   // Expected: ❌
]);
```

## Step-by-Step Workflow

### Step 1: Define JSON Schema
Define exact output format with field types.

### Step 2: Write System Prompt
- Clear role definition
- Task description
- Output format (JSON schema)
- Few-shot examples
- Constraints

### Step 3: Configure Kimi
```typescript
temperature: 0.6,
thinking: { type: 'disabled' },
response_format: { type: 'json_object' }
```

### Step 4: Test with A/B Script
Verify config before production.

### Step 5: Add safeInt() Wrappers
Prevent 22P02 errors on integer fields.

### Step 6: Validate JSON Before DB
```typescript
try {
  const parsed = JSON.parse(response);
  // Validate with Zod schema
} catch (e) {
  // Handle parse error
}
```

---

# Part 3: Quality Standards

## Pre-Delivery Checklist

- [ ] Temperature set to exactly 0.6
- [ ] Thinking mode set to `{ type: 'disabled' }`
- [ ] `response_format: { type: 'json_object' }` configured
- [ ] JSON schema documented in prompt
- [ ] Few-shot examples included
- [ ] `safeInt()` wrapper used on all integer fields
- [ ] JSON validated before database insert
- [ ] A/B test passed (0.6 + disabled)
- [ ] Retry logic implemented for API failures
- [ ] Timeout configured (~30s for vision, ~5s for text)

## Validation Commands

```bash
# Verify Kimi config (temp + thinking)
scaffold-validator kimi-prompt-engineer --check config --file lib/ai/config.ts

# Check JSON schema defined
scaffold-validator kimi-prompt-engineer --check json-schema --file prompts/

# Verify safeInt usage
scaffold-validator kimi-prompt-engineer --check safeint-usage --path lib/
```

## Common Errors

| Symptom | Cause | Fix |
|---------|-------|-----|
| Garbage JSON | Wrong temp + thinking combo | Use 0.6 + disabled |
| `invalid temperature` | Temp not 0.6 or 1.0 | Use exactly 0.6 |
| Slow response | Thinking enabled | Use disabled |
| Missing fields | Prompt unclear | Add JSON schema to prompt |
| Float in integer field | LLM outputs decimals | Use safeInt() |
| JSON parse error | Invalid JSON output | Add try/catch + retry |

---

# Part 4: Technical Reference

## Test Endpoint Template

Create `app/api/test-kimi/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { text, imageBuffers, config } = await req.json();

  const response = await fetch('https://api.moonshot.cn/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.KIMI_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: 'kimi-k2.5',
      messages: [
        {
          role: 'user',
          content: text || 'Responde con JSON: {"status": "ok", "test": true}'
        }
      ],
      temperature: config?.temperature ?? 0.6,
      thinking: config?.thinking ?? { type: 'disabled' },
      response_format: { type: 'json_object' }
    }),
  });

  const data = await response.json();
  return NextResponse.json({
    config: {
      temperature: config?.temperature ?? 0.6,
      thinking: config?.thinking ?? { type: 'disabled' }
    },
    response,
    content: data.choices?.[0]?.message?.content
  });
}
```

## Integration with Other Skills

| Skill | When to Use Together | Pattern |
|-------|---------------------|---------|
| `m1-m9-pipeline` | Photo analysis | Kimi Vision → Extraction → Draft |
| `irce-engineer` | Intent classification | LLM fallback for complex intents |
| `api-error-handler` | API resilience | Retry logic + error handling |
| `db-guardian` | Data insertion | safeInt() before INSERT |

## Cost Structure

| Component | Cost | When |
|-----------|------|------|
| Kimi K2.5 (text) | ~$0.001-0.003/call | 0.6 + disabled |
| Kimi Vision | ~$0.005-0.01/image | Photo analysis |
| Kimi with thinking | 2-3x cost | Not recommended for extraction |

## Constraints

- **Temperature MUST be 0.6** for structured extraction
- **Thinking MUST be disabled** for fast responses
- **Always validate JSON** before writing to DB
- **Always use safeInt()** on integer fields
- **Test config with A/B script** before production

---

*Skill: kimi-prompt-engineer — v2.0*  
*Template: 4-Part Structure*  
*Golden Rule: 0.6 + Disabled for Extraction*
