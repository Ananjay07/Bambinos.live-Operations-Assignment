# 📊 Deliverable C: Working Prototype — Reconciliation & Auto-Approval Engine

> **Bambinos.live • AI Program Manager (Operations) Assessment**  
> **Topic:** Technical Architecture, Design Decisions & Test Results of the Working Prototype  
> **Prepared For:** Head of Operations & Technical Evaluator  

---

## 1. What Was Built

A production-ready, zero-dependency reconciliation engine (`/engine/reconciliation_engine.js`) written in Node.js that ingests all 5 masked production CSV tables and runs a deterministic + AI-ready exception triage pipeline.

> [!NOTE]
> A fully equivalent Python version (`reconciliation_engine.py`) is also provided for teams using Python-based tooling. Both produce identical output.

### Technology Choices & Justification

| Concern | Decision | Why |
| :--- | :--- | :--- |
| **Language** | Node.js (Primary) + Python (Secondary) | Node.js is available without install in most CI/CD environments. Python aligns with the data science / analytics stack most Ops teams use. |
| **Dependencies** | Zero external packages (`npm install` not needed) | Evaluators can run `node engine/reconciliation_engine.js` immediately. No `pip install`, no virtual envs. |
| **Data Parsing** | Custom CSV parser with quoted-comma support | The `preferred_days` column contains `"Tue,Thu"` (commas inside quotes). Built-in string splits would corrupt this. |
| **Fuzzy Matching** | Sorted-character transposition distance | Catches teacher typos like `84931 → 84913` without an external library. 2-character edit with identical character sets = transposition. |
| **LLM Integration Point** | Modular JSON payload design | The `prompts/` folder contains production-ready system prompts. Any call to the AI exception router can be replaced by a live Gemini / GPT-4o API call in the n8n flow with zero code change to the core engine. |

---

## 2. Engine Architecture & Decision Tree

```
reconciliation_engine.js
│
├── loadCSV()           → Loads and parses all 5 tables into typed JS objects
├── isTransposition()   → Checks if two ID strings differ only by transposed adjacent digits
│
├── PASS 1: Faculty Form Processing Loop (12 submitted forms)
│   ├── Deduplication Check     → ERR_DUPLICATE_FORM_SUBMISSION
│   ├── Invoice Resolution      → Fuzzy match invoice ID to student via transposition
│   │   └── On failure          → ERR_INVOICE_NOT_FOUND
│   ├── Typo Warning            → WARN_TRANSPOSITION_TYPO_RESOLVED (logs, then continues)
│   ├── Payment Status Guard    → ERR_PAYMENT_FAILED_GHOST_SUBMISSION [HALT]
│   ├── Installment Guard       → ERR_INSTALLMENT_SESSION_MISMATCH [HALT + prorate]
│   ├── Cross-Parent Match      → INFO_CROSS_PARENT_IDENTITY_VERIFIED [log, continue]
│   ├── Batch Change Guard      → ACTION_BATCH_CHANGE_SELECTION_REQUIRED [HALT + notify]
│   ├── Capacity Guard          → ERR_BATCH_CAPACITY_EXCEEDED [HALT + alt-slot notify]
│   └── Discount Flag           → INFO_AUTHORIZED_SCHOLARSHIP_APPLIED [log, continue]
│       └── All Guards Pass     → autoApprovedBookings.push() + notification drafted ✅
│
└── PASS 2: Orphan Invoice Scan
    └── For every Paid invoice NOT seen in Pass 1:
        → ERR_ORPHANED_INVOICE_CRITICAL_24H + teacher WhatsApp nudge + Ops Slack alert
```

---

## 3. Verified Test Results (13-Invoice Production Dataset)

Running `node ops/engine/reconciliation_engine.js` produces:

```json
{
  "total_invoices_audited": 13,
  "total_forms_submitted": 12,
  "auto_approved_zero_touch": 7,
  "total_flagged_exceptions": 10,
  "critical_exceptions": 3,
  "high_severity_exceptions": 1,
  "medium_severity_exceptions": 2,
  "info_warnings_resolved": 4,
  "notifications_drafted": 18
}
```

### Auto-Approved Bookings (7 — Zero Human Touch)

| Booking ID | Invoice | Student | Root Resolution |
| :--- | :--- | :--- | :--- |
| AUTO-BKG-1001 | 84901 | Aa\*\*\*v Sh\*\*\*a | Standard happy path |
| AUTO-BKG-1002 | 84902 | Di\*\*\*a Na\*\*\*r | Recovered previously stuck Ops drop-off |
| AUTO-BKG-1003 | 84903 | Ka\*\*\*r Ve\*\*\*a | Pending Ops approval auto-cleared by rules |
| AUTO-BKG-1004 | 84904 | An\*\*\*a Iy\*\*\*r | ✅ Cross-parent fuzzy identity match (mother's UPI) |
| AUTO-BKG-1005 | 84907 | Vi\*\*\*n Jo\*\*\*i | ✅ 20% retention scholarship auto-verified |
| AUTO-BKG-1006 | 84908 | Si\*\*\*a Ma\*\*\*a | ✅ Browser-freeze duplicate pruned; booking proceeds |
| AUTO-BKG-1007 | 84913 | De\*\*\*a Me\*\*\*a | ✅ Transposed invoice ID (`84931→84913`) auto-corrected |

### Exceptions Correctly Identified & Triaged (10 Flags)

| Severity | Code | Record | Auto-Remediation Dispatched |
| :--- | :--- | :--- | :--- |
| 🔴 CRITICAL | `ERR_PAYMENT_FAILED_GHOST_SUBMISSION` | 84911 | Parent WhatsApp retry link + Ops Slack |
| 🔴 CRITICAL | `ERR_ORPHANED_INVOICE_CRITICAL_24H` | 84906 | Teacher 1-tap confirm WhatsApp |
| 🔴 CRITICAL | `ERR_ORPHANED_INVOICE_CRITICAL_24H` | 84910 | Teacher 1-tap confirm WhatsApp |
| 🟠 HIGH | `ERR_BATCH_CAPACITY_EXCEEDED` | 84905 | Parent alternative slot picker dispatched |
| 🟡 MEDIUM | `ACTION_BATCH_CHANGE_SELECTION_REQUIRED` | 84912 | Parent self-serve scheduling link |
| 🟡 MEDIUM | `ERR_INSTALLMENT_SESSION_MISMATCH` | 84909 | Sessions prorated to 12; reminder queued |
| 🟢 LOW | `WARN_TRANSPOSITION_TYPO_RESOLVED` | 84913 | Auto-corrected; logged for educator tracking |
| 🟢 LOW | `INFO_CROSS_PARENT_IDENTITY_VERIFIED` | 84904 | CRM family profile auto-linked |
| 🟢 LOW | `INFO_AUTHORIZED_SCHOLARSHIP_APPLIED` | 84907 | Discount verified; no action needed |
| 🟢 LOW | `ERR_DUPLICATE_FORM_SUBMISSION` | Form 3309 | Duplicate pruned; primary form 3308 proceeds |

---

## 4. Output Files Generated

| File | Contents |
| :--- | :--- |
| [`output/auto_approved_bookings.csv`](../output/auto_approved_bookings.csv) | 7 clean records ready for direct insertion into `class_schedule_bookings` table |
| [`output/flagged_exceptions.csv`](../output/flagged_exceptions.csv) | 10 exception records with error codes, severity levels, descriptions, and responsible parties |
| [`output/drafted_notifications.csv`](../output/drafted_notifications.csv) | 18 personalized WhatsApp/Slack message payloads in tabular format |
| [`output/drafted_notifications.json`](../output/drafted_notifications.json) | Same 18 messages in JSON format, ready for direct dispatch via WhatsApp API or Slack Webhook |
| [`output/reconciliation_summary.json`](../output/reconciliation_summary.json) | Aggregated run metrics for dashboard KPI integration |

---

## 5. How to Run (Evaluator Instructions)

### Pre-requisites
* Node.js v18+ (or Python 3.9+) — no external packages needed
* All 5 CSVs present in `./ops/data/`

### Run the Engine (Node.js)
```bash
cd d:\LPU\Placements\Bambinos
node ops/engine/reconciliation_engine.js
```

### Run the Engine (Python)
```bash
cd d:\LPU\Placements\Bambinos
python ops/engine/reconciliation_engine.py
```

### Open the Live Ops Dashboard
Open `ops/dashboard/index.html` directly in any modern browser. No server required (pure HTML/JS).

---

## 6. How the AI Layer Plugs In

The engine is designed with **AI injection points** at exactly 2 decision nodes:

**Node A — Fuzzy Entity Resolution:**
Replace the `isTransposition()` function call in Pass 1 with a call to the LLM agent defined in `prompts/reconciliation_fuzzy_match.md`. The JSON input/output contract is already specified and production-ready.

**Node B — Exception Communication Drafting:**
Replace the hardcoded message strings in `draftedNotifications.push()` with a call to the LLM agent defined in `prompts/exception_triage_agent.md`, passing the full exception payload. The LLM will draft contextually personalized messages adapting to parent name, student name, specific error, and available alternatives.

This modular design means the engine works fully today without any AI API costs, but can be upgraded to full LLM-powered resolution by adding one API call per injection point.

