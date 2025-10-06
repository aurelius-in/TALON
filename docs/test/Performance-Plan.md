# Performance Test Plan

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-05  

---

## 1) Purpose
Define how we validate TALON’s latency, throughput, and stability under realistic and extreme loads. Results guide capacity, SLOs, and release readiness.

## 2) Objectives
- Verify SLOs for API and Perception pipelines.  
- Establish headroom for peak + failures.  
- Detect regressions before release.  
- Produce repeatable scripts and dashboards.

## 3) Scope
- **In scope:** control‑plane API, perception workers, infra runners, policy engine, queues.  
- **Out of scope:** third‑party model training, non‑TALON plant systems.

## 4) SLIs & Targets
| SLI | Target (Prod) |
|---|---|
| API p95 latency | ≤ 250 ms |
| Perception p95 latency | ≤ 800 ms |
| Error rate (5xx) | ≤ 0.5% |
| Plan→Apply success | ≥ 99% (approved) |
| Queue wait p95 | ≤ 2 s |
| Soak stability | ≤ 0.2% error over 8 h |
| Resource saturation | CPU/GPU p95 ≤ 75% |

> Adjust per Service Definition if different.

## 5) Workload Model
Traffic drivers:
- **Shift pattern:** Day > Evening > Night multipliers.  
- **Changeover spikes:** 10–15 min bursts.  
- **Mix:** 80% perception jobs, 15% read‑only API, 5% infra plans (applies gated).

### Mix by Scenario
| Scenario | Weight |
|---|---:|
| Submit Perception Job | 35% |
| Poll Job Status | 35% |
| Retrieve Result | 10% |
| List Jobs | 10% |
| Create Infra Plan (dry‑run) | 8% |
| Apply (approved path only in stage) | 2% |

## 6) Test Types
- **Baseline:** single‑user smoke to set floor latency.  
- **Load:** steady RPS to target SLOs.  
- **Stress:** ramp until first SLO breach.  
- **Spike:** sudden 2×–3× RPS for 2–5 min.  
- **Soak:** 6–8 h steady load; watch leaks and drift.  
- **Chaos adjunct (optional):** node loss, registry delay, policy deny storm.

## 7) Environments
| Env | Purpose | Data | Notes |
|---|---|---|---|
| dev | script debug | synthetic | no paging |
| stage | pre‑prod validation | masked | canary applies |
| prod | limited, off‑peak | synthetic only | CAB approval |

## 8) Entry/Exit Criteria
**Entry**  
- Env healthy, GitOps synced, baseline green.  
- Monitoring and alerts active.  
- Synthetic data loaded.

**Exit**  
- Targets met for test type.  
- No critical errors in logs.  
- Report and comparison against last run published.

## 9) Tooling
- Generator: k6 or Locust; JMeter allowed where needed.  
- Observability: dashboards for latency, errors, saturation, queue depth.  
- Tracing: enable sampling during tests.  
- Reports: JSON + HTML summaries stored in repo.

## 10) Data & Seeding
- Synthetic frames for gauges with labeled ranges.  
- Fixed seeds for reproducibility.  
- Do not use production PII.

## 11) Test Design
### Target Load Levels (stage)
| Level | RPS | Notes |
|---|---:|---|
| Baseline | 1 | floor latency |
| Nominal | 25 | expected average |
| Peak | 60 | expected peak |
| Burst | 120 | spike window |
| Soak | 30 | 8 h run |

### Ramp Profile (example)
- Warmup 5 min → Nominal 20 min → Peak 15 min → Burst 3 min → Nominal 10 min → Cooldown 5 min.

## 12) k6 Skeleton (example)
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '5m', target: 25 },   // warmup to nominal
    { duration: '20m', target: 25 },
    { duration: '5m', target: 60 },   // ramp to peak
    { duration: '10m', target: 60 },
    { duration: '3m', target: 120 },  // burst
    { duration: '10m', target: 25 },
    { duration: '5m', target: 0 },    // cooldown
  ],
  thresholds: {
    http_req_failed: ['rate<0.005'],
    http_req_duration: ['p(95)<250'], // API target
  },
};

const BASE = __ENV.BASE_URL || 'https://talon.example/api';

export default function () {
  // Submit perception job
  const payload = JSON.stringify({ type: 'perception', image_id: 'synthetic-001' });
  const headers = { 'Content-Type': 'application/json', 'Authorization': `Bearer ${__ENV.TOKEN}` };
  const res = http.post(`${BASE}/v1/jobs`, payload, { headers });
  check(res, { 'status 202/201': (r) => r.status === 202 || r.status === 201 });
  sleep(1);
}
```

## 13) Metrics to Capture
- Latency: p50/p90/p95/p99 per endpoint.  
- Errors: rate by code; top failure reasons.  
- Saturation: CPU, memory, GPU util, disk, network.  
- Queue depth: p95 and growth during bursts.  
- Policy decisions: pass/deny counts.  
- GC pauses, thread pools, DB/registry timings if applicable.

## 14) Observability Hooks
- Correlate by `job_id`, `plan_id`, `digest`.  
- Tag test runs with `test_id`, `scenario`, `git_sha`.  
- Dashboards: *Performance Test* folder; export snapshots post‑run.

## 15) Pass/Fail
- **Pass:** All SLO thresholds met at Nominal and Peak. Spike recovers within 2 min to Nominal. Soak shows no leak trend (>5% growth in RSS/handles).  
- **Conditional:** one target narrowly missed with mitigation plan and capacity notes.  
- **Fail:** SLO(s) missed with no viable mitigation.

## 16) Risks & Mitigations
- Data not representative → build better synthetic set.  
- Shared env noise → reserve window; isolate resources.  
- Policy denies inflate errors → pre‑load approved policies for tests.

## 17) Execution Checklist
- [ ] Change approved (if Stage/Prod).  
- [ ] Dashboards/reset counters ready.  
- [ ] Seed data loaded.  
- [ ] Tokens valid.  
- [ ] Pager muted in non‑prod.  
- [ ] Start scripts; monitor SLO panels.  
- [ ] Record run IDs and links.

## 18) Reporting
- Store artifacts under `perf/reports/<date>-<git_sha>/`.  
- Include: test config, graphs (PNG), raw metrics, summary, analysis, and comparison vs. last run.  
- Create an issue for regressions with owners and due dates.

## 19) Ownership
- **Performance Lead (RAIN)** — scripts, execution, report.  
- **SRE (RAIN)** — env health, observability, capacity input.  
- **Dev Lead (RAIN)** — endpoint contracts, fixes.  
- **Eng Director (BER)** — acceptance of results.

## 20) Revision History
| Version | Date | Change |
|---|---|---|
| 0.1.0 | 2025‑09‑05 | Initial plan |
