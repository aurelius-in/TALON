# Capacity Plan

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025-09-09 

---

## 1) Purpose
Define how TALON forecasts demand, sizes environments, and triggers scale actions so that SLIs/SLOs are consistently met at optimal cost.

## 2) Scope
Covers control-plane, perception workers (GPU/CPU), infra runners, storage, and network for `dev`, `stage`, and `prod` (single site). Multi‑site notes included in §12.

## 3) Service Goals (Targets)
- **Availability**: ≥ 99.5% monthly (prod).  
- **Latency** (Perception p95): ≤ 800 ms.  
- **Latency** (API p95): ≤ 250 ms.  
- **Throughput**: sustain 6 jobs/sec average, burst to 12 jobs/sec for 10 min.  
- **Error rate**: ≤ 0.5% 5‑minute rolling.

## 4) Capacity Units & KPIs
| Layer | Unit | Primary KPIs |
|---|---|---|
| Control‑plane | vCPU / GiB RAM / replicas | API p95 latency, RPS, CPU%, mem% |
| Perception workers | GPU count / SM util / VRAM | job latency p95, GPU util%, queued jobs |
| Infra runners | concurrent plans/applies | queue wait, plan duration p95 |
| Storage | TiB, IOPS, throughput | read/write latency p95, occupancy% |
| Network | Gbps | egress/ingress, packet drops, RTT |

## 5) Demand Model
We model **jobs per hour** and **payload complexity**:
- **Perception**: jobs/hr × (avg frames × model complexity).  
- **Infra**: change requests/week × average resource delta.  
Traffic exhibits **shift‑based seasonality** (Day/Evening/Night) and **changeover spikes**.

## 6) Baselines (Week‑0 after pilot)
| Metric | Dev | Stage | Prod |
|---|---:|---:|---:|
| Perception jobs/hr (avg/peak) | 30 / 80 | 60 / 140 | 200 / 480 |
| API RPS (avg/peak) | 5 / 15 | 10 / 30 | 40 / 120 |
| GPU util% (avg/p95) | 22 / 55 | 35 / 65 | 58 / 85 |
| Queue depth (p95) | 2 | 4 | 12 |
| Storage occupancy% | 22 | 35 | 61 |
| Network egress Gbps (p95) | 0.2 | 0.5 | 1.6 |

> Replace with your first‑week measurements; use them as seeds for forecasting.

## 7) Forecasting Method
- **Short‑term (1–2 weeks)**: Exponentially Weighted Moving Average (EWMA) on jobs/hr and latency.  
- **Seasonal effects**: Day‑of‑week and shift multipliers (e.g., ×1.3 at changeovers).  
- **Project events**: For known campaigns or line additions, add step functions (Δ demand).  
- **Headroom policy**: keep **25–35%** spare capacity on the most constrained resource (GPU or CPU).

### Example (Perception GPUs)
```
needed_gpus = ceil( (peak_jobs_per_sec × cost_per_job_ms / 1000) / (gpu_capacity_fraction × concurrency) × (1 + headroom) )
```
Where **cost_per_job_ms** is measured average inference time per job on a single worker; **gpu_capacity_fraction** is safe utilization (e.g., 0.75).

## 8) Scaling Triggers (Prod)
Triggers fire only when **two conditions** are met for **10 consecutive minutes** to avoid flapping.

### Perception (GPU workers)
- **Scale‑out**: (GPU util p95 ≥ 75% **AND** job latency p95 ≥ 800 ms) → +1 worker (or +N to clear queue).  
- **Scale‑in**: (GPU util p95 ≤ 45% **AND** queue depth p95 ≤ 2) for 30 min → −1 worker.  
- **Queue‑based**: queue depth p95 ≥ 25 → +1 worker immediately (burst rule).

### Control‑Plane (API)
- **Scale‑out**: (CPU p95 ≥ 70% **OR** mem p95 ≥ 75%) **AND** API p95 ≥ 250 ms → +1 replica.  
- **Scale‑in**: CPU p95 ≤ 35% for 30 min → −1 replica (min=2).

### Infra Runners
- **Scale‑out**: wait time p95 ≥ 2 min **OR** backlog ≥ 10 plans → +1 runner (max N).  
- **Throttling**: if policy denials spike, cap concurrency and alert SecEng.

### Storage & Network Alerts
- Storage occupancy ≥ 80% → expand volume by next window; ≥ 90% → emergency expand.  
- Network egress p95 ≥ 80% link cap for 15 min → investigate & upgrade path.

## 9) Provisioning & Reservations
- **GPU pools**: maintain N+1 spare GPUs in prod; one spare worker in stage.  
- **Reserved capacity**: 20% of runner slots reserved for emergency changes.  
- **Burst strategy**: allow temporary scale to cloud burst workers where permitted.

## 10) Cost Guardrails
- Monthly budget caps with alerts at **70/85/100%**.  
- Prefer **scale‑in** over upgrades when SLOs are green ≥ 7 days.  
- Night hours: allow deeper scale‑in with stricter queue limits.

## 11) Dashboards & Alerts
- **Capacity**: GPU/CPU util, queue depth, storage%, network Gbps, per‑env panels.  
- **Forecast vs Actual**: EWMA forecast, measured demand, error band ±15%.  
- **Alert Policies**: all triggers in §8 encoded as rules; page on p95 breaches.

## 12) Multi‑Site Notes
- Keep site‑local buffers; never drain a site below 20% headroom if others are full.  
- Cross‑site failover entails doubled control‑plane capacity for 1 hour (test quarterly).

## 13) Change Management
- Encode triggers in HPA/Autoscaler/Jobs controller as code.  
- All threshold changes by PR with review from SA (RAIN) and Eng Dir (BER).  
- Record trigger fires and outcomes in the ops report; tune monthly.

## 14) Review Cadence
- **Weekly**: check forecast error; adjust multipliers.  
- **Monthly**: right‑size baseline; revisit headroom policy.  
- **Quarterly**: load test at +50% expected peak; validate cost guardrails.

## 15) Open Items (fill in)
- Target GPU model capacity (ms/job) per workload: _______.  
- Max concurrent infra applies allowed: _______.  
- Storage growth rate (GiB/day): _______.  
- Network headroom target (%): _______.


