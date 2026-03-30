---
name: feature-reporter
description: Creates standardized technical reports for completed features. Documents architecture, API routes, workflow, and transplantability for future reference or migration.
trigger: "feature complete", "document feature", "technical report", "feature manual"
---

# Feature Reporter

Document a feature AFTER it's working, for future reference, maintenance, or transplantation.

---

## When to Activate

- User says "document this feature", "create technical report", "feature is done"
- Need to understand a feature's internals for debugging
- Planning to transplant a feature to another project
- Handoff documentation for maintenance

---

## Step 1: Gather Feature Information

Ask the user or inspect the codebase to collect:

- **Feature name** — short, descriptive identifier
- **Project name** — where this feature lives
- **Status** — production, beta, or experimental
- **Authors** — who built it
- **Purpose** — what problem this solves
- **Target users** — admin, client, public, bot

---

## Step 2: Analyze Codebase Structure

Map the implementation by identifying:

- **Pages/Routes** — UI entry points and their file paths
- **API Routes** — endpoints, methods, and handlers
- **Components** — reusable UI pieces with props
- **Hooks** — state management and side effects
- **Lib/Utils** — helper functions and utilities
- **External services** — third-party integrations
- **Database tables** — data persistence layer

---

## Step 3: Generate Feature Report

Create file: `FEATURE-REPORT-{feature-name}-{project}.md`

Use this template:

```markdown
# Feature Report: {Feature Name}

> **Project:** {Project Name}  
> **Status:** ✅ Production / 🟡 Beta / 🔴 Experimental  
> **Created:** {Date}  
> **Authors:** {Who built it}  
> **Last Updated:** {Version/Date}

---

## 1. EXECUTIVE SUMMARY

| Aspect | Detail |
|--------|--------|
| **Purpose** | What problem does this solve? |
| **User** | Who uses this? (admin, client, public, bot) |
| **Business Value** | Why does this exist? |
| **Cost Impact** | 🟢 Low / 🟡 Medium / 🔴 High per use |

---

## 2. ARCHITECTURE OVERVIEW

```
[User Input]
    ↓
[Component/Interface]
    ↓
[API Route] → [External Service]
    ↓
[Database/Storage]
    ↓
[Output/Result]
```

---

## 3. TECHNICAL STACK

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| Frontend | React/Next.js/etc | X.X | UI layer |
| State Management | Hook/Context/etc | — | Data flow |
| Styling | Tailwind/etc | — | Appearance |
| API | REST/GraphQL/etc | — | Communication |
| External | Service name | — | Third-party |
| Database | PostgreSQL/etc | — | Persistence |
| Storage | Supabase/etc | — | Files/images |

---

## 4. FILE INVENTORY

### Pages/Routes
| Path | File | Purpose |
|------|------|---------|
| `/path` | `app/.../page.tsx` | What it shows |

### API Routes
| Endpoint | Method | File | Purpose |
|----------|--------|------|---------|
| `/api/...` | POST | `route.ts` | What it does |

### Components
| Component | File | Props | Purpose |
|-----------|------|-------|---------|
| `Name` | `path.tsx` | `{...}` | What it renders |

### Hooks
| Hook | File | Returns | Purpose |
|------|------|---------|---------|
| `useName` | `path.ts` | `{...}` | What it manages |

### Lib/Utils
| File | Exports | Purpose |
|------|---------|---------|
| `lib/...ts` | `{...}` | What it does |

---

## 5. WORKFLOW (Step-by-Step)

### User Journey
1. **Step 1:** User action → System response
2. **Step 2:** Next action → Response
3. **Step 3:** Final action → Result

### Data Flow
```
Input: {...}
  ↓ [Transformation]
Process: {...}
  ↓ [Transformation]
Output: {...}
```

---

## 6. CONFIGURATION

### Environment Variables
```bash
KEY=value  # What it's for
```

### Required Setup
- [ ] Step 1
- [ ] Step 2
- [ ] Step 3

---

## 7. LIMITS & CONSTRAINTS

| Constraint | Value | What happens if exceeded |
|------------|-------|-------------------------|
| Max items | N | Error/behavior |
| Timeout | Ns | Error/behavior |
| File size | N MB | Error/behavior |
| Rate limit | N/min | Error/behavior |

---

## 8. TRANSPLANTABILITY

### Can this be moved to another project?
- [ ] Yes, standalone (no dependencies)
- [ ] Yes, with minimal dependencies
- [x] Yes, with scaffold dependencies
- [ ] No, tightly coupled

### Dependencies on other features
| Dependency | Required? | Notes |
|------------|-----------|-------|
| `feature-name` | Yes/No | Why needed |

### Steps to Transplant
1. Copy files: `path/...`
2. Install dependencies: `npm i ...`
3. Setup environment variables
4. Configure external services
5. Test all workflows

---

## 9. TESTING CHECKLIST

### Functional Tests
- [ ] Test case 1
- [ ] Test case 2
- [ ] Test case 3

### Edge Cases
- [ ] Empty state
- [ ] Max limit
- [ ] Error handling
- [ ] Timeout

---

## 10. INCIDENTS & LESSONS LEARNED

| Date | Issue | Solution | Lesson |
|------|-------|----------|--------|
| YYYY-MM-DD | What broke | How fixed | What learned |

---

## 11. COST ANALYSIS

| Resource | Cost per use | Scaling risk |
|----------|--------------|--------------|
| Vercel | $X.XXXX | 🟢 Low |
| Supabase | $X.XXXX | 🟡 Medium |
| External API | $X.XXXX | 🔴 High |
| **Total** | **$X.XX** | **Risk level** |

---

## 12. FUTURE IMPROVEMENTS

- [ ] Idea 1
- [ ] Idea 2
- [ ] Idea 3

---

*Feature Report generated by feature-reporter skill*  
*Template version: 1.0*
```

---

## Step 4: Review and Deliver

Ask the user:
> "Feature report generated. Review for accuracy — any missing sections or corrections needed?"

### Documentation Rules

1. **Fill ALL sections** — Empty sections indicate incomplete documentation
2. **Be specific** — "Path: `app/admin/feature/page.tsx`" not "Somewhere in admin"
3. **Include code snippets** for critical logic
4. **Note transplantability** — Will we need this elsewhere?
5. **Record incidents** — Future you will thank past you

---

## Information Gaps — Catastro

If any of the following is unclear, STOP and ask the user before proceeding:

| Gap | Question to Ask |
|-----|-----------------|
| **Feature scope** | "What specific feature should I document?" |
| **Feature status** | "Is this in production, beta, or experimental?" |
| **Code location** | "Where are the source files for this feature?" |
| **Authors** | "Who built this feature? (for attribution)" |
| **Dependencies** | "What other features or services does this depend on?" |
| **Transplant needs** | "Do we plan to move this to another project later?" |
| **Known issues** | "Are there any incidents or lessons learned I should record?" |

---

*Skill: feature-reporter v2.0*  
*Part of: Nadistudio Scaffold Documentation System*
