# 📋 Deliverable D: Ops Dashboard, 30-60-90 Rollout Plan & Operations SOP

> **Bambinos.live • AI Program Manager (Operations) Assessment**  
> **Prepared For:** Head of Operations  

---

## Section 1: 30-60-90 Day Change Management & Rollout Plan

> [!IMPORTANT]
> The phased rollout ensures that live student classes are **never disrupted** during the transition. The old manual process runs in parallel ("shadow mode") in Phase 1 while the AI engine is validated.

### Days 1–30: Shadow Mode Calibration & Pilot

| Goal | Key Activities | Success Metric |
| :--- | :--- | :--- |
| **Validate AI engine accuracy** against real production data without impacting Ops flow | Run automated reconciliation engine **in parallel** (read-only) alongside existing manual Ops. Compare outputs daily. | Auto-approve rate on pilot cohort > 65%; zero false positives (no failed payment approved) |
| **Deploy pre-tagged payment links** for 20 pilot teachers | CRM team generates new renewal links with metadata tokens (`student_id`, `batch_id`, `educator_id`) embedded. Soft-launch with 20 volunteer teachers. | 100% of pilot teacher links carry metadata by Day 15 |
| **Instrument the SLA dashboards** | Live Ops Dashboard deployed (`/ops/dashboard/index.html`). Ops head begins daily morning check-in with radar view. | Dashboard live with real data by Day 10 |
| **Train Ops team on exception-only workflow** | 2 workshops: (1) Understanding reason codes, (2) Using 1-click Slack approval buttons. SOP document distributed. | All 10 Ops team members certified on exception review workflow |
| **WhatsApp API integration tested** | End-to-end WhatsApp template message delivery for Batch Full, Orphaned Invoice, and Payment Retry scenarios. | 100% delivery rate; < 5 min response time from parents on slot-picker tests |

---

### Days 31–60: Live Rollout (Zero-Touch Pipeline Active for Happy Path)

| Goal | Key Activities | Success Metric |
| :--- | :--- | :--- |
| **Deprecate manual faculty form for clean renewals** | All 200+ educators receive new 1-tap renewal confirm system. `faculty_form_unapproved` submission is deprecated for same-batch, clean-payment renewals. | > 75% of renewals processed via zero-touch path |
| **Ops team transitions to Exception Queue only** | Ops agents stop manually auditing every form submission. Daily workflow: Ops morning check on Stuck Invoices Radar + Exception Triage Queue only. | Ops time per 100 renewals drops from 20h to under 5h |
| **Automate batch capacity pre-checks** | Batch capacity shown to educators before they pitch renewal. If batch is 5/6 or 6/6, educator is auto-redirected to next available equivalent batch. | Batch overflow error rate drops from 100% to < 5% |
| **Teacher Quick-Confirm integration** | Educators receive WhatsApp with pre-computed renewal summary and 1-tap confirm link. Eliminates form for clean cases. | Teacher form-fill rate drops by 80% |
| **A/B Testing parent communications** | Test 2 variants of WhatsApp renewal notification messages. Measure re-booking rate and time-to-slot-selection. | Identify best-performing template for FOMO urgency + warmth balance |

---

### Days 61–90: Full Scale, Proactive Renewals & Analytics

| Goal | Key Activities | Success Metric |
| :--- | :--- | :--- |
| **Proactive renewal pipeline (pre-expiry)** | Automated outreach to teachers **7 days before** a student's last class: *"Student X has 2 classes left. Renew now to maintain momentum."* | Renewal conversation starts 7 days before expiry; reduces term-gap churn |
| **Self-serve parent batch scheduling portal** | Parents who request batch changes can browse real-time batch availability, pick a slot, and confirm in < 60 seconds without Ops involvement. | Parent self-serve scheduling > 60% of batch change requests |
| **SLA Breach alerting** | Every critical SLA metric (Orphaned Invoice > 60m, TAT > 2h for exceptions) triggers automated Slack alert to Ops lead. | Zero undetected SLA breaches lasting > 2 hours |
| **Instructor dashboard** | Educators see a real-time view of their student renewal statuses. Know which students have paid vs. pending without WhatsApp chasing. | Orphaned invoice rate < 1% |
| **Monthly AI audit & exception analysis** | Monthly report on exception codes frequency, resolution time, and false-positive/negative rates to continuously improve AI rules. | FPAR grows from 75% $\to$ > 90% within 90 days |

---

## Section 2: Operations SOP — Exception Playbook for the 10-Person Ops Team

### 2.1 Daily Morning Routine (10 minutes)

Each Ops specialist follows this check-in:

1. Open **Ops Cockpit**: [Bambinos Ops Dashboard](ops/dashboard/index.html)
2. **Scan:** "Stuck Invoices Radar" — any paid invoice > 2 hours without a booking.
3. **Scan:** "Critical Exceptions" — ghost payments, capacity overflow.
4. **Action each exception** (15 minutes/exception maximum) using the playbook below.

### 2.2 Exception Playbooks (Sorted by Severity)

---

**🚨 ERR_ORPHANED_INVOICE_CRITICAL_24H — Orphaned Invoice (Paid, No Form)**

| Step | Action |
| :---: | :--- |
| 1 | Automated WhatsApp reminder already sent to educator. Confirm in notification log. |
| 2 | If no educator response in 90 minutes, review Master Reference table for student's current batch. |
| 3 | If same batch has open capacity: Use **1-click Auto-Approve button** in Ops Dashboard to book student directly into next available class date. |
| 4 | Send parent confirmation WhatsApp. |
| 5 | Log the orphan incident against the educator in the monthly report. |

---

**🚨 ERR_PAYMENT_FAILED_GHOST_SUBMISSION — Failed Payment + Form Submitted**

| Step | Action |
| :---: | :--- |
| 1 | NEVER approve the faculty form or booking request. Mark form as `Rejected - Failed Gateway`. |
| 2 | Check if parent has since retried and paid: Refresh invoice record or check Razorpay console. |
| 3 | If parent now has a new successful invoice: Link to new invoice ID and re-initiate the auto-approval flow. |
| 4 | Notify educator not to take payment screenshots as confirmation. |

---

**⚠️ ERR_BATCH_CAPACITY_EXCEEDED — Batch Full (6/6)**

| Step | Action |
| :---: | :--- |
| 1 | Automated WhatsApp with 2 alternative slots already sent. Confirm in notification log. |
| 2 | Put a 24-hour hold on one available seat in the alternative batch for this student. |
| 3 | If parent does not respond in 24 hours: Escalate to educator to re-engage via phone call. |
| 4 | Only release held seat after 48 hours if no response. |

---

**⚠️ ERR_INSTALLMENT_SESSION_MISMATCH — Partial Payment vs. Full Sessions**

| Step | Action |
| :---: | :--- |
| 1 | Auto-correction has already set sessions to 12. Verify in the auto-approved bookings log. |
| 2 | Confirm installment 2 reminder is queued in the CRM for the date of class 11. |
| 3 | Notify educator that only 12 sessions are confirmed until second payment clears. |

---

**ℹ️ WARN_TRANSPOSITION_TYPO_RESOLVED — Auto-Corrected Digit Typo**

| Step | Action |
| :---: | :--- |
| 1 | AI engine auto-matched correctly. Review the reconciliation note in flagged_exceptions log. |
| 2 | No action required unless confidence score < 0.90. |
| 3 | Log to teacher's accuracy record (triggers re-training reminder if 3+ typos). |

---

## Section 3: Teacher On-Boarding & Change Communications

For the 200+ educators across geographies (India, GCC, US/UK), the transition to 1-tap renewal confirmations is managed via:

1. **WhatsApp Announcement Broadcast (Day 1):**
   > *"Hi Team Bambinos Educators! 🎉 Great news — we have launched the new 1-Tap Renewal System. When a parent pays for renewal, you will receive a WhatsApp confirmation with your student's details. Simply tap [Confirm] and their next 24 classes are booked automatically. No more Google Forms! Detailed instructions: [link]"*

2. **5-Minute Video Walkthrough:** Screen recording of the new teacher experience.

3. **Educator Hotline (Ops lead on-call, Days 1–14):** Dedicated Ops point-of-contact number for teacher questions during transition.

4. **Performance Recognition:** Teachers with the highest renewal confirmation rate (<30-minute response time) are highlighted in the monthly educator newsletter.

