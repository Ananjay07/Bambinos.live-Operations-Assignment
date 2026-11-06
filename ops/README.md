# 🎯 Bambinos.live — AI PM (Operations) Take-Home Assignment Submission

> **Candidate Submission** | Operations Problem Statement  
> **Assignment:** Re-Engineering the Student Renewal → Class Scheduling Pipeline  
> **Dataset:** 13 masked production records across 5 MySQL production tables

---

## 📐 Repository Structure

```
ops/
├── data/                                    ← 5 production CSV tables (provided)
│   ├── 1_invoices.csv
│   ├── 2_faculty_form_unapproved.csv
│   ├── 3_booking_requests.csv
│   ├── 4_class_schedule_bookings.csv
│   └── reference_active_batches_and_students.csv
│
├── engine/                                  ← Deliverable C: Working Prototype
│   ├── reconciliation_engine.js            ← Node.js engine (zero dependencies)
│   └── reconciliation_engine.py            ← Python equivalent
│
├── output/                                  ← Generated outputs from engine run
│   ├── auto_approved_bookings.csv          ← 7 zero-touch approved class schedules
│   ├── flagged_exceptions.csv              ← 10 triaged exceptions with reason codes
│   ├── drafted_notifications.csv           ← 18 auto-drafted messages (tabular)
│   ├── drafted_notifications.json          ← 18 auto-drafted messages (API-ready JSON)
│   └── reconciliation_summary.json         ← Aggregated run metrics
│
├── dashboard/
│   └── index.html                          ← Deliverable D: Ops Head Morning Cockpit
│
├── prompts/                                 ← LLM system prompts (AI injection layer)
│   ├── reconciliation_fuzzy_match.md       ← Entity resolution agent prompt
│   └── exception_triage_agent.md           ← Communication drafting agent prompt
│
├── reports/                                 ← All 4 written deliverables
│   ├── DELIVERABLE_A_LEAKAGE_AUDIT.md
│   ├── DELIVERABLE_B_WORKFLOW_DESIGN.md
│   ├── DELIVERABLE_C_PROTOTYPE_ENGINE.md
│   └── DELIVERABLE_D_SOP_AND_ROLLOUT.md
│
└── README.md                                ← This file
```

---

## ⚡ Quick Start (Run the Engine in 60 Seconds)

```bash
# Node.js (recommended — no installs needed)
node ops/engine/reconciliation_engine.js

# Python 3.9+
python ops/engine/reconciliation_engine.py

# Open the dashboard in any browser
start ops/dashboard/index.html
```

---

## 📦 The 4 Deliverables — At a Glance

### Deliverable A — Root-Cause Problem Definition & Data Leakage Audit
📄 [`reports/DELIVERABLE_A_LEAKAGE_AUDIT.md`](reports/DELIVERABLE_A_LEAKAGE_AUDIT.md)

**8 distinct failure modes** identified from forensic analysis of the 13 production records:

| ID | Failure Mode | Evidence | Severity |
| :---: | :--- | :--- | :--- |
| FM-1 | **Orphaned Invoices** — Parent paid but teacher never filed form | Invoices 84906 (₹18,000, 44h), 84910 (₹11,000, 25h) | 🔴 Critical |
| FM-2 | **Ghost Renewals** — Form submitted on a failed/unverified payment | Invoice 84911, gateway_payment_id = NULL | 🔴 Critical |
| FM-3 | **Batch Capacity Blind Spots** — Teacher pitches full batch | Invoice 84905, Batch 9205 at 6/6 | 🟠 High |
| FM-4 | **Cross-Parent Payer ID Mismatch** — Mother's UPI vs father's CRM number | Invoice 84904, mobile mismatch | 🟡 Medium |
| FM-5 | **Installment Session Hallucination** — Teacher requests 24 sessions on 12-session payment | Invoice 84909, Installment_1_of_2 | 🟡 Medium |
| FM-6 | **Batch Change Friction** — No self-serve scheduling; every change requires Ops | Invoice 84912, `continue_same_batch = No` | 🟡 Medium |
| FM-7 | **Digit Transposition Typos** — Manual form entry errors break FK joins | Invoice 84913, teacher typed `84931` | 🟢 Low-Med |
| FM-8 | **Duplicate Form Submissions** — Browser freeze causing double-submit | Forms 3308 & 3309 for student 68108 | 🟢 Low |

**Key SLA Targets Defined:**

| Metric | Current State | Target |
| :--- | :--- | :--- |
| Payment-to-Schedule TAT (Happy Path) | 1.75h – 48h | **< 30 seconds** |
| Orphaned Invoice Rate | 15.4% | **0%** at 2 hours |
| First-Pass Auto-Approval Rate | 0% | **> 75%** |
| Ops Hours per 100 Renewals | ~20 hours | **< 3.3 hours** (83.5% reduction) |
| Revenue at Risk (> 24h stuck) | ₹29,000 (19.3%) | **₹0** at 12 hours |

---

### Deliverable B — AI & No-Code Re-Engineered Workflow
📄 [`reports/DELIVERABLE_B_WORKFLOW_DESIGN.md`](reports/DELIVERABLE_B_WORKFLOW_DESIGN.md)

The **Zero-Touch Pipeline** eliminates the teacher form entirely for clean same-batch renewals by embedding metadata in the payment link itself (`student_id`, `batch_id`, `educator_id`). When the payment webhook fires, the pipeline:

1. **Runs deterministic rules** (payment verified, same batch, seat available) → **Auto-books in < 15 seconds**
2. **Routes exceptions** through a 3-tier AI triage engine:
   - **Tier 1 (Rules):** Ghost payments, installment splits, capacity overflow
   - **Tier 2 (LLM Fuzzy):** Payer identity mismatch, transposed invoice IDs, name variations
   - **Tier 3 (Automated Dispatch):** WhatsApp slot pickers, teacher 1-tap confirms, Ops Slack alerts

**No-Code Stack:** n8n (orchestration) + Interakt (WhatsApp API) + Metabase (analytics) + Gemini Flash / GPT-4o-mini (entity resolution)

> [!TIP]
> The critical architectural insight is that **the teacher form shouldn't exist for clean renewals**. It was introduced as a band-aid to solve a metadata problem that smart payment link design solves upstream, eliminating 80% of Ops workload.

---

### Deliverable C — Working Prototype Engine
📄 [`reports/DELIVERABLE_C_PROTOTYPE_ENGINE.md`](reports/DELIVERABLE_C_PROTOTYPE_ENGINE.md)  
⚙️ [`engine/reconciliation_engine.js`](engine/reconciliation_engine.js)  
🐍 [`engine/reconciliation_engine.py`](engine/reconciliation_engine.py)

A fully functional, zero-dependency reconciliation engine that processes all 5 CSVs in a single run:

```
13 invoices audited  →  7 auto-approved (54%)  +  10 flagged with exact reason codes
                     →  18 personalized WhatsApp/Slack messages drafted & API-ready
```

The engine correctly handles **all 8 failure modes** including:
- ✅ Deduplication of browser-freeze double submits
- ✅ Transposition typo auto-correction via character-sort distance (`84931 → 84913`)
- ✅ Cross-parent identity fuzzy match (mother's phone on father's CRM record)
- ✅ Ghost payment firewall (gateway status `Failed` → booking halted)
- ✅ Batch capacity overflow with alternative slot generation
- ✅ Installment proration (24 sessions → 12 sessions + reminder queued)
- ✅ Orphaned invoice detection (every paid invoice with no form after threshold)

---

### Deliverable D — Ops Dashboard, SOP & 30-60-90 Day Rollout
📄 [`reports/DELIVERABLE_D_SOP_AND_ROLLOUT.md`](reports/DELIVERABLE_D_SOP_AND_ROLLOUT.md)  
📊 [`dashboard/index.html`](dashboard/index.html)

**Live Ops Cockpit** (open `dashboard/index.html` in any browser):
- 5 real-time KPI cards: Total Invoices, Zero-Touch Scheduled, Orphaned Invoices, Exceptions, Hours Saved
- 4 tabbed views: Auto-Approved, Exception Triage Queue, Stuck Invoice Radar, Comms Activity Log
- 1-click action buttons: Auto-Approve into available batch, Resend teacher nudge

**30-60-90 Day Rollout (Zero Class Disruption):**
- **Days 1–30:** Shadow mode — engine runs in parallel with manual Ops; validates accuracy; 20-teacher pilot
- **Days 31–60:** Live rollout — teacher form deprecated for clean renewals; Ops manages exceptions only
- **Days 61–90:** Full autonomy — proactive pre-expiry renewal triggers; self-serve parent batch scheduling

**Operations SOP:** Step-by-step exception playbooks for each of the 8 error codes, designed for the 10-person Ops team to process any exception in under 15 minutes.

---

## 🧠 The Central Insight (Why Manual Steps Were Wrong, Not Just Slow)

The root problem isn't that Ops is too slow — it's that the **teacher form exists at all** for straightforward renewals. The 4-step pipeline was built on a false assumption: that teachers need to manually communicate payment confirmation to Ops.

A payment link that **carries the renewal metadata** (student, batch, educator) makes teacher re-entry entirely redundant for 75%+ of cases. The gateway webhook becomes the single source of truth, triggering auto-booking, auto-confirmation, and auto-notification without any human in the loop.

The teacher only needs to interact when something genuinely requires human judgment: a parent requesting a different schedule, a batch being full, or a disputed payment.

**Current State:** Every renewal requires human touch.  
**Target State:** Only genuine exceptions require human touch.
