---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue: test failures, bugs in production, unexpected behavior, performance problems, build failures, integration issues.

**Use ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully** — don't skip, read stack traces completely
2. **Reproduce Consistently** — if not reproducible, gather more data first
3. **Check Recent Changes** — git diff, recent commits, new dependencies
4. **Gather Evidence in Multi-Component Systems**

   For EACH component boundary:
   - Log what data enters/exits the component
   - Verify environment/config propagation
   - Check state at each layer
   - Run once to see WHERE it breaks, then investigate that component

5. **Trace Data Flow** — where does bad value originate? Keep tracing up until you find the source. Fix at source, not symptom.

### Phase 2: Pattern Analysis

1. Find working examples (similar working code in same codebase)
2. Compare against references — read reference implementation COMPLETELY
3. Identify differences — list every difference, however small
4. Understand dependencies — what config/environment does it assume?

### Phase 3: Hypothesis and Testing

1. **Form Single Hypothesis** — "I think X is root cause because Y"
2. **Test Minimally** — smallest possible change, one variable at a time
3. **Verify Before Continuing** — worked? → Phase 4. Didn't work? Form NEW hypothesis. Don't stack fixes.
4. **When You Don't Know** — say so, ask for help, research more

### Phase 4: Implementation

1. **Create Failing Test Case** — must have before fixing
2. **Implement Single Fix** — address root cause only, no "while I'm here" changes
3. **Verify Fix** — test passes? No other tests broken?
4. **If Fix Doesn't Work:**
   - < 3 attempts: Return to Phase 1 with new information
   - **≥ 3 attempts: STOP — question the architecture (see below)**

### If 3+ Fixes Failed: Question Architecture

Pattern: each fix reveals new coupling/shared state in a different place.

STOP and ask:
- Is this pattern fundamentally sound?
- Should we refactor architecture vs. keep fixing symptoms?

**Discuss with human partner before attempting more fixes.**

## Red Flags — STOP and Follow Process

- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- Proposing solutions before tracing data flow
- **"One more fix attempt" after already trying 2+**

**ALL of these mean: STOP. Return to Phase 1.**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic is FASTER than guess-and-check thrashing. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+) | 3+ failures = architectural problem. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## Related Skills

- `superpowers:verification-before-completion` — verify fix worked before claiming success
