# Training — Operator Quickstart

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-12

---

## Who this is for
Line operators, shift leads, and support technicians who **run TALON jobs** and **read results**. No admin access required.

## What you’ll do
- Submit **Perception Jobs** (e.g., gauge reads, presence checks).  
- Submit **Infra Jobs** (dry‑run a change plan or request an apply).  
- Interpret results: **value, confidence, latency, pass/fail policy**.  
- Handle common errors and know when to escalate.

---

## 1) Quick setup (one time)
- You have a TALON username and token from your supervisor.  
- Install the lightweight CLI on your workstation or use the web console.  
- Confirm connectivity: you can see the **Jobs** dashboard and your line/area.

> If login fails or you don’t see your area, contact support.

---

## 2) Perception Job — submit & read
Use this when you need a reading from the production line (e.g., a gauge or indicator).

### A) Submit
**Console:** Click **New Job → Perception**. Choose the camera/ROI template, enter optional notes, press **Run**.  
**CLI:**

```bash
talon jobs create perception   --image-id cam12.frame   --roi preset:gauge_a   --note "Batch 47 changeover check"
```

### B) What you’ll see
- **value** — the measured reading (e.g., “62.1 PSI”).  
- **confidence** — 0–1; higher is better.  
- **latency_ms** — time to process.  
- **policy.status** — `pass` or `deny` based on thresholds (min confidence, max latency, valid range).  
- **explain** — short note (e.g., “below min confidence 0.85”).

### C) Pass/Fail examples
- **PASS**: value 62.1, confidence 0.92, within range → continue work.  
- **FAIL**: value 61.8, confidence 0.63 → repeat once after cleaning lens; if still low, escalate.

### D) Tips
- If glare or blur is visible, wipe lens or re‑aim camera and retry.  
- For borderline confidence (0.75–0.85), take a second reading and attach a photo if asked.

---

## 3) Infra Job — dry‑run & apply
Use this to preview (or request) a change to infrastructure (e.g., allowlisted IP, queue size).

### A) Dry‑run (Plan)
**Console:** **New Job → Infra (Plan)**. Fill the form (what you need changed) and submit.  
**CLI:**

```bash
talon jobs create infra   --change-request '{
    "action":"update_queue",
    "target":"line-b-eu01",
    "params":{"max_depth": 2000}
  }' --plan
```

**Result shows:** a **summary** with resources to `add/change/destroy`, policy status, and a plan `id` for reference.

### B) Request Apply
If the plan looks correct and policies pass:

- **Console:** Click **Request Apply** on the plan.  
- **CLI:**

```bash
talon jobs apply --plan-id PLAN-12345
```

**Note:** Some applies need a maintenance window or approval. You’ll see **pending** until approved.

---

## 4) Interpreting results
### Perception
| Field | What it means | What to do |
|---|---|---|
| value | The reading (unit shown next to value) | Compare to spec; log a note if out of band |
| confidence | 0–1 model certainty | < 0.80: retake; persistent low → escalate |
| latency_ms | Processing time | If very high repeatedly, note in ticket |
| policy.status | pass/deny from thresholds | `deny`: follow on‑screen guidance or escalate |
| explain | Reason for pass/deny | Use text to decide retake vs. escalate |

### Infra
| Field | What it means | What to do |
|---|---|---|
| summary.add/change/destroy | What will change | If unexpected, **do not apply**; ask support |
| policy.status | pass/deny from rules | `deny`: missing tags, unsafe size, etc. |
| artifact_ref | Signed plan reference | Keep for your ticket or shift log |
| job_id/plan_id | Tracking IDs | Include in any emails/tickets |

---

## 5) Common errors & quick fixes
- **Auth failed / token expired** → log out/in; ask supervisor to re‑issue token.  
- **Policy deny (perception)** → low confidence or out‑of‑range reading → clean lens, re‑aim, retake.  
- **Policy deny (infra)** → missing tags or too risky change → review form; if correct, escalate for admin review.  
- **No cameras found** → ensure your station profile is selected; if still empty, call support.  
- **Job stuck in pending** → network hiccup or approval needed; wait 60s and refresh; if >10m, escalate.

---

## 6) When to escalate
- 3 consecutive **denies** for the same step.  
- Measurements outside safety limits.  
- Infra plan differs from what you requested.  
- Any apply request pending > 30 minutes during a change window.

**How:** open a ticket with **job_id**, a screenshot, and a short description.

---

## 7) Shift checklist (operators)
- Start shift: log in; check **Jobs** and **Alerts** panels for your area.  
- During shift: submit readings at required checkpoints; watch for **denies**.  
- End of shift: ensure all jobs are **Completed** or escalated; hand over notes.

---

## 8) FAQ
- **Do I need admin rights?** No. Operators can submit jobs and review results.  
- **What if I typed the wrong parameters?** Cancel and resubmit; only admins can edit policies.  
- **Can I run jobs offline?** If the console shows **offline mode**, you can queue jobs; they sync later.

---

## 9) Glossary
- **Confidence** — model’s certainty (0–1).  
- **Policy** — rules that automatically approve/deny results or plans.  
- **Plan** — preview of infra changes; **Apply** executes the plan.  
- **Artifact** — signed file (image/plan) TALON can verify.

