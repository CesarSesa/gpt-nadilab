---
name: 4-step-docs
description: "Pre-coding 4-document framework for new products/features: PRD, App Flow, Design Doc, Backend Doc"
trigger: "new product, new feature, start project, plan feature, PRD, product requirements, app flow, design doc, backend doc, 4 docs, 4 step, pre-coding docs"
---

# 4-Step Product Documentation Framework

Before writing ANY code for a new product or major feature, generate these 4 documents sequentially. Each document must be reviewed and approved by the user before proceeding to the next.

---

## 1. Identificación

**¿Qué es este skill?**
Framework de 4 documentos pre-código que deben existir antes de escribir cualquier implementación para un nuevo producto o feature.

**¿Cuándo usarlo?**
- Cuando el usuario dice: "new product", "new feature", "start project", "plan feature"
- Cuando se menciona PRD, product requirements, app flow, design doc, backend doc
- Antes de escribir cualquier código de implementación

**Contexto esperado:**
- El usuario quiere crear un nuevo producto o feature
- No existen los 4 documentos aún (o están incompletos)
- Se necesita planificación antes de implementación

---

## 2. Checklist de Ejecución

### Step 1: Product Requirements Document (PRD)

- [ ] Crear archivo: `docs/1-PRD.md`
- [ ] Incluir: Product name and one-line description
- [ ] Incluir: Problem statement — what pain does this solve?
- [ ] Incluir: Target users — who uses this and in what context?
- [ ] Incluir: Core features — bulleted list, prioritized (P0/P1/P2)
- [ ] Incluir: Non-goals — what this product explicitly does NOT do
- [ ] Incluir: Success criteria — how do we know it works?
- [ ] Incluir: Constraints — timeline, tech limitations, dependencies
- [ ] Preguntar al usuario: "Review the PRD. Any changes before I move to App Flow?"
- [ ] NO proceder al siguiente paso sin aprobación explícita del usuario

### Step 2: App Flow Document

- [ ] Crear archivo: `docs/2-APP-FLOW.md`
- [ ] Incluir: Entry points — how does the user arrive? (URL, redirect, link, etc.)
- [ ] Incluir: Screen-by-screen flow — every screen the user sees, in order
- [ ] Incluir: User interactions — every button, input, toggle, gesture, and what it triggers
- [ ] Incluir: State transitions — what changes in the app state at each step
- [ ] Incluir: Edge cases — empty states, errors, loading, permissions denied
- [ ] Incluir: Navigation map — how screens connect (can be ASCII diagram)
- [ ] Preguntar al usuario: "Review the App Flow. Any changes before I move to Design?"
- [ ] NO proceder al siguiente paso sin aprobación explícita del usuario

### Step 3: Design Document

- [ ] Crear archivo: `docs/3-DESIGN.md`
- [ ] Incluir: Visual style — reference the ui-ux-pro-max skill if available, or ask user for style preference
- [ ] Incluir: Color palette — primary, secondary, accent, semantic colors (success/error/warning)
- [ ] Incluir: Typography — font pairing, sizes for headings/body/captions
- [ ] Incluir: Component inventory — list every UI component needed (buttons, cards, modals, forms, tables, etc.)
- [ ] Incluir: Layout structure — per screen, describe the layout (sidebar + main, full-width, grid, etc.)
- [ ] Incluir: Responsive behavior — how it adapts to mobile/tablet/desktop
- [ ] Incluir: Reference screenshots — ask user if they have any visual references
- [ ] Preguntar al usuario: "Review the Design Doc. Any changes before I move to Backend?"
- [ ] NO proceder al siguiente paso sin aprobación explícita del usuario

### Step 4: Backend Document

- [ ] Crear archivo: `docs/4-BACKEND.md`
- [ ] **Section A — Tech Stack & Frameworks:**
  - [ ] Framework (Next.js, React, etc.)
  - [ ] Database (Supabase, Postgres, etc.)
  - [ ] Auth method
  - [ ] Hosting/deployment
  - [ ] External APIs or services
- [ ] **Section B — Backend Structure:**
  - [ ] Database schema — tables, columns, types, relationships (use SQL or markdown tables)
  - [ ] Row Level Security (RLS) — policies per table
  - [ ] API routes / Edge functions — endpoint, method, purpose, auth required
  - [ ] Data flow — how data moves from UI → API → DB and back
  - [ ] Third-party integrations — webhooks, OAuth, external APIs
- [ ] Preguntar al usuario: "Review the Backend Doc. All 4 docs are complete — ready to start coding?"
- [ ] NO iniciar implementación sin aprobación explícita del usuario

### Rules (globales)

- [ ] NEVER skip a step. All 4 documents must exist before writing implementation code.
- [ ] NEVER proceed to the next document without explicit user approval.
- [ ] Write documents to files, not inline. This preserves them for future sessions and agent delegation.
- [ ] If the project already has some of these docs, read them first and update rather than overwrite.
- [ ] These docs become the source of truth for any agent (Kimi swarm, subagents) that works on this feature.

---

## 3. Information Gaps — Catastro

Antes de comenzar, verificar si falta información crítica que bloquee el proceso:

| Gap | Impacto | Mitigación |
|-----|---------|------------|
| No hay claridad sobre el problema que resuelve el producto | 🔴 CRÍTICO - El PRD será vago | Preguntar: "¿Qué problema específico resuelve esto? ¿Para quién?" |
| El usuario no sabe quién es el usuario target | 🔴 CRÍTICO - No se puede definir el flujo | Preguntar: "¿Quién usará esto? ¿En qué contexto/situación?" |
| No hay priorización de features (P0/P1/P2) | 🟡 MEDIO - Scope puede crecer infinitamente | Forzar priorización: "¿Cuáles son los 3 features imprescindibles para el MVP?" |
| No hay preferencias de tech stack | 🟡 MEDIO - No se puede completar el Backend Doc | Preguntar stack por defecto o proponer uno basado en el contexto |
| El proyecto ya tiene documentos parciales | 🟢 BAJO - Riesgo de sobreescribir | Leer documentos existentes primero, actualizar en lugar de reemplazar |
| El usuario quiere saltarse un paso | 🔴 CRÍTICO - Rompe el framework | Recordar: "Nunca saltamos pasos. Los 4 docs deben existir antes del código." |
| No hay referencias visuales para el diseño | 🟡 MEDIO - Diseño puede no alinearse con expectativas | Pedir screenshots, links, o describir estilo deseado (minimalista, corporativo, etc.) |

---

## 4. Output Definition

**Archivos generados:**
1. `docs/1-PRD.md` — Product Requirements Document
2. `docs/2-APP-FLOW.md` — App Flow Document  
3. `docs/3-DESIGN.md` — Design Document
4. `docs/4-BACKEND.md` — Backend Document

**Criterios de éxito:**
- Los 4 archivos existen en el directorio `docs/`
- Cada documento tiene todos sus campos requeridos completados
- El usuario ha aprobado explícitamente cada documento antes de pasar al siguiente
- Los documentos son guardados en archivos, no inline
- Los documentos se convierten en source of truth para cualquier agente que trabaje en el feature

**Próximos pasos después de este skill:**
- Una vez aprobados los 4 docs, proceder a la implementación del código
- Cualquier agente (Kimi swarm, subagents) debe referirse a estos documentos
- Si hay cambios durante implementación, actualizar los documentos correspondientes
