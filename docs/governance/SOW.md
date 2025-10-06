# Statement of Work (SOW)

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-14  

---

## 1. Background
Blue Eagle Robotics seeks to deploy TALON, an agentic platform that orchestrates perception workloads and infrastructure automation at factory sites. RAIN will design, implement, and pilot TALON to demonstrate measurable improvements in reliability, security, and time‑to‑value.

## 2. Objectives
- Establish a secure, repeatable build/sign/deploy pipeline for TALON.  
- Deliver an on‑prem pilot that performs line‑readiness checks and infrastructure changes via policy.  
- Prove success against agreed SLIs/SLOs and hand over runbooks and training.

## 3. Scope
### 3.1 In Scope
- Architecture: HLD/LLD, threat model, observability plan.  
- Platform: GitOps, CI/CD, image signing, SBOM, policy evaluation.  
- Application: Perception and infra job APIs; minimal reference UI.  
- Infrastructure as Code: baseline modules and environments (dev/stage/prod).  
- Compliance & Ops: runbooks (deploy/rollback/triage), cutover plan, acceptance.  

### 3.2 Out of Scope
- Broad feature development beyond pilot use cases.  
- 24×7 production support.  
- Legacy system rewrites.  
- Non‑standard hardware procurement.

## 4. Deliverables
- **Design**: HLD, LLD, API (OpenAPI), data contracts.  
- **Security & Compliance**: image signing & SBOM SOP, STRIDE threat model, data classification & handling.  
- **Platform**: CI/CD design, policy pack (thresholds & OPA), IaC modules.  
- **Operations**: runbooks (deploy/rollback/triage), observability plan, KPIs & dashboard links.  
- **Project Governance**: communication & meeting cadence, RACI matrix, 30/60/90 plan, RAID log.  
- **Acceptance Package**: pilot results, acceptance certificate, cutover plan.

## 5. Milestones & Timeline
| Milestone | Description | Target Date |
|---|---|---|
| M1 | HLD approved; environments provisioned (dev/stage) | 2025‑09‑26 |
| M2 | CI/CD + signing + policy gates functional | 2025‑10‑03 |
| M3 | Pilot features complete; dashboards live | 2025‑10‑17 |
| M4 | UAT & acceptance testing completed | 2025‑10‑24 |
| M5 | Cutover executed; acceptance certificate signed | 2025‑10‑31 |

## 6. Success Criteria & Acceptance
- TALON deploys via GitOps with signed artifacts and passing policy checks.  
- Perception job returns value/confidence within agreed latency percentiles.  
- Infra job produces a plan, passes policy, and applies with zero drift on re‑check.  
- Dashboards show SLIs/SLOs; alerting active for error/latency thresholds.  
- Runbooks validated during dry‑run and pilot incident simulations.  
- All deliverables reviewed and accepted by BER Engineering Director.

**Acceptance Process**  
1) BER reviews deliverables against this SOW and linked criteria.  
2) Variances are logged in RAID with owners and due dates.  
3) On closure of variances, BER issues the Acceptance Certificate.

## 7. Assumptions & Dependencies
- BER provides network access, credentials, and test data.  
- Air‑gapped transfers follow the agreed offline bundle process.  
- Standard change management via CAB for production‑impacting changes.  
- Licensing for required third‑party components is available.

## 8. Roles & Responsibilities (summary)
- **RAIN Solution Architect** — owns architecture, policy design, handover.  
- **RAIN SRE/Platform** — owns CI/CD, GitOps, infra modules, observability.  
- **RAIN Dev Lead** — owns API/services, reference implementations.  
- **BER Engineering Director** — final technical authority and acceptance.  
- **BER Product Owner** — use‑case priorities and UAT coordination.  
- **BER IT Security** — policy alignment and access approvals.

## 9. Change Control
- Scope, schedule, or deliverable changes require a change request (CR) with impact analysis.  
- CRs are reviewed in weekly project sync and, if production‑impacting, in CAB.  
- Approved CRs update this SOW’s version and the project plan.

## 10. Communication
- Cadence per “Communication & Meeting Cadence” doc (daily stand‑up; weekly sync; monthly steering).  
- Status shared via dashboards and RAID log; decisions captured as ADRs.

## 11. Handover & Training
- Admin and operator runbooks delivered.  
- Knowledge transfer sessions recorded; artifacts stored in the project repo.

## 12. Warranty & Support (pilot phase)
- Defect remediation for pilot scope during the pilot window.  
- Severity triage and response per runbook; production support is out of scope.

## 13. Pricing & Invoicing (placeholder)
- Time & Materials or Fixed‑Fee sections may be inserted here if required by BER.  
- Invoices aligned to milestones M1–M5 or monthly, as agreed.

## 14. Intellectual Property & Open Source
- RAIN retains ownership of pre‑existing materials.  
- BER receives a non‑exclusive right to use deliverables for internal operations.  
- Open‑source components and obligations are listed in the IP & Open Source Annex.

