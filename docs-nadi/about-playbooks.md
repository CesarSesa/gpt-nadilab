# About Playbooks — Consolidation Plan

> Last updated: 2026-03-26
> Status: Planning phase — mapping existing docs to final 9 playbooks

---

## The 9 Playbooks (Final Structure)

Consolidation from 13 scattered docs + 2 new additions → 9 focused playbooks.

| # | Playbook | Focus Area | Order Logic |
|---|----------|------------|-------------|
| 1 | Vertical Lego: Multi-Tenant Expansion | Architecture | Foundations |
| 2 | Database Architecture & Schema Evolution | Data | Foundations |
| 3 | Media Pipeline: Ingestion to Storage | Workflow | Core flows |
| 4 | Drafts & Publishing Workflow | Workflow | Core flows |
| 5 | Conversational Interface: Bots & IRCE | Interface | User-facing |
| 6 | API Design & Backend Patterns | Interface | Developer-facing |
| 7 | Product Features & Tier Strategy | Business | Strategy |
| 8 | Onboarding & First Value Activation | Operations | NEW |
| 9 | Incident Response & 3 AM Runbook | Operations | NEW |

---

## Existing Material → Playbook Mapping

### Playbook 1: Vertical Lego
**Status: ~80% written**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `00.vertical-lego-template-v2.1.md` | 15KB | Primary source — the current template |
| `00.vertical-lego-template.v2.0` | 9KB | Deprecate — v2.1 supersedes |
| `legos v3 posibilidad.txt` | 13KB | Cherry-pick ideas, rest is exploration |

**Linked skill:** `vertical-factory`
**Action:** Light editing — v2.1 is solid, add cross-refs to Playbooks 2-5.

---

### Playbook 2: Database Architecture
**Status: ~60% written (scattered)**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `final-state-unified-db.md` | 9KB | DB snapshot — schema reference |
| Sections from `Playbook-scaffold-master-patterns-and-gotchas.md` | ~20KB est. | DB-related sections (ghost columns, RLS, naming) |

**Linked skills:** `db-guardian`, `migration-generator`
**Action:** Extract DB sections from the 101KB mega-playbook + unified-db into one coherent doc. Add schema evolution patterns, migration safety checklist.

---

### Playbook 3: Media Pipeline
**Status: ~90% written**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `00. About-product-batches-and-architecture-playbook.md` | 13KB | Batch architecture, dual-write, storage paths |
| `Playbook-two-phase-async-processing.md` | 8KB | Two-phase pattern (accumulate → process) |

**Linked skill:** `m1-m9-pipeline`
**Action:** Merge both docs. The m1-m9-pipeline skill (22KB) is actually MORE complete than these — consider the skill as the canonical source and this playbook as the "why + decisions" companion.

---

### Playbook 4: Drafts & Publishing
**Status: ~85% written**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| Parts from `About-product-batches` | ~5KB est. | Draft lifecycle, approval flows |

**Linked skill:** `draft-workflow-pipeline` (1,248 lines — very complete)
**Action:** The merged skill IS the playbook essentially. Write a thin playbook that covers the "why" (high-ticket vs low-ticket model differences, Fashion N:1 vs Real Estate 1:N) and links to the skill for implementation.

---

### Playbook 5: Conversational Interface (Bots & IRCE)
**Status: ~95% written (most material exists, needs merge)**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `00.Playbook-IRCE-pipeline-for-verticals.md` | 16KB | Primary IRCE playbook — excellent |
| `Playbook-bot-architecture-and-vertical-expansion.md` | 15KB | Bot architecture + expansion patterns |
| `IRCE-cross-llm-compilation-by-antigravity.md` | 20KB | Multi-LLM compilation — historical/reference |
| `IRCE-transplant-plan-from-proyecto-miche.md` | 13KB | File-by-file transplant plan — mostly done |

**Linked skill:** `irce-engineer` (enhanced, 400 lines)
**Action:** Merge the two main playbooks (IRCE + bot-architecture). The transplant plan and cross-LLM compilation become appendices or move to `past-history/`.

---

### Playbook 6: API Design & Backend Patterns
**Status: ~70% written (needs extraction from mega-doc)**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `Playbook-scaffold-master-patterns-and-gotchas.md` | 101KB | The mega-doc — extract API/patterns sections |
| `qué hacer con el error y los logs.txt` | 12KB | Error handling philosophy, logging patterns |

**Linked skill:** `api-error-handler`
**Action:** This is the hardest consolidation. The 101KB mega-playbook needs to be SPLIT — DB sections go to Playbook 2, API/patterns sections stay here, bot sections go to Playbook 5. What remains becomes this playbook + the "gotchas" appendix.

---

### Playbook 7: Product Features & Tier Strategy
**Status: ~40% written**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `Scaffold-Improvements.md` | 8KB | Feature roadmap (needs updating) |
| `possibilities-about-generating-webs-for-the-future.md` | 8KB | Vision for web generation |

**Linked skill:** `feature-expert`
**Action:** Needs significant new writing — tier strategy (Starter/Pro/Enterprise), per-rubro feature matrix, brandization capabilities. Directly relevant to current BRANDIZATION focus.

---

### Playbook 8: Onboarding & First Value Activation
**Status: NEW — needs writing**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `Post IRCE planes, Groq, comandos y pgvector.txt` | 5KB | Some UX flow ideas |

**Action:** Write from scratch. Focus: `/start CODIGO` → first product published in < 5 min. Per-rubro activation flows. Re-engagement triggers. Directly relevant to BRANDIZATION focus.

---

### Playbook 9: Incident Response & 3 AM Runbook
**Status: NEW — needs writing**

| Source doc | Size | What to keep |
|-----------|------|-------------|
| `For refactoring legacy projects.txt` | 9KB | Some debugging patterns |
| M1-M9 debugging decision tree | In skill | Reusable triage pattern |

**Action:** Write from scratch. Triage flowchart, rollback procedures, "qué decirle a Miche cuando todo está en llamas". The m1-m9-pipeline debugging tree is a great starting template for structured triage.

---

## Docs NOT mapped to any playbook

These live outside the 9-playbook system:

| Doc | Disposition |
|-----|-------------|
| `nadistudio-castle-v8.0-opus-update.md` | Constitutional doc — stays as-is |
| `SOUL-v2.0-operational-state.md` | Operational state — stays as-is |
| `ARCHITECTURE-RULES.md` | Reference — stays, update stale sections |
| `AGENTS.md` | Agent coordination — stays as-is |
| `5-ideas-to-MCP.md` | MCP exploration — stays as-is |
| `Nadistudio-mcp.md` | MCP reference — stays as-is |
| `about-different-skills.md` | Skills meta-doc — stays as-is |
| `about-ethnographies-of-artificial-cognition-and-future-ideas.md` | Meta/philosophy — stays in 00.claude/ |
| `for-opus-v2-robots-edition.md` | LLM coordination — stays in 00.claude/ |
| `1. Las guidelines del mage.txt` | Kimi guidelines — stays in 00.claude/ |
| `Audit-inmobiliario-2026-03-15.md` | Snapshot — move to `past-history/` |
| `Audit-retail-2026-03-15.md` | Snapshot — move to `past-history/` |
| `inconsistencias-*.md` | Audit artifacts — actions extracted, then archive |
| `00. Future-pgvector-memory-system.md` | Future reference — linked from Playbook 5 |

---

## The 101KB Elephant: Splitting the Mega-Playbook

`Playbook-scaffold-master-patterns-and-gotchas.md` (73 sections, 101KB) needs to be decomposed:

| Sections about... | Goes to |
|-------------------|---------|
| DB patterns, ghost columns, RLS, naming | Playbook 2 (Database) |
| Storage, uploads, image paths | Playbook 3 (Media Pipeline) |
| Draft lifecycle, approval | Playbook 4 (Drafts) |
| Bot patterns, webhooks, Telegram | Playbook 5 (Bots & IRCE) |
| API design, error handling, anti-patterns | Playbook 6 (API & Backend) |
| Everything else (gotchas, cross-cutting) | Playbook 6 appendix |

This is the single biggest consolidation task. Estimated effort: 3-4 hours with LLM assistance.

---

## Priority Order (aligned with Nadi's current focus)

### Phase A: Brandization & Polish (current focus)
1. **Playbook 7** — Features & Tiers (write tier strategy, brandization capabilities)
2. **Playbook 8** — Onboarding (write activation flows per rubro)
3. **Playbook 4** — Drafts (thin wrapper, skill does the heavy lifting)

### Phase B: Infrastructure Consolidation
4. **Playbook 6** — API & Backend Patterns (extract from mega-doc)
5. **Playbook 2** — Database Architecture (extract from mega-doc)
6. **Playbook 1** — Vertical Lego (light edit of v2.1)

### Phase C: Bot & Pipeline
7. **Playbook 5** — Bots & IRCE (merge two existing docs)
8. **Playbook 3** — Media Pipeline (merge two docs + link to skill)

### Phase D: Operations
9. **Playbook 9** — Incident Response (write from scratch)

---

## Playbook ↔ Skill Cross-Reference

| Playbook | Primary Skill(s) |
|----------|-----------------|
| 1. Vertical Lego | `vertical-factory` |
| 2. Database | `db-guardian`, `migration-generator` |
| 3. Media Pipeline | `m1-m9-pipeline` |
| 4. Drafts | `draft-workflow-pipeline` |
| 5. Bots & IRCE | `irce-engineer` |
| 6. API & Backend | `api-error-handler`, `security-auditor` |
| 7. Features & Tiers | `feature-expert` |
| 8. Onboarding | (no skill yet) |
| 9. Incident Response | (no skill yet — could become `incident-responder`) |

---

## nadi.cafe Separation (Relevant to Playbooks 8 & 9)

The planned separation of `/ops` and `/wizard` to nadi.cafe affects:
- **Playbook 8** (Onboarding) — wizard flows need to document which domain they live on
- **Playbook 9** (Incident Response) — ops monitoring lives on nadi.cafe, runbooks need to reference the correct domain
- **Playbook 1** (Vertical Lego) — tenant creation wizard moves to nadi.cafe

This separation should be documented BEFORE writing Playbooks 8 and 9.
