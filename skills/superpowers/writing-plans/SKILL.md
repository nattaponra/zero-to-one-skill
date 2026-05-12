---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch, code, how to test. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`

## Scope Check

If the spec covers multiple independent subsystems, suggest breaking into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each is responsible for.

- Design units with clear boundaries and well-defined interfaces
- Prefer smaller, focused files over large ones that do too much
- Files that change together should live together
- In existing codebases, follow established patterns

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" — step
- "Run it to make sure it fails" — step
- "Implement the minimal code to make the test pass" — step
- "Run the tests and make sure they pass" — step
- "Commit" — step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.ts`
- Modify: `exact/path/to/existing.ts:123-145`
- Test: `tests/exact/path/to/test.ts`

- [ ] **Step 1: Write the failing test**

```typescript
test('specific behavior', () => {
  const result = fn(input)
  expect(result).toBe(expected)
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pnpm test path/to/test.ts`
Expected: FAIL with "fn is not defined"

- [ ] **Step 3: Write minimal implementation**

```typescript
export function fn(input: string): string {
  return expected
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pnpm test path/to/test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.ts src/path/file.ts
git commit -m "feat: add specific feature"
```
````

## No Placeholders

**Never write:**
- "TBD", "TODO", "implement later"
- "Add appropriate error handling"
- "Write tests for the above" (without actual test code)
- Steps that describe what to do without showing how (code required)
- References to types/functions not defined in any task

## Self-Review

After writing the complete plan:

1. **Spec coverage** — can you point to a task for every requirement?
2. **Placeholder scan** — search for patterns from "No Placeholders" above
3. **Type consistency** — do method names/types match across all tasks?

Fix issues inline. No re-review needed — just fix and move on.

## Execution Handoff

After saving the plan, offer:

```
Plan complete and saved to docs/superpowers/plans/<filename>.md

Two execution options:
1. Subagent-Driven (recommended) — fresh subagent per task, review between tasks
2. Inline Execution — execute tasks in this session with checkpoints

Which approach?
```

**If Subagent-Driven chosen:** Use `superpowers:subagent-driven-development`
**If Inline chosen:** Execute tasks in session with checkpoints for review
