---
name: env-validator
description: Validates environment variables — checks for required keys, validates formats, and prevents runtime errors from missing config.
trigger: "check env", "env vars", "environment", "setup local", "config error", "missing env", ".env"
origin: Nadistudio
cost_aware: false
validation:
  - scaffold-validator env-validator --check required
  - scaffold-validator env-validator --check format
  - scaffold-validator env-validator --check secrets
---

# Part 1: Goals

## When to Activate

- User says "check env vars", "debug config error", or "setup local dev"
- Missing env vars causing crashes
- Onboarding new environment
- Deploying to new environment (Vercel, etc.)
- CI/CD pipeline failing due to missing config

## When NOT to Use

- **Runtime configuration** → Use feature flags or database config
- **Secret rotation** → Use secret management service
- **Different env per tenant** → Use tenant settings in database

## Core Principles

1. **Fail Fast at Startup** — Validate on boot, not at runtime
2. **Never Log Secrets** — Log presence, never values
3. **Document Everything** — `.env.example` is the source of truth
4. **Format Validation** — URLs look like URLs, tokens like tokens
5. **Clear Error Messages** — Tell exactly what's missing

---

# Part 2: Execution

## Quick Start: Required Variables

### Supabase
| Variable | Format | Required |
|---------|--------|----------|
| `NEXT_PUBLIC_SUPABASE_URL` | https://xxx.supabase.co | ✅ |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | eyJ... | ✅ |
| `SUPABASE_SERVICE_ROLE_KEY` | eyJ... | ✅ |

### Kimi
| Variable | Format | Required |
|---------|--------|----------|
| `KIMI_API_KEY` | sk-... | ✅ |

### Telegram
| Variable | Format | Required |
|---------|--------|----------|
| `TELEGRAM_BOT_TOKEN_{NAME}` | 123456:ABC-... | Per bot |
| `TELEGRAM_WEBHOOK_SECRET` | Random string | ✅ |

### Site
| Variable | Format | Required |
|---------|--------|----------|
| `NEXT_PUBLIC_SITE_URL` | https://xxx.vercel.app | ✅ |
| `NEXT_PUBLIC_WHATSAPP_NUMBER` | +56912345678 | Per vertical |
| `DEV_TENANT_SLUG` | demo-moda | Dev only |
| `DEPLOY_TARGET` | `public` or `ops` | Optional (default: `ops`)

## Implementation Patterns

### Pattern 1: Validation Function

```typescript
// lib/env.ts
const required = [
  'NEXT_PUBLIC_SUPABASE_URL',
  'SUPABASE_SERVICE_ROLE_KEY',
  'KIMI_API_KEY'
];

export function validateEnv() {
  const missing = required.filter(key => !process.env[key]);
  if (missing.length > 0) {
    throw new Error(`Missing env vars: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

### Pattern 2: Debug Command

```bash
# List all env vars (without secrets)
grep -E "^[^#]" .env.example | cut -d= -f1 | sort
```

## Step-by-Step Workflow

### Step 1: Check .env.example
Verify all required vars are documented.

### Step 2: Run Validation
```typescript
validateEnv();
```

### Step 3: Fix Missing Vars
Add missing environment variables.

### Step 4: Verify Formats
- URLs: Must start with https://
- Keys: Must match expected prefix
- Numbers: Must be valid format

---

# Part 3: Quality Standards

## Pre-Delivery Checklist

- [ ] All required vars present
- [ ] Format validated (URLs are URLs, tokens are tokens)
- [ ] `.env.example` documented with all vars
- [ ] No hardcoded secrets in code
- [ ] Validation runs at startup
- [ ] Clear error messages for missing vars
- [ ] Secrets never logged

## Validation Commands

```bash
# Check required vars present
scaffold-validator env-validator --check required

# Validate formats
scaffold-validator env-validator --check format

# Check no secrets hardcoded
scaffold-validator env-validator --check secrets --path lib/

# List env vars from .env.example
grep -E "^[^#]" .env.example | cut -d= -f1 | sort
```

## Common Errors

| Error | Symptom | Cause | Fix |
|-------|---------|-------|-----|
| **Missing env var** | `undefined` in code | Var not set | Add to `.env` and environment |
| **Wrong format** | Connection fails | URL malformed | Fix format (https:// required) |
| **Secret in logs** | Security leak | Logged actual value | Log presence only, never values |
| **Runtime error** | Crash on first use | Validation not at startup | Call `validateEnv()` at boot |

---

# Part 4: Technical Reference

## Variable Categories

```
Required (All Environments):
- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY
- SUPABASE_SERVICE_ROLE_KEY
- KIMI_API_KEY
- TELEGRAM_WEBHOOK_SECRET
- NEXT_PUBLIC_SITE_URL

Per Bot (As needed):
- TELEGRAM_BOT_TOKEN_{NAME}

Per Vertical (As needed):
- NEXT_PUBLIC_WHATSAPP_NUMBER

Development Only:
- DEV_TENANT_SLUG

Optional:
- DEPLOY_TARGET (default: 'ops')
```

## Integration with Other Skills

| Skill | When to Use Together | Pattern |
|-------|---------------------|---------|
| `4-step-docs` | New project setup | Document env vars in setup guide |
| `security-auditor` | Security review | Check no secrets in code |
| `vertical-factory` | New vertical | Add per-vertical env vars |

## Constraints

- **Never log actual values** of secrets
- **Validate at startup**, not at runtime
- **Document in `.env.example`** — source of truth
- **Fail fast** — throw on missing required vars

---

*Skill: env-validator — v2.0*  
*Template: 4-Part Structure*  
*Core Rule: Validate at Startup, Never Log Secrets*
