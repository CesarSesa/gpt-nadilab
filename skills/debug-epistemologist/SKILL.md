---
name: debug-epistemologist
description: Systematic debugging protocol that distinguishes REGRESSION vs NEVER-WORKED, applies binary search method, and documents bugs as LX-### before attempting fixes. Prevents scatter debugging and premature fixes.
trigger: "no funciona", "se rompio", "debug", "que paso aqui", "error raro", "bug", "falla", "no entiendo por que", "antes funcionaba", "dejo de funcionar"
origin: Nadistudio
---

# Debug Epistemologist

> *"The difference between a surgeon and a butcher is knowing what NOT to cut. The same applies to debugging."*

## Job-to-be-Done

Solve bugs systematically by understanding before fixing, distinguishing regression from never-worked, and documenting findings to prevent repeated mistakes.

Activate when:
- Something "stopped working" or "never worked"
- Multiple symptoms that might be related (or not)
- Temptation to "try changing a few things to see if it helps"
- 3 AM debugging with no clear hypothesis
- Error messages that don't clearly point to root cause

---

## The Process

### Step 1: Answer the 5 Questions (Before Touching Code)

Answer these in writing. Not in your head. In writing.

#### Q1: ¿Funcionaba antes? (The Regression Test)

```
YES -> REGRESSION
    ├── Git log: What changed recently in this file?
    ├── Env diff: Local vs Prod, Before vs After
    └── Deploy timestamp: Did it break after last deploy?

NO -> NEVER-WORKED
    ├── Is this a new feature that was never validated?
    ├── Did it only work in the "happy path"?
    └── Did someone assume it worked without testing?

MAYBE / DON'T KNOW -> FIND OUT
    └── Check git history, logs, ask "when did you last see it work?"
```

**Why this matters:**
- REGRESSION -> Focus on what changed (git diff)
- NEVER-WORKED -> Focus on the logic itself (design flaw)

#### Q2: ¿Es consistente o intermitente?

```
CONSISTENT (fails 100% of time)
    ├── Easier to debug
    ├── Reproducible
    └── Likely: Logic error, missing data, wrong config

INTERMITTENT (fails sometimes)
    ├── Harder to debug
    ├── Race condition, timing issue, external dependency
    └── Likely: Async timing, caching, network, state leakage
```

**The Rule:**
- If you can't reproduce it 3 times in a row, it's **intermittent**
- Intermittent bugs need **logging**, not code changes

#### Q3: ¿Qué es lo último que cambió cerca de aquí?

```bash
# Git: Last changes to this file
git log -5 --oneline -- path/to/file.ts

# Git: What changed in last deploy
git diff HEAD~1 HEAD --stat

# Git: Who touched this last
git blame -L 10,30 path/to/file.ts
```

**The Correlation Principle:**
> "The last thing that changed is the first thing to suspect."

Not because it's always guilty, but because ruling it out is the fastest path to knowledge.

#### Q4: ¿Hay un LX-### similar?

```bash
# Search existing bug documentation
grep -r "LX-" . --include="*.md" | grep -i "error\|bug\|fail"

# Search system_logs for similar error codes
SELECT component, operation, message, COUNT(*)
FROM system_logs
WHERE level = 'error'
  AND created_at > NOW() - INTERVAL '7 days'
GROUP BY component, operation, message
ORDER BY COUNT(*) DESC;
```

**The Pattern Principle:**
> "If it happened once, it'll happen again."

Documented bugs (LX-###) contain wisdom. Don't relearn what you already learned.

#### Q5: ¿Qué NO es el problema?

List 3 things that are definitely NOT causing this:

```
This is NOT caused by:
1. Database connection (because other queries work)
2. Auth/RLS (because user can access other pages)
3. Frontend (because API returns same error in Postman)
```

**The Negative Knowledge Principle:**
> "Knowing what isn't wrong is as valuable as knowing what is."

This prevents the "scatter debugging" where you change auth, database, and frontend simultaneously.

---

### Step 2: Apply the Binary Search Method

When you have no idea where the bug is, don't guess. Divide.

```
Function with bug:
├── Part A (lines 1-50)
├── Part B (lines 51-100)
└── Part C (lines 101-150)

Step 1: Comment out Part B + C
        Run -> Does it still fail?
        
        YES -> Bug is in Part A
        NO  -> Bug is in Part B or C

Step 2: If bug in Part B:
        Comment out half of Part B
        Run -> Does it still fail?
        
        YES -> Bug is in first half of Part B
        NO  -> Bug is in second half of Part B

Continue until you isolate the exact line/function.
```

**The Rule:**
- Never change more than 1 variable at a time
- Each test must eliminate 50% of possibilities
- If you're not eliminating possibilities, you're guessing

---

### Step 3: Validate with the 3-Data-Point Rule

**Do not attempt a fix until you have:**

1. **ONE case where it FAILS** (documented with input/output/error)
2. **ONE case where it WORKS** (if such exists)
3. **The EXACT DIFFERENCE** between case 1 and 2

```
FAILS: User "maria" + Property "depto-123" + Photo batch > 10MB
WORKS: User "maria" + Property "depto-123" + Photo batch < 5MB
DIFF:  File size threshold

HYPOTHESIS: 10MB limit is enforced somewhere
TEST: Send exactly 9.9MB -> should work
TEST: Send exactly 10.1MB -> should fail
```

**Without 3 data points, you're not debugging. You're hoping.**

---

### Step 4: Document Before Fixing (LX-### Protocol)

Before attempting a fix, document the bug.

#### Template: New Bug Discovery

```markdown
# LX-XXX: [Short Description]

## Symptom
[What the user sees / what breaks]

## 5 Questions Answers
- **Q1 Regression?** YES / NO / UNKNOWN
- **Q2 Consistent?** YES / INTERMITTENT
- **Q3 Last change?** [Commit hash / deploy time]
- **Q4 Similar bugs?** NONE / LX-YYY, LX-ZZZ
- **Q5 NOT caused by?** Auth, Database, Frontend...

## Binary Search Progress
- [ ] Tested: [what you tested]
- [ ] Eliminated: [what you ruled out]
- [ ] Isolated to: [file/function/line]

## Hypothesis
[What you think is happening and why]

## Fix Attempt
[What you tried and what happened]

## Status
- [ ] DISCOVERED
- [ ] ISOLATED
- [ ] FIXED
- [ ] VALIDATED
- [ ] DOCUMENTED
```

#### Where to store
- File: `.docs/ERRORS/LX-XXX-bug-name.md`
- Link from: Castle docs, relevant SKILL.md
- Reference in: Commit messages (`Fix: LX-042 batch timeout`)

---

## Constraints & Anti-Patterns

### Constraints

- **Never guess.** If you don't know, say "I don't know yet."
- **Never change more than 1 thing.** Isolate variables.
- **Never fix without understanding.** The fix teaches; the code doesn't.
- **Never skip documentation.** Today's "obvious" fix is tomorrow's "why did we do this?"

### Anti-Patterns (What NOT to Do)

#### Scatter Debugging
```
"Let me try:
- Restarting the server
- Clearing the cache
- Changing the timeout
- Adding console.logs everywhere
- Reverting yesterday's commit
- Asking in Discord"
```
**Problem:** If one of these works, you won't know which. And it'll break again.

#### Premature Fixation
```
"I think it's the database connection. Let me fix that."
```
**Problem:** You "thought" for 5 seconds. You didn't ask the 5 questions.

#### Cargo Cult Debugging
```
"I found a Stack Overflow answer that mentions this error. Let me apply that fix."
```
**Problem:** Their context != Your context. Same error, different cause.

#### Fix Without Validation
```
"I changed something and now it works. Ship it!"
```
**Problem:** You don't know what you changed or why it worked. It'll break again.

---

## Information Gaps — Catastro

When debugging fails, it's often because critical information is missing. Recognize these gaps **before** they become blockers:

### The 6 Critical Information Gaps

| Gap | Symptom | Fix |
|-----|---------|-----|
| **1. Environment** | "Works on my machine" | Document exact env: OS, Node version, DB state, env vars |
| **2. Reproduction Steps** | "Sometimes it happens" | Get exact steps, input data, timestamps, user context |
| **3. Error Scope** | "Everything is broken" | Isolate: one user? one feature? one endpoint? |
| **4. Recent Changes** | "It was working before" | Git diff, deploy logs, dependency updates |
| **5. Expected vs Actual** | "It doesn't work" | Define expected behavior precisely, with examples |
| **6. Related Systems** | "Database error" | Check: migrations, RLS, connection pools, external APIs |

### The Catastro Rule

> **If you've been debugging for >30 minutes without progress, you have an Information Gap.**

Stop debugging. Start gathering:
1. Ask: "What information am I missing?"
2. Check the 6 gaps above
3. Get that information before continuing

### Information Gathering Checklist

```markdown
## Before Debugging: Information Checklist

- [ ] Exact error message (copy-paste, not "it says error")
- [ ] Environment: Local / Staging / Production
- [ ] User context: Which user, what permissions
- [ ] Timestamp: When did it first occur
- [ ] Frequency: Every time / sometimes / once
- [ ] Related data: Input that triggers it
- [ ] Related logs: Last 50 lines before error
- [ ] Recent changes: Last deploy, last commit to this area
```

---

## Quick Reference

### Debugging Decision Tree

```
Bug discovered
    │
    ├──> Can you reproduce it 3 times consistently?
    │       │
    │       NO -> INTERMITTENT -> Add logging, wait for pattern
    │       │
    │       YES -> CONSISTENT
    │               │
    │               ├──> Did it work before?
    │               │       │
    │               │       YES -> REGRESSION
    │               │       │       └──> Git diff since last known good
    │               │       │
    │               │       NO -> NEVER-WORKED
    │               │               └──> Design flaw, needs rework
    │               │
    │               └──> Binary search to isolate
    │                       │
    │                       └──> Found the line?
    │                               │
    │                               YES -> Fix + validate
    │                               NO -> Log as LX-###, add instrumentation
    │
    └──> Document findings (even if not fixed)
            └──> Update LX-### with learnings
```

### The 3 AM Protocol

When debugging at night, tired:

1. **STOP** if you've tried 3 things without progress
2. **DOCUMENT** current state (LX-###) even if unfinished
3. **SLEEP** on it
4. **ASK** another LLM/terminal in the morning with fresh context

**The Myth of Heroic Debugging:**
> "I'll stay up until I fix this" usually produces:
> - 10 changed files
> - 3 new bugs
> - 0 understanding of the original bug

**Sleep is a debugging tool.**

### Integration with Other Skills

| Skill | Relationship |
|-------|-------------|
| `api-error-handler` | Use to classify errors before debugging |
| `db-guardian` | Use to verify schema isn't the issue (ghost columns) |
| `irce-engineer` | When debugging Layer 1-6 issues |
| `m1-m9-pipeline` | When debugging photo processing |
| `nadi-operational` | "El problema no es técnico, es timing" |

---

## The Epistemological Oath

> "I will not attempt to fix what I do not understand.
> I will not change code without a hypothesis.
> I will document what I learn, even if I don't solve the problem.
> I will respect the 3-data-point rule.
> I will sleep before I'm desperate."

---

*Debug Epistemologist: Because fixing without understanding is just moving bugs around.*
