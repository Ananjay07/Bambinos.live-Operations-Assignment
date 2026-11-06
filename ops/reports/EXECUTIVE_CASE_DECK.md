---
title: "Re-Engineering the Student Renewal → Class Scheduling Pipeline"
subtitle: "Bambinos.live · AI PM (Operations) Take-Home Assessment"
date: "November 2026"
author: "Operations Problem Statement Submission"
---

# Executive Summary

At Bambinos.live, the student renewal pipeline represents the highest-leverage business process for revenue retention and lifetime value. Yet today, only **1 in 13 paying renewals** (7.7%) successfully reaches a scheduled class without getting stuck, delayed, or dropped by the current 4-step manual handoff.

This submission delivers a complete re-engineering of that pipeline across all 4 required dimensions:

---

**Key Finding:** The teacher renewal form — the core of the current process — is an unnecessary bottleneck for 75%+ of renewals. Payment links that carry metadata (student ID, batch ID, educator ID) make teacher data re-entry redundant, eliminating 3 of the 4 manual pipeline steps for the happy path.

**Key Result (Prototype):** Processing all 13 production records, the engine auto-approved 7 bookings in under 30 seconds, correctly identified every exception with a typed reason code, and generated 18 personalised WhatsApp/Slack messages — with zero external dependencies.

---

# Part 1: The Problem (Deliverable A)

## The 4-Table Manual Handoff

The current pipeline requires 4 sequential human-to-human touches:

1. **Invoice created** when parent pays via Razorpay/Stripe payment link
2. **Teacher manually submits** `faculty_form_unapproved` — copying invoice link, student ID, fee amount
3. **Ops manually verifies** the form and approves it
4. **Ops manually creates** calendar rows in `class_schedule_bookings`

Over weekends, public holidays, or during teacher fatigue, any of these steps can silently stall — and the paying parent has no visibility into why their child's classes haven't started.

## 8 Failure Modes — From 13 Production Records

| ID | Failure Mode | Evidence | Severity |
|:--:|:--|:--|:--|
| FM-1 | **Orphaned Invoices** | Invoices 84906 (₹18,000, 44h), 84910 (₹11,000, 25h) | 🔴 Critical |
| FM-2 | **Ghost Renewals on Failed Payments** | Invoice 84911 — gateway_payment_id = NULL | 🔴 Critical |
| FM-3 | **Batch Capacity Blind Spots** | Invoice 84905 — Batch 9205 at 6/6 | 🟠 High |
| FM-4 | **Cross-Parent Payer ID Mismatch** | Invoice 84904 — Mother's UPI vs father's CRM | 🟡 Medium |
| FM-5 | **Installment Session Hallucination** | Invoice 84909 — 24 sessions on 12-session payment | 🟡 Medium |
| FM-6 | **Batch Change Friction** | Invoice 84912 — no self-serve scheduling | 🟡 Medium |
| FM-7 | **Digit Transposition Typos** | Invoice 84913 — teacher typed `84931` | 🟢 Low-Med |
| FM-8 | **Duplicate Form Submissions** | Forms 3308 & 3309 — browser freeze | 🟢 Low |

## Operational SLA Framework

| Metric | Current State | Target |
|:--|:--|:--|
| Payment-to-Schedule TAT (Happy Path) | 1.75h – 48h | **< 30 seconds** |
| Orphaned Invoice Rate | 15.4% | **0%** at 2 hours |
| First-Pass Auto-Approval Rate (FPAR) | 0% | **> 75%** |
| Ops Hours per 100 Renewals | ~20 hours | **< 3.3 hours** |
| Revenue at Risk (> 24h stuck) | ₹29,000 (19.3%) | ₹0 at 12 hours |

---

# Part 2: The Solution (Deliverable B)

## The Zero-Touch Architecture

The architectural principle: payment links carry renewal metadata, so the gateway webhook triggers auto-booking with zero teacher re-entry.

```
Payment Link → [student_id, batch_id, educator_id embedded]
         ↓
Razorpay/Stripe Webhook (HMAC-verified)
         ↓
Stage 1: Deterministic Rules Engine
  ├── payment_status = 'Paid'?
  ├── continue_same_batch = True?
  └── batch_capacity < max_batch_size?
         ↓               ↓
   [ALL PASS]        [FLAG RAISED]
         ↓               ↓
  < 15s Auto-Book   AI Exception Router
                         ↓
            Tier 1: Rules (ghost, installment, capacity)
            Tier 2: LLM (identity, typos, name matching)
            Tier 3: Dispatch (WhatsApp, Slack, 1-tap confirm)
```

## What Gets Eliminated for Happy-Path Renewals

- ❌ Teacher Google Form / web form (re-entry of data the system already knows)
- ❌ Manual Ops invoice URL verification (gateway HMAC signature replaces this)
- ❌ Manual Ops calendar session creation (automated DB trigger from batch schedule pattern)

## Smart Exception Handling — 3 Real Examples

**Scenario: Payer Phone Mismatch (Record 84904)**
Mother pays from her UPI. LLM agent cross-checks student surname, educator ID, and grade level → 0.99 confidence match → Auto-approves, auto-links mother's phone as secondary CRM contact.

**Scenario: Batch Full (Record 84905)**
System halts before DB collision. Queries identical curriculum batches with open seats. Dispatches WhatsApp to parent: *"Reply 1 for Tue-Thu 5 PM or 2 for Sat-Sun 10 AM."* Parent reply triggers instant booking. Ops never involved.

**Scenario: Orphaned Invoice (Records 84906, 84910)**
Cron scanner runs every 15 minutes. Paid invoice > 60min with no booking → WhatsApp to educator: *"Tap Confirm to continue [Student Name] in Batch 9206."* If unanswered at 4h and seat is open → auto-books defensively.

## No-Code Tech Stack

| Component | Tool | Role |
|:--|:--|:--|
| Orchestration | n8n (self-hosted) or Make.com | Webhook handling, flow branching, error retries |
| AI Entity Resolution | Gemini Flash / GPT-4o-mini | Fuzzy matching, identity resolution, typo correction |
| Messaging | Interakt / Wati (WhatsApp Business API) | Interactive templates with Quick-Reply buttons |
| Ops Alerts | Slack Webhook API | Real-time escalations with 1-click action buttons |
| Analytics | Metabase / Looker Studio | Live SLA dashboard for Ops Head |

---

# Part 3: Working Prototype (Deliverable C)

## Engine Results — 13 Production Invoices

| Outcome | Count | Detail |
|:--|:--|:--|
| Zero-Touch Auto-Approved | **7** (54%) | Direct to class_schedule_bookings in < 30s |
| Critical Exceptions Caught | **3** | Ghost payment, 2× Orphaned invoices |
| High/Medium Exceptions Triaged | **3** | Batch full, batch change, installment mismatch |
| Info/Warnings Auto-Resolved | **4** | Typo corrected, duplicate pruned, scholarship verified, identity linked |
| WhatsApp/Slack Messages Drafted | **18** | Personalised, API-ready JSON payload |

## Auto-Approved Bookings — What Was Recovered

| Booking ID | Student | Root Mechanism |
|:--|:--|:--|
| AUTO-BKG-1001 | Aa\*\*\*v Sh\*\*\*a | Standard happy path |
| AUTO-BKG-1002 | Di\*\*\*a Na\*\*\*r | Recovered previously stuck Ops drop-off |
| AUTO-BKG-1003 | Ka\*\*\*r Ve\*\*\*a | Pending approval auto-cleared by rules |
| AUTO-BKG-1004 | An\*\*\*a Iy\*\*\*r | ✅ Cross-parent fuzzy identity match |
| AUTO-BKG-1005 | Vi\*\*\*n Jo\*\*\*i | ✅ 20% retention scholarship auto-verified |
| AUTO-BKG-1006 | Si\*\*\*a Ma\*\*\*a | ✅ Browser-freeze duplicate pruned |
| AUTO-BKG-1007 | De\*\*\*a Me\*\*\*a | ✅ Transposed typo 84931→84913 auto-corrected |

## Key Design Decisions

**Zero external dependencies:** Runs with `node reconciliation_engine.js` or `python reconciliation_engine.py` — no npm install, no pip, no virtual env.

**Modular AI injection:** Two clearly marked injection points where the hardcoded rules can be replaced by a live Gemini/GPT-4o API call. The LLM system prompts are production-ready in `/prompts/`.

**Idempotent & safe:** Every guard runs before any write. Ghost payments are detected and blocked before a single class session record is created.

---

# Part 4: Ops Cockpit, SOP & Rollout (Deliverable D)

## 30-60-90 Day Rollout Plan

### Days 1–30: Shadow Mode (Zero Risk)

- Run AI engine in parallel with existing manual Ops — read-only, no writes
- Deploy pre-tagged payment links for 20 pilot educators
- Stand up Ops Head Dashboard with live SLA metrics
- Train 10 Ops team members on exception-only review workflow
- Test WhatsApp API end-to-end for all exception message types

### Days 31–60: Live Rollout (Happy Path Automated)

- Deprecate teacher manual form for same-batch, clean-payment renewals
- Ops team transitions from "process every renewal" to "resolve exceptions only"
- Batch capacity pre-check shown to educators before they pitch renewal
- **Target:** > 75% auto-approval rate; Ops hours/100 renewals < 5h

### Days 61–90: Full Scale (Proactive & Predictive)

- Automated outreach 7 days before student's last class
- Self-serve parent batch scheduling portal (no Ops call needed for reschedules)
- Monthly AI exception audit to continuously improve FPAR
- **Target:** FPAR > 90%; Revenue at Risk at 12h = ₹0

## Ops Exception Playbooks (5-Minute Reference)

**ERR_ORPHANED_INVOICE_CRITICAL_24H:**
1. Confirm automated WhatsApp was sent to educator
2. If no response in 90 min + same-batch seat available → 1-click Auto-Approve in Dashboard
3. Send parent confirmation

**ERR_PAYMENT_FAILED_GHOST_SUBMISSION:**
1. NEVER approve — mark form `Rejected - Failed Gateway`
2. Check if parent has since retried with a new successful invoice
3. If yes → link new invoice and re-initiate auto-approval flow

**ERR_BATCH_CAPACITY_EXCEEDED:**
1. Confirm automated alternative-slot WhatsApp sent to parent
2. Hold a seat in the alternative batch for 24 hours
3. If no parent response in 24h → escalate to educator to call

## Teacher Change Communications

A 3-part rollout to 200+ educators across India, GCC, and US/UK:
1. **WhatsApp broadcast** on Day 1: *"Good news — no more Google Forms for renewals! You'll get a 1-tap WhatsApp to confirm and that's it."*
2. **5-minute screen recording** of the new teacher experience
3. **Performance recognition**: Highest renewal confirmation speed educators featured in monthly newsletter

---

# Conclusion: The Central Insight

The root problem isn't that Ops is too slow — it's that the teacher form **shouldn't exist** for straightforward renewals. It was introduced as a band-aid to solve a metadata problem that smart payment link design solves upstream.

**Before:** Every renewal requires 4 human touches, 1–48 hours, 3 teams.
**After:** Happy path takes 15 seconds and zero human touches. Exceptions are routed to the right person automatically, with the data they need already computed.

The 10-person Ops team doesn't get smaller — they get dramatically more effective, because their attention is redirected from mechanical form verification to genuinely difficult edge cases that benefit from human judgment.

