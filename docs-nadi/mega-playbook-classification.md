# Mega-Playbook Classification Index

> **Source:** `Playbook-scaffold-master-patterns-and-gotchas.md` (101KB, 73 sections)
> **Date:** 2026-03-26
> **Classified by:** kimi-docs

---

## Summary

| Target Playbook | Sections | Confidence |
|-----------------|----------|------------|
| 1. Vertical Lego | 2 | High |
| 2. Database Architecture | 8 | High |
| 3. Media Pipeline | 8 | High |
| 4. Drafts & Publishing | 6 | High |
| 5. Bots & IRCE | 12 | High |
| 6. API Design & Backend | 24 | High |
| 7. Product Features & Tiers | 2 | Medium |
| 8. Onboarding & Activation | 3 | High |
| 9. Incident Response | 4 | High |
| **Cross-cutting / Appendix** | 4 | N/A |

---

## Classification Table

| # | Section Title | Target Playbook | Confidence | Notes |
|---|---------------|-----------------|------------|-------|
| 1 | Tenant Resolution — The Three Tiers | 6. API & Backend | High | Core routing pattern |
| 2 | getUserTenant() — Cross-Rubro Isolation | 5. Bots & IRCE | High | Bot security pattern |
| 3 | Image Systems — Legacy vs Future | 3. Media Pipeline | High | Dual architecture |
| 4 | Two-Phase Async Processing | 3. Media Pipeline | High | Core pipeline pattern |
| 5 | Server/Client Component Split | 6. API & Backend | High | React pattern |
| 6 | Admin Auth — Cookie-Based | 6. API & Backend | High | Auth implementation |
| 7 | Bot Webhook — Message Processing Order | 5. Bots & IRCE | High | Bot routing |
| 8 | Bot Webhook — Media Accumulation | 3. Media Pipeline | High | Queue pattern |
| 9 | Bot Webhook — Callback Queries | 5. Bots & IRCE | High | Bot UI pattern |
| 10 | Bot Webhook — Idempotence | 6. API & Backend | High | API safety pattern |
| 11 | Theme System | 6. API & Backend | Medium | UI/theming |
| 12 | Supabase Clients — Which One When | 2. Database | High | DB client pattern |
| 13 | LX Tenant Resolver | 6. API & Backend | High | API routing |
| 14 | Status Values — The Source of Many Bugs | 4. Drafts & Publishing | High | State management |
| 15 | WhatsApp URL Construction | 6. API & Backend | Medium | Integration pattern |
| 16 | Entity Images — No FK | 3. Media Pipeline | High | DB design quirk |
| 17 | Vercel Limits & maxDuration | 6. API & Backend | High | Infrastructure |
| 18 | Danger Zones — Known Weak Points | 9. Incident Response | High | Risk catalog |
| 19 | IRCE Pipeline — What's Missing | 5. Bots & IRCE | High | Intent engine |
| 20 | Phase Boundaries — Migration Markers | 1. Vertical Lego | Medium | Migration planning |
| 21 | Kimi API Configuration | 5. Bots & IRCE | High | LLM settings |
| 22 | Soft Delete vs Hard Delete | 4. Drafts & Publishing | High | Data lifecycle |
| 23 | Draft Button Actions — Complete Map | 4. Drafts & Publishing | High | UI actions |
| 24 | Session Routing | 5. Bots & IRCE | High | Bot conversation |
| 25 | Entity Resolver — Fuzzy Matching | 5. Bots & IRCE | High | NLP matching |
| 26 | Confirmation Builder — Permissive Parser | 5. Bots & IRCE | High | UX pattern |
| 27 | Operation Executor — Atomicity | 6. API & Backend | High | Transaction safety |
| 28 | Multi-Tenant Bot — Activation Handshake | 5. Bots & IRCE | High | Bot onboarding |
| 29 | Architecture Decision Records | 1. Vertical Lego | Medium | ADR format |
| 30 | Master Pre-Flight Checklist | 9. Incident Response | High | Validation list |
| 31 | Serverless Architecture | 6. API & Backend | High | Vercel patterns |
| 32 | Storage Bucket Organization | 3. Media Pipeline | High | File structure |
| 33 | Draft State Transitions | 4. Drafts & Publishing | High | State machine |
| 34 | RLS Policy Template | 2. Database | High | Security |
| 35 | Supabase Query Robustness | 2. Database | High | Query patterns |
| 36 | Kimi Configuration — Validation Matrix | 5. Bots & IRCE | High | LLM config |
| 37 | Bot Message Pipeline Order | 5. Bots & IRCE | High | Processing flow |
| 38 | Dirty State & Navigation Guards | 6. API & Backend | Medium | UI pattern |
| 39 | MCP — Model Context Protocol | 9. Incident Response | Medium | Tooling |
| 40 | FK Migration — Don't Orphan Historical Data | 2. Database | High | Migration safety |
| 41 | Circuit Breaker — Kimi API Protection | 6. API & Backend | High | Resilience |
| 42 | Rate Limiting — Per-User Abuse Prevention | 6. API & Backend | High | API protection |
| 43 | Memory Leak Prevention — Bounded Collections | 6. API & Backend | High | Serverless |
| 44 | Voice Correction — Chilean Spanish | 5. Bots & IRCE | Medium | Localization |
| 45 | Image Optimization — Sharp Before Kimi | 3. Media Pipeline | High | Performance |
| 46 | Cross-Browser Drag & Drop | 6. API & Backend | Medium | Frontend |
| 47 | Split-Brain Prevention — Verify Working Directory | 9. Incident Response | Medium | Dev workflow |
| 48 | SSR Cache Data Leak | 6. API & Backend | High | React/Next.js |
| 49 | SQL Forensics — Diagnostic Queries | 9. Incident Response | High | Debugging |
| 50 | Race Condition Detection — Optimistic Locking | 2. Database | High | Concurrency |
| 51 | Supabase Connection Architecture | 2. Database | High | DB connections |
| 52 | Fuzzy Command Matching | 5. Bots & IRCE | High | Bot UX |
| 53 | Debugging Methodology — The Guard Analogy | 9. Incident Response | High | Methodology |
| 54 | TypeScript Build — The Silent Killer | 6. API & Backend | High | Build safety |
| 55 | PowerShell Gotchas — Windows Dev | 9. Incident Response | Low | Environment |
| 56 | Turbopack Cache Corruption | 9. Incident Response | Medium | Dev tooling |
| 57 | Supabase Migration Pattern | 2. Database | High | DB migrations |
| 58 | Security Pre-Checklist | 9. Incident Response | High | Security |
| 59 | Rubro Expansion — 7-Step Recipe | 1. Vertical Lego | High | Vertical creation |
| 60 | Client Onboarding — 5 Minutes Start to Finish | 8. Onboarding | High | Tenant setup |
| 61 | Intelligence Layer — pgvector RAG Architecture | 7. Features & Tiers | High | Future feature |
| 62 | Image System Unification — The Lego Vision | 3. Media Pipeline | High | Architecture vision |
| 63 | New Bot Creation — Complete Webhook Template | 5. Bots & IRCE | High | Bot template |
| 64 | Anti-Hardcode Comprehensive Reference | 6. API & Backend | High | Code patterns |
| 65 | Ghost Columns — Silent PostgREST Failures | 2. Database | High | DB debugging |
| 66 | safeInt() — Kimi Returns Floats for Integer Columns | 5. Bots & IRCE | High | Data coercion |
| 67 | CHECK Constraints Must Match All Code Paths | 2. Database | High | DB integrity |
| 68 | Dual-Table Archives | 4. Drafts & Publishing | High | Archive pattern |
| 69 | Empty Body → request.json() Crash | 6. API & Backend | High | API robustness |
| 70 | Polling Must Match Writer Status | 4. Drafts & Publishing | High | Sync pattern |
| 71 | Session Size Limits — Photo Upload Budget | 3. Media Pipeline | High | Resource limits |
| 72 | Cron Jobs — Vercel Scheduled Functions | 6. API & Backend | Medium | Automation |
| 73 | Batch-Complete Ghost Columns — The Silent Product Black Hole | 2. Database | High | DB debugging |

---

## Sections by Playbook

### Playbook 1: Vertical Lego (2 sections)
- §20: Phase Boundaries — Migration Markers
- §29: Architecture Decision Records
- §59: Rubro Expansion — 7-Step Recipe

### Playbook 2: Database Architecture (8 sections)
- §12: Supabase Clients — Which One When
- §34: RLS Policy Template
- §35: Supabase Query Robustness
- §40: FK Migration — Don't Orphan Historical Data
- §49: SQL Forensics — Diagnostic Queries
- §50: Race Condition Detection — Optimistic Locking
- §51: Supabase Connection Architecture
- §57: Supabase Migration Pattern
- §65: Ghost Columns — Silent PostgREST Failures
- §67: CHECK Constraints Must Match All Code Paths
- §73: Batch-Complete Ghost Columns

### Playbook 3: Media Pipeline (8 sections)
- §3: Image Systems — Legacy vs Future
- §4: Two-Phase Async Processing
- §8: Bot Webhook — Media Accumulation
- §16: Entity Images — No FK
- §32: Storage Bucket Organization
- §45: Image Optimization — Sharp Before Kimi
- §62: Image System Unification — The Lego Vision
- §71: Session Size Limits — Photo Upload Budget

### Playbook 4: Drafts & Publishing (6 sections)
- §14: Status Values — The Source of Many Bugs
- §22: Soft Delete vs Hard Delete
- §23: Draft Button Actions — Complete Map
- §33: Draft State Transitions
- §68: Dual-Table Archives
- §70: Polling Must Match Writer Status

### Playbook 5: Bots & IRCE (12 sections)
- §2: getUserTenant() — Cross-Rubro Isolation
- §7: Bot Webhook — Message Processing Order
- §9: Bot Webhook — Callback Queries
- §19: IRCE Pipeline — What's Missing
- §21: Kimi API Configuration
- §24: Session Routing
- §25: Entity Resolver — Fuzzy Matching
- §26: Confirmation Builder — Permissive Parser
- §28: Multi-Tenant Bot — Activation Handshake
- §36: Kimi Configuration — Validation Matrix
- §37: Bot Message Pipeline Order
- §44: Voice Correction — Chilean Spanish
- §52: Fuzzy Command Matching
- §63: New Bot Creation — Complete Webhook Template
- §66: safeInt() — Kimi Returns Floats

### Playbook 6: API Design & Backend Patterns (24 sections)
- §1: Tenant Resolution — The Three Tiers
- §5: Server/Client Component Split
- §6: Admin Auth — Cookie-Based
- §10: Bot Webhook — Idempotence
- §11: Theme System
- §13: LX Tenant Resolver
- §15: WhatsApp URL Construction
- §17: Vercel Limits & maxDuration
- §27: Operation Executor — Atomicity
- §31: Serverless Architecture
- §38: Dirty State & Navigation Guards
- §41: Circuit Breaker — Kimi API Protection
- §42: Rate Limiting — Per-User Abuse Prevention
- §43: Memory Leak Prevention — Bounded Collections
- §46: Cross-Browser Drag & Drop
- §48: SSR Cache Data Leak
- §54: TypeScript Build — The Silent Killer
- §64: Anti-Hardcode Comprehensive Reference
- §69: Empty Body → request.json() Crash
- §72: Cron Jobs — Vercel Scheduled Functions

### Playbook 7: Product Features & Tiers (2 sections)
- §61: Intelligence Layer — pgvector RAG Architecture

### Playbook 8: Onboarding & Activation (3 sections)
- §60: Client Onboarding — 5 Minutes Start to Finish

### Playbook 9: Incident Response (6 sections)
- §18: Danger Zones — Known Weak Points
- §30: Master Pre-Flight Checklist
- §39: MCP — Model Context Protocol
- §47: Split-Brain Prevention — Verify Working Directory
- §53: Debugging Methodology — The Guard Analogy
- §55: PowerShell Gotchas — Windows Dev
- §56: Turbopack Cache Corruption
- §58: Security Pre-Checklist

---

## Ambiguous Classifications (Medium Confidence)

| Section | Ambiguity | Resolution |
|---------|-----------|------------|
| §11: Theme System | UI vs API | → Playbook 6 (API implementation) |
| §20: Phase Boundaries | Migration vs Architecture | → Playbook 1 (vertical expansion) |
| §29: ADRs | Generic vs Specific | → Playbook 1 (architecture decisions) |
| §39: MCP | Tooling vs Debugging | → Playbook 9 (dev tooling) |
| §44: Voice Correction | Bot vs Localization | → Playbook 5 (bot-specific) |
| §55: PowerShell | Dev env vs Incident | → Playbook 9 (debugging context) |

---

## Recommended Extraction Order

1. **First Pass (Foundation):** Playbook 2 (DB) + Playbook 3 (Media)
2. **Second Pass (Core Flows):** Playbook 4 (Drafts) + Playbook 6 (API)
3. **Third Pass (Bots):** Playbook 5 (IRCE)
4. **Fourth Pass (Business):** Playbook 1 (Vertical) + Playbook 7 (Features)
5. **Fifth Pass (Ops):** Playbook 8 (Onboarding) + Playbook 9 (Incident)

---

## Notes for Rewriting

- §4 (Two-Phase Async) overlaps heavily with existing `Playbook-two-phase-async-processing.md`
- §19 (IRCE Pipeline) should merge with `Playbook-IRCE-pipeline-for-verticals.md`
- §62 (Image System Unification) is the "why" — the skill has the "how"
- §64 (Anti-Hardcode) is cross-cutting — consider duplicating in Playbook 2 (DB) and Playbook 6 (API)
- §§65, 67, 73 are all DB debugging — group in Playbook 2 appendix
