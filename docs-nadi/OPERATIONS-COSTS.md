# Operations & Cost Model — Nadistudio Scaffold

> **Document Type:** Operational cost reference for Vercel + Supabase serverless stack  
> **Assumed Standard:** Vercel Pro + Supabase Pro (post-incident egress upgrade)  
> **Last Updated:** 2026-03-28  
> **Incident Logged:** Egress bomb via recursive logs (244% quota)

---

## 1. CURRENT STACK STANDARD

| Service | Plan | Monthly Base | Why This Tier |
|---------|------|--------------|---------------|
| **Vercel** | Pro | $20 | Wildcard domains (*.nadistudio.cl), team features |
| **Supabase** | Pro | $25 | Egress 250GB (vs 10GB free), storage auto-scale |
| **Kimi API** | Pay-as-you-go | ~$0 | $10 credits, ROI star (Vision ~$0.02-0.03/call) |
| **Domains** | External | ~$10-15 | nadistudio.cl + wildcards |

**Base monthly burn:** ~$55-60 (before variable usage)

---

## 2. COST STRUCTURE ARRAYS

### 2.1 Vercel Pro — Cost Drivers

| Metric | Free Limit | Pro Limit | What Triggers It | Scalability |
|--------|------------|-----------|------------------|-------------|
| **Function Invocations** | 125K/mo | 1M/mo | API calls, SSR renders | ✅ Linear with users |
| **Fluid Active CPU** | 1000 GB-hrs | 4000 GB-hrs | Long-running functions | ⚠️ Heavy AI jobs |
| **Memory Provisioned** | 1000 GB-hrs | 4000 GB-hrs | RAM × time | ✅ Predictable |
| **Edge Transfer** | 100 GB | 1 TB | CDN assets (cached) | ✅ Cache-dependent |
| **Origin Transfer** | 100 GB | 1 TB | API responses, SSR | ⚠️ Data-heavy queries |
| **Build Minutes** | 6000 min | 15000 min | Deploys | ✅ Rarely limits |

**Current usage (single dev):**
- Function Invocations: 4.48K (~0.4% of Pro)
- Fluid CPU: 4 min (~0.01% of Pro)
- Memory: 0.63 GB-hrs (~0.02% of Pro)
- Edge: 62 MB (<1%)
- Origin: 53 MB (<1%)

### 2.2 Supabase Pro — Cost Drivers

| Metric | Free Limit | Pro Limit | What Triggers It | Incident Risk |
|--------|------------|-----------|------------------|---------------|
| **Egress (Total)** | 10 GB | 250 GB | ALL outbound traffic | 🔴 **CRITICAL** |
| **Database Size** | 500 MB | 8 GB + auto-scale | Data accumulation | 🟡 Watch |
| **Storage Size** | 1 GB | 100 GB | Images, files | 🟡 Grow slowly |
| **Storage Egress** | Included in total | Included | Image downloads | 🔴 With many users |
| **Edge Functions** | 500K invocations | 5M | Serverless functions | 🟢 Generous |
| **Image Transformations** | Unavailable | 1000 origins | Dynamic resizing | 🟡 Check usage |

**Incident:** Logs recursion → 12.177 GB egress (244% of free)  
**Lesson:** Egress is the silent killer. Not storage size, not DB size. **Bandwidth out.**

### 2.3 Kimi API — The ROI Star

| Operation | Input | Output | Cost | Notes |
|-----------|-------|--------|------|-------|
| **Vision Analysis** | 30 images (10MB each) | JSON ~2K tokens | ~$0.02-0.03 | Main cost per property |
| **Text Completion** | 4K context | 1K output | ~$0.001 | IRCE, prompts |
| **Embedding** | 1K tokens | vector | ~$0.0001 | Future RAG feature |

**Current credits:** $10 (sufficient for ~300-500 property analyses)  
**Scaling:** Linear with properties processed. No quota limits, just wallet.

---

## 3. UNIT ECONOMICS BY BUSINESS EVENT

### "One Property Published" (EasyProp Bridge flow)

| Step | Component | Cost | Driver |
|------|-----------|------|--------|
| Upload 30 photos | Supabase Storage PUT | ~$0.0001 | Storage size |
| Store metadata | Supabase DB write | ~$0.0001 | DB ops |
| Kimi Vision analysis | Kimi API | ~$0.025 | Tokens + images |
| Process/Store result | Vercel Function (5s) | ~$0.0002 | CPU time |
| Serve photos (N views) | Supabase Storage Egress | ~$0.001 per view | Downloads |
| **Total per property** | — | **~$0.03 + egress** | — |

### "One Bot Interaction" (Telegram IRCE)

| Step | Component | Cost | Notes |
|------|-----------|------|-------|
| Webhook receive | Vercel Edge | ~$0.00001 | Minimal |
| Intent classification (regex) | Vercel Function | ~$0.00001 | 80% of cases |
| Intent classification (LLM) | Vercel + Kimi | ~$0.001 | 20% fallback |
| DB query | Supabase | ~$0.00001 | Indexed queries |
| Response | Vercel | ~$0.00001 | — |
| **Total per message** | — | **~$0.0001-0.001** | — |

---

## 4. OPERATIONAL THRESHOLDS (When to Act)

### 4.1 Vercel Triggers

| Metric | Green | Yellow | Red | Action |
|--------|-------|--------|-----|--------|
| GB-hrs usage | <50% | 50-80% | >80% | Review long functions, cache more |
| Function duration | <5s avg | 5-10s | >10s | Optimize or split async |
| Origin transfer | <50% | 50-80% | >80% | Add CDN caching, reduce payloads |

### 4.2 Supabase Triggers

| Metric | Green | Yellow | Red | Action |
|--------|-------|--------|-----|--------|
| Egress (the big one) | <30% | 30-60% | >60% | **Move images to R2** |
| DB Size | <30% | 30-70% | >70% | Archive old data, partition |
| Storage Size | <30% | 30-70% | >70% | Clean temp files, compress |
| Connection Pool | <50% | 50-80% | >80% | Add pooling, optimize queries |

### 4.3 Kimi Triggers

| Metric | Green | Yellow | Red | Action |
|--------|-------|--------|-----|--------|
| Credits remaining | >$5 | $2-5 | <$2 | Top up or reduce Vision usage |
| Avg cost per analysis | <$0.03 | $0.03-0.05 | >$0.05 | Optimize prompts, reduce images |

---

## 5. INCIDENT REGISTRY

### INC-001: Egress Bomb via Recursive Logs

**Date:** 2026-03-28  
**Status:** Resolved (forced upgrade to Pro)  
**Cost:** $0 (caught before billing), but forced $25/mo Pro upgrade  

#### What Happened
```
Bug in logging code → Recursive loop
    ↓
Each log entry triggered new log write
    ↓
Cascading egress: 12 GB in hours (244% of 5 GB free)
    ↓
Supabase Free suspended until upgrade
```

#### Root Cause
- Code bug: Log handler calling itself
- Not caught because: Free tier has no alerting
- Assumption: "Logs are cheap" (false when recursive)

#### Prevention
- [ ] Add circuit breakers to logging
- [ ] Rate limit on egress (Impossible on Free)
- [ ] Monitor: Alert at 50% egress (Pro has this)
- [ ] Code review: No function should call itself without base case

#### Lesson
**Any data egress can explode.** Not just images. Not just user traffic. Logs, metrics, health checks — all count.

---

## 6. SCALING SCENARIOS (Pre-Launch Projections)

### Scenario A: Soft Launch (Month 1-3)
- **Tenants:** 5
- **Properties/month:** 50
- **Bot messages/day:** 100
- **Kimi cost:** ~$1.50
- **Vercel usage:** <5% of Pro
- **Supabase egress:** <10 GB
- **Total variable:** ~$5-10

### Scenario B: Pilot Growth (Month 4-6)
- **Tenants:** 20
- **Properties/month:** 400
- **Bot messages/day:** 500
- **Kimi cost:** ~$12
- **Vercel usage:** <20% of Pro
- **Supabase egress:** ~40 GB (watch threshold)
- **Total variable:** ~$40-60

### Scenario C: Traction (Month 7-12)
- **Tenants:** 100
- **Properties/month:** 2000
- **Bot messages/day:** 2500
- **Kimi cost:** ~$60
- **Vercel usage:** ~50% of Pro
- **Supabase egress:** ~150 GB (migrate images to R2)
- **Total variable:** ~$150-200

### Scenario D: Viral Stress Test
- **Tenants:** 1000
- **Properties/month:** 20000
- **Bot messages/day:** 25000
- **Kimi cost:** ~$600
- **Vercel usage:** >100% of Pro (need Enterprise)
- **Supabase egress:** >250 GB (R2 mandatory)
- **Total variable:** ~$1500+

**Decision point:** At 100 tenants, evaluate Vercel Enterprise + R2 migration.

---

## 7. COST OPTIMIZATION PLAYBOOK

### Immediate (Now)
- ✅ Upgrade to Supabase Pro (egress monitoring)
- ✅ Set up Vercel Pro (higher limits)
- ⏳ Add egress alert at 50% (Pro feature)

### Near-term (Pre-launch)
- ⏳ Implement image compression before upload
- ⏳ Cache Kimi results (don't re-analyze same photos)
- ⏳ Add client-side rate limiting

### Growth-phase (10+ tenants)
- 📋 Migrate images to Cloudflare R2 ($0.015/GB vs Supabase egress)
- 📋 Evaluate Vercel Edge Functions vs Serverless (cost/latency tradeoff)
- 📋 Kimi caching layer (Redis) for repeated queries

### Scale-phase (100+ tenants)
- 📋 Dedicated Supabase instance
- 📋 Vercel Enterprise (dedicated support, higher limits)
- 📋 Multi-region (if expanding beyond Chile)

---

## 8. FEATURE COST TRACKING

When adding new features, estimate:

```markdown
### Feature: [Name]
- **Vercel impact:** Low/Med/High (function duration, invocations)
- **Supabase impact:** Low/Med/High (egress, DB size)
- **Kimi impact:** Low/Med/High (tokens per use)
- **Risk:** What could make costs explode?
- **Circuit breaker:** How do we limit abuse?
```

**Example: EasyProp Bridge**
- Vercel: Medium (35s functions)
- Supabase: Low (no egress in flow)
- Kimi: High ($0.03 per use, main cost)
- Risk: User uploads 100 photos (bypass client limit)
- Circuit breaker: Server-side max 30 photos

---

## Appendix A: Quick Reference Card

```
┌─────────────────────────────────────────┐
│  NADISTUDIO COST DASHBOARD ( mental )   │
├─────────────────────────────────────────┤
│  Base burn:      $60/mo                 │
│  Per property:   ~$0.03 + egress        │
│  Per bot msg:    ~$0.0005               │
├─────────────────────────────────────────┤
│  WATCH: Egress (Supabase)               │
│  WATCH: Credits (Kimi)                  │
│  IGNORE: Build minutes                  │
└─────────────────────────────────────────┘
```

---

*Document: OPERATIONS-COSTS.md*  
*Part of: Nadistudio Scaffold Operations*  
*Related: feature-expert skill (cost-aware features)*
