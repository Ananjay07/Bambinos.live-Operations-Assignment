# 📈 Sales Track Assets — Post-Demo Call Intelligence & Conversion Engine

> **⚠️ Important:** Refer to and use the assets in this folder **ONLY** if you have been shortlisted for the **Sales Problem Statement (PS)**. Please read the Sales Problem Statement PDF shared with you in your shortlist email along with the root [`../README.md`](../README.md).

---

## 📂 Assets in This Folder

We have provided a multi-table masked dataset (`./data/`) and 6 post-demo academic counsellor call transcripts (`./call_transcripts/`) that mirror our real Demo-to-Enrollment sales funnel across India, GCC (UAE/Qatar), and International (US/UK) markets.

### 1. Multi-Table CRM, Demo & Payment Dataset (`./data/`)
All PII (`parent_name`, `student_name`, `mobile`, `email`) has been masked (`Di***a Na***r`, `+91-9820XXXX33`).

| File | What It Represents | Key Columns |
| :--- | :--- | :--- |
| **[`data/1_crm_leads.csv`](./data/1_crm_leads.csv)** | **CRM Leads Table (12 post-demo leads)** — Contains lead details, geo/timezone, course pitched, assigned counsellor, and the lazy/vague manual dispositions logged by reps (plus uncalled high-intent leads and duplicate leads). | `id`, `lead_code`, `user_id`, `student_id`, `student_name`, `student_age`, `student_grade`, `parent_name`, `mobile`, `email`, `country`, `city_timezone`, `lead_source`, `class_id`, `course_pitched`, `assigned_counsellor_id`, `assigned_counsellor_name`, `crm_lead_stage`, `manual_rep_disposition`, `manual_rep_notes`, `last_call_at`, `next_followup_at` |
| **[`data/2_demo_class_evaluations.csv`](./data/2_demo_class_evaluations.csv)** | **Demo Teacher Evaluation Notes** — Filled by the demo educator right after the trial class. Captures the child's strengths, specific learning gaps, and the exact **"Demo Magic Moment"** when the child shined. | `demo_id`, `lead_id`, `student_id`, `class_id`, `category_name`, `demo_educator_id`, `demo_educator_name`, `demo_completed_at`, `attended_demo`, `demo_batch_size`, `child_engagement_score_1_to_5`, `current_skill_level_assessed`, `skill_strengths_observed`, `learning_gaps_observed`, `demo_magic_moment_for_parent`, `recommended_course_track`, `recommended_batch_type` |
| **[`data/3_sales_call_logs.csv`](./data/3_sales_call_logs.csv)** | **Telephony & Call Metadata Log** — Links `LEAD-801` through `LEAD-806` to their call recording transcripts, call duration, rep-vs-parent talk-time ratio, and language detected. | `call_id`, `lead_id`, `counsellor_id`, `counsellor_name`, `call_started_at`, `call_duration_sec`, `rep_talk_time_pct`, `parent_talk_time_pct`, `language_detected`, `transcript_filename`, `rep_logged_disposition`, `rep_logged_crm_note` |
| **[`data/4_live_orders_and_invoices.csv`](./data/4_live_orders_and_invoices.csv)** | **Live Payment Gateway & Order Status** — Real-time status of payment links sent to parents (`Paid`, `Payment_Failed`, `Unpaid_Flagged_Discount_Anomaly`, `Unpaid_Link_Opened_Twice`). | `order_id`, `invoice_number`, `lead_id`, `student_id`, `parent_mobile`, `class_id`, `plan_sessions`, `currency`, `standard_amount`, `discount_applied`, `net_payable_amount`, `payment_plan_type`, `payment_gateway`, `payment_link_status`, `gateway_error_reason`, `paid_at` |
| **[`data/reference_courses_pricing_and_compliance_policy.csv`](./data/reference_courses_pricing_and_compliance_policy.csv)** | **Official Course Catalog, Pricing & Compliance Policy** — Ground-truth fees (INR/AED/USD), allowed discount caps (10%), installment rules, 14-day refund policy, and forbidden sales claims. | `class_id`, `course_name`, `category`, `target_age_grades`, `sessions_included`, `standard_fee_india_inr`, `standard_fee_gcc_aed`, `standard_fee_usa_usd`, `max_rep_discount_allowed_pct`, `installment_plan_available`, `official_refund_policy`, `forbidden_compliance_red_flags` |

### 2. Post-Demo Academic Counsellor Call Transcripts (`./call_transcripts/`)
1. **[`call_01_LEAD801_aarav_english_spouse_objection.txt`](./call_transcripts/call_01_LEAD801_aarav_english_spouse_objection.txt)** — Grade 3 Public Speaking (India); strong consultative call; Spouse Decision Objection (Father wasn't on Zoom).
2. **[`call_02_LEAD802_zoya_maths_dubai_price_dumping.txt`](./call_transcripts/call_02_LEAD802_zoya_maths_dubai_price_dumping.txt)** — Grade 5 Brainiac Maths (Dubai GCC); premature price dumping by rep; Parent comparing AED 1,200 fee to AED 350 local worksheet tuition while child struggles with application word problems.
3. **[`call_03_LEAD803_vihaan_phonics_compliance_redflag.txt`](./call_transcripts/call_03_LEAD803_vihaan_phonics_compliance_redflag.txt)** — Grade 2 Phonics (India); 84% rep talk-time monologue; severe **Compliance Red Flags** (fake "Rank 1 / Harry Potter in 30 days" guarantee, unauthorized 42% discount to Rs. 5,500, off-the-record 6-month refund promise, high-pressure threat).
4. **[`call_04_LEAD804_meera_gita_usa_schedule_and_paid_trap.txt`](./call_transcripts/call_04_LEAD804_meera_gita_usa_schedule_and_paid_trap.txt)** — Grade 4 Little Yogi Gita (USA - PST); weekday gymnastics schedule conflict resolved with Sat-Sun 9:30 AM PST batch; **check `4_live_orders_and_invoices.csv` before triggering any follow-up!**
5. **[`call_05_LEAD805_kabir_english_hinglish_competitor_emi.txt`](./call_transcripts/call_05_LEAD805_kabir_english_hinglish_competitor_emi.txt)** — Grade 4 Public Speaking (Delhi NCR); **Hinglish (Hindi + English)** conversation; Competitor Comparison (PlanetSpark 1-on-1 vs. small group) + School Exam pressure + 2-part Installment request (with a failed UPI mandate in `4_live_orders_and_invoices.csv`).
6. **[`call_06_LEAD806_ananya_chess_child_hesitant_rep_gaming_rubric.txt`](./call_transcripts/call_06_LEAD806_ananya_chess_child_hesitant_rep_gaming_rubric.txt)** — Grade 1 Chess (Hyderabad); **Rep gaming the AI rubric** by mechanically reading checklist questions in the first 30 seconds while ignoring that 6-year-old Ananya felt intimidated by 9-year-old boys in the demo (`Child Not Interested / Demo Batch Mismatch`).

---

## 💡 Instructions for Your Prototype

- Combine the **Call Transcripts** (`./call_transcripts/`) with the **Demo Teacher Evaluations** (`2_demo_class_evaluations.csv`), **Live Orders/Payment Status** (`4_live_orders_and_invoices.csv`), and **Official Pricing/Compliance Rules** (`reference_courses_pricing_and_compliance_policy.csv`).
- Run your **AI Call Audit + CRM Structuring + Personalized WhatsApp Follow-Up** pipeline across all **6 recorded calls** and audit the broader **12-lead CRM table** (`1_crm_leads.csv`) for pipeline leaks (e.g., high-intent demos never called, duplicate leads, failed payment links, or parents who already paid).
- Save your prompts (`.md`), structured CRM output table (CSV/JSON/Markdown), call audit scorecards, compliance alerts, and generated 3-touch WhatsApp sequences in your submission GitHub repository.
