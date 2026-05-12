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

### Value-First Build Order

> **หลักการ:** พิสูจน์ value ก่อนลงทุนกับ infrastructure
> ห้ามให้ user รอ 1–2 สัปดาห์โดยไม่ได้เห็น product ทำงานจริง

```
V0: Value Core    ← BUILD นี้ก่อน — ใช้ mock auth + mock data
   │ 👤 Human Checkpoint: "นี่คือ product ที่ต้องการไหม?"
   ▼
M0: Foundation    ← เฉพาะหลัง V0 ผ่านแล้ว — real auth, real DB, CI/CD
   ▼
M1: Core Features ← build V0 features ใหม่บน real infrastructure
   ▼
M2: Payments      ← Stripe + subscription gate
   ▼
M3: Polish & Launch
```

**V0 คืออะไร:**
- Core feature ทำงานได้จริง (real UI + real AI/logic)
- Mock auth — hardcoded user session ไม่ต้องสมัครสมาชิก
- Mock data — in-memory หรือ hardcoded ไม่ต้องมี DB จริง
- Deploy ได้ทันทีบน Vercel เป็น demo URL
- เป้าหมาย: **แสดงให้ user/stakeholder เห็นใน 2–3 วัน**

**ห้ามทำสิ่งเหล่านี้ใน V0:**
- Real authentication (Supabase Auth)
- Database migrations และ RLS
- Stripe / payments
- CI/CD pipeline
- Error handling ครบถ้วน

**เริ่ม M0 ก็ต่อเมื่อ:** Human approve V0 แล้วเท่านั้น

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
> **Stack:** Supabase cloud + Vercel — ไม่ต้องใช้ Docker หรือ local Supabase

### Supabase Projects (แยก dev / prod)

| Environment | Supabase Project | Vercel |
|-------------|-----------------|--------|
| Development | `<project>-dev` | preview branches |
| Production | `<project>-prod` | main branch |

> สร้าง 2 Supabase projects แยกกันตั้งแต่เริ่ม — ป้องกัน dev data ปน prod

### `.env.development` (ใช้ระหว่าง dev — ไม่ commit)
```bash
# Supabase — Dev project (https://supabase.com/dashboard)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-dev-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-dev-service-role-key

# OpenRouter
OPENROUTER_API_KEY=sk-or-dev-xxxx
OPENROUTER_BASE_URL=https://openrouter.ai/api/v1
OPENROUTER_DEFAULT_MODEL=anthropic/claude-3-haiku   # ใช้ model ถูกระหว่าง dev

# Stripe — Test mode (https://dashboard.stripe.com/test/apikeys)
STRIPE_SECRET_KEY=sk_test_xxxx
STRIPE_WEBHOOK_SECRET=whsec_test_xxxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxxx

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### `.env.example` (commit ขึ้น repo)
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
OPENROUTER_DEFAULT_MODEL=anthropic/claude-3-5-sonnet

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

## DEVELOPMENT SETUP

> ไม่ต้องใช้ Docker — รัน local + test บน Supabase cloud + Vercel preview

---

### CREDENTIAL REQUEST GUIDE

> AI จะถามเฉพาะ credential ที่ต้องใช้ตอนนั้น — ไม่ถามทุกอย่างพร้อมกันตั้งแต่ต้น

```
V0 scaffold  → ถาม: OpenRouter API key
M0 auth/DB   → ถาม: Supabase dev project keys + Vercel setup
M2 payments  → ถาม: Stripe test keys + webhook
```

---

#### เมื่อถึง V0 (#1 scaffold) — ถาม:

```
ต้องการ OpenRouter API key สำหรับรัน AI feature

1. ไปที่ openrouter.ai → Sign in → Keys → Create Key
   ตั้งชื่อ: <app-name>-dev
2. copy key มาให้: sk-or-xxxx

จะใส่ใน .env.development ให้อัตโนมัติ
```

---

#### เมื่อถึง M0 (#4 Project Setup) — ถาม:

**ส่วนที่ 1: GitHub + Vercel**
```
ต้องการ GitHub repo และ Vercel project

GitHub:
1. ไปที่ github.com → New repository → ตั้งชื่อ → Create
2. copy repo URL มาให้

GitHub CLI login (ถ้ายังไม่เคย):
3. รันคำสั่งนี้บน terminal:
   gh auth login
   (เลือก GitHub.com → HTTPS → Login with a web browser)
   ใช้สำหรับ: create/label/comment/close issues และ open PRs

Vercel CLI login (ถ้ายังไม่เคย):
4. รันคำสั่งนี้บน terminal:
   npm install -g vercel
   vercel login
   (เลือก login ด้วย GitHub / Email ตามต้องการ)

Vercel Project:
4. ไปที่ vercel.com → Add New Project → Import repo นั้น
5. Framework: Next.js → Deploy (จะ fail ได้ — ไม่เป็นไร)
6. copy Project URL มาให้ (เช่น https://my-app.vercel.app)
```

**ส่วนที่ 2: Supabase**
```
ต้องการ Supabase 2 projects (dev + prod)

DEV PROJECT:
1. supabase.com → New project
   - ชื่อ: <app-name>-dev
   - Region: Southeast Asia (Singapore)
   - ตั้ง Database Password → จำไว้
2. รอ ~2 นาที → Settings → API → copy:
   - Project URL
   - anon public key
   - service_role key  ⚠️ เก็บเป็นความลับ

PROD PROJECT:
3. ทำซ้ำ → ชื่อ: <app-name>-prod → copy keys เก็บไว้แยกกัน

copy ค่าทั้งหมดมาให้ — จะตั้ง .env.development และ Vercel env vars ให้
```

**ส่วนที่ 3: Vercel + Supabase Integration (inject env vars อัตโนมัติ)**
```
เชื่อม Vercel กับ Supabase เพื่อ sync keys อัตโนมัติ:

1. vercel.com/integrations/supabase → Add Integration
2. เลือก Vercel project → เลือก Supabase dev project
   → environment: Preview → Connect
3. ทำซ้ำ → เลือก Supabase prod project
   → environment: Production → Connect

Vercel จะ inject SUPABASE_URL, ANON_KEY, SERVICE_ROLE_KEY ให้อัตโนมัติทุก deploy
```

**ส่วนที่ 4: Supabase CLI link**
```bash
# รันบน local ครั้งเดียว
supabase login
supabase link --project-ref <project-ref>
# project-ref อยู่ใน URL: supabase.com/dashboard/project/<project-ref>
```

---

#### เมื่อถึง M2 (#20 Stripe setup) — ถาม:

```
ต้องการ Stripe test keys สำหรับ payment

1. stripe.com → Login → ตรวจสอบว่าอยู่ใน Test mode (toggle มุมบนขวา)
2. Developers → API keys → copy:
   - Publishable key (pk_test_...)
   - Secret key (sk_test_...)        ⚠️ เก็บเป็นความลับ

copy มาให้ — จะตั้ง .env.development และ Vercel env vars ให้
```

```
Stripe Webhook (หลัง deploy preview แล้ว):

1. Stripe dashboard → Developers → Webhooks → Add endpoint
2. Endpoint URL: https://<vercel-preview-url>/api/stripe/webhook
3. Select events:
   - checkout.session.completed
   - customer.subscription.updated
   - customer.subscription.deleted
4. copy Signing secret (whsec_...) มาให้
```

---

### Prerequisites (ติดครั้งเดียวบนเครื่อง dev)

```bash
npm install -g pnpm

# Supabase CLI — สำหรับ push migrations และ gen types เท่านั้น (ไม่ต้อง Docker)
brew install supabase/tap/supabase

# Stripe CLI — สำหรับ test webhooks บน local (M2+)
brew install stripe/stripe-cli/stripe
```

---

### V0 Setup (2 นาที)

> V0 ใช้ mock auth + mock data — ต้องการแค่ OpenRouter key

```bash
git clone <repo-url> && cd <project>
pnpm install
cp .env.example .env.development
# แก้ OPENROUTER_API_KEY=sk-or-xxxx

pnpm dev   # → http://localhost:3000
```

---

### M0+ Setup (full stack — หลัง V0 ผ่านแล้ว)

```bash
# 1. สร้าง Supabase dev project ที่ https://supabase.com/dashboard
#    → copy URL + ANON_KEY + SERVICE_ROLE_KEY ใส่ .env.development

# 2. Link CLI กับ cloud project (ครั้งแรกครั้งเดียว)
supabase link --project-ref <project-ref>

# 3. Push migrations ขึ้น cloud
pnpm db:push

# 4. Seed ข้อมูลทดสอบ (optional)
pnpm db:seed

# 5. Generate TypeScript types จาก schema
pnpm db:types

# 6. รัน dev server
pnpm dev   # → http://localhost:3000 (ต่อ Supabase cloud dev project)
```

---

### M2+ Stripe Webhook (local testing)

```bash
# Terminal 1
pnpm dev

# Terminal 2
stripe login
stripe listen --forward-to localhost:3000/api/stripe/webhook
# copy "whsec_..." → STRIPE_WEBHOOK_SECRET ใน .env.development
```

---

### Vercel Preview (test บน cloud ก่อน merge)

```
Push branch → GitHub → Vercel สร้าง preview URL อัตโนมัติ
Preview URL ใช้ env vars จาก Vercel dashboard (ชี้ไป Supabase dev project)
```

**Vercel env vars ที่ต้องตั้ง (Settings → Environment Variables):**
- Preview & Development: ใช้ Supabase dev project keys
- Production: ใช้ Supabase prod project keys

---

### Test Commands

```bash
pnpm test            # unit + integration (Vitest)
pnpm test:watch      # watch mode
pnpm test:e2e        # E2E (Playwright) — ต้องมี pnpm dev รันอยู่
pnpm test:e2e:ui     # Playwright visual mode
pnpm type-check      # TypeScript
pnpm lint            # ESLint
```

### `package.json` scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint . --ext .ts,.tsx",
    "type-check": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "db:push": "supabase db push",
    "db:seed": "supabase db seed",
    "db:types": "supabase gen types typescript --linked > src/types/database.ts"
  }
}
```

---

### Dev Checklist (ก่อน push ทุกครั้ง)

```bash
pnpm lint && pnpm type-check && pnpm test
```

- [ ] `pnpm dev` รันได้ ต่อ Supabase cloud dev project ได้
- [ ] feature ทำงานได้บน local + Vercel preview URL
- [ ] `pnpm test` ผ่านทุกอัน
- [ ] ไม่มี secret ติดมาใน commit

---

## PHASE A: WRITE PLAN TO GITHUB ISSUES

> **REQUIRED SKILL:** ใช้ `superpowers:writing-plans` ในขั้นตอนนี้
> บันทึก plan ไปที่ `docs/superpowers/plans/YYYY-MM-DD-<feature>.md`

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

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` to execute this plan task-by-task.

## Milestone Overview
| Milestone | Issues | เป้าหมาย | Sprint | Gate |
|-----------|--------|---------|--------|------|
| **V0: Value Core** | #1–#3 | core feature ทำงานได้ด้วย mock — demo-able | Sprint 0 (2–3 วัน) | 👤 Human must approve ก่อน M0 |
| M0: Foundation | #4–#9 | repo setup จริง, auth จริง, DB schema, CI/CD | Sprint 1 | — |
| M1: Core Features | #10–#19 | P0 features บน real infrastructure | Sprint 2–3 | — |
| M2: Payments | #20–#22 | Stripe integration + plan gating | Sprint 3 | — |
| M3: Polish & Launch | #23–#28 | error handling, loading states, deploy | Sprint 4 | — |

## Dependency Graph
```
V0 (#1–#3: mock core feature) ──→ 👤 Human Checkpoint
                                         │
                                         ▼
#4 (setup) → #5 (DB schema) → #6 (auth) → #10 (core feature real)
                                         → #11 (core feature B real)
#5 → #20 (stripe) → #21 (webhook) → #22 (subscription gate)
```

## Value Priority Rule
> เมื่อมี 2 issues ที่ไม่มี blocker — เลือก issue ที่ user เห็น value ก่อนเสมอ
> infrastructure issue ที่ไม่ block value delivery → ทำทีหลัง

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

### After Plan Approved — Create Issues on GitHub

> **เมื่อ Plan Reviewer บอก APPROVED — ต้องสร้าง issues บน GitHub ทันที ด้วย `gh` CLI**
> ห้ามแค่บันทึก plan.md แล้วหยุด — issues ต้องอยู่บน GitHub เพื่อให้ Orchestrator track ได้

```bash
# 1. ตรวจว่า gh auth ผ่านแล้ว
gh auth status

# 2. สร้าง labels (ถ้ายังไม่มี — ดู GitHub Issue Management section)

# 3. สร้างทุก issue จาก plan — ตัวอย่าง:
gh issue create \
  --title "#1: Next.js scaffold + project structure" \
  --body "$(cat docs/superpowers/plans/issues/issue-01.md)" \
  --label "V0,status: open"

# ทำซ้ำทุก issue ใน plan
# ลำดับ: V0 issues (#1-#3) ก่อน, แล้ว M0 (#4+)

# 4. ตรวจว่า issues ขึ้นครบ
gh issue list --state open
```

**ห้ามข้ามขั้นตอนนี้** — ถ้าไม่มี issues บน GitHub, Orchestrator จะไม่มี queue ให้ track

---

## PHASE B: EXECUTE (Subagent-Driven Development)

> **REQUIRED SKILL:** ใช้ `superpowers:subagent-driven-development` ในขั้นตอนนี้
> Fresh subagent ต่อ 1 issue + two-stage review (spec compliance → code quality) ทุกอัน

### GitHub Issue Management — `gh` CLI Commands

> Orchestrator ต้องใช้ `gh` CLI ทุกครั้งที่ต้องการ create/update/close issue
> **ห้ามอธิบายว่า "ควรจะ update issue" — ต้องรัน `gh` command จริงๆ เสมอ**

#### ก่อนใช้ `gh` ครั้งแรก — ตรวจและแนะนำ user

```bash
# ตรวจว่า gh ติดตั้งแล้วหรือยัง
gh --version
```

ถ้าได้ error `command not found` → แจ้ง user ให้ติดตั้งก่อน:

```
GitHub CLI ยังไม่ได้ติดตั้ง — กรุณาติดตั้งก่อนดำเนินการต่อ:

macOS:
  brew install gh

Windows:
  winget install --id GitHub.cli
  หรือ https://cli.github.com/

Linux (apt):
  sudo apt install gh

หลังติดตั้งแล้ว รัน: gh auth login
```

```bash
# ตรวจว่า login แล้วหรือยัง
gh auth status
```

ถ้าได้ error `You are not logged into any GitHub hosts` → แจ้ง user ให้ login:

```
GitHub CLI ยังไม่ได้ login — กรุณารันคำสั่งนี้ก่อน:

  gh auth login

ขั้นตอน:
  1. เลือก: GitHub.com
  2. เลือก: HTTPS
  3. เลือก: Login with a web browser
  4. copy one-time code → เปิด browser → paste → Authorize
```

> **Orchestrator**: ตรวจ `gh --version` และ `gh auth status` ก่อนเริ่ม Phase B เสมอ
> ถ้า check ไม่ผ่าน → หยุดและแสดงข้อความแนะนำข้างบน ก่อนดำเนินการต่อ

---

```bash
# --- SETUP (ครั้งเดียว ต้องทำก่อน Phase B) ---
gh auth login                              # login ด้วย GitHub account

# --- สร้าง Labels ที่ใช้ใน pipeline ---
gh label create "status: open"      --color 0075ca
gh label create "status: in-progress" --color fbca04
gh label create "status: done"      --color 0e8a16
gh label create "status: blocked"   --color e4e669
gh label create "status: failed"    --color d93f0b
gh label create "qa-passed"         --color 0e8a16
gh label create "qa-failed"         --color d93f0b
gh label create "V0" --color 5319e7
gh label create "M0" --color 1d76db
gh label create "M1" --color 1d76db
gh label create "M2" --color 1d76db
gh label create "M3" --color 1d76db

# --- สร้าง Issue ---
gh issue create \
  --title "Issue #1: [ชื่อ]" \
  --body "$(cat <<'EOF'
## Description
...

## Acceptance Criteria
- [ ] ...

## Test Cases
...
EOF
)" \
  --label "V0,status: open"

# --- อ่าน Issues ที่ยังเปิดอยู่ ---
gh issue list --state open

# --- เริ่ม work บน issue (เปลี่ยน label + comment) ---
gh issue edit 1 --remove-label "status: open" --add-label "status: in-progress"
gh issue comment 1 --body "🤖 Subagent started — $(date)"

# --- issue done (QA passed) ---
gh issue edit 1 --remove-label "status: in-progress" --add-label "status: done,qa-passed"
gh issue comment 1 --body "✅ Done — files changed: [...], tests: passed"
gh issue close 1

# --- issue QA failed ---
gh issue edit 1 --add-label "qa-failed"
gh issue comment 1 --body "❌ QA Failed — [รายละเอียด test ที่ fail]"

# --- issue blocked ---
gh issue edit 1 --remove-label "status: in-progress" --add-label "status: blocked"
gh issue comment 1 --body "🚫 Blocked: [เหตุผล] — needs human input"

# --- open PR ---
gh pr create \
  --title "feat: [ชื่อ feature]" \
  --body "Closes #1, #2" \
  --base main \
  --head feat/issue-1-slug
```

### Orchestrator System Prompt

```
You are an AI engineering orchestrator.
You manage a queue of GitHub Issues and dispatch them to subagents for implementation.

Your responsibilities:
1. Pick the next executable issue (no unresolved blockers)
2. Spawn a subagent with full context for that issue
3. Collect the result and update the issue status using `gh` CLI commands
4. Open a PR when a feature group is complete
5. Invoke the PR Review Agent on each PR

You do NOT write implementation code directly.
You ONLY orchestrate, track status, and escalate blockers.

CRITICAL: Every status update, label change, and comment MUST be done via `gh` CLI.
Do NOT just say "I would update the issue" — actually run the commands.
Use the GitHub Issue Management section above for exact commands.
```

### Execution Loop

```
WHILE issues remain in queue:

  1. SELECT next issue where status = open and all blockers = closed
     → gh issue list --state open --label "status: open"

  2. RUN (ทันที — ห้ามแค่บอกว่าจะทำ):
     gh issue edit <N> --remove-label "status: open" --add-label "status: in-progress"
     gh issue comment <N> --body "🤖 Subagent started — $(date)"

  3. SPAWN subagent with:
     - Issue spec (full content from: gh issue view <N>)
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

  5a. If SUCCESS + QA_PASSED → RUN:
      gh issue edit <N> --remove-label "status: in-progress" --add-label "status: done,qa-passed"
      gh issue comment <N> --body "✅ Done — files: [...], tests: passed\n[QA report]"
      gh issue close <N>

  5b. If SUCCESS + QA_FAILED → RUN:
      gh issue edit <N> --add-label "qa-failed"
      gh issue comment <N> --body "❌ QA Failed\n[รายละเอียด test ที่ fail]"
      (Subagent ใช้ superpowers:systematic-debugging → แก้ → loop กลับ step 4)
      หลัง 2 รอบยังไม่ผ่าน → ทำ step 5c

  5c. If BLOCKED → RUN:
      gh issue edit <N> --remove-label "status: in-progress" --add-label "status: blocked"
      gh issue comment <N> --body "🚫 Blocked: [เหตุผล] — needs human input"
      STOP this issue, move to next

  5d. If FAILED (after 2 retries) → RUN:
      gh issue edit <N> --add-label "status: failed"
      gh issue comment <N> --body "💥 Failed after 2 retries\n[error log]"
      STOP and alert human

  6. WHEN V0 milestone complete:
     - Deploy demo URL ไป Vercel preview
     - 🛑 STOP — รอ Human Checkpoint ก่อน
     - แจ้ง human: "V0 demo พร้อมแล้วที่ [URL] — core feature ทำงานได้
       ก่อนลงทุนกับ infrastructure (auth, DB, CI/CD) ขอ confirm ว่านี่คือ
       product direction ที่ต้องการ?"
     - รอ approve → ถึงจะเริ่ม M0
     - ถ้า reject → กลับ Phase A ปรับ direction

  7. WHEN milestone M0/M1/M2/M3 complete → OPEN PR (see below)
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
- **ก่อน report SUCCESS ทุกครั้ง:** ใช้ `superpowers:verification-before-completion`
  (รัน verification commands จริง อ่าน output จริง — ห้ามอ้างว่า done โดยไม่มี evidence)
- **ถ้า QA_FAILED:** ใช้ `superpowers:systematic-debugging` — หา root cause ก่อน ห้าม patch

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

> **REQUIRED SKILL:** เมื่อ milestone complete ให้ใช้ `superpowers:finishing-a-development-branch`
> (verify tests → detect environment → present merge/PR/keep/discard options → execute)

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

## VALUE CORE ISSUES (V0 — ทำก่อนทุกอย่าง)

> เป้าหมาย: demo-able ใน 2–3 วัน — ใช้ mock ทุกอย่างที่ไม่ใช่ core value

```markdown
#1 Project Scaffold (minimal)
- Init Next.js App Router + TypeScript (ไม่ต้องมี ESLint/Husky ใน V0)
- Install MUI + OpenRouter SDK เท่านั้น
- สร้าง mock session: `lib/mock-session.ts` — hardcoded user object
  (ใช้แทน Supabase Auth ใน V0)
- สร้าง mock DB: `lib/mock-db.ts` — in-memory data store
  (ใช้แทน Supabase ใน V0)
- Deploy ขึ้น Vercel ทันที (ไม่รอ CI/CD)

#2 Core Value Feature — UI
- สร้าง screen หลักที่ deliver value จาก design brief
- ใช้ mock session + mock data
- ต้องทำงานได้จริง — ไม่ใช่ placeholder หรือ static mockup
- path: /app/[core-feature]

#3 Core Value Feature — Logic / AI
- implement business logic / AI call จริงด้วย OpenRouter
- ต่อเข้ากับ UI จาก #2
- ผล: user สามารถทำ [core job-to-be-done] ได้ครบจบในหน้าเดียว

---
🛑 HUMAN CHECKPOINT — deploy V0 demo → รอ approve ก่อนทำ #4 เป็นต้นไป
---
```

---

## FOUNDATION ISSUES (M0 — เริ่มหลัง V0 ผ่าน human checkpoint แล้ว)

> AI จะ generate issues เพิ่มเติมจาก design brief แต่ M0 นี้คือ baseline ทุก project

```markdown
#4 Project Setup (full)
- ต่อยอดจาก V0 scaffold — เพิ่ม ESLint, Prettier, Husky, lint-staged
- Install Supabase, Stripe, Vitest, Playwright, Testing Library, msw
- สร้าง .env.example และ .env.development (ชี้ไป Supabase cloud dev project)
- เพิ่ม scripts ทุกตัวใน package.json (dev, test, test:e2e, lint, type-check, db:*)
- `supabase link --project-ref <dev-project-ref>` — link CLI กับ cloud
- ตั้ง Vercel env vars: Preview → dev keys, Production → prod keys
- ยืนยัน `pnpm dev` รันได้ + ต่อ Supabase cloud ได้

#5 Database Schema & Migrations
- Design Supabase tables จาก PRD
- Write migration files ใน `supabase/migrations/`
- Enable RLS บนทุก table
- `pnpm db:push` — push migrations ขึ้น Supabase cloud dev project
- `pnpm db:types` — generate TypeScript types จาก schema

#6 Authentication
- Supabase Auth — email/password + magic link
- แทนที่ mock session จาก V0 ด้วย real auth
- Middleware: protect /app routes
- Auth UI pages: /login, /signup, forgot password
- Session handling via @supabase/ssr

#7 Theme & Design System
- MUI ThemeProvider — colors จาก design brief
- Typography: font family, sizes
- Shared layout: header, sidebar/nav
- Dark mode support (optional)

#8 CI/CD + QA Pipeline
- GitHub Actions: lint + type-check + vitest on every PR
- GitHub Actions: playwright E2E on PR to main
- QA Agent trigger: comment on PR เมื่อ checks pass
- Vercel preview deployment per PR (QA ใช้ URL นี้ทดสอบ)
- .env injection for preview (Vercel env vars)

#9 Pricing & Plan Infrastructure
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
- [ ] V0 issues (#1–#3) อยู่ใน queue ก่อนทุกอย่าง และ deploy ได้ใน 2–3 วัน
- [ ] V0 ไม่มี real auth / real DB / Stripe — ถ้ามีแสดงว่า scope ใหญ่เกินไป
- [ ] M0 issues (#4–#9) อยู่ใน queue หลัง V0 เท่านั้น
- [ ] Feature Matrix จาก Step 02 Section 7.3 ถูก implement ครบใน `user_plans` + server utilities
- [ ] ทุก feature ที่ lock มี server-side guard — ไม่ใช่แค่ hide UI
- [ ] `.env.example` มีทุก var ที่ใช้ใน codebase
- [ ] Dependency graph ไม่มี circular dependency
- [ ] ไม่มี issue ที่ใหญ่กว่า 2 วัน
- [ ] CI/CD pipeline (#5) รัน vitest + playwright ก่อน QA Agent ทำงาน