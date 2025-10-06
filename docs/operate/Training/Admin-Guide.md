# Training — Admin Guide

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-13

---

## 1) Purpose & Audience
This guide trains platform **administrators** to operate TALON day‑to‑day: user and access management, configuration, deployments, policy administration, monitoring, backup/restore, and incident troubleshooting.

## 2) Concepts (Quick Ref)
- **TALON Control‑Plane**: API + agents coordinating perception jobs and infra jobs.  
- **GitOps**: Desired state is defined in Git. Controllers reconcile cluster/envs to Git.  
- **Signed Artifacts**: Container images and SBOMs are signed; policy requires valid signatures.  
- **Policy Pack**: Rego rules + thresholds for security, safety, and quality gates.  
- **Observability**: Logs, metrics, traces and dashboards for both app and platform.

## 3) Roles & Permissions
| Role | Scope | Typical Personas |
|---|---|---|
| Admin | Full read/write on platform and namespaces | RAIN SRE, BER Platform Admin |
| Operator | Deploy apps, view dashboards, run runbooks | BER Ops |
| Viewer | Read‑only dashboards and logs | Stakeholders |
| Security | Policy management, signing keys, audits | BER IT Sec / RAIN SecEng |

> RBAC is managed via groups. Always grant the minimum role that enables the task.

## 4) Prerequisites
- Admin workstation with terminal, Git, container runtime, access to the GitOps repo.  
- Credentials for the orchestration cluster(s) and signing infrastructure.  
- Access to monitoring (dashboards) and artifact registry.  
- Approved change ticket if a production change is required.

## 5) Environment Matrix
| Env | Purpose | Change Window | Promotion Path |
|---|---|---|---|
| `dev` | Developer integration & smoke tests | Anytime | feature → `dev` |
| `stage` | Pre‑prod test & UAT | Daily window | `dev` → `stage` |
| `prod` | Manufacturing/line runtime | CAB‑approved | `stage` → `prod` |

## 6) Day‑to‑Day Admin Tasks
1. **Review Dashboards** (AM/PM): error rates, latency, job throughput, policy denials.  
2. **Check GitOps Status**: all apps/envs should be `Synced` and `Healthy`.  
3. **Rotate On‑Call Notes**: known issues, temporary mitigations, pending CRs.  
4. **Backlog Grooming**: convert recurring toil into automation tasks.

## 7) User & Access Management
- Onboard: add user to the correct group; verify least‑privilege.  
- Offboard: remove from groups; revoke tokens; rotate shared secrets if used.  
- Service Accounts: issue short‑lived credentials; restrict to namespace/scope.  
- Key Management: store signing keys in HSM/KMS; enable key rotation schedule.

## 8) Configuration Management
- All configuration is stored in Git as code.  
- To change config, **create a PR** against the environment folder.  
- Ensure: schema validated; policy checks pass; reviewers approved.  
- Merge PR → GitOps reconciles automatically per environment cadence.

## 9) Deployments (Admin Workflow)
1. **Build & Sign**: CI builds image, generates SBOM, signs both.  
2. **Promote via GitOps**: update image tag/digest in `dev` kustomization/helm values.  
3. **Smoke Tests**: run platform smoke (health, auth, policy), then app smoke (perception job, infra job dry‑run).  
4. **Promote to `stage`**: repeat tests + UAT sign‑off.  
5. **Promote to `prod`**: CAB approval, maintenance window, run canary if configured.

### Quick Commands (examples)
```bash
# verify image signature
cosign verify --key cosign.pub ghcr.io/rain/talon@sha256:<digest>

# reconcile now (if your controller supports manual sync)
git pull && kubectl apply -k envs/dev
# or: argocd app sync talon-dev
```

## 10) Policy Administration
- Thresholds (e.g., max latency, min confidence) and safety/infra rules live in the **policy pack**.  
- To change a policy: PR → unit tests → policy eval in CI → review → merge.  
- In `prod`, policy changes follow CAB if they can impact availability/safety.  
- Always document rationales and link to incidents or experiments that justify the change.

## 11) Monitoring & Alerting
- **SLIs**: request latency (pXX), error rate, confidence distribution, job throughput, plan→apply success.  
- **SLOs**: e.g., p95 latency < 800 ms in prod; error rate < 0.5%.  
- **Dashboards**: platform health, job pipelines, policy decisions, resource capacity.  
- **Alerts** (page): SLO burn, policy deny spikes, unsynced GitOps app, signing verification failures.  
- **Alerts** (ticket): low‑priority drifts, nearing capacity, SBOM scan warnings.

## 12) Backups & Restore
- **What**: Git repositories, manifests, policy pack, signing keys (in KMS/HSM), critical databases, dashboard configs.  
- **When**: nightly in `prod`, retained per data policy; weekly in `stage`.  
- **Restore Drill**: quarterly; document RTO/RPO achieved.  
- **Test**: validate a restore by reconciling a staging environment from backup.

## 13) Troubleshooting Playbooks
### A) High Latency (Perception)
1. Check dashboard: p95 latency, GPU/CPU utilization, queue depth.  
2. Inspect recent deploys or policy changes.  
3. Scale workers or adjust batch/parallelism settings.  
4. Roll back to last good digest if regression confirmed.

### B) Low Confidence (Perception)
1. Inspect sample images for glare, occlusion; validate camera focus/zoom.  
2. Compare model version; run A/B with last known good.  
3. Increase sampling; route cases to human review queue if configured.  
4. Open model‑tuning ticket with evidence.

### C) Policy Deny (Infra or Safety)
1. Open policy decision log; identify failing rule and input values.  
2. If false positive: fix inputs or adjust threshold in PR.  
3. If valid deny: follow change path or security process.  
4. Re‑run evaluation after merge.

### D) GitOps Sync Failed
1. View app status (`OutOfSync/Degraded`) and health messages.  
2. Validate manifest schema and image digest; check RBAC/service account.  
3. Revert last PR if urgent; otherwise fix and merge a correction.  
4. Force sync if needed when the fix is merged.

### E) Image Signature Verification Failed
1. Confirm digest and provenance; ensure public key matches trust policy.  
2. Check CI logs—was the image actually signed? re‑sign if needed.  
3. Inspect registry permissions and transparency log (if used).  
4. Block promote until signature verifies; open incident if in prod.

### F) SBOM/Scan Finding
1. Verify severity and exploitability; check if a fix version exists.  
2. Patch or pin dependency; rebuild and re‑sign image.  
3. Add temporary policy exception only with time‑boxed expiry.

## 14) Run & Maintenance Checklists
**Daily**
- Review dashboards & alerts; acknowledge/resolve pages.  
- Confirm GitOps apps are healthy; zero critical policy denials.  
- Check remaining capacity (GPU/CPU, storage, queue depth).

**Weekly**
- RAID review; close stale items.  
- Rotate minor credentials/tokens as scheduled.  
- Review open policy exceptions; renew or remove.

**Monthly**
- Restore drill in staging; sample audit of access logs.  
- Steering update: SLIs/SLOs, incidents, improvement plan.

## 15) Audit & Reporting
- Keep immutable logs of deploys, policy decisions, and sign/verify events.  
- Tag releases with signed SBOM references.  
- Export weekly ops report: uptime, SLO status, incidents, MTTR, exceptions.

## 16) Decommissioning
- Drain workloads; archive artifacts and dashboards.  
- Revoke credentials; rotate shared keys; remove policies.  
- Snapshot Git repos and mark archived; update inventory.
