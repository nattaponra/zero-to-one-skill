---
name: zero-to-one-skill
description: Use when turning a raw product idea into a shipped MVP. Triggers on new product ideas, startup concepts, or any feature that needs to go from vague notion to working software with a defined market and real users.
---

# Zero-to-One Skill
> ไอเดีย → Working MVP ใน 4 ขั้นตอน

## FIRST: ตรวจสอบสถานะโปรเจกต์ก่อนทำอะไรทั้งนั้น

> **ทำทันทีที่ skill นี้ถูกเรียก — ก่อนถามหรือทำอะไรทั้งสิ้น**

```bash
# ตรวจไฟล์ output ของแต่ละ step
ls *.md 2>/dev/null
ls docs/superpowers/plans/ 2>/dev/null
cat PROGRESS.md 2>/dev/null
gh issue list --state all 2>/dev/null | head -20
```

จากนั้นประเมินสถานะตาม decision tree นี้:

```
มี PROGRESS.md?
├── YES → อ่าน PROGRESS.md → แจ้ง "โปรเจกต์นี้อยู่ที่ [milestone] ต่อจาก: [Next Action]"
│          → ถามว่า "ต้องการทำต่อจากตรงนี้เลยไหม?"
│
└── NO → ตรวจไฟล์ตามลำดับ:
    │
    ├── ไม่มีไฟล์อะไรเลย
    │   → แจ้ง "เริ่มต้นใหม่ — Step 00 Brainstorm"
    │   → เริ่ม step-00-brainstorm.md
    │
    ├── มี 00_idea_card.md แต่ไม่มี 01_discovery_report.md
    │   → แจ้ง "idea card พร้อมแล้ว — ต่อด้วย Step 01 Discovery"
    │   → เริ่ม step-01-discovery.md
    │
    ├── มี 01_discovery_report.md แต่ไม่มี 02_definition_report.md
    │   → แจ้ง "discovery เสร็จแล้ว — ต่อด้วย Step 02 Define"
    │   → เริ่ม step-02-define.md
    │
    ├── มี 02_definition_report.md แต่ไม่มี 03_design_brief.md
    │   → แจ้ง "definition เสร็จแล้ว — ต่อด้วย Step 03 Design"
    │   → เริ่ม step-03-design.md
    │
    ├── มี 03_design_brief.md แต่ไม่มี docs/superpowers/plans/
    │   → แจ้ง "design brief พร้อมแล้ว — ต่อด้วย Step 04A Plan"
    │   → เริ่ม Phase A ใน step-04-build.md
    │
    └── มี docs/superpowers/plans/ และ/หรือมี gh issues
        → แจ้ง "plan เสร็จแล้ว — ต่อด้วย Step 04B Build"
        → รัน SESSION RESUME ใน step-04-build.md
```

**แจ้งผลให้ user เห็นชัดเจน เช่น:**
```
พบว่าโปรเจกต์นี้ทำมาถึง Step 02 Define แล้ว
ไฟล์ที่มี: 00_idea_card.md ✅  01_discovery_report.md ✅  02_definition_report.md ✅
ขั้นตอนถัดไป: Step 03 Design — ออกแบบ screens + v0 prompts

ต้องการทำต่อเลยไหม?
```

---

## Overview

Pipeline เปลี่ยนไอเดีย 1–2 ประโยคให้กลายเป็น working MVP พร้อม deploy โดย AI agents ทำงานแต่ละ step และส่ง HANDOFF ให้ขั้นถัดไปโดยอัตโนมัติ

**Fixed Tech Stack:** Next.js 16+ · MUI · Supabase · OpenRouter · Stripe · Vercel

---

## Pipeline

```
[IDEA]  (raw — แค่ keyword หรือ 1–2 ประโยคก็พอ)
   │
   ▼
Step 00: Brainstorm             ──→  00_idea_card.md          (15–30 นาที)
   │
   ▼
Step 01: Discovery & Research  ──→  01_discovery_report.md   (2–3 ชม.)
   │
   ▼
Step 02: Define & Validate     ──→  02_definition_report.md  (2–4 ชม.)
   │
   ▼
Step 03: Design & Prototype    ──→  03_design_brief.md       (3–5 ชม.)
   │
   ▼
Step 04: Build MVP             ──→  Working code + PRs       (2–4 สัปดาห์)
```

---

## Quick Reference

| Step | Role | Input | Output | ไฟล์รายละเอียด |
|------|------|-------|--------|---------------|
| 00 Brainstorm | Product Strategist | Raw idea | `00_idea_card.md` | [step-00-brainstorm.md](step-00-brainstorm.md) |
| 01 Discovery | Research Analyst | `00_idea_card.md` | `01_discovery_report.md` | [step-01-discovery.md](step-01-discovery.md) |
| 02 Define | Product Manager | `01_discovery_report.md` | `02_definition_report.md` | [step-02-define.md](step-02-define.md) |
| 03 Design | UX Designer | `02_definition_report.md` | `03_design_brief.md` | [step-03-design.md](step-03-design.md) |
| 04 Build | Eng Lead + Subagents | `03_design_brief.md` | Working MVP | [step-04-build.md](step-04-build.md) |

---

## วิธีเริ่มใช้งาน

1. มีไอเดีย ไม่ต้องสมบูรณ์ — keyword เดียวก็พอ
2. เปิด **[step-00-brainstorm.md](step-00-brainstorm.md)** → ทำ dialogue กับ Claude จนได้ `00_idea_card.md` ที่ approve แล้ว
3. เปิด **[step-01-discovery.md](step-01-discovery.md)** → copy System Prompt + Prompt Template → วาง idea card → รันกับ Claude
3. ตรวจ **Validation Checklist** ในแต่ละ step ก่อนส่งต่อ
4. แต่ละ step มี `🔗 HANDOFF` section ด้านบน output — **ต้องกรอกให้ครบก่อนส่งต่อเสมอ**
5. ทำซ้ำ Step 02 → 03 → 04

---

## Tech Stack Lock

> ห้ามเปลี่ยน stack ต่อไปนี้ เว้นแต่มีเหตุผล document ไว้ใน GitHub Issue

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16+ (App Router) |
| UI | MUI v6+ |
| Backend / Auth / Storage | Supabase |
| LLM API | OpenRouter + SDK |
| Payment | Stripe |
| Deploy | Vercel |

---

## Human Checkpoints

คุณต้องตรวจและ approve ก่อนเสมอที่ขั้นตอนเหล่านี้:

- **หลัง Step 00:** Direction ชัด? IS/IS NOT ของ MVP ตกลงแล้ว?
- **หลัง Step 01:** Problem Statement สมเหตุสมผล? Market signal แข็งพอ?
- **หลัง Step 02:** P0 features ≤ 5? Riskiest assumption ระบุชัด?
- **หลัง Step 03:** Design brief พร้อม dev ทำงานได้ทันที?
- **Phase A ของ Step 04:** Plan ผ่าน Plan Reviewer ก่อน execute
- **หลัง V0 (Step 04):** ดู demo — "นี่คือ product ที่ต้องการไหม?" → approve ก่อน M0
