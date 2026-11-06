# Bambinos.live — AI PM (Operations) Take-Home Submission

> **Track:** Operations — Student Renewal → Class Scheduling Pipeline  
> **Dataset:** 13 masked production records across 5 MySQL production tables  
> **Submission Folder:** [`./ops/`](./ops/)

---

## 🧠 Central Insight

The core problem is **not** that Ops is too slow — it's that the teacher renewal form **shouldn't exist** for straightforward renewals. It was introduced as a workaround to solve a metadata problem that smart payment link design solves upstream.

> *If the student is continuing in the same batch with the same educator and has paid the required fee, no human should ever touch the record.*

By embedding `student_id`, `batch_id`, and `educator_id` into the renewal payment link itself, the gateway webhook can trigger auto-booking in under 15 seconds — eliminating 3 of the 4 manual pipeline steps for the happy path.

---

## ⚡ Quick Start (Run the Prototype in 60 Seconds)

```bash
# Node.js — no install needed
node ops/engine/reconciliation_engine.js

# Python 3.9+
python ops/engine/reconciliation_engine.py
```

Then open **`ops/dashboard/index.html`** in any browser for the live Ops Head cockpit.

---

## 📊 Engine Results (13 Production Invoices)

| Metric | Result |
| :--- | :--- |
| **Zero-Touch Auto-Approved** | **7 bookings** (54%) in < 30 seconds each |
| **Critical Exceptions Caught** | 3 (ghost payment firewall, 2× orphaned invoices) |
| **All Exceptions Triaged** | 10 flags with typed reason codes + severity levels |
| **Automated Messages Drafted** | 18 personalised WhatsApp/Slack messages (API-ready JSON) |
| **Projected Ops Time Reduction** | 83.5% (20h → 3.3h per 100 renewals) |

---

## 📁 Repository Structure

```
ops/
├── data/                             ← 5 production CSV tables (provided by Bambinos)
│   ├── 1_invoices.csv
│   ├── 2_faculty_form_unapproved.csv
│   ├── 3_booking_requests.csv
│   ├── 4_class_schedule_bookings.csv
│   └── reference_active_batches_and_students.csv
│
├── engine/                           ← DELIVERABLE C: Working Prototype
│   ├── reconciliation_engine.js     ← Node.js (zero dependencies)
│   └── reconciliation_engine.py     ← Python equivalent
│
├── output/                           ← Generated outputs from one engine run
│   ├── auto_approved_bookings.csv   ← 7 zero-touch bookings (ready for DB insert)
│   ├── flagged_exceptions.csv       ← 10 triaged exceptions with reason codes
│   ├── drafted_notifications.csv    ← 18 auto-drafted messages (tabular)
│   ├── drafted_notifications.json   ← Same 18 messages in API-dispatch format
│   └── reconciliation_summary.json  ← Aggregated run metrics
│
├── dashboard/
│   └── index.html                   ← DELIVERABLE D: Ops Head Morning Cockpit
│
├── prompts/                          ← LLM system prompts (AI injection layer)
│   ├── reconciliation_fuzzy_match.md   ← Entity resolution agent (Gemini/GPT-4o)
│   └── exception_triage_agent.md       ← Communication drafting agent
│
├── reports/                          ← All 4 written deliverables
│   ├── DELIVERABLE_A_LEAKAGE_AUDIT.md
│   ├── DELIVERABLE_B_WORKFLOW_DESIGN.md
│   ├── DELIVERABLE_C_PROTOTYPE_ENGINE.md
│   ├── DELIVERABLE_D_SOP_AND_ROLLOUT.md
│   └── EXECUTIVE_CASE_DECK.md       ← Full case in one document (PDF source)
│
└── README.md
```

---

## 📋 The 4 Deliverables

### A — Root-Cause Leakage Audit · [`→ Read`](ops/reports/DELIVERABLE_A_LEAKAGE_AUDIT.md)

Forensic analysis of all 13 production records revealed **8 distinct failure modes**:

| # | Failure Mode | Evidence | Severity |
|:-:|:--|:--|:--|
| FM-1 | Orphaned Invoices — parent paid, teacher never filed form | Invoices 84906 (₹18,000, 44h), 84910 (₹11,000, 25h) | 🔴 Critical |
| FM-2 | Ghost Renewals — form filed on a failed/unverified payment | Invoice 84911 — `gateway_payment_id = NULL` | 🔴 Critical |
| FM-3 | Batch Capacity Blind Spot — teacher pitches a full batch | Invoice 84905 — Batch 9205 at 6/6 | 🟠 High |
| FM-4 | Cross-Parent Payer ID Mismatch — mother UPI vs father CRM | Invoice 84904 — mobile number mismatch | 🟡 Medium |
| FM-5 | Installment Session Hallucination — 24 sessions on 12-session payment | Invoice 84909 — `Installment_1_of_2` | 🟡 Medium |
| FM-6 | Batch Change Friction — no self-serve scheduling exists | Invoice 84912 — `continue_same_batch = No` | 🟡 Medium |
| FM-7 | Digit Transposition Typos — manual entry breaks FK joins | Invoice 84913 — teacher typed `84931` | 🟢 Low-Med |
| FM-8 | Duplicate Form Submissions — browser freeze double-submit | Forms 3308 & 3309 for student 68108 | 🟢 Low |

**5 SLAs defined:** Payment-to-Schedule TAT, Orphaned Invoice Rate, First-Pass Auto-Approval Rate, Ops Touch Hours per 100 renewals, Revenue at Risk.

---

### B — Re-Engineered Zero-Touch Workflow · [`→ Read`](ops/reports/DELIVERABLE_B_WORKFLOW_DESIGN.md)

The new pipeline embeds metadata into payment links and uses a 3-tier exception engine:

- **Tier 1 — Deterministic Rules:** Ghost payments, installment splits, capacity overflow (sub-millisecond)
- **Tier 2 — LLM Fuzzy Resolution:** Cross-parent identity, transposed IDs, name variations (Gemini Flash / GPT-4o-mini)
- **Tier 3 — Automated Dispatch:** WhatsApp interactive slot pickers, educator 1-tap confirms, Ops Slack alerts with pre-computed diffs

**No-code stack:** n8n (orchestration) + Interakt (WhatsApp API) + Gemini Flash (AI) + Metabase (analytics dashboard)

---

### C — Working Prototype Engine · [`→ Read`](ops/reports/DELIVERABLE_C_PROTOTYPE_ENGINE.md) | [`→ Code`](ops/engine/reconciliation_engine.js)

Zero-dependency engine that processes all 5 CSVs in a single run. Key design decisions:

- **No external packages** — evaluators can run it immediately with no setup
- **Quoted-CSV parser** — handles `"Tue,Thu"` preferred_days without corruption
- **Modular AI injection points** — 2 clearly marked locations where live Gemini/GPT-4o calls plug in; LLM prompts in `prompts/` are production-ready

Correctly handles all 8 failure modes including transposition typo auto-correction, cross-parent fuzzy match, duplicate deduplication, and ghost payment firewall.

---

### D — Ops Dashboard, SOP & 30-60-90 Rollout · [`→ Read`](ops/reports/DELIVERABLE_D_SOP_AND_ROLLOUT.md) | [`→ Dashboard`](ops/dashboard/index.html)

**Live Ops Cockpit** (open `ops/dashboard/index.html` — no server needed):
- 5 KPI cards · 4 tabbed views · 1-click action buttons per stuck invoice
- Tabs: Auto-Approved · Exception Triage Queue · Stuck Invoice Radar · Comms Activity Log

**Rollout** designed for zero class disruption:
- Days 1–30: Shadow mode (engine runs in parallel, Ops unaffected)
- Days 31–60: Live rollout — teacher form deprecated for clean renewals
- Days 61–90: Proactive pre-expiry renewal triggers + parent self-serve scheduling

**SOP:** Step-by-step exception playbooks for each of the 8 error codes — any exception resolved in < 15 minutes.

---

## 🔑 The Core Argument

| | Current State | Target State |
|:--|:--|:--|
| **Process** | 4 manual human touches, 3 teams | 0 touches (happy path), 1 touch (exceptions) |
| **TAT** | 1.75h – 48h | < 30 seconds (happy path) |
| **Ops visibility** | None for orphaned invoices | Real-time SLA radar + automated alerts |
| **Teacher burden** | Fill out full renewal form per student | Tap 1 WhatsApp button to confirm |
| **Parent experience** | No confirmation until Ops manually books | Auto-WhatsApp within 30 seconds of payment |
| **Revenue at Risk** | ₹29,000 (19.3%) stuck > 24h | ₹0 at 12 hours |
