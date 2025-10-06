# Risk/Assumption/Issue/Decision (RAID) Log

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-15  

---

## Purpose
A single source of truth to record and track **Risks, Assumptions, Issues, and Decisions** for TALON. Each entry has a clear owner and a next action.

## How to Use
1. Add an entry with a unique **ID** (e.g., R-001 for Risk, A-003 for Assumption, I-012 for Issue, D-004 for Decision).  
2. Set **Owner** to one person. Add **Due/Review** and **Next Action**.  
3. Update **Status** at least weekly; close with an outcome note and link to artifacts (ADRs, PRs, runbooks).

## Fields
- **ID** — Unique identifier (prefix: R/A/I/D).  
- **Type** — Risk | Assumption | Issue | Decision.  
- **Title** — Short, action‑oriented summary.  
- **Description** — What, where, when. For risks include cause→event→impact.  
- **Impact** — Business/technical effect if it occurs (e.g., delay, cost, quality, safety).  
- **Probability** (Risks) — Low | Medium | High.  
- **Severity** — Low | Medium | High (or compute RPN = Probability×Impact, 1–9).  
- **Owner** — Single accountable person.  
- **Contributors** — Optional consulted stakeholders.  
- **Status** — Open | Monitoring | Mitigating | Blocked | Resolved | Closed.  
- **Due/Review** — Next check‑in date; for decisions this is the decision date.  
- **Next Action** — The very next step to advance this entry.  
- **Links** — URLs to ADRs, Jira, PRs, dashboards, or docs.

## Status & SLA Guidance
- **Open**: triage within 1 business day.  
- **Monitoring**: review at least weekly.  
- **Mitigating**: active work; update progress twice weekly.  
- **Blocked**: escalate to PM/Steering within 24 hours.  
- **Closed**: capture a brief post‑note (what worked/failed).

## Example Entries

### Risks
| ID | Type | Title | Impact | Prob. | Severity | Owner | Next Action | Due | Status | Links |
|---|---|---|---|---|---|---|---|---|---|---|
| RAID‑001 | Risk | GPU supply delay for edge cluster | High | Medium | High | SA (RAIN) | Confirm alternate SKU with vendor | 2025‑10‑10 | Mitigating | JIRA‑123 |
| RAID‑002 | Risk | Model drift on gauge OCR | Medium | Medium | Medium | Dev Lead (RAIN) | Enable shadow eval on 5% of runs | 2025‑10‑15 | Monitoring | RAP‑Eval Doc |

### Assumptions
| ID | Type | Title | Owner | Next Action | Due | Status |
|---|---|---|---|---|---|---|
| RAID‑010 | Assumption | BER provides camera RTSP access | PM (RAIN) | Verify credentials in staging | 2025‑09‑20 | Open |

### Issues
| ID | Type | Title | Impact | Owner | Next Action | Due | Status | Links |
|---|---|---|---|---|---|---|---|---|
| RAID‑020 | Issue | CI signing step intermittently fails | Medium | SRE (RAIN) | Pin cosign version; add retry | 2025‑09‑18 | Mitigating | PR‑88 |

### Decisions
| ID | Type | Title | Owner | Decision | Date | Links |
|---|---|---|---|---|---|---|
| RAID‑030 | Decision | Adopt TALON policy pack v0.1 | SA (RAIN) | Approved after pilot results | 2025‑09‑25 | ADR‑000 |

## RAID Master Table (All Items)
> Use this single table if you prefer one log instead of per‑section tables.

| ID | Type | Title | Impact | Prob. | Severity | Owner | Contributors | Next Action | Due | Status | Created | Updated | Links |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| RAID‑001 | Risk | GPU supply delay for edge cluster | High | Medium | High | SA (RAIN) | PM (RAIN) | Confirm alternate SKU with vendor | 2025‑10‑10 | Mitigating | 2025‑09‑15 | 2025‑09‑15 | JIRA‑123 |
| RAID‑010 | Assumption | BER provides camera RTSP access | – | – | – | PM (RAIN) | SA (RAIN) | Verify credentials in staging | 2025‑09‑20 | Open | 2025‑09‑15 | 2025‑09‑15 | – |
| RAID‑020 | Issue | CI signing step intermittently fails | Medium | – | – | SRE (RAIN) | SecEng (RAIN) | Pin cosign version; add retry | 2025‑09‑18 | Mitigating | 2025‑09‑15 | 2025‑09‑15 | PR‑88 |
| RAID‑030 | Decision | Adopt TALON policy pack v0.1 | – | – | – | SA (RAIN) | Eng Dir (BER) | — | 2025‑09‑25 | Closed | 2025‑09‑25 | 2025‑09‑25 | ADR‑000 |

## Workflows
- **Intake:** New items enter as *Open* with owner and next action. Label by type and add to the RAID project board.  
- **Review:** Weekly project sync: update status, escalate blockers, and close items with outcome notes.  
- **Escalation:** High‑severity risks/issues escalate to Steering; change‑related items route via CAB.  
- **Closure:** When resolved, set *Closed*, record outcome, and link evidence (PRs, tickets, dashboards).

## Ownership & Cadence
- **Log Owner:** SA (RAIN) maintains the log; PM (RAIN) ensures weekly review.  
- **Meetings:** Daily stand‑up (spot‑check), weekly project sync (full RAID), monthly Steering, CAB as needed.  

## Templates
Copy/paste the desired row and replace values.

**Risk Row Template**  
`RAID-### | Risk | <title> | <Impact> | <Prob.> | <Severity> | <Owner> | <Contributors> | <Next Action> | <Due YYYY‑MM‑DD> | <Status> | <Created> | <Updated> | <Links>`

**Assumption Row Template**  
`RAID-### | Assumption | <title> | <Owner> | <Next Action> | <Due YYYY‑MM‑DD> | <Status>`

**Issue Row Template**  
`RAID-### | Issue | <title> | <Impact> | <Owner> | <Next Action> | <Due YYYY‑MM‑DD> | <Status> | <Links>`

**Decision Row Template**  
`RAID-### | Decision | <title> | <Owner> | <Decision> | <Date YYYY‑MM‑DD> | <Links>`


