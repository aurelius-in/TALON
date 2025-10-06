# SRE Playbook

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-05  

---

## 1) Purpose
Give Site Reliability Engineering (SRE) a single, actionable reference for operating TALON: on‑call, incident response, SLIs/SLOs and error budgets, alerting, change management, and continuous improvement.

## 2) Scope
Applies to control‑plane services, perception workers, infra runners, CI/CD and GitOps, registry/signing, observability stack, and data stores across dev, stage, and prod.

---

## 3) Roles & On‑Call
- **Primary On‑Call (P1):** 24×7 for prod within rotation. Acknowledges pages within **5 minutes**.  
- **Secondary On‑Call (P2):** Escalation within **10 minutes** if P1 does not respond.  
- **Incident Commander (IC):** Coordinates comms/decisions during Sev‑1/Sev‑2.  
- **Comms Lead:** External updates to stakeholders.  
- **Scribe:** Timeline and actions in the incident doc.

**Handover (daily):** known issues, feature flags, open CRs, and risk watchlist.  
**Paging windows:** prod 24×7; stage business hours; dev non‑paged.

---

## 4) Severity Matrix
| Sev | Description | Examples | Initial Target |
|---|---|---|---|
| **Sev‑1** | Critical outage or safety risk | API down; policy engine failing closed; data loss | Page P1 immediately; IC within 5 min |
| **Sev‑2** | Major degradation | P95 latency breach > 30m; sustained 5xx > 1% | Page P1; IC as needed |
| **Sev‑3** | Minor impact/workaround | Single line affected; intermittent errors | Ticket; page only if prolonged |
| **Sev‑4** | Low impact | Cosmetic or tooling | Backlog only |

---

## 5) SLIs, SLOs & Error Budgets
### Platform SLIs
- **Availability** (prod monthly): control‑plane 99.5%+.  
- **API Latency (p95):** ≤ 250 ms.  
- **Perception Job Latency (p95):** ≤ 800 ms.  
- **Perception Confidence:** % jobs ≥ threshold.  
- **Infra Plan→Apply Success:** ≥ 99% when approved.

### Error Budgets
- Each SLO has an error budget (e.g., 0.5% monthly unavailability).  
- **Burn policies:** if burn rate > 2× for 1 hour, freeze non‑urgent changes and trigger a mitigation review.

---

## 6) Monitoring & Alerting
**Golden signals:** latency, traffic, errors, saturation (plus confidence for perception).  
**Page (urgent):**  
- API availability below SLO; p95 > threshold for 15 min.  
- Perception latency p95 > 800 ms for 15 min **and** queue depth rising.  
- Admission denials for unsigned images or failed signature verification.  
- GitOps app Degraded > 10 min in prod.

**Ticket (non‑urgent):** capacity trending to 80%, policy exception expiry soon, SBOM critical with fixed patch available.

**Alert engineering:** deduplicate by service; include runbook link and **next action** line in every alert.

---

## 7) Incident Response
1. **Acknowledge** page (P1). Start incident doc with timestamp.  
2. **Stabilize:** rollback/canary/scale per runbook.  
3. **Communicate:** IC sets severity, updates #incident channel and stakeholders. Cadence: 15–30 min.  
4. **Diagnose:** check dashboards, logs, policy decisions, recent deploys. Form a hypothesis, test safely.  
5. **Mitigate:** implement least‑risky fix first.  
6. **Close:** verify SLO recovery; capture root cause, actions, and follow‑ups.

**Comms templates** (examples):
- *Initial:* “Sev‑2 incident impacting API latency in prod since 14:32. Mitigation in progress; next update 14:50.”  
- *Resolved:* “Incident resolved at 15:08; cause: cache regression; fix: config revert; follow‑ups: PR‑123, add alert.”

---

## 8) Standard Runbooks (Quick)
### A) API Latency Spike
- Check **API p95**, **CPU/mem**, **RPS**; correlate with deploys.  
- If CPU>75% with scale room → **scale out**; else rollback last change.  
- Verify DB/cache health; clear hot path contention.  
- Confirm policy engine reachable.

### B) Perception Latency/Confidence Degradation
- Inspect **GPU util**, **queue depth**, **camera health**.  
- Add worker if GPU util p95 ≥ 75% and queue rising; check model version and ROI.  
- If confidence drop isolated to one line: clean lens, adjust light; compare to last good model bundle.

### C) GitOps App Degraded
- Review app diff; verify image digest and signature.  
- Revert last PR if urgent; fix manifest and re‑sync.  
- Ensure service account and RBAC intact.

### D) Unsigned Image Attempted
- Admission webhook should deny. Capture digest; open incident if in prod.  
- Verify CI signing; force re‑build/re‑sign; audit trust store.

---

## 9) Change Management
- **Pre‑prod checks:** unit/integration tests, policy eval, signature verification, smoke.  
- **Release strategy:** canary 10% → 50% → 100% for API; blue/green or rolling for workers.  
- **Freeze rules:** active when error budget burn > threshold or during cutover windows.  
- **CAB:** only for production‑impacting or policy/risk changes.

---

## 10) Capacity & Autoscaling (Ops Notes)
- Headroom target 25–35% on constrained resource (GPU/CPU).  
- Triggers encoded in HPA/queues as code; use scale‑in protections (min replicas, stabilization window).  
- Weekly review forecast vs. actual; adjust multipliers.

---

## 11) Reliability Engineering
- **Toil cap:** ≤ 50% per SRE; automate repeated manual tasks.  
- **Game days:** quarterly chaos tests (node loss, registry outage, policy deny storm).  
- **Blameless postmortems:** due within 5 business days, with action owners and deadlines.  
- **Error budget policy:** sustained burn pauses feature rollout until fixed.

---

## 12) Security & Compliance Touchpoints
- Enforce signed artifacts and SBOM checks at admission.  
- Policies and thresholds versioned; changes via PR with tests.  
- Access: least‑privilege groups, short‑lived tokens, auditable approvals.

---

## 13) Dashboards & Logs (Pointers)
- **Platform Health:** API/queue/worker health, capacity.  
- **Perception Pipeline:** latency, confidence, throughput.  
- **Infra Pipeline:** plan/apply success, queue wait.  
- **Policy Decisions:** pass/deny, top rules.  
- **Audit/Signing:** sign/verify events.

Every dashboard panel should link to logs for pivot (correlated by `job_id`, `digest`, or `plan_id`).

---

## 14) Readiness & Checklists
**Start of Shift**
- Verify pager, dashboards, GitOps health.  
- Review open incidents and feature flags.

**Pre‑Change**
- Backout plan ready; SLO impact understood; run smoke in lower env.

**Post‑Change**
- Watch key SLIs for 30–60 min; roll back on regression.

**Weekly**
- RAID review, action closure, and SLO trend report.

---

## 15) Contacts & Escalations
- P1 On‑Call: `+1-XXX-XXX-XXXX`, `oncall@rain.example`  
- P2 On‑Call: `+1-YYY-YYY-YYYY`  
- IC bridge: `meet.example/incidents`  
- Vendors: registry, DNS, GPU platform (list here)

---

## 16) Artifacts & References
- Capacity Plan • Disaster Recovery Plan • Runbooks (Deploy/Rollback/Triage) • Observability Plan • Policy Pack • Image Signing & SBOM SOP • 30/60/90 Plan.

---

## 17) Revision History
| Version | Date | Change |
|---|---|---|
| 0.1.0 | 2025‑09‑05 | Initial SRE playbook |

---
