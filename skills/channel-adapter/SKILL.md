---
name: channel-adapter
description: Connects new communication channels (WhatsApp, Email, SMS, Slack) to existing Castle pipelines with minimal changes. The "1-document methodology" for when IRCE/M1-M9 logic is already solved and only the input channel changes.
trigger: ["add whatsapp", "new channel", "email bot", "sms integration", "slack bot", "adapter", "bridge", "connect channel"]
version: 2.0
origin: Nadistudio
---

# Channel Adapter

> *"Don't rebuild the castle, just build a new bridge to it."*

---

## Part 1: Goals

### When to Activate (The Bridge Test)

**Ask yourself: "¿Estamos conectando un nuevo canal, o creando lógica nueva?"**

| Question | If YES → Channel Adapter | If NO → Use 4-Step Docs |
|----------|-------------------------|------------------------|
| Does IRCE engine already work? | ✅ Channel Adapter | ❌ 4-Step Docs |
| Is M1-M9 pipeline functional? | ✅ Channel Adapter | ❌ 4-Step Docs |
| Only the "input method" changes? | ✅ Channel Adapter | ❌ 4-Step Docs |
| Do we reuse 90%+ of existing code? | ✅ Channel Adapter | ❌ 4-Step Docs |
| Are we creating new business logic? | ❌ Use 4-Step Docs | ✅ 4-Step Docs |

**The 30-Second Test:**
> "If I turn off this new channel, does the core product still work 100%?"
> - **YES** → It's a bridge (use this skill)
> - **NO** → It's a feature (use 4-Step Docs)

### When NOT to Use (Anti-Patterns: Bridge vs Feature)

| Scenario | Why It's Not a Bridge | What to Use |
|----------|----------------------|-------------|
| "WhatsApp should have voice calls" | New feature, new UX | 4-Step Docs |
| "Email should create calendar events" | New integration (Google Calendar) | 4-Step Docs |
| "SMS should handle payments" | New execution layer (payment gateway) | 4-Step Docs |
| "Slack bot needs admin commands" | New permissions, new intents | 4-Step Docs + Security Auditor |
| "WhatsApp should work offline" | New architecture (PWA, local storage) | 4-Step Docs |

### When YES to Use

| Scenario | Why It's a Bridge |
|----------|------------------|
| "Add WhatsApp as input option" | Same IRCE, same flows, different transport |
| "Send email notifications for drafts" | Same triggers, different output format |
| "SMS alerts for urgent inquiries" | Same events, shorter message format |
| "Slack bot for team notifications" | Same data, different UI (channels vs DMs) |

### Core Principles

#### The 3-Change Rule

A true channel adapter requires **exactly 3 changes** — no more, no less:

```
┌─────────────────────────────────────────────────────────────┐
│  CHANGE 1: Input Layer                                      │
│  Different API/format → Normalize to IRCE Input             │
├─────────────────────────────────────────────────────────────┤
│  CHANGE 2: Identity Mapping                                 │
│  Channel ID → tenant_id (different lookup field)            │
├─────────────────────────────────────────────────────────────┤
│  CHANGE 3: Output Format                                    │
│  IRCE Response → Channel-specific formatting                │
└─────────────────────────────────────────────────────────────┘
```

**If you need more than 3 changes, it's not a bridge — it's a new vertical.**

#### The Bridge Metaphor

```
        [User A] ──Telegram──┐
                              ├──→ [IRCE Engine] ──→ [Castle Core]
        [User B] ──WhatsApp──┘
                              ↑
                         You are here
                         (Channel Adapter)
                              
        [User C] ──Email─────┐ (future bridge)
```

**The Castle doesn't care how you enter. The bridge just gets you there.**

---

## Part 2: Execution

### Quick Start

#### Prerequisites Checklist (Before Building)

- [ ] Verified IRCE works end-to-end in existing channel
- [ ] Verified M1-M9 pipeline processes photos correctly
- [ ] Identified the 3 changes (input, identity, output)
- [ ] Confirmed no new business logic needed
- [ ] Created Bridge Spec (1 document)
- [ ] Estimated < 4 hours implementation
- [ ] La Hermana can explain it in 1 sentence

### Implementation Patterns

#### The Bridge Spec (1-Document Template)

Create: `docs/bridge-{channel-name}.md`

##### Section 1: The Cable (Input Mapping)

Map the incoming data to IRCE Input format:

```markdown
| Source Field | IRCE Input Field | Transform |
|--------------|------------------|-----------|
| WhatsApp `from` | `userId` | Extract phone: `549123456789` |
| WhatsApp `body` | `text` | Passthrough |
| WhatsApp `imageMessage` | `photos` | Download + upload to temp |
| Email `envelope.from` | `userId` | Extract email |
| Email `subject` | `text` | `Subject: ${subject}` |
| Email `html` | `htmlBody` | Extract text via cheerio |
```

##### Section 2: The 3 Changes (Implementation)

**Change 1: Input Normalization**

```typescript
// lib/channels/{channel}/adapter.ts

interface WhatsAppMessage {
  from: string;        // "549123456789@s.whatsapp.net"
  body?: string;
  media?: WhatsAppMedia[];
}

function normalizeToIRCE(msg: WhatsAppMessage): IRCEInput {
  return {
    userId: normalizePhone(msg.from),     // "+549123456789"
    text: msg.body || '',
    photos: msg.media?.filter(m => m.type === 'image').map(m => m.url),
    voice: msg.media?.find(m => m.type === 'audio'),
    source: 'whatsapp',
    raw: msg // for debugging
  };
}
```

**Change 2: Identity Resolution**

```typescript
// Lookup tenant by channel-specific ID

// For WhatsApp (phone number)
const { data: user } = await supabase
  .from('telegram_users') // or user_phones table
  .select('tenant_id, tenant:tenants(*)')
  .eq('whatsapp_number', input.userId)
  .maybeSingle();

// For Email
const { data: user } = await supabase
  .from('user_emails')
  .select('tenant_id')
  .eq('email', input.userId)
  .maybeSingle();

// For SMS (same as WhatsApp — phone number)
```

**Change 3: Output Formatting**

```typescript
// Format IRCE Response for channel constraints

function formatForWhatsApp(response: IRCEResponse): WhatsAppMessage {
  return {
    text: `*${response.title}*\n\n${response.body}`,
    buttons: response.actions?.slice(0, 3).map(a => ({
      buttonId: a.id,
      buttonText: { displayText: a.label }
    }))
  };
}

function formatForEmail(response: IRCEResponse): EmailPayload {
  return {
    subject: response.title,
    html: emailTemplate(response), // HTML rich
    text: response.body // Plain text fallback
  };
}

function formatForSMS(response: IRCEResponse): SMSPayload {
  return {
    body: `${response.title}: ${response.body.slice(0, 140)}...`,
    link: response.shortUrl // Shortened URL
  };
}
```

##### Section 3: Reuse Checklist

Verify these are **unchanged** (100% reuse):

- [ ] IRCE Layer 2 (Intent Classification) — Regex + LLM
- [ ] IRCE Layer 3 (Resolution) — Fuzzy matching
- [ ] IRCE Layer 4 (Confirmation) — Sí/No logic
- [ ] IRCE Layer 5 (Execution) — Atomic transactions
- [ ] M1-M9 Pipeline — Photo processing, Kimi Vision
- [ ] Draft Workflow — Batch review, approval
- [ ] Error Handler — Same `api-error-handler`
- [ ] DB Schema — Same tables, maybe 1 new column

##### Section 4: The Connector (Minimal Service)

```typescript
// channel-connector.ts (Railway/VPS/Edge)
// This is the ONLY new runtime component

import { ChannelClient } from './channel-client';
import { normalizeToIRCE } from './adapter';
import { formatForChannel } from './formatter';

const CHANNEL = process.env.CHANNEL_TYPE; // 'whatsapp' | 'email' | 'sms'
const CASTLE_URL = process.env.CASTLE_API_URL;

const client = new ChannelClient({
  // Baileys, nodemailer, twilio, etc.
});

client.on('message', async (rawMessage) => {
  // 1. Normalize to IRCE
  const input = normalizeToIRCE(rawMessage);
  
  // 2. Send to Castle (same endpoint for all channels)
  const response = await fetch(`${CASTLE_URL}/api/webhook/${CHANNEL}`, {
    method: 'POST',
    headers: { 
      'Content-Type': 'application/json',
      'x-webhook-secret': process.env.WEBHOOK_SECRET
    },
    body: JSON.stringify(input)
  });
  
  const ircResponse = await response.json();
  
  // 3. Format for channel
  const reply = formatForChannel(ircResponse);
  
  // 4. Send back
  await client.send(rawMessage.from, reply);
});

// Health check
setInterval(() => {
  fetch(`${CASTLE_URL}/api/health`).catch(() => {
    console.error('Castle unreachable');
  });
}, 30000);
```

### Step-by-Step Guide

#### Step 1: Identify Channel Requirements

Analyze the new channel's:
- Authentication method
- Message format
- Media handling capabilities
- Rate limits
- Formatting constraints

#### Step 2: Create Input Adapter

Map channel-specific format to standardized IRCE Input.

#### Step 3: Implement Identity Resolution

Determine how to lookup tenant_id from channel-specific identifier.

#### Step 4: Create Output Formatter

Transform IRCE Response to channel-compatible format.

#### Step 5: Deploy Connector Service

Host the minimal connector (Railway, VPS, or Edge Function).

#### Step 6: Configure Webhook Endpoint

Add channel-specific route to Castle.

---

## Part 3: Quality Standards

### Pre-Delivery Checklist

#### Before Building Checklist

- [ ] Verified IRCE works end-to-end in existing channel
- [ ] Verified M1-M9 pipeline processes photos correctly
- [ ] Identified the 3 changes (input, identity, output)
- [ ] Confirmed no new business logic needed
- [ ] Created Bridge Spec (1 document)
- [ ] Estimated < 4 hours implementation
- [ ] La Hermana can explain it in 1 sentence

#### After Building Checklist

- [ ] 30-minute test passes (ping, identity, pipeline, response, error)
- [ ] Works alongside existing channel (Telegram still works)
- [ ] No changes needed to IRCE Layer 2-6
- [ ] No changes needed to M1-M9 pipeline
- [ ] Same error handling as original channel
- [ ] Documented in Bridge Spec

### Validation Commands / 30-Minute Test

- [ ] **Ping:** Send test message → appears in `system_logs`
- [ ] **Identity:** Message resolves to correct tenant
- [ ] **Pipeline:** Photo/message flows through IRCE
- [ ] **Response:** Reply received in channel, formatted correctly
- [ ] **Error:** Invalid user gets "activate your account" message

**If all pass → Bridge is operational.**

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "User not found" | Identity lookup failing | Check phone/email format normalization |
| "Message not delivered" | Output formatter issue | Verify channel constraints (length, format) |
| "IRCE timeout" | Pipeline stuck | Check M1-M9 pipeline health |
| "Webhook 500" | Schema mismatch | Validate input normalization |
| "Duplicate messages" | Idempotency missing | Add message deduplication |
| "Media not showing" | URL format issue | Check temp URL generation |

---

## Part 4: Technical Reference

### API Reference

#### IRCE Input Format

```typescript
interface IRCEInput {
  userId: string;           // Normalized identifier
  text: string;             // Message text
  photos?: string[];        // Photo URLs
  voice?: any;              // Voice message data
  source: string;           // Channel name
  raw: any;                 // Original message (debug)
}
```

#### IRCEResponse Format

```typescript
interface IRCEResponse {
  title: string;
  body: string;
  actions?: Array<{
    id: string;
    label: string;
  }>;
  shortUrl?: string;
}
```

#### Webhook Endpoint (Castle Side)

```typescript
// app/api/webhook/{channel}/route.ts
// One file per channel, minimal code

import { ircePipeline } from '@/lib/bot/irce-pipeline';
import { withErrorHandler } from '@/lib/api/errorHandler';

export const POST = withErrorHandler(async (req) => {
  const channel = req.url.split('/').pop(); // 'whatsapp' | 'email' | 'sms'
  
  // 1. Parse normalized input (from connector)
  const input = await req.json();
  
  // 2. Resolve tenant (channel-specific lookup)
  const tenantId = await resolveTenant(input.userId, channel);
  if (!tenantId) {
    return Response.json({ action: 'send_activation', userId: input.userId });
  }
  
  // 3. Route to IRCE (SAME for all channels)
  const result = await ircePipeline.process({
    ...input,
    tenantId,
    channel
  });
  
  // 4. Return formatted response
  return Response.json(result);
}, 'webhook/{channel}');
```

### Schema Changes (Minimal)

#### Option A: Extend Existing Table (MVP)

```sql
-- For WhatsApp/SMS (phone-based)
ALTER TABLE telegram_users 
ADD COLUMN IF NOT EXISTS whatsapp_number TEXT UNIQUE,
ADD COLUMN IF NOT EXISTS preferred_channel TEXT DEFAULT 'telegram'
  CHECK (preferred_channel IN ('telegram', 'whatsapp', 'sms', 'email'));

-- Index for fast lookup
CREATE INDEX IF NOT EXISTS idx_telegram_users_whatsapp 
ON telegram_users(whatsapp_number) 
WHERE whatsapp_number IS NOT NULL;
```

#### Option B: New Table (Multi-channel per user)

```sql
-- For users with multiple channels
CREATE TABLE IF NOT EXISTS user_channels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id) NOT NULL,
  user_id UUID REFERENCES telegram_users(id),
  channel_type TEXT CHECK (channel_type IN ('telegram', 'whatsapp', 'sms', 'email')),
  channel_id TEXT NOT NULL, -- phone, email, telegram_id
  is_primary BOOLEAN DEFAULT false,
  is_verified BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_user_channels_lookup 
ON user_channels(channel_type, channel_id);
```

**Decision:** Option A for first bridge (speed), Option B when adding 3+ channels.

### Integration with Other Skills

| Skill | Relationship |
|-------|-------------|
| `irce-engineer` | Reuses Layer 2-6 (no changes) |
| `m1-m9-pipeline` | Reuses photo processing (no changes) |
| `api-error-handler` | Same error handling |
| `db-guardian` | Minimal schema changes (1 column or table) |
| `4-step-docs` | Alternative when NOT a bridge |
| `feature-expert` | Check if feature exists before bridging |

### Constraints

1. **Never modify IRCE core** — If Layer 2-6 needs changes, it's not a bridge
2. **Never modify M1-M9** — If photo pipeline needs changes, it's not a bridge
3. **Max 1 new table** — If you need 2+ tables, use 4-Step Docs
4. **Max 4 hours implementation** — If it takes longer, scope is wrong
5. **Must coexist** — New channel cannot break existing ones

### Examples from the Castle

#### ✅ WhatsApp Bridge (Real Example)

```yaml
Channel: WhatsApp
Library: @whiskeysockets/baileys
Hosting: Railway ($5/mes)

Change 1 - Input:
  WhatsAppMessage → IRCEInput
  - from: "549123456789@s.whatsapp.net"
  - body: "vendí 2 jeans"
  - imageMessage: [...]

Change 2 - Identity:
  Lookup: telegram_users.whatsapp_number
  Fallback: Send "ACTIVAR CÓDIGO" message

Change 3 - Output:
  IRCE Response → WhatsApp format
  - Markdown: *bold* only (no _italic_)
  - Buttons: Max 3 reply buttons
  - Media: One image per message

Reuse: 100%
  - Same IRCE Layer 2-6
  - Same M1-M9 pipeline
  - Same draft workflow
  - Same error handling
```

#### 🔄 Email Bridge (Future Example)

```yaml
Channel: Email
Library: nodemailer + IMAP
Hosting: Zapier/n8n or Railway

Change 1 - Input:
  Email envelope → IRCEInput
  - from: "corredor@email.com"
  - subject: "Nueva propiedad Ñuñoa"
  - html: "<p>Depto 3D2B...</p>"

Change 2 - Identity:
  Lookup: user_emails.email
  - Extract from: header
  - Validate SPF/DKIM

Change 3 - Output:
  IRCE Response → Email template
  - HTML rich with property cards
  - "Reply to approve" functionality

Reuse: 95%
  - Same IRCE (new intent: EMAIL_FORWARD)
  - Same draft workflow
```

#### 🔄 SMS Bridge (Future Example)

```yaml
Channel: SMS
Library: Twilio
Hosting: Vercel Edge (stateless)

Change 1 - Input:
  Twilio webhook → IRCEInput
  - From: "+56912345678"
  - Body: "BUSCAR NUNOA" (160 char max)

Change 2 - Identity:
  Lookup: Same as WhatsApp (phone number)

Change 3 - Output:
  IRCE Response → SMS
  - Truncate to 160 chars
  - Shortened links (bit.ly)
  - "Reply 1 for details"

Reuse: 95%
  - Same IRCE (short commands)
  - Search only (no photos)
```

---

## Information Gaps — Catastro

**Esta sección documenta información que falta o necesita ser completada:**

1. **Canales adicionales probados**: Solo WhatsApp tiene ejemplo real documentado. Email y SMS son teóricos.

2. **Detalles de hosting específicos**: Falta documentación detallada sobre configuración de Railway, Vercel Edge, etc.

3. **Manejo de errores específicos por canal**: No hay ejemplos de cómo manejar rate limits, bloqueos, o reconexiones.

4. **Testing automatizado**: Falta información sobre cómo escribir tests para channel adapters.

5. **Monitoreo y observabilidad**: No se documenta cómo monitorear la salud de múltiples canales.

6. **Migración de usuarios entre canales**: Falta guía sobre cómo permitir que usuarios cambien de canal preferido.

---

*Channel Adapter: The minimalist methodology for maximal reuse.*
*Channel Adapter v2.0 — Formato 4-Partes*
