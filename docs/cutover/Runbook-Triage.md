# Runbook — Triage

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑09‑27  

---

## Purpose

This runbook defines the process for triaging operational incidents in the **TALON** platform, focusing on three common failure modes: high latency, low confidence scores, and policy evaluation failures.  It guides responders through diagnosis, remediation and escalation while keeping the system secure and compliant.  The document contains no confidential information and can be shared publicly as an example of structured triage procedures.

## Overview

Incidents can manifest as slow responses from inference or infrastructure agents, unusually low confidence scores returned by perception jobs, or failed policy checks that block job execution.  Timely triage is essential to maintain service levels and trust.  This runbook provides a decision tree and concrete steps to identify root causes and stabilise the system.

## Pre‑requisites

- Access to observability tools, including logs (e.g. via `kubectl logs` or the logging service), metrics dashboards, and tracing tools for TALON microservices.
- Knowledge of baseline performance metrics for latency and confidence.  Establish thresholds for acceptable latency (e.g. p50 <2s, p95 <5s) and confidence (e.g. ≥0.92).
- Access to the policy engine logs (e.g. OPA or equivalent) to review evaluation results.
- Ability to execute diagnostic commands and, if authorised, restart services or scale infrastructure.
- Contact details for on‑call engineers and escalation paths.

## Decision Tree

1. **Alert Received** – An alert triggers on one of the following: response latency exceeds thresholds, confidence scores drop below acceptable range, or a policy evaluation fails.
2. **Confirm Scope** – Determine whether the issue is isolated to a specific job, endpoint, environment or affects all traffic.
3. **Categorize Incident**
   - If the alert relates to slow responses from `/job/perception` or `/job/infra`, follow **Latency Triage**.
   - If the alert relates to low confidence scores from perception jobs, follow **Confidence Triage**.
   - If the alert relates to policy engine failures (status “fail” in job output), follow **Policy Failure Triage**.

## Latency Triage

1. **Check System Health**
   - Review the metrics dashboard for CPU, memory and GPU utilisation of the relevant pods.  Look for resource exhaustion.
   - Inspect the number of queued jobs.  A growing backlog may indicate insufficient concurrency.
   - Check for recent changes (deployments, configuration updates) that may have affected performance.
2. **Examine Logs**
   - Use `kubectl logs` or your logging tool to view application logs around the time of the alert.  Look for timeouts, retry attempts or errors connecting to dependent services.
   - If using gRPC or HTTP, check for increased latency at upstream dependencies (e.g. model server, database, external APIs).
3. **Assess Hardware/Infrastructure**
   - Verify that nodes are healthy.  Look for node-level issues (disk pressure, network saturation) in the cluster dashboard.
   - If running on edge hardware, ensure cameras and capture devices are connected and not saturating the bus.
4. **Mitigation Steps**
   - **Scale resources** – If capacity is constrained, scale up pods or nodes via the autoscaler.  For GPU workloads, ensure GPUs are not oversubscribed.
   - **Restart stuck pods** – Rolling restart of the affected services may clear hung connections or memory leaks.
   - **Throttle or queue management** – Temporarily reduce input rate or increase queue worker concurrency to clear backlog.
   - **Configuration rollback** – If a recent deployment caused the regression, consider rolling back to a previous version using the **Runbook — Rollback**.
5. **Validation**
   - After remediation, monitor latency metrics and confirm they return within acceptable thresholds.  Perform ad‑hoc requests to verify improvement.

## Confidence Triage

1. **Validate Inputs**
   - Retrieve the offending job’s request, including `image_id` and `roi`, and visually inspect the captured frame if possible.  Poor lighting, blur or occlusions can lead to low confidence scores.
   - Check the correctness of the region of interest (ROI) coordinates.  An incorrect ROI may cause the model to analyse the wrong part of the image.
2. **Review Model Version and Data Drift**
   - Confirm which model version produced the result.  Ensure that the expected model version is deployed and that the model has not been inadvertently downgraded.
   - Investigate whether there has been a shift in the data distribution (e.g. new gauge designs or camera placement) that the model was not trained to handle.
3. **Check Metrics**
   - Use monitoring tools to examine trends in confidence scores over time.  A systemic decline may indicate model drift or hardware degradation (e.g. camera autofocus failure).
4. **Mitigation Steps**
   - **Retry** – For transient issues, re-run the job to see if confidence improves.
   - **Adjust ROI or settings** – Update the ROI or camera settings to improve image quality.  Coordinate with onsite personnel if cameras need adjustment.
   - **Model update** – If drift is suspected, schedule a retraining or deploy a model better suited to current conditions.  Evaluate models offline before promoting them.
   - **Escalate** – If the issue persists across multiple devices or locations, open an incident and notify the machine learning team.
5. **Validation**
   - Execute test jobs or use a labeled dataset to confirm that confidence scores return to expected levels after remediation.

## Policy Failure Triage

1. **Identify the Failed Policy**
   - Examine the job response’s `policy` section to determine which rule triggered the failure.  Look for specific violation messages.
   - Review policy engine logs for details of the evaluation.  Policies may check naming conventions, tag compliance, or specific thresholds (e.g. minimum confidence, maximum latency).
2. **Determine Input Issues**
   - For **infra** jobs: Confirm that the requested change includes all required fields (e.g. tags, correct resource prefixes) and complies with naming and CIDR conventions.
   - For **perception** jobs: Verify that the inference result meets the expected confidence and latency thresholds defined in the policy.
3. **Review Recent Policy Changes**
   - Check if new rules or threshold changes were recently introduced.  Policy updates without proper testing can cause unexpected rejections.
4. **Mitigation Steps**
   - **Correct the input** – Amend the change request or adjust the job parameters to meet policy requirements.
   - **Adjust policy thresholds** – If the policy is too strict, propose a policy update through the normal change management process.  Ensure proper peer review and testing before deployment.
   - **Roll back policy** – If a recent policy change is causing widespread failures, revert to the previous version by updating the policy repository.
   - **Bypass (exception)** – In emergency situations, a break‑glass procedure may allow temporary bypass of specific rules.  All exceptions must be logged and reviewed.
5. **Validation**
   - Re-run the job after correction and verify that it passes policy evaluation.  Monitor for additional policy failures to ensure systemic issues are resolved.

## Escalation and Documentation

1. **Notify On‑Call** – If the issue cannot be resolved within 30 minutes or has customer impact, page the on‑call engineer and the responsible team via the designated channel.
2. **Open an Incident** – Create an incident ticket in the incident management system (e.g. Jira, PagerDuty) with a description, severity, time of occurrence, and actions taken.
3. **Post‑Incident Review** – After resolution, conduct a blameless post‑mortem to document the root cause, corrective actions and preventive measures.  Update runbooks and monitoring thresholds accordingly.
4. **Audit Logs** – Ensure that all steps, commands, and policy decisions are recorded.  Retain logs in accordance with compliance requirements.

## Notes

- Baselines and thresholds should be revisited regularly as the system evolves.  Adjust monitoring alerts to minimize false positives and false negatives.
- Sensitive data (e.g. images containing proprietary information) should only be accessed by authorized personnel and handled according to privacy guidelines.
- Use profiling tools to examine latency and performance bottlenecks in AI inference.  Consistent monitoring of latency, throughput and resource usage enables proactive tuning.
- Implement structured logging and include correlation IDs to trace a job through the various components of TALON.  This aids in troubleshooting and root‑cause analysis.
