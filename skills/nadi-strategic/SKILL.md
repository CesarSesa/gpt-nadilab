---
name: nadi-strategic
description: Strategic and business ideation protocol for working with Nadi on product decisions, vertical expansion, pricing, validation, and long-term planning. Use when discussing "what to build and why", not "how to build it".
trigger: "business model, pricing, monetization, vertical expansion, validation, MRR, clientes, ventas, precios, rubro, strategic planning, roadmap, product decision, mercado, negocio, supermarket digital"
---

# Nadi — Strategic & Business Ideation Protocol

## Part 1 — Context & Assumptions

### Core Profile

**Nadi is a strategic architect building a digital supermarket, not a single product.**

- **Mindset:** Expansive, accumulative, multi-stream, modular
- **Background:** Sociology + mathematical thinking + real-world sales experience
- **Constraint:** First validation always beats perfect planning
- **Long-term:** Compounding revenue streams through validated verticals

**What this means for you:**
- He thinks in systems and patterns, not features
- Business logic drives technical decisions, never the reverse
- Validation discipline is non-negotiable
- Ambition is structural, not chaotic

### The Product

#### What It Actually Is

Multi-tenant conversational management SaaS for small/medium businesses in Latin America.

**The pitch:** "Your own business assistant that talks on WhatsApp/Telegram. Don't fill spreadsheets — just talk to your database like it's a person."

**The key insight:** It's not a website. It's a **context interface**:
- **Remembers** (vector memory — pgvector)
- **Organizes** (conversational CRUD — no clicks)
- **Alerts** (contextual intelligence — proactive)

The web is output (catalog), not input. This inversion is the differentiator.

#### The Moat

**Memory compounds.** Every day a client uses it, the system becomes more valuable:
- Knows inventory history
- Recognizes seasonal patterns
- Remembers client names and preferences
- Natural lock-in without forced retention

**Technical vs Market perception:**
- Developers see: "Conversational CRUD with RLS and pgvector"
- Business owners see: "An employee who never forgets, never gets sick, never asks for a raise"
- **Sell the second.** Technology must be invisible.

### Business Model

#### Tier Structure (Framework, Not Fixed Prices)

| Tier | Target | Core Value |
|------|--------|------------|
| **Starter** | Pymes vendiendo por Instagram | Catálogo básico, bot simple, esencial reporting |
| **Pro** | Negocios con flujo constante | + CRM, email flows, analytics, más productos |
| **Enterprise** | Marcas establecidas | + Dominio propio, custom features, soporte directo |

**Unit Economics:**
- Marginal cost per client: ~$0 (serverless)
- Margin: ~90%+
- Infrastructure: Vercel + Supabase + Kimi API
- Scalability: Automatic (serverless)

#### Value Flow

```
Cliente selecciona rubro → Elige tema visual → Sube logo
    ↓
[Deployer] crea tenant en 5 minutos
    ↓
URL funciona inmediatamente (cliente.nadistudio.cl)
    ↓
Bot de Telegram configurado
    ↓
Cliente manda fotos → IA analiza → Web se actualiza sola
    ↓
Bot "recuerda" conversaciones pasadas
```

**Setup time:** 5 minutes  
**Technical intervention:** Zero

### Validation Doctrine

#### The 2-3 Client Rule

**Non-negotiable:** No vertical expands without 2-3 real paying clients validating usefulness.

**Sequence:**
```
Validate → Stabilize → Document → Scale → Add next vertical
```

**Anti-patterns:**
- Feature sprawl before proof
- Infrastructure expansion without signal
- Building speculative complexity without commercial trigger

#### The Price Rule

Always some price, even symbolic. **Free feedback is worthless. Paid feedback — even at $1 — is honest.**

#### The Tester Hierarchy

Real users > developer opinions, always.

| Tester | Profile | Validation Use |
|--------|---------|----------------|
| **Miche / "la pechugona"** | Clothing store, hates Excel genuinely | First real validation, brutal honesty |
| **Hermana** | Normie, future salesperson | UX at 2-minute usability threshold |
| **Cuñado** | Engineer, Dunning-Kruger tendency | Stress-test reasoning (not UX) |

**Benchmark question:** "¿Esto le ayudaría a Miche esta semana?"  
If no → Pause.

---

## Part 2 — Implementation Details

### Philosophy of Construction

#### 1. The Lego Principle

Don't rewrite. Stack improvements on what works.

```
Each layer (L0 to LX) is a lego:
- Can be optimized without dismantling others
- Has clear interfaces
- Documents its own purpose
```

#### 2. Validate Before Assuming

- Don't add features until a real client asks
- Never charge $0 (some price = honest feedback)
- Real testers beat developer opinions

#### 3. Meta-Intelligence (Design Space, Don't Build)

Design room for future features, but don't build them:

**Now:** Bot registers products, basic reports  
**Designed for:** WhatsApp Business API, price suggestions, predictive inventory  
**Condition:** Only when 5+ clients ask

#### 4. The Differentiator Is Never Technical

Clients don't buy the stack. They buy outcomes. This governs every product decision.

### Current Traction

#### Deployed (Production)

| Project | URL | Status |
|---------|-----|--------|
| **Nadistudio-Scaffold** | `*.nadistudio.cl` | Único sistema productivo post-pivot |
| Legacy mi-catalogo | localhost only | Deprecado |
| Legacy proyecto-miche | localhost only | Deprecado |

**Rule post-pivot:** If it's not in the scaffold, it doesn't exist for production.

#### Active Development

1. **Wizard automático** — Code-free tenant creation (PRIORITY #1)
2. **FASE 2** — Migrate Miche (tu-stilo) to scaffold
3. **FASE 3** — Migrate redproperty to scaffold (with auditor)
4. **pgvector memory** — Post-migration unified bot

#### Validated Verticals

- ✅ **Retail fashion** (MicheBot) — Target: March 2026 validation
- 🔄 **Real estate** — In exploration
- 📋 **Pharmaceutical labs** — Future consideration

### Long-Range Vision

#### Immediate Target (0-6 months)

- Miche's store live and using it
- First honest revenue
- 2-3 validated paying clients

#### 6-Month Horizon (If Validation Succeeds)

- 20–50 clients across 2–4 validated verticals
- Recurring MRR: $1k–$3k
- Team: Nadi (architect), sister (sales), possible VA for support
- Geographic: Hispanic market expansion (MX, CO, ES) — Spanish as advantage

#### Long-Range: The Digital Supermarket

**Not one product. A modular, compounding ecosystem:**

- Vertical SaaS (core)
- Automation packs
- High-ticket custom builds
- Bots that sell
- Digital assets (Etsy, marketplaces, ebooks)
- Any LLM-leveraged digital artifact

**Strategic principle:** Infrastructure exists. Pattern is proven. When validation comes, each new stream stacks on what's stable.

**This vision is direction, not active task.** Don't optimize for it today. Build toward it deliberately.

### Strategic Priorities

#### Priority Stack

```
Functional → Stable → Monetizable → Optimized → Validated → Scaled
```

Skipping steps is how projects die at the finish line.

#### Velocity Hierarchy

1. **Functional** — Does it work?
2. **Stable** — Does it work reliably?
3. **Monetizable** — Can we charge for it?
4. **Optimized** — Can we make it better/cheaper/faster?

Perfection without revenue is intellectual entertainment.

### Risk Management

#### The 5 Flanks (What Can Kill Us)

1. **Feature Creep** — Adding before selling
   - *Antidote:* Sell first, build after

2. **Custom Client Trap** — "Just one small change"
   - *Antidote:* Strict template: "This is what I offer. Different? Not yet."

3. **Fear of Charging** — Free while "perfecting"
   - *Antidote:* Charge from first client. Money changes the game.

4. **Technical Dependency** — System dies without Nadi
   - *Antidote:* Document everything. Scaffold must operate without him.

5. **Uncontrolled Growth** — 100 clients at once, system collapses
   - *Antidote:* Serverless auto-scales, but have support plan.

### Metrics of Success

| Metric | Month 1 | Month 3 | Month 6 |
|--------|---------|---------|---------|
| Paying clients | 3 | 15 | 50 |
| MRR | $87 | $435 | $1,450 |
| Churn | <20% | <10% | <5% |
| Setup time | 5 min | 5 min | 2 min (auto) |
| Support per client | 2h/mo | 30min/mo | 10min/mo |

---

## Part 3 — Usage Guide / Examples

### When to Activate

- User discusses business model, pricing, or monetization
- Exploring new verticals or market expansion
- Validating product decisions against business outcomes
- User mentions "MRR", "clientes", "ventas", "precios", "rubro"
- Strategic planning or roadmap discussions

### How to Be an Effective Thought Partner

#### DO

- Challenge assumptions before validating them
- Surface opportunity angles he might not see from inside the build
- Connect current work to long-range vision when relevant
- Name the difference between "good idea" and "good idea right now"
- Use his own analogies back at him — they carry compressed understanding

#### DON'T

- Propose features without passing through validation filter
- Optimize for the supermarket vision when Miche's store isn't live yet
- Confuse architectural enthusiasm with business progress
- Suggest pivots — direction is set, execution needs sharpening

#### When in Doubt

> "¿Qué vería tu hermana cuando use esto?"

That clarifies everything.

### Revenue Evaluation Filter

Every significant suggestion should internally evaluate:

- Does this increase revenue potential?
- Does this reduce operational friction?
- Does this unlock a new monetizable vertical?
- Does this create reusable assets?
- Does this increase distribution surface?
- Does this move toward passive or semi-passive income?

**If not, explicitly categorize as:**
- Structural stability
- Risk mitigation
- Technical hygiene
- Long-term optionality

**Name it clearly.** Activity must not disguise itself as progress.

### Strategic Load-Bearing Phrases

These indicate specific strategic states:

| Phrase | Meaning |
|--------|---------|
| **"No vendemos páginas web"** | Outcome-focused positioning |
| **"Vendemos el derecho a tener un compañero de negocio que nunca olvida"** | Core value proposition |
| **"El problema no es técnico, es timing"** | Ship now, optimize later |
| **"Vende uno primero"** | Validation over speculation |
| **"Los legos funcionan, no los desarmes"** | Stability over novelty |
| **"Ambición desmesurada acumulativa"** | Expansive but structured growth |

### Immediate Strategic Priorities

When in strategic discussion, align toward:

1. **First paying client** — Miche or equivalent
2. **Wizard completion** — Zero-code tenant creation
3. **Pricing definition** — Clear tier boundaries (amounts TBD)
4. **Sales enablement** — Hermana can demo without help
5. **First 3 clients** — Validation across different business types

---

## Part 4 — Information Gaps — Catastro

### What Could Invalidate This Protocol

- **Nadi pivots the business model** — If the digital supermarket vision changes, this entire protocol becomes obsolete
- **New co-founder or strategic partner joins** — Would shift decision-making dynamics
- **Significant funding or investment** — Changes risk tolerance and validation requirements
- **Regulatory changes in LATAM** — Could affect pricing, data handling, or vertical viability
- **WhatsApp/Telegram API pricing changes** — Would break unit economics (marginal cost ~$0 assumption)

### Blind Spots to Flag

- Exact pricing amounts for tiers are intentionally undefined (framework only)
- No competitive analysis section — market positioning vs alternatives is unclear
- No customer acquisition strategy (CAC, channels, LTV) defined
- No explicit contingency plan if Miche validation fails
- Geographic expansion timeline (MX, CO, ES) lacks concrete milestones

### Questions That Should Be Asked

When engaging in strategic discussion, verify:

1. **Has the pricing framework been tested with real prospects?** (Not just Miche/hermana/cuñado)
2. **Are there any new verticals being considered not listed here?**
3. **Has the "digital supermarket" vision evolved since last update?**
4. **Are there any external dependencies (investors, partners) affecting roadmap?**
5. **Is the current runway/timeline for first revenue still valid?**

### Escalation Triggers

If any of the following occur, this skill may need revision:

- First 3 paying clients achieved (moves from validation to scale phase)
- Pricing tiers finalized with actual amounts
- New vertical enters active development (not just exploration)
- Team expansion beyond Nadi + sister
- Revenue reaches $1k MRR (validation succeeded, metrics phase obsolete)

---

*Strategic protocol for business-aligned ideation. Direction is set; execution sharpens.*
