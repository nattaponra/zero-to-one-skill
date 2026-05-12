---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development

Execute plan by dispatching fresh subagent per task, with two-stage review after each: spec compliance review first, then code quality review.

**Why subagents:** You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Core principle:** Fresh subagent per task + two-stage review (spec then quality) = high quality, fast iteration

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping. The only reasons to stop are: BLOCKED status you cannot resolve, ambiguity that genuinely prevents progress, or all tasks complete. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the plan, so execute it.

## The Process

1. Read plan → create TodoWrite task per item
2. For each task (in order, no parallel implementers):
   - Dispatch implementer subagent with full isolated context
   - Handle status (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED)
   - Dispatch spec compliance reviewer
   - If spec review passes → dispatch code quality reviewer
   - Mark task complete → continue to next

## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

- **Mechanical implementation tasks** (isolated functions, clear specs, 1-2 files): fast, cheap model
- **Integration and judgment tasks** (multi-file, pattern matching, debugging): standard model
- **Architecture, design, and review tasks**: most capable available model

## Handling Implementer Status

- **DONE:** Proceed to spec compliance review.
- **DONE_WITH_CONCERNS:** Read concerns before proceeding.
- **NEEDS_CONTEXT:** Provide missing context and re-dispatch.
- **BLOCKED:** Assess and resolve the blocker before retrying.

## Prompt Templates

### Implementer Prompt

```
You are an AI developer implementing a specific task.

## Task
[Full task spec from plan]

## Codebase Context
[Relevant files only — not full repo]

## Constraints
- Implement exactly what the spec describes
- Do not add features not in the spec
- Follow the tech stack constraints
- Write tests as specified
- Branch: feat/issue-[N]-[slug]

## Required Skills
- Use `frontend-design` skill (skills/frontend-design/SKILL.md) for any UI work
- Use `superpowers:verification-before-completion` before reporting SUCCESS
- Use `superpowers:systematic-debugging` if you hit a bug

## Output
Report one of: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
Include: files changed, test results, verification output
```

### Spec Compliance Reviewer Prompt

```
You are a spec compliance reviewer.

## Original Spec
[Full task spec]

## Implementation
[Files changed by implementer]

Check: Does implementation satisfy every acceptance criterion?
Report: APPROVED | NEEDS_REVISION
List any gaps with specific line references.
```

### Code Quality Reviewer Prompt

```
You are a code quality reviewer.

## Implementation
[Files changed]

## Tech Stack
[Stack constraints]

Check: Code quality, security, performance, maintainability.
Report: APPROVED | NEEDS_REVISION
List specific issues with file:line references.
```

## Red Flags

**Never:**
- Skip spec compliance review before code quality review
- Dispatch parallel implementers for the same task
- Inherit session context in subagent (construct context explicitly)
- Stop between tasks to ask "should I continue?"
- Mark complete without both reviews passing

**Always:**
- Fresh subagent per task
- Spec review before code quality review
- Construct full context explicitly for each subagent
- Continue until BLOCKED or all tasks done
