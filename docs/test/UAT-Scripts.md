# User Acceptance Test (UAT) Scripts

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-05  

---

## 1) Purpose
Validate that TALON meets business requirements for **Perception** (gauge/indicator reads) and **Provisioning** (policy‑gated infrastructure changes) using clear, repeatable scripts that BER stakeholders can execute and sign off.

## 2) Roles
- **UAT Lead (RAIN):** facilitates sessions, records results.  
- **Business Owner (BER):** approves outcomes against acceptance criteria.  
- **Operator (BER):** executes perception flows.  
- **Platform Admin (RAIN/BER):** executes provisioning flows.  
- **Observer/Scribe:** notes, evidence links, defects.

## 3) Environment & Preconditions
- **Environment:** `stage` (pre‑prod) with current release tag.  
- **Access:** operator and admin accounts provisioned; tokens valid.  
- **Data:** synthetic frames for gauges with known truth values; change targets prepared (e.g., test namespace).  
- **Health:** dashboards green; GitOps synced; alerts quiet.

## 4) Evidence & Recording
- Capture screenshots or console output for each step.  
- Record `job_id`, `plan_id`, and image/plan **digest** when shown.  
- Store artifacts in `uat/evidence/<date>-<run-id>/`.

---

## 5) UAT Scripts — Perception

### P‑1: Single Gauge Read — PASS within Threshold
**Goal:** Operator submits a perception job and receives an in‑range value with sufficient confidence.  
**Preconditions:** Camera preset `gauge_a` available. Truth value for test frame: **62 ± 2 PSI**. Thresholds: **min_confidence=0.85**, **max_latency=800 ms**.

**Steps & Expected Results**  
1. **Submit Job**
   - *Action:* Console → **New Job → Perception**. Choose `cam12`, ROI `gauge_a`, note “UAT P‑1”. Click **Run**.  
   - *Expect:* Job status `queued → running → completed` within 5s.
2. **Review Result**
   - *Action:* Open job details.  
   - *Expect:* `value` between 60–64, `confidence ≥ 0.85`, `latency_ms ≤ 800`, `policy.status = pass`.
3. **Evidence**
   - *Action:* Save screenshot; record `job_id`.  
   - *Expect:* Evidence stored and linked.

**Pass/Fail:** Pass if all expectations met. Otherwise log defect with screenshots.

---

### P‑2: Low Confidence — DENY and Operator Guidance
**Goal:** System denies a result with insufficient confidence and provides actionable guidance.  
**Preconditions:** Use test frame with glare; expected confidence **< 0.80**.

**Steps & Expected Results**  
1. Submit perception job with “glare” frame. → **Expect:** `policy.status = deny`, `explain` mentions low confidence.  
2. Follow **on‑screen guidance** (clean/adjust), resubmit with “clean” frame. → **Expect:** `pass` with confidence ≥ 0.85.  
3. Evidence saved for both attempts.

**Pass/Fail:** Pass if first attempt denies, second passes, and guidance is displayed.

---

### P‑3: Latency Spike — Alert and Recovery
**Goal:** Detect a latency breach and confirm auto‑recovery.  
**Preconditions:** Enable load script to create temporary backlog.

**Steps & Expected Results**  
1. Run load to push p95 > 800 ms for ≤ 5 min. → **Expect:** alert fires; jobs complete with `policy.status = pass` once backlog clears.  
2. Stop load; verify p95 returns below threshold within 10 min.  
3. Evidence: latency panel snapshot before/after; alert entry.

**Pass/Fail:** Pass if alert triggered and latency recovers within time window.

---

## 6) UAT Scripts — Provisioning (Infra)

### I‑1: Plan (Dry‑Run) Requires Mandatory Tags
**Goal:** Plan fails policy when required tags are missing and passes when provided.  
**Preconditions:** Target `line-b-eu01`, policy requires tags: `owner`, `env`, `change_ticket`.

**Steps & Expected Results**  
1. **Submit Plan** without tags. → **Expect:** `policy.status = deny`, `explain` lists missing tags.  
2. **Resubmit Plan** with required tags. → **Expect:** `policy.status = pass`, plan summary shows intended changes.  
3. Evidence: plan output and policy decisions linked.

**Pass/Fail:** Pass when deny→pass behavior confirmed with clear reasons.

---

### I‑2: Apply Requires Approval
**Goal:** Approved plan applies successfully; unapproved remains pending.  
**Preconditions:** Maintenance window open; approver available.

**Steps & Expected Results**  
1. Request **Apply** for previously approved plan. → **Expect:** status `pending_approval`.  
2. Approver grants approval. → **Expect:** `apply in progress → completed`, drift check = zero.  
3. Evidence: approval record, apply logs, drift check screenshot.

**Pass/Fail:** Pass when approval gating works and apply completes without drift.

---

### I‑3: Rollback to Last Signed Digest
**Goal:** Restore previous version using signed artifact reference.  
**Preconditions:** Known good digest `DIGEST_A` recorded in evidence.

**Steps & Expected Results**  
1. Initiate **Rollback** to `DIGEST_A`. → **Expect:** GitOps syncs; service healthy; signature verified.  
2. Evidence: admission log showing verified signature; health dashboard green.

**Pass/Fail:** Pass when rollback completes and health checks succeed.

---

## 7) Negative/Edge Cases
- **Unsigned Image Attempted:** Admission deny; verify event in logs.  
- **Policy Bundle Not Loaded:** API returns clear error; operator sees “temporarily unavailable.”  
- **Network Loss (Operator):** Console shows queued state; jobs sync after reconnect.

---

## 8) Acceptance Criteria (Summary)
| Area | Criteria |
|---|---|
| Perception | P‑1 pass; P‑2 deny→pass sequence; P‑3 alert & recovery within window |
| Provisioning | I‑1 deny→pass; I‑2 approval‑gated apply; I‑3 rollback verifies signature |
| Evidence | All `job_id`/`plan_id`/digest recorded with screenshots |
| Documentation | Operator Quickstart and Admin Guide referenced during UAT |
| Sign‑off | Business Owner approves; defects Sev‑1/2 resolved or exception filed |

---

## 9) Defects & Variances Log
Use the RAID log ID for cross‑reference.

| ID | Scenario | Severity | Description | Owner | Due | Status |
|---|---|---|---|---|---|---|
| UAT‑001 | P‑2 | 3 | Guidance text unclear | Dev Lead | 2025‑09‑12 | Open |

---

## 10) Sign‑Off
| Role | Name | Date | Signature |
|---|---|---|---|
| Business Owner (BER) |  |  |  |
| Engineering Director (BER) |  |  |  |
| UAT Lead (RAIN) |  |  |  |
