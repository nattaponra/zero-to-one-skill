# SKILL 04 — Build MVP

> **Role:** AI Engineering Lead + Subagent Orchestrator
> **Input:** `03_design_brief.md`
> **Output:** Working MVP + GitHub Issues + PRs
> **Tech Stack:** Next.js 16+ · MUI · Supabase · OpenRouter · Stripe
> **เวลาโดยประมาณ:** 2–4 สัปดาห์ (sprint-based)

---

## Overview

Skill นี้แบ่งเป็น 2 phase:

```
Phase A: PLAN  — อ่าน design brief → แตก GitHub Issues → review plan
Phase B: BUILD — execute issues ทีละอัน → update status → open PR → agent review
```

---

## FIXED TECH STACK

> ห้ามเปลี่ยน stack ต่อไปนี้ เว้นแต่มีเหตุผลที่ document ไว้ใน issue

| Layer | Technology | Version | เหตุผล |
|-------|-----------|---------|--------|
| Framework | Next.js (App Router) | 16+ | SSR, file-based routing, server actions |
| UI Library | MUI (Material UI) | v6+ | design system พร้อม, Thai font support |
| Database / Auth / Storage | Supabase | latest | BaaS ครบ, realtime, row-level security |
| LLM API | OpenRouter + SDK | latest | multi-model, fallback, cost control |
| Payment | Stripe | latest | webhook, subscription, Thai baht support |
| Language | TypeScript | 5+ | type safety ลด bug |
| Deployment | Vercel | - | Next.js native, preview per PR |

### Package Conventions

```bash
# Core
next@latest
@mui/material @mui/icons-material @emotion/react @emotion/styled
@supabase/supabase-js @supabase/ssr
openrouter  # หรือ openai SDK ชี้ไป openrouter base_url
stripe

# Dev
typescript tsx @types/node
eslint prettier husky lint-staged

# Testing
vitest @vitest/ui
@testing-library/react @testing-library/user-event @testing-library/jest-dom
playwright @playwright/test
msw                          # mock API / Supabase / OpenRouter ใน tests
```

---

## ENV MANAGEMENT

> **Rule:** ไม่มี hardcode secret ใน codebase ทุกค่าต้องมาจาก env

### `.env.development` (ใช้ระหว่าง dev — ไม่ commit)
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-local-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-local-service-role-key

# OpenRouter
OPENROUTER_API_KEY=sk-or-dev-xxxx
OPENROUTER_BASE_URL=https://openrouter.ai/api/v1
OPENROUTER_DEFAULT_MODEL=anthropic/claude-3-haiku   # ใช้ model ถูกระหว่าง dev

# Stripe
STRIPE_SECRET_KEY=sk_test_xxxx
STRIPE_WEBHOOK_SECRET=whsec_test_xxxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxxx

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### `.env.example` (commit ขึ้น repo — user copy ไปตั้งค่าเอง)
```bash
# ========================================
# SUPABASE — https://supabase.com/dashboard
# ========================================
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# ========================================
# OPENROUTER — https://openrouter.ai/keys
# ========================================
OPENROUTER_API_KEY=
OPENROUTER_BASE_URL=https://openrouter.ai/api/v1
OPENROUTER_DEFAULT_MODEL=anthropic/claude-3-5-sonnet   # เปลี่ยนได้

# ========================================
# STRIPE — https://dashboard.stripe.com/apikeys
# ========================================
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# ========================================
# APP CONFIG
# ========================================
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

> **AI Rule:** เมื่อใดก็ตามที่ต้องใช้ env var ใหม่ — ต้อง update `.env.example` ด้วยทันที

---

## PHASE A: WRITE PLAN TO GITHUB ISSUES

> ดัดแปลงจาก superpowers/writing-plans + plan-document-reviewer-prompt

### System Prompt (Phase A)

```
You are a senior engineering lead.
Your job is to read a Design Brief and produce a structured set of GitHub Issues
that AI subagents can execute independently.

Rules:
- Each issue = one atomic, independently testable unit of work
- Issues must have explicit acceptance criteria and verification commands
- Dependencies between issues must be declared (blockers)
- No issue should take more than 1 day to implement
- Use the Fixed Tech Stack — do not suggest alternatives
- Language in issues: English (code) + ภาษาไทย (description/context)
```

### Prompt Template (Phase A)

```
[SYSTEM PROMPT ด้านบน]

## Input: Design Brief
[วางเนื้อหาทั้งหมดของ 03_design_brief.md]

## Instructions
1. Produce a PLAN DOCUMENT (see schema below)
2. Then produce individual GITHUB ISSUE SPECS for each task
3. Flag any design gaps that block implementation as [BLOCKER] issues
```

---

### Plan Document Schema (`plan.md`)

```markdown
# MVP Build Plan
> Input: 03_design_brief.md | Tech Stack: Next.js · MUI · Supabase · OpenRouter · Stripe

## Milestone Overview
| Milestone | Issues | เป้าหมาย | Sprint |
|-----------|--------|---------|--------|
| M0: Foundation | #1–#5 | repo setup, auth, DB schema | Sprint 1 |
| M1: Core Features | #6–#15 | P0 features ทำงานได้ | Sprint 2–3 |
| M2: Payments | #16–#18 | Stripe integration | Sprint 3 |
| M3: Polish & Launch | #19–#25 | error handling, loading states, deploy | Sprint 4 |

## Dependency Graph
```
#1 (setup) → #2 (DB schema) → #3 (auth) → #4 (core feature A)
                                         → #5 (core feature B)
#2 → #16 (stripe) → #17 (webhook) → #18 (subscription gate)
```

## Risk Register
| Risk | Issue | Mitigation |
|------|-------|------------|
| Supabase RLS complexity | #2 | ทำ policy ทีละ table, มี test ทุกอัน |
| OpenRouter rate limit | #10 | implement retry + fallback model |
```

---

### GitHub Issue Spec Schema

> AI จะ generate issues ตาม template นี้ ทุกอัน

```markdown
---
## Issue #[N]: [ชื่อ feature/task]

**Labels:** `[milestone]`, `[type: feature|bug|chore]`, `[priority: p0|p1|p2]`
**Milestone:** [M0–M3]
**Blocked by:** #[N] (ถ้ามี)
**Estimated:** [0.5 | 1 | 2] วัน

### Context
[อธิบายว่า task นี้คืออะไร และทำไมต้องทำ — ภาษาไทย]

### User Story (ถ้า feature)
> เป็น [persona] ฉันต้องการ [action] เพื่อ [benefit]

### Technical Spec

**Files to create/modify:**
- `src/[path]` — [ทำอะไร]
- `src/[path]` — [ทำอะไร]

**Implementation notes:**
- [ข้อสังเกตหรือ pattern ที่ต้องใช้]
- [edge case ที่ต้องระวัง]
- [integration กับ stack ที่ต้องทำ]

**Database changes (ถ้ามี):**
```sql
-- migration ที่ต้องรัน
```

**ENV vars required:**
- `VAR_NAME` — [ใช้ทำอะไร]

### Acceptance Criteria
- [ ] [เงื่อนไข 1 — ตรวจสอบได้]
- [ ] [เงื่อนไข 2]
- [ ] [เงื่อนไข 3]
- [ ] `.env.example` updated (ถ้ามี env ใหม่)
- [ ] TypeScript — no `any` types
- [ ] No hardcoded secrets

### Test Cases

> **QA Agent จะใช้ section นี้ในการตรวจสอบ — ต้องกรอกครบทุกอัน**
> Format: `TC-[N]-[ISSUE_NUMBER]` | Type: `unit` | `integration` | `e2e` | `manual`

#### TC-[N]-[ISSUE]: [ชื่อ test case — happy path]
- **Type:** `e2e` / `unit` / `integration` / `manual`
- **Priority:** `critical` / `high` / `medium`
- **Pre-condition:** [สิ่งที่ต้องมีก่อน test นี้ทำงานได้]

**Test Steps:**
1. Given [สถานะเริ่มต้น / context]
2. When [action ที่ user หรือ system ทำ]
3. Then [ผลลัพธ์ที่คาดหวัง — ตรวจสอบได้]

**Expected Result:** [อธิบายผลที่ถูกต้องอย่างละเอียด]
**Test Data:** [ข้อมูลที่ใช้ทดสอบ เช่น email, amount, input]

---

#### TC-[N+1]-[ISSUE]: [ชื่อ test case — sad path / edge case]
- **Type:** `e2e` / `unit`
- **Priority:** `high`
- **Pre-condition:** [...]

**Test Steps:**
1. Given [...]
2. When [action ผิดพลาด / ข้อมูลไม่ถูกต้อง / network error]
3. Then [error message ที่ถูกต้อง / fallback behavior]

**Expected Result:** [...]
**Test Data:** [invalid data, edge values]

[ทำซ้ำสำหรับทุก test case — ขั้นต่ำ: 1 happy path + 1 sad path ต่อ feature]

### Verification Commands
```bash
# รันคำสั่งนี้เพื่อยืนยันว่า task เสร็จแล้ว
[คำสั่ง 1]
[คำสั่ง 2]

# Run unit/integration tests สำหรับ issue นี้
pnpm vitest run [test-file-path]

# Run E2E tests (ถ้ามี)
pnpm playwright test [spec-file-path]
```

### Definition of Done
- [ ] Code ผ่าน `pnpm lint && pnpm type-check`
- [ ] ถ้ามี logic → มี unit/integration test ครอบทุก test case ใน issue
- [ ] ถ้าเป็น UI → มี E2E test หรือ manual test ตาม steps ที่กำหนด
- [ ] ถ้าเป็น UI → screenshot/recording แนบใน PR
- [ ] Acceptance criteria ผ่านทุกข้อ
- [ ] **QA Agent ตรวจผ่าน** — status: `qa-passed`
---
```

---

### Plan Reviewer Prompt

> ก่อน execute ให้รัน prompt นี้กับ plan ที่เพิ่ง generate — เพื่อ review quality

```
You are a critical plan reviewer. Read the plan and issues below.
Flag any of the following problems:

BLOCKERS (must fix before executing):
- Issue ที่ไม่มี acceptance criteria วัดได้
- Issue ที่ไม่มี Test Cases (ทุก issue บังคับ ≥ 1 happy path + ≥ 1 sad path)
- Issue ที่ depend on อีก issue แต่ไม่ได้ declare blocker
- Issue ที่ใหญ่เกินไป (> 2 วัน) — ต้อง break down
- Missing migration หรือ schema change ที่ feature อื่นต้อง depend on
- ENV var ที่ใช้แต่ไม่อยู่ใน .env.example

WARNINGS (should fix):
- Issue ที่ description คลุมเครือ
- Missing error handling spec
- Test case ที่ไม่มี step ละเอียดพอ (ต้องมี Given/When/Then หรือ numbered steps)
- ไม่มี negative/edge case test สำหรับ feature ที่มี validation หรือ payment
- ไม่มี loading/empty state spec สำหรับ UI issues
- Tech decisions ที่ขัดกับ fixed stack

Output format:
## BLOCKERS
- Issue #N: [ปัญหา] → [วิธีแก้]

## WARNINGS
- Issue #N: [ปัญหา] → [วิธีแก้]

## APPROVED ISSUES
[list issues ที่ผ่านโดยไม่มี blocker]

## Verdict: APPROVED / NEEDS REVISION
```

---

## PHASE B: EXECUTE (Subagent-Driven Development)

> ดัดแปลงจาก superpowers/executing-plans + subagent-driven-development

### Orchestrator System Prompt

```
You are an AI engineering orchestrator.
You manage a queue of GitHub Issues and dispatch them to subagents for implementation.

Your responsibilities:
1. Pick the next executable issue (no unresolved blockers)
2. Spawn a subagent with full context for that issue
3. Collect the result and update the issue status
4. Open a PR when a feature group is complete
5. Invoke the PR Review Agent on each PR

You do NOT write implementation code directly.
You ONLY orchestrate, track status, and escalate blockers.
```

### Execution Loop

```
WHILE issues remain in queue:

  1. SELECT next issue where:
     - status = 'open'
     - all blockers = 'closed'
     - priority = highest

  2. UPDATE issue:
     - Add label: `status: in-progress`
     - Add comment: "🤖 Subagent started — [timestamp]"

  3. SPAWN subagent with:
     - Issue spec (full content)
     - Codebase context (relevant files only)
     - Tech stack constraints
     - .env.development values

  4. SUBAGENT executes:
     - Implement on branch: `feat/issue-[N]-[slug]`
     - Write tests ตาม Test Cases ที่กำหนดใน issue
     - Run verification commands
     - Report: SUCCESS | BLOCKED | FAILED

  4.5 SPAWN QA AGENT (ทันทีหลัง subagent SUCCESS):
     - QA Agent รับ: issue spec + test cases + code ที่ implement
     - QA Agent รัน test cases ทุกอัน (automated + manual steps)
     - QA Agent รายงาน: QA_PASSED | QA_FAILED

  5a. If SUCCESS + QA_PASSED:
      - UPDATE issue: label `status: done`, `qa-passed`, close issue
      - Add comment with: files changed, test results, QA report, screenshot (if UI)

  5b. If SUCCESS + QA_FAILED:
      - UPDATE issue: label `status: in-progress`, `qa-failed`
      - Add comment: QA report + failed test cases
      - Subagent แก้ไขตาม QA report → loop กลับ step 4
      - หลัง 2 รอบยังไม่ผ่าน → label `status: blocked` → alert human

  5c. If BLOCKED:
      - UPDATE issue: label `status: blocked`
      - Add comment: "🚫 Blocked: [reason] — needs human input"
      - STOP this issue, move to next

  5d. If FAILED (after 2 retries):
      - UPDATE issue: label `status: failed`
      - Add comment: error log + what was attempted
      - STOP and alert human

  6. WHEN milestone complete → OPEN PR (see below)
```

---

### Subagent Prompt Template

> Orchestrator inject ค่านี้สำหรับแต่ละ issue

```
You are an AI developer. Implement exactly what the issue describes.
Do not add features not in the spec. Do not change the tech stack.

## Your Task
[วาง issue spec ทั้งหมดที่นี่]

## Codebase Context
[วาง content ของ files ที่เกี่ยวข้อง]

## Tech Stack Constraints
- Next.js App Router — use server components by default, client only when needed
- MUI — use sx prop, not inline style. Theme via ThemeProvider
- Supabase — always use @supabase/ssr for server-side, never expose service_role to client
- OpenRouter — use SDK, never fetch() directly. Handle rate limit with exponential backoff
- Stripe — webhook must verify signature before processing
- Stripe — ใช้ `subscription.status` และ `price_id` จาก webhook เพื่ออัปเดต `user_plans` table ใน Supabase เสมอ
- Feature gating — check plan ที่ **server side เท่านั้น** (Route Handler / Server Action) — ห้าม trust client
- Usage limits — นับและ check ใน `usage_logs` table ก่อนทุก operation ที่มี quota — return 403 ถ้าเกิน
- TypeScript — strict mode, no `any`, no `as unknown as X`

## ENV Available
[วาง .env.development ที่นี่ — ลบ value ที่เป็น secret จริง]

## Output Required
1. All file contents (complete, not diff)
2. Migration SQL (if any)
3. Verification command results
4. List of acceptance criteria: PASS / FAIL

## Branch
Work on: feat/issue-[N]-[slug]
Do NOT touch: main, .env.development
```

---

### Issue Status Labels

```
open          → รอดำเนินการ
in-progress   → subagent กำลัง implement
qa-in-progress → QA Agent กำลังตรวจ
qa-failed     → test cases ไม่ผ่าน — subagent ต้องแก้
qa-passed     → ผ่าน QA ทุก test case
blocked       → ติดปัญหา รอ human
failed        → ล้มเหลว 2 ครั้ง
done          → เสร็จ ผ่าน QA และ criteria ทุกข้อ
```

---

## PR WORKFLOW

### เมื่อไหรจะเปิด PR
- เมื่อ milestone ครบ (M0, M1, M2, M3)
- หรือเมื่อ feature group สมบูรณ์ (> 3 issues done บน branch เดิม)

### PR Template

```markdown
## Summary
[อธิบายสิ่งที่ implement ใน milestone นี้ — 2–3 ประโยค]

## Issues Closed
Closes #[N], #[N], #[N]

## Changes
| File | Change |
|------|--------|
| | |

## Screenshots / Recordings (UI changes)
[แนบ screenshot ทุก screen ที่เปลี่ยน]

## Verification
- [ ] `pnpm lint` — passed
- [ ] `pnpm type-check` — passed
- [ ] `pnpm test` — passed ([X] tests)
- [ ] `pnpm playwright test` — passed ([X] E2E tests)
- [ ] All acceptance criteria in linked issues — passed
- [ ] **QA Agent report attached** — all test cases: PASSED
- [ ] `.env.example` up to date
- [ ] No secrets in codebase (`git grep -i "sk_live\|sk_test\|service_role"` — clean)

## QA Summary
> copy จาก QA Agent report

| Issue | Test Cases | Passed | Failed | Coverage |
|-------|-----------|--------|--------|----------|
| #[N] | [total] | [n] | 0 | [%] |

**Failed test cases:** None / [รายการถ้ามี]

## ENV Changes
[NEW vars added to .env.example — หรือ "None"]

## Migration
[SQL migration ที่ต้องรัน — หรือ "None"]

## Known Issues / Follow-ups
[สิ่งที่ยังไม่ได้ทำ หรือ technical debt]
```

---

## QA AGENT

> รันทันทีหลัง subagent implement เสร็จ **ก่อน** เปิด PR
> ทำงานคู่กับ PR Review Agent — ต้องผ่านทั้งคู่จึง merge ได้

### QA Agent System Prompt

```
You are a QA engineer. Your job is to validate that an implementation
matches every test case defined in the GitHub Issue.

You do NOT review code quality or architecture — that is the Code Review Agent's job.
You ONLY verify: does the feature behave exactly as the test cases specify?

Be strict. A test case PASSES only when every step produces exactly the expected result.
Partial pass = FAIL.
```

### QA Agent Prompt Template

```
You are a QA Agent. Execute all test cases for this issue.

## Issue Being Tested
[วาง issue spec ทั้งหมด — โดยเฉพาะ Test Cases section]

## Implementation Context
[วาง: files changed, migration run, app URL (Vercel preview)]

## Available Test Environment
- App URL: [Vercel preview URL ของ branch นี้]
- Test DB: [Supabase local / staging]
- Test accounts: [email/password ของ test users]
- Stripe test cards: 4242424242424242 (success), 4000000000000002 (decline)

## Instructions
For each test case in the issue:
1. Execute every step exactly as written
2. Verify the expected result
3. Record actual result
4. Mark: PASS / FAIL / BLOCKED

Output the QA Report using the schema below.
```

### QA Report Schema

```markdown
# QA Report — Issue #[N]
> QA Agent | Branch: feat/issue-[N]-[slug] | Date: [วันที่]

## Summary
| Total TCs | Passed | Failed | Blocked | Result |
|-----------|--------|--------|---------|--------|
| [N] | [N] | [N] | [N] | ✅ QA_PASSED / ❌ QA_FAILED |

---

## Test Case Results

### TC-[N]-[ISSUE]: [ชื่อ test case]
- **Status:** ✅ PASS / ❌ FAIL / ⚠️ BLOCKED
- **Type:** unit / e2e / manual
- **Executed steps:**
  1. Given [...] → ✅ confirmed
  2. When [...] → ✅ executed
  3. Then [...] → ✅ matched / ❌ got [actual result] instead
- **Actual Result:** [สิ่งที่เกิดขึ้นจริง]
- **Expected Result:** [สิ่งที่ควรเกิด]
- **Evidence:** [screenshot URL / console log / response body]
- **Failure Reason:** [ถ้า FAIL — อธิบายสาเหตุ]

[ทำซ้ำทุก TC]

---

## Regression Check
> ตรวจว่า implementation นี้ไม่ทำลาย feature เดิม

| Area | Status | หมายเหตุ |
|------|--------|---------|
| Authentication flow | ✅ / ❌ | |
| [Feature ที่เกี่ยวข้อง] | ✅ / ❌ | |

---

## Bugs Found (นอกเหนือจาก test cases)
| Bug | Severity | Steps to Reproduce |
|-----|----------|-------------------|
| [ถ้าพบ] | critical/high/medium/low | |

---

## QA Decision

### ✅ QA_PASSED
> ทุก test case ผ่าน — ready for Code Review

### ❌ QA_FAILED — must fix before PR:
- TC-[N]: [สิ่งที่ต้องแก้]
- TC-[N]: [สิ่งที่ต้องแก้]
```

---

## PR REVIEW AGENT

> รันหลังจาก open PR **คู่กับ QA Agent** — ต้องผ่านทั้งคู่จึง merge ได้

```
PR merged ต่อเมื่อ:
  QA Agent    → QA_PASSED   ✅
  Code Review → APPROVE     ✅
```

### Review Agent Prompt

```
You are a senior code reviewer. Review this PR strictly.
Note: QA testing has already been done by the QA Agent — you do NOT re-run test cases.
Your job is code quality, security, architecture, and spec compliance.
Output: APPROVE or REQUEST_CHANGES with specific comments.

## Review Criteria

### Must REJECT if:
- [ ] QA Agent report missing หรือ status = QA_FAILED
- [ ] TypeScript errors หรือ `any` types
- [ ] Hardcoded secret หรือ API key
- [ ] Supabase service_role key ถูกส่งไป client
- [ ] Missing error handling บน async operations
- [ ] ENV var ใหม่ไม่อยู่ใน .env.example
- [ ] Acceptance criteria ข้อใดข้อหนึ่งไม่ผ่าน
- [ ] SQL migration ที่ทำลาย existing data
- [ ] Test files ขาดหายสำหรับ test cases ใน issue

### Should REQUEST CHANGES if:
- [ ] Component ขนาดใหญ่เกิน 200 บรรทัดโดยไม่มีเหตุผล
- [ ] Logic ซ้ำซ้อนที่ควร extract เป็น utility
- [ ] Missing loading state หรือ empty state บน UI
- [ ] OpenRouter call ที่ไม่มี error handling / retry
- [ ] Stripe webhook ที่ไม่มี signature verification
- [ ] เปลี่ยน tech stack โดยไม่มี documented reason

### May APPROVE with comments if:
- [ ] Style issues เล็กน้อย (ไม่ block merge)
- [ ] Nice-to-have improvements สำหรับ iteration ถัดไป

## Output Format

### Decision: APPROVE | REQUEST_CHANGES | REJECT

### Summary
[2–3 ประโยคสรุป quality ของ PR]

### Issues Found
| Severity | File | Line | Issue | Fix Required |
|----------|------|------|-------|-------------|
| BLOCKER | | | | Yes |
| WARNING | | | | No |

### For REQUEST_CHANGES — must fix before merge:
- [ ] [สิ่งที่ต้องแก้ 1]
- [ ] [สิ่งที่ต้องแก้ 2]

### Positive Notes
[สิ่งที่ทำได้ดี — เพื่อ learning]
```

### Merge Rules

```
QA_PASSED  + APPROVE          → Auto-merge to main ✅
QA_PASSED  + REQUEST_CHANGES  → Subagent แก้ code → re-review (QA ไม่ต้องรันใหม่)
QA_FAILED  + (any)            → Subagent แก้ → QA รันใหม่ → re-review
QA_PASSED  + REJECT           → Alert human immediately 🚨
QA_FAILED  + REJECT           → Alert human immediately 🚨
```

---

## PROJECT STRUCTURE (Baseline)

```
my-product/
├── .env.example              ← commit นี้
├── .env.development          ← ห้าม commit
├── .gitignore
├── package.json
├── tsconfig.json
├── next.config.ts
│
├── src/
│   ├── app/                  ← App Router pages
│   │   ├── page.tsx          ← / Landing Page (public)
│   │   ├── pricing/
│   │   │   └── page.tsx      ← /pricing
│   │   ├── (auth)/           ← route group — ไม่มีผลต่อ URL
│   │   │   ├── login/
│   │   │   │   └── page.tsx  ← /login
│   │   │   └── signup/
│   │   │       └── page.tsx  ← /signup
│   │   ├── app/              ← /app/* — protected routes ทั้งหมด
│   │   │   ├── layout.tsx    ← auth guard (redirect ถ้าไม่มี session)
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx  ← /app/dashboard
│   │   │   ├── [feature]/
│   │   │   │   └── page.tsx  ← /app/[feature]
│   │   │   └── settings/
│   │   │       ├── page.tsx  ← /app/settings
│   │   │       └── billing/
│   │   │           └── page.tsx ← /app/settings/billing
│   │   ├── api/
│   │   │   ├── stripe/
│   │   │   │   └── webhook/route.ts
│   │   │   └── [feature]/route.ts
│   │   └── layout.tsx        ← root layout (font, theme, providers)
│   │
│   ├── components/           ← UI components (MUI-based)
│   │   ├── ui/               ← reusable primitives
│   │   └── [feature]/        ← feature-specific
│   │
│   ├── lib/                  ← core utilities
│   │   ├── supabase/
│   │   │   ├── client.ts     ← browser client
│   │   │   └── server.ts     ← server client (SSR)
│   │   ├── openrouter/
│   │   │   ├── client.ts
│   │   │   └── models.ts
│   │   └── stripe/
│   │       ├── client.ts
│   │       └── webhook.ts
│   │
│   ├── types/                ← TypeScript types (generated from Supabase)
│   └── utils/
│
├── tests/
│   ├── unit/                 ← Vitest — logic, utils, hooks
│   │   └── [feature]/
│   │       └── [feature].test.ts
│   ├── integration/          ← Vitest + msw — API routes, DB interactions
│   │   └── [feature]/
│   │       └── [feature].integration.test.ts
│   ├── e2e/                  ← Playwright — full user flows
│   │   └── [feature]/
│   │       └── [feature].spec.ts
│   └── fixtures/             ← shared test data, mocks, factories
│       ├── users.ts
│       ├── supabase.mock.ts
│       └── stripe.mock.ts
│
├── supabase/
│   ├── migrations/           ← SQL migrations
│   └── seed.sql
│
└── docs/
    ├── plan.md               ← Phase A output
    └── decisions.md          ← Architecture decision log
```

---

## FOUNDATION ISSUES (M0 — ทุก project ต้องมี)

> AI จะ generate issues เพิ่มเติมจาก design brief แต่ M0 นี้คือ baseline ทุก project

```markdown
#1 Project Setup
- Init Next.js 16+ App Router + TypeScript strict
- Install MUI, Supabase, OpenRouter, Stripe
- Install Vitest, Playwright, Testing Library, msw
- Setup ESLint + Prettier + Husky
- Create .env.example และ .env.development
- Setup Vercel project + connect GitHub

#2 Database Schema & Migrations
- Design Supabase tables จาก PRD
- Write migration SQL
- Enable RLS บนทุก table
- Generate TypeScript types (supabase gen types)

#3 Authentication
- Supabase Auth — email/password + magic link
- Middleware: protect /dashboard routes
- Auth UI pages: login, signup, forgot password
- Session handling via @supabase/ssr

#4 Theme & Design System
- MUI ThemeProvider — colors จาก design brief
- Typography: font family, sizes
- Shared layout: header, sidebar/nav
- Dark mode support (optional)

#5 CI/CD + QA Pipeline
- GitHub Actions: lint + type-check + vitest on every PR
- GitHub Actions: playwright E2E on PR to main
- QA Agent trigger: comment on PR เมื่อ checks pass
- Vercel preview deployment per PR (QA ใช้ URL นี้ทดสอบ)
- .env injection for preview (Vercel env vars)

#6 Pricing & Plan Infrastructure
- สร้าง Stripe Products + Prices ตาม tier ใน Feature Matrix (Step 02 Section 7.3)
- DB migration: table `user_plans` (user_id, plan, stripe_customer_id, stripe_subscription_id, current_period_end)
- DB migration: table `usage_logs` (user_id, feature, count, period_start) + RLS
- Stripe webhook handler `/api/stripe/webhook`:
  - `checkout.session.completed` → upsert `user_plans`
  - `customer.subscription.updated` → update plan + period
  - `customer.subscription.deleted` → downgrade to Free
- Server utility `getPlan(userId)` → return plan tier จาก `user_plans`
- Server utility `checkUsage(userId, feature)` → return { used, limit, allowed }
- Server utility `incrementUsage(userId, feature)` → เพิ่ม count ใน `usage_logs`
- Middleware: inject plan ใน session context เพื่อให้ทุก Route Handler เรียกได้เลย
```

---

## VALIDATION CHECKLIST (คุณตรวจก่อน execute)

- [ ] Plan document ผ่าน Plan Reviewer — verdict: APPROVED
- [ ] ทุก issue มี acceptance criteria ≥ 3 ข้อ
- [ ] **ทุก issue มี test cases ≥ 2 อัน (1 happy path + 1 sad path)**
- [ ] ทุก test case มี steps ครบ (Given/When/Then) และ Expected Result
- [ ] M0 issues (#1–#6) อยู่ใน queue ก่อนทุกอย่าง
- [ ] Feature Matrix จาก Step 02 Section 7.3 ถูก implement ครบใน `user_plans` + server utilities
- [ ] ทุก feature ที่ lock มี server-side guard — ไม่ใช่แค่ hide UI
- [ ] `.env.example` มีทุก var ที่ใช้ใน codebase
- [ ] Dependency graph ไม่มี circular dependency
- [ ] ไม่มี issue ที่ใหญ่กว่า 2 วัน
- [ ] CI/CD pipeline (#5) รัน vitest + playwright ก่อน QA Agent ทำงาน