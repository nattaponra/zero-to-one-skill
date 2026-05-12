# Zero-to-One Skill

> เปลี่ยนไอเดีย 1–2 ประโยค ให้กลายเป็น Working MVP พร้อม deploy ใน 4 ขั้นตอน

---

## Pipeline Overview

```mermaid
flowchart TD
    START(["💡 Raw Idea\nแค่ keyword ก็พอ"])

    START --> S00

    subgraph S00["⚙️ Step 00 · Brainstorm"]
        direction LR
        A0["AI Product Strategist\nถามคำถาม clarifying ทีละข้อ\nเสนอ 2-3 directions + trade-off\nผลลัพธ์: 00_idea_card.md"]
    end

    S00 --> H0

    H0{"👤 Human Checkpoint 0\n\nเลือก direction แล้ว?\nIS / IS NOT ของ MVP ชัด?\nTop 3 assumptions ระบุแล้ว?"}

    H0 -->|"✅ Approve"| S01
    H0 -->|"🔄 ปรับ direction"| S00

    subgraph S01["⚙️ Step 01 · Discovery & Research"]
        direction LR
        A1["AI Research Analyst\nวิเคราะห์ตลาด · competitors · target users\nผลลัพธ์: 01_discovery_report.md"]
    end

    S01 --> H1

    H1{"👤 Human Checkpoint 1\n\nปัญหาสมเหตุสมผล?\nMarket signal แข็งพอ?\nCompetitor gap ชัดเจน?"}

    H1 -->|"✅ Approve"| S02
    H1 -->|"🔄 ปรับ idea"| S00

    subgraph S02["⚙️ Step 02 · Define & Validate"]
        direction LR
        A2["AI Product Manager\nเขียน PRD · User stories · Pricing tiers\nFeature matrix · Business model\nผลลัพธ์: 02_definition_report.md"]
    end

    S02 --> H2

    H2{"👤 Human Checkpoint 2\n\nP0 features ≤ 5?\nPricing tiers สมเหตุสมผล?\nRiskiest assumption ระบุชัด?"}

    H2 -->|"✅ Approve"| S03
    H2 -->|"🔄 ปรับ scope"| S02

    subgraph S03["⚙️ Step 03 · Design & Prototype"]
        direction LR
        A3["AI UX Designer\nออกแบบ screens · flows · design system\nPricing page · Paywall component\nผลลัพธ์: 03_design_brief.md + v0/Bolt prompts"]
    end

    S03 --> H3

    H3{"👤 Human Checkpoint 3\n\nทุก P0 screen มี spec ครบ?\nv0/Bolt prompts พร้อม paste?\nUpgrade flow ชัดเจน?"}

    H3 -->|"✅ Approve"| S04A
    H3 -->|"🔄 ปรับ design"| S03

    subgraph S04A["⚙️ Step 04A · Plan"]
        direction LR
        A4A["AI Engineering Lead\nแตก GitHub Issues · dependency graph\nplan.md · M0–M3 milestones"]
    end

    S04A --> H4

    H4{"👤 Human Checkpoint 4\n\nทุก issue มี acceptance criteria?\nTest cases ≥ 2 ต่อ issue?\nM0 Foundation issues ครบ?"}

    H4 -->|"✅ Approve"| S04B
    H4 -->|"🔄 ปรับ plan"| S04A

    subgraph S04B["⚙️ Step 04B · Build"]
        direction TB
        V0["🚀 V0: Value Core\n#1 scaffold · #2 UI · #3 logic/AI\nmock auth + mock data\ngoal: demo in 2–3 days"]
        HV0{"👤 V0 Checkpoint\n\nดู demo URL\nนี่คือ product\nที่ต้องการไหม?"}
        M0M3["M0→M3: Real Infrastructure\nauth · DB · payments · CI/CD\nfull feature build"]
        QA["QA Agent + Code Review\nรัน test cases · ตรวจ code quality"]
        V0 --> HV0
        HV0 -->|"✅ ใช่เลย"| M0M3
        HV0 -->|"🔄 ปรับ direction"| V0
        M0M3 --> QA
        QA -->|"QA_FAILED → systematic-debugging"| M0M3
    end

    S04B --> MERGE["Merge to main ✅"]
    QA -->|"QA_PASSED + APPROVE"| MERGE

    MERGE --> H5

    H5{"👤 Human Checkpoint 5\n\nMVP ทำงานได้ตาม spec?\nReady to ship?"}

    H5 -->|"✅ Ship it!"| DONE(["🚀 MVP Live"])
    H5 -->|"🔄 Fix issues"| S04B

    style START fill:#f0f4ff,stroke:#4f6ef7
    style DONE fill:#f0fff4,stroke:#22c55e
    style H0 fill:#fff7ed,stroke:#f97316
    style H1 fill:#fff7ed,stroke:#f97316
    style H2 fill:#fff7ed,stroke:#f97316
    style H3 fill:#fff7ed,stroke:#f97316
    style H4 fill:#fff7ed,stroke:#f97316
    style H5 fill:#fff7ed,stroke:#f97316
```

---

## Human Checkpoints สรุป

| # | จุด | สิ่งที่ต้องตรวจ | ใช้เวลา |
|---|-----|----------------|---------|
| 0 | หลัง Brainstorm | Direction ชัด? IS/IS NOT ของ MVP ตกลง? Top 3 assumptions ระบุแล้ว? | 15–30 นาที |
| 1 | หลัง Discovery | Problem จริง? Market signal แข็ง? Competitor gap ชัด? | 30 นาที |
| 2 | หลัง Define | P0 ≤ 5 features? Pricing tiers สมเหตุสมผล? Riskiest assumption ระบุชัด? | 45 นาที |
| 3 | หลัง Design | ทุก P0 screen มี spec? v0/Bolt prompt ใช้ได้? Upgrade flow ครบ? | 1 ชั่วโมง |
| 4 | หลัง Plan | ทุก issue มี test cases ≥ 2? V0 issues แค่ #1-#3? ไม่มี circular dep? | 30 นาที |
| 4.5 | หลัง V0 demo | ดู demo URL — "นี่คือ product ที่ต้องการ?" → approve ก่อน M0 | 30 นาที |
| 5 | หลัง Build | MVP ทำงานได้ตาม spec? Ready to ship? | ตามขนาด MVP |

---

## URL Structure Convention

> ทุก product ที่สร้างด้วย skill นี้ใช้ pattern นี้เสมอ

```
/              → Landing Page (public)
/pricing       → Pricing page (public)
/login         → Login (public)
/signup        → Sign Up (public)

/app/*         → App — protected routes ทั้งหมด (ต้องการ auth)
/app/dashboard → Dashboard หลัก
/app/[feature] → Feature pages
/app/settings  → Settings
/app/settings/billing → Billing & Plan management
```

---

## ไฟล์ในนี้

| ไฟล์ | บทบาท |
|------|-------|
| `SKILL.md` | Entry point สำหรับ Claude agent |
| `step-00-brainstorm.md` | Brainstorm dialogue — ตกผลึก idea ก่อน research |
| `step-01-discovery.md` | System prompt + output schema สำหรับ Step 01 |
| `step-02-define.md` | System prompt + PRD schema + pricing matrix |
| `step-03-design.md` | Design brief schema + screen specs + v0 prompts |
| `step-04-build.md` | Engineering plan + subagent orchestration + QA |

---

## วิธีใช้งาน

### เริ่มต้น (Step 00)

1. มีไอเดีย ไม่ต้องสมบูรณ์ — keyword เดียวก็พอ เช่น `"แอปช่วย SME ทำ content"`

2. เปิด `step-00-brainstorm.md` → เริ่ม dialogue กับ Claude → ได้ `00_idea_card.md` ที่ approve แล้ว

3. ตรวจ **Validation Checklist** ในแต่ละ step ก่อนส่งต่อเสมอ

### ส่งต่อระหว่าง steps

แต่ละ step มี `🔗 HANDOFF` section ที่ด้านบนของ output — **กรอกให้ครบก่อนส่งต่อเสมอ**

```
Step 00 output → วาง 00_idea_card.md ใน Step 01 prompt
Step 01 output → วาง 01_discovery_report.md ทั้งหมดใน Step 02 prompt
Step 02 output → วาง 02_definition_report.md ทั้งหมดใน Step 03 prompt
Step 03 output → วาง 03_design_brief.md ทั้งหมดใน Step 04 Phase A prompt
```

### Superpowers skills ที่ใช้ใน Step 04

| Skill | ใช้เมื่อไหร่ |
|-------|------------|
| `superpowers:writing-plans` | Phase A — เขียน plan |
| `superpowers:subagent-driven-development` | Phase B — execute issues |
| `superpowers:verification-before-completion` | ก่อน subagent report SUCCESS ทุกครั้ง |
| `superpowers:systematic-debugging` | เมื่อ QA_FAILED — หา root cause ก่อนแก้ |
| `superpowers:finishing-a-development-branch` | เมื่อ milestone complete — merge/PR |

---

## Tech Stack (Fixed)

| Layer | Technology | เหตุผล |
|-------|-----------|--------|
| Framework | Next.js 16+ (App Router) | SSR, file-based routing |
| UI | MUI v6+ | Design system พร้อม, Thai font support |
| Backend / Auth / Storage | Supabase | BaaS ครบ, RLS, realtime |
| LLM API | OpenRouter + SDK | Multi-model, cost control |
| Payment | Stripe | Subscription, Thai baht, webhook |
| Deploy | Vercel | Next.js native, preview per PR |

> ห้ามเปลี่ยน stack เว้นแต่มีเหตุผล document ไว้ใน GitHub Issue

---

## เวลารวมโดยประมาณ

| Phase | เวลา AI | เวลาคุณ (review) |
|-------|---------|-----------------|
| Step 01 Discovery | 2–3 ชม. | 30 นาที |
| Step 02 Define | 2–4 ชม. | 45 นาที |
| Step 03 Design | 3–5 ชม. | 1 ชม. |
| Step 04 Plan | 1–2 ชม. | 30 นาที |
| Step 04 Build | 2–4 สัปดาห์ | ตาม milestone |
