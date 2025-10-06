# Service Definition

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-05  

---

## 1) Overview
This document defines the TALON service as consumed by BER: what is provided, when and how support is delivered, and the target response/restore commitments (SLAs/OLAs). It also outlines request types, escalation, and reporting.

## 2) Service Scope
- **Included**: TALON control‑plane API, perception/infra job execution, CI/CD & GitOps pipeline, policy evaluation, observability stack, signed artifact supply chain, and administrator/operator support.  
- **Adjacent** (coordination only): plant network, camera hardware, third‑party model training pipelines.  
- **Excluded**: custom feature development outside agreed backlog, 24×7 on‑site support, legacy system rewrites.

## 3) Service Tiers
| Tier | Environment | Purpose | Support Window |
|---|---|---|---|
| T1 | **Prod** | Line runtime | 24×7 paging for critical incidents; change windows per cutover plan |
| T2 | **Stage** | UAT & pre‑prod | Business hours with best‑effort after‑hours for cutovers |
| T3 | **Dev** | Integration & testing | Business hours only |

## 4) Support Hours
- **Business hours**: Mon–Fri 08:00–18:00 local site time (excluding holidays).  
- **After hours** (prod): on‑call rotation for Sev‑1/Sev‑2 only.  
- Planned changes follow the **Communication & Meeting Cadence** and **Cutover Plan** documents.

## 5) Request Types
| Type | Examples | Channel | Target Handling |
|---|---|---|---|
| **Incident** | Outage, SLO breach, security event | Pager + incident bridge | Per SLA below |
| **Service Request** | Access request, new camera onboarding, policy tweak | Ticket | First response in 1 business day |
| **Change** | Config change, deploy, infrastructure update | Ticket + PR | As per CAB or change policy |
| **Information** | How‑to, docs, training | Ticket | First response in 2 business days |

## 6) Severity & SLAs (Response/Restore)
| Severity | Definition | Initial Response (Prod) | Restore/Workaround (Prod) | Initial Response (Non‑Prod) |
|---|---|---:|---:|---:|
| **Sev‑1** | Complete outage/safety risk; business stopped | 5 min | 2 h | 30 min |
| **Sev‑2** | Major degradation; material SLO breach | 15 min | 4 h | 4 h |
| **Sev‑3** | Minor impact with workaround | 4 h | 2 business days | 1 business day |
| **Sev‑4** | Low impact/cosmetic | 1 business day | 5 business days | 2 business days |

> “Response” = human acknowledgment + triage start. “Restore” = service functional (may be in DR mode).

## 7) OLAs (Internal Targets)
- Pipeline restore (CI/CD/GitOps): 2 h Sev‑1, 4 h Sev‑2.  
- Policy bundle distribution: 1 h Sev‑1.  
- Registry/signing verification: deny unsigned within 1 min; restore signing within 4 h.  
- Observability stack: dashboards available within 4 h; logs may be delayed during DR.

## 8) Maintenance & Change Windows
- **Standard window**: Tue/Thu 02:00–04:00 local.  
- **Freeze**: during peak production or when error‑budget burn is high.  
- **Emergency changes**: allowed with IC approval; must include backout plan.  
- **CAB**: required for production‑impacting changes or policy/risk changes.

## 9) Escalation Path
1. Primary On‑Call (P1) → Secondary (P2).  
2. Incident Commander (IC) (PM/SA).  
3. BER Engineering Director.  
4. Vendor support (registry/DNS/GPU) when applicable.

Contacts are maintained in the **SRE Playbook** and kept current.

## 10) SLOs & Error Budgets (Summary)
- **Availability (Prod)**: ≥ 99.5% monthly for control‑plane.  
- **API Latency (p95)**: ≤ 250 ms.  
- **Perception Latency (p95)**: ≤ 800 ms and queue depth controlled.  
- **Plan→Apply Success**: ≥ 99% when approved.  
- Error budget policy: if burn > 2× threshold over 1 h, freeze non‑urgent changes and initiate mitigation review.

## 11) Reporting
- **Weekly**: incident summary, SLO status, open risks/actions (RAID).  
- **Monthly**: trend of SLIs, capacity vs. forecast, policy exceptions, top incidents and MTTR.  
- **Quarterly**: service review with improvement plan and roadmap alignment.

## 12) Dependencies
- BER provides plant network connectivity, camera endpoints, directory/SSO (if used), and any site approvals.  
- External vendors: container registry, DNS/traffic manager, GPU platform, object storage provider.  
- All dependencies must have documented SLAs or OLAs and contact points.

## 13) Guardrails & Policies
- Signed artifacts and SBOM verification at admission.  
- Policy pack governs safety/infra thresholds; changes via PR only.  
- Least‑privilege access; short‑lived tokens; auditable approvals.  
- Data handling per **Data Classification & Handling**.

## 14) Service Catalog (High‑Level)
| Service | What You Get | Request Path |
|---|---|---|
| **TALON Runtime** | API, workers, policy engine, observability | Incident/Change/Ticket |
| **Job Execution** | Perception & infra jobs with policy enforcement | Console/CLI + Ticket if change |
| **Release Pipeline** | Build, sign, promote via GitOps | PR + Change request |
| **Observability** | Dashboards, alerts, logs & traces | Access request via ticket |
| **Training** | Operator quickstart and admin guide | Training request via ticket |

## 15) Roles & Responsibilities (Summary)
- **RAIN SRE/Platform**: uptime, scaling, deploys, on‑call.  
- **RAIN Solution Architect**: architecture, policy and SLO ownership, change approval.  
- **RAIN Dev Lead**: API and worker services.  
- **BER Engineering Director**: acceptance and production approvals.  
- **BER Product Owner**: prioritization and UAT coordination.  
- **BER IT Security**: access, policy alignment, audits.

## 16) Service Onboarding
- Submit service request with site details, contacts, and change windows.  
- Configure connectivity and RBAC; import users/groups.  
- Run smoke tests; confirm dashboards and paging.  
- Record SLOs and publish service entry in catalog.

## 17) Offboarding
- Drain/disable jobs; snapshot artifacts and dashboards.  
- Revoke credentials; remove access; archive Git repos.  
- Provide final report with SLO attainment and incident history.

## 18) Financials (Placeholder)
- Pricing and billing model (T&M vs. fixed‑fee), tied to tiers and environments.  
- Optional service credits for missed SLAs to be defined in the MSA/SOW.

## 19) Versioning & Review
- This service definition is versioned in the repo; changes via PR.  
- Review cadence: quarterly, or when SLO/architecture changes materially.

## 20) References
- SRE Playbook • Capacity Plan • Disaster Recovery Plan • Communication & Meeting Cadence • Runbooks • Policy Pack • Data Classification & Handling • RACI Matrix.
