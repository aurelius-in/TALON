# KPIs & Dashboard Links

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑09-25

---

## Purpose

This document defines key performance indicators (KPIs) for the TALON platform and provides guidance on how to monitor them.  These KPIs measure the health of two core workflows – **perception** (AI‑driven gauge readings) and **provisioning** (infrastructure plan and apply) – as well as the underlying platform.  Clear metrics and dashboards help engineers and operators observe system behaviour, detect anomalies quickly and drive continuous improvement.

## KPI categories

### Perception workflow

| KPI | Definition | Goal |
| --- | --- | --- |
| **Accuracy rate** | Percentage of perception jobs where the AI inference matches ground truth. This can be measured periodically against a labelled validation set. | ≥95 % on pilot lines |
| **Latency (p50 / p95)** | Time in milliseconds from receipt of a perception job to a completed inference. Break down latency into median (p50) and tail (p95) to identify long‑running outliers. | p50 ≤ 2 s; p95 ≤ 5 s |
| **Throughput** | Number of perception jobs processed per minute. Monitored to ensure system can handle expected load. | Scales linearly with input volume |
| **Confidence distribution** | Distribution of model confidence scores over time. Unexpected drops may indicate data drift or hardware issues. | Stable mean and variance |
| **Policy pass/fail rate** | Ratio of perception results that pass defined thresholds (e.g., minimum confidence, maximum latency) versus those that require review. | Pass ≥ 90 % |
| **Error rate** | Percentage of perception jobs that end in error due to missing frames, model errors or system exceptions. | < 1 % |

### Provisioning workflow

| KPI | Definition | Goal |
| --- | --- | --- |
| **Plan generation time** | Time to generate the infrastructure plan artifact (including dependency resolution) for a change request. | < 10 s for typical plans |
| **Policy violations per plan** | Number of failed policy checks (e.g. naming/tagging rules, cost diffs) in a generated plan. Zero means the plan meets all guardrails. | Zero critical violations |
| **Approval lead time** | Time from plan creation to approval by an authorised reviewer.  Shorter lead times indicate efficient governance. | < 24 h |
| **Apply success rate** | Percentage of applies that complete without errors or rollbacks. | ≥98 % |
| **Change failure rate** | Ratio of infrastructure changes that cause service degradation.  Lower values indicate stable rollouts. | < 3 % |

### Platform & system metrics

| KPI | Definition | Goal |
| --- | --- | --- |
| **CPU / memory / GPU utilisation** | Resource usage on nodes running TALON control plane and AI model inference. | < 80 % sustained utilisation |
| **Queue backlog** | Number of unprocessed jobs in the message queue.  A growing backlog may indicate bottlenecks. | Maintain backlog near zero |
| **Cost per job** | Estimated cost (compute, storage, licensing) per perception or provisioning job. | Within budgeted cost envelope |
| **User feedback score** | Rating collected from operators for responsiveness and ease of use (1–5 stars or Net Promoter Score). | Score ≥ 4/5 |
| **Incident MTTR (Mean Time to Recovery)** | Average time to detect and remediate incidents that impact TALON. | < 4 h |

## Dashboards and observability

To effectively monitor these KPIs, TALON uses a set of dashboards.  Each dashboard visualises the metrics above and enables drill‑down into anomalies.  Example dashboards include:

- **Perception Overview**: Shows p50 and p95 latency over time, accuracy rate on validation sets, confidence distribution histograms, policy pass/fail ratio and throughput.  Alerts trigger when latency or accuracy breach defined thresholds.
- **Provisioning Pipeline**: Displays plan generation durations, number of policy violations per plan, approval lead times, and apply success rates.  Includes a bar chart of violations by rule type to highlight common infra guardrail breaches.
- **Resource Utilisation**: Monitors CPU, memory and GPU usage across control plane nodes and inference hosts.  Incorporates queue backlog length to correlate resource saturation with job processing.
- **Cost & Usage**: Tracks cost per perception job and cost diffs per infra change.  Charts show cumulative cost against allocated budget and highlight trends that require optimisation.
- **User Feedback & Incident Response**: Aggregates user satisfaction ratings and mean time to recovery.  Also features an incident timeline to correlate outages with infrastructure changes or perception anomalies.

### Accessing dashboards

The following links (internal to the RAIN/BER environment) provide access to the live dashboards:

- `https://dashboards.rain.ber/perception-overview`
- `https://dashboards.rain.ber/provisioning-pipeline`
- `https://dashboards.rain.ber/resource-utilisation`
- `https://dashboards.rain.ber/cost-and-usage`
- `https://dashboards.rain.ber/feedback-and-incidents`

> *Note*:  If viewing this document outside the internal network, these links will not resolve.  When publishing this document in a public portfolio, replace the links with screenshots or anonymised copies of the dashboards.

## Designing a dashboard

To build effective dashboards for TALON:

1. **Define metrics clearly**:  Use concise names and descriptions.  Align dashboards with the KPIs defined above.
2. **Include time series & distribution views**:  Plot latency and throughput over time with percentile bands.  Use histograms for confidence and cost distributions.
3. **Group by dimension**:  Segment metrics by line, location or model version to identify which deployments drive anomalies.
4. **Provide drill‑down**:  Enable click‑through to logs, traces or plan artifacts for detailed investigation.
5. **Alerting & SLOs**:  Configure alerts based on SLO thresholds.  For example, trigger notifications if p95 latency > 5 s for over five minutes.
6. **Review regularly**:  Update dashboards and KPIs as the product evolves and new patterns emerge.

By tracking these KPIs and reviewing the dashboards regularly, Blue Eagle Robotics and RAIN can ensure the TALON platform delivers reliable, efficient and auditable performance.
