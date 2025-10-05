# RACI Matrix

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments
**Version/Date:** v0.1 • 2025-09-16

---

## Purpose

Defines who is **Responsible (R)**, **Accountable (A)**, **Consulted (C)**, and **Informed (I)** for key TALON deliverables and activities.

## Roles

* **SA (RAIN):** Solution Architect
* **PM (RAIN):** Project/Delivery Manager
* **Dev Lead (RAIN):** Engineering Lead for app/agent code
* **SRE (RAIN):** Platform/SRE lead (GitOps, clusters, CI/CD)
* **SecEng (RAIN):** Security Engineering (policies, signing, SBOM, IAM)
* **PO (BER):** Product Owner / Business lead
* **Eng Dir (BER):** Engineering Director / Technical decision-maker
* **IT Sec (BER):** Information Security / Compliance
* **Ops (BER):** Plant/Line Operations & Support

## Matrix

| Item                            | R               | A             | C                                         | I                           |
| ------------------------------- | --------------- | ------------- | ----------------------------------------- | --------------------------- |
| High‑Level Design (HLD)         | SA (RAIN)       | Eng Dir (BER) | Dev Lead (RAIN), PO (BER), IT Sec (BER)   | Ops (BER), PM (RAIN)        |
| Low‑Level Design (LLD)          | Dev Lead (RAIN) | SA (RAIN)     | SRE (RAIN), SecEng (RAIN)                 | PO (BER), Ops (BER)         |
| OpenAPI Spec                    | Dev Lead (RAIN) | SA (RAIN)     | PO (BER)                                  | PM (RAIN), Eng Dir (BER)    |
| IaC Modules                     | SRE (RAIN)      | SA (RAIN)     | Dev Lead (RAIN), IT Sec (BER)             | Eng Dir (BER)               |
| CI/CD Design                    | SRE (RAIN)      | SA (RAIN)     | SecEng (RAIN), Dev Lead (RAIN)            | PM (RAIN), Eng Dir (BER)    |
| Image Signing & SBOM SOP        | SecEng (RAIN)   | SA (RAIN)     | SRE (RAIN), IT Sec (BER)                  | Eng Dir (BER), Ops (BER)    |
| Policy Pack (Thresholds & OPA)  | SecEng (RAIN)   | SA (RAIN)     | Dev Lead (RAIN), IT Sec (BER)             | PO (BER)                    |
| Threat Model (STRIDE)           | SecEng (RAIN)   | SA (RAIN)     | Dev Lead (RAIN), SRE (RAIN)               | Eng Dir (BER), IT Sec (BER) |
| Observability Plan              | SRE (RAIN)      | SA (RAIN)     | Dev Lead (RAIN), SecEng (RAIN)            | PO (BER), Ops (BER)         |
| KPIs & Dashboard Links          | SRE (RAIN)      | SA (RAIN)     | PO (BER), Dev Lead (RAIN)                 | Eng Dir (BER)               |
| Data Classification & Handling  | SecEng (RAIN)   | IT Sec (BER)  | SA (RAIN), PM (RAIN)                      | Eng Dir (BER), Ops (BER)    |
| Data Contracts (JSON Schemas)   | Dev Lead (RAIN) | SA (RAIN)     | SRE (RAIN)                                | PO (BER), Eng Dir (BER)     |
| Runbook — Deploy                | SRE (RAIN)      | SA (RAIN)     | Dev Lead (RAIN), Ops (BER)                | PO (BER), Eng Dir (BER)     |
| Runbook — Rollback              | SRE (RAIN)      | SA (RAIN)     | SecEng (RAIN), Ops (BER)                  | PO (BER), Eng Dir (BER)     |
| Runbook — Triage                | SRE (RAIN)      | SA (RAIN)     | Dev Lead (RAIN), SecEng (RAIN), Ops (BER) | PO (BER)                    |
| Cutover Plan                    | PM (RAIN)       | Eng Dir (BER) | SA (RAIN), Ops (BER)                      | PO (BER), IT Sec (BER)      |
| Acceptance Certificate          | PO (BER)        | Eng Dir (BER) | SA (RAIN), PM (RAIN)                      | Dev Lead (RAIN), SRE (RAIN) |
| Communication & Meeting Cadence | PM (RAIN)       | SA (RAIN)     | Eng Dir (BER), PO (BER)                   | All                         |
| 30/60/90 Plan                   | SA (RAIN)       | Eng Dir (BER) | PM (RAIN), PO (BER)                       | Dev Lead (RAIN), SRE (RAIN) |

## Notes

* **Accountability (A)** is singular per item to ensure clear decision‑making.
* **Consulted (C)** implies two‑way input during drafting or change. **Informed (I)** are notified on publish or status change.
* Adjust roles per program reality; this matrix is a living artifact owned by **SA (RAIN)** and reviewed in the Steering Committee monthly.
* For CAB‑governed changes, **IT Sec (BER)** and **Ops (BER)** must be at least **C**; escalate to **A** for security‑critical policies if required.
