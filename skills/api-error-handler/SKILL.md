---
name: api-error-handler
description: Centralized error handling for API routes — discriminates Supabase, Kimi, Zod, and PostgREST errors with consistent JSON responses and the withErrorHandler HOC.
trigger: "API error", "error handler", "guardSupabase", "withErrorHandler", "try catch", "API route", "HTTP status", "Supabase error"
origin: Nadistudio
cost_aware: false
validation:
  - scaffold-validator api-error-handler --check guard-usage
  - scaffold-validator api-error-handler --check error-codes
  - scaffold-validator api-error-handler --check hoc-coverage
---

# Part 1: Goals

## When to Activate

- Creating a new API endpoint
- Existing routes have scattered `try/catch` with inconsistent error formats
- Debugging silent failures (ghost columns, RLS denials, Kimi garbage)
- Migrating routes to centralized error handling
- User says "add error handling", "standardize API errors", "guard Supabase"

## When NOT to Use

- **Client-side error handling** → Use UI error boundaries
- **Background job errors** → Use `m1-m9-pipeline` error handling
- **Database migrations** → Use `migration-generator` checks
- **Simple static routes** → Overkill for non-DB routes

## Core Principles

1. **Never Expose Internals** — Stack traces stay in logs, never in responses
2. **Discriminate Then Respond** — Know the error source before choosing HTTP status
3. **Fail Fast, Fail Clear** — Throw early with context, handle once at boundary
4. **Consistent Format** — All errors: `{ success: false, error: string, code?: string }`
5. **Gradual Adoption** — Wrap routes one by one, don't big-bang refactor

---

# Part 2: Execution

## Quick Start: Error Handler Setup

```typescript
// lib/api/errorHandler.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

// Error Classes
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public code?: string
  ) {
    super(message);
    Object.setPrototypeOf(this, ApiError.prototype);
  }
}

export class SupabaseError extends ApiError {
  constructor(public pgCode: string, message: string) {
    const status = SUPABASE_STATUS_MAP[pgCode] || 500;
    super(status, message, `PG-${pgCode}`);
  }
}

// PG Code → HTTP Status Mapping
const SUPABASE_STATUS_MAP: Record<string, number> = {
  '23505': 409, // unique_violation
  '23503': 400, // foreign_key_violation
  '22P02': 422, // invalid_input_syntax (Kimi float→int)
  '42501': 403, // insufficient_privilege (RLS)
  '42703': 400, // undefined_column (ghost column)
  'PGRST116': 404, // .single() no rows
};

// Error Discriminator
export function errorHandler(error: unknown, context?: string) {
  // 1. Known operational errors
  if (error instanceof ApiError) {
    return NextResponse.json(
      { success: false, error: error.message, code: error.code },
      { status: error.statusCode }
    );
  }

  // 2. Zod validation
  if (error instanceof z.ZodError) {
    return NextResponse.json(
      {
        success: false,
        error: 'Validation failed',
        details: error.errors.map(e => ({
          field: e.path.join('.'),
          message: e.message,
        })),
      },
      { status: 400 }
    );
  }

  // 3. SyntaxError (empty POST body)
  if (error instanceof SyntaxError) {
    return NextResponse.json(
      { success: false, error: 'Invalid or empty request body' },
      { status: 400 }
    );
  }

  // 4. Unknown — log with context, never expose internals
  console.error(`[API${context ? `:${context}` : ''}] Unexpected:`, error);
  return NextResponse.json(
    { success: false, error: 'Internal server error' },
    { status: 500 }
  );
}
```

## Implementation Patterns

### Pattern 1: guardSupabase() Wrapper

```typescript
// CORRECT ✅ — Centralized error handling
export function guardSupabase<T>(
  result: { data: T | null; error: any },
  context?: string
): T {
  if (result.error) {
    throw new SupabaseError(
      result.error.code || 'UNKNOWN',
      result.error.message || `Supabase error in ${context}`
    );
  }
  if (result.data === null) {
    throw new ApiError(404, `No data found${context ? ` for ${context}` : ''}`);
  }
  return result.data;
}

// Usage
const products = guardSupabase(result, 'products.list');
```

### Pattern 2: withErrorHandler() HOC

```typescript
// CORRECT ✅ — Wrap route handler
export function withErrorHandler(
  handler: (req: NextRequest) => Promise<NextResponse>,
  routeName?: string
) {
  return async (req: NextRequest) => {
    try {
      return await handler(req);
    } catch (error) {
      return errorHandler(error, routeName);
    }
  };
}

// Usage
export const POST = withErrorHandler(async (req) => {
  // Handler logic throws freely
}, 'products/list');
```

### Pattern 3: Kimi-Safe Wrapper

```typescript
// CORRECT ✅ — Handle Kimi failures
export async function callKimiSafe<T>(
  fn: () => Promise<T>,
  schema?: z.ZodType<T>
): Promise<T> {
  try {
    const result = await fn();
    if (schema) return schema.parse(result);
    return result;
  } catch (error) {
    if (error instanceof z.ZodError) {
      throw new ApiError(422, 'Kimi output failed validation', 'L1-AI-004');
    }
    if (error instanceof TypeError && error.message.includes('fetch')) {
      throw new ApiError(502, 'Kimi API unreachable', 'L1-AI-002');
    }
    throw new ApiError(502, 'Kimi processing failed', 'L1-AI-001');
  }
}
```

## Step-by-Step Migration

### Step 1: Create errorHandler.ts
Copy template to `lib/api/errorHandler.ts`

### Step 2: Wrap One Route
```typescript
import { withErrorHandler, guardSupabase } from '@/lib/api/errorHandler';

export const POST = withErrorHandler(async (req) => {
  const result = await supabase.from('products').select('*');
  const products = guardSupabase(result, 'products.list');
  return NextResponse.json({ success: true, data: products });
}, 'products/list');
```

### Step 3: Migrate Routes Gradually
- Pick one route at a time
- Replace `{ data, error }` checks with `guardSupabase()`
- Test before moving to next route

---

# Part 3: Quality Standards

## Pre-Delivery Checklist

- [ ] `lib/api/errorHandler.ts` created with all error classes
- [ ] Routes wrapped with `withErrorHandler()`
- [ ] Supabase queries use `guardSupabase()` not manual error checks
- [ ] Error codes reference `ERRORS/INDEX.md` (LX-XXX, L1-XXX, etc.)
- [ ] Stack traces never exposed in production responses
- [ ] Webhook routes ALWAYS return Response (even on error)
- [ ] Kimi calls wrapped with `callKimiSafe()` or equivalent
- [ ] Empty POST body handled (SyntaxError caught)

## Validation Commands

```bash
# Check guardSupabase usage across codebase
scaffold-validator api-error-handler --check guard-usage --path app/api/

# Verify error codes reference valid LX-### codes
scaffold-validator api-error-handler --check error-codes --path lib/api/

# Check HOC coverage (which routes are wrapped)
scaffold-validator api-error-handler --check hoc-coverage --path app/api/
```

## Common Errors

| Error Source | Symptom | Detection | HTTP Status |
|--------------|---------|-----------|-------------|
| **Supabase PostgREST** | `{ data: null, error: {...} }` | Check `error` field | 400/404/409 |
| **Supabase RLS** | Empty `{ data: [], error: null }` | No rows + authenticated = RLS | 403 |
| **Ghost Column** | `.select('nonexistent')` silent fail | Verify columns in schema | 400 |
| **Kimi API** | Network timeout, garbage output | Wrap in try/catch + validate | 502/422 |
| **Kimi Float→Int** | `22P02` on integer column | Use `safeInt()` wrapper | 422 |
| **Zod Validation** | Throws `ZodError` | `instanceof z.ZodError` | 400 |
| **Empty POST body** | `request.json()` crash | SyntaxError caught | 400 |
| **Next.js params** | `params` not awaited | `await params` | 500 |

---

# Part 4: Technical Reference

## Error Taxonomy

```
ApiError (base)
├── SupabaseError (pgCode mapping)
├── ZodError (validation)
├── SyntaxError (JSON parse)
└── Unknown → 500 (logged, not exposed)
```

## PG Code Mapping

| Code | Meaning | HTTP Status |
|------|---------|-------------|
| 23505 | unique_violation | 409 |
| 23503 | foreign_key_violation | 400 |
| 22P02 | invalid_input_syntax | 422 |
| 42501 | insufficient_privilege (RLS) | 403 |
| 42703 | undefined_column (ghost) | 400 |
| PGRST116 | .single() no rows | 404 |

## Integration with Other Skills

| Skill | When to Use Together | Pattern |
|-------|---------------------|---------|
| `db-guardian` | Database queries | `guardSupabase()` + `.eq('tenant_id')` |
| `kimi-prompt-engineer` | AI calls | `callKimiSafe()` with Zod validation |
| `irce-engineer` | Bot handlers | Wrap handlers with `withErrorHandler()` |
| `debug-epistemologist` | Debugging errors | Map errors to LX-### codes |
| `m1-m9-pipeline` | Async processing | Error handling in machine callbacks |

## Common Patterns

| Scenario | Pattern |
|----------|---------|
| Supabase query fails | `guardSupabase(result, 'context')` |
| Optional row lookup | Use `.maybeSingle()` NOT `.single()` |
| Webhook secret | `if (secret !== expected) throw new ApiError(401, ...)` |
| Fire-and-forget | Use `after()` from `next/server` |
| Kimi returns floats | Pipe through `safeInt()` BEFORE DB insert |

---

## Information Gaps — Catastro

| Sección | Estado | Qué falta definir |
|---------|--------|-------------------|
| Validation Commands | ⚠️ | Crear `scaffold-validator` CLI |
| Template file path | ⚠️ | Verificar `templates/lib/api/errorHandler.ts` existe |
| Safe-int wrapper | ⚠️ | Crear skill `safe-int-wrapper` o integrar aquí |

---

*Skill: api-error-handler — v2.0*  
*Template: 4-Part Structure*  
*Pattern: Fail Fast, Handle Once*
