# Runbook — Rollback

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑09-28

---

## Purpose

This runbook describes the procedure for rolling back a deployment of **TALON** to the last known good release.  It provides step‑by‑step instructions for locating the most recent signed artifact, reverting the infrastructure via GitOps, validating the restoration and documenting the incident.  The steps herein are non‑confidential and suitable for public sharing.

## Overview

Rollbacks may be necessary when a new release introduces severe defects, performance regressions or misconfigurations.  The rollback process uses a GitOps workflow to revert the manifest to reference the last signed digest.  After rolling back, smoke tests are run to confirm stability.  An incident record is created to document the event and feed into continuous improvement.

## Pre‑requisites

- Access to the GitOps repository and permission to create and merge pull requests.
- Knowledge of the last known good version/digest (e.g. from release notes, change log or previous GitOps commit).  If unclear, coordinate with the release manager to identify the correct digest.
- Availability of the signed container image and SBOM in the registry.  If the registry has been cleaned, retrieve the image from the backup registry or local cache.
- Defined smoke tests or health checks for TALON APIs and agents.
- Notification channels (e.g. Slack/Teams) to inform stakeholders.

## Step‑by‑Step Procedure

1. **Identify the target version**
   - Review the release history and locate the version tag or image digest of the last stable release.  Confirm that it has a valid signature and SBOM.
   - Document the digest and the corresponding GitOps commit for reference.

2. **Open a rollback pull request**
   - Navigate to the GitOps repository and open the manifest that specifies the TALON image.
   - Change the image tag or digest to the previously stable value.  Also update any version references in associated Helm charts or Kustomize overlays.
   - Commit your changes on a new branch and open a pull request titled “Rollback TALON to <version/digest>”.  Include a brief description of the issue prompting the rollback and the hash of the signed bundle for traceability.

3. **Review and merge**
   - Request reviews from the on‑call engineer and, if relevant, the security team.  Ensure the attestation for the target digest is available and that the manifest changes are limited to the rollback.
   - Once approved, merge the pull request.  The GitOps agent (Argo CD/Flux) will detect the change and begin reconciling the environment.

4. **Monitor rollback deployment**
   - Use the GitOps dashboard to watch the deployment status.  Confirm that the agent reports a successful sync and healthy state for all TALON components.
   - If the GitOps agent reports errors (e.g. image pull errors, failed health checks), investigate immediately.  Ensure that the old image exists in the registry and that credentials are valid.
   - Use `kubectl get pods` or the platform console to verify that pods are terminating the faulty version and starting the previous one.  Check for pods stuck in crash loops or pending state.

5. **Run smoke tests**
   - Execute the standard smoke test suite against the TALON endpoints to ensure core functions are operational.  Verify that latency, error rates and resource usage are within normal ranges.
   - Confirm that signed attestations continue to verify successfully, indicating the integrity of the rolled‑back version.

6. **Communicate status**
   - Notify stakeholders of the rollback via the chosen communication channel.  Include the reason for the rollback, the version restored and the success of smoke tests.
   - If users were impacted, provide guidance on any actions they may need to take (e.g. retrying jobs, refreshing sessions).

7. **Post‑rollback actions**
   - **Incident documentation** – Open an incident ticket describing the problem, rollback steps, duration, and impact.  Capture logs, screenshots and error messages for root‑cause analysis.
   - **Root‑cause analysis (RCA)** – Schedule an RCA meeting with engineering, operations and product.  Determine the underlying cause of the failed deployment and develop an action plan to prevent recurrence.
   - **Cleanup** – Remove unused manifests or tags related to the failed deployment if they are no longer needed.  Ensure the environment reflects the stable state.
   - **Update runbooks** – Adjust this runbook or others to incorporate lessons learned from the event.

## Notes

- Always ensure that rollback targets have been properly signed and verified.  Never roll back to an unsigned or unaudited image.
- Keep at least one previous version available in the container registry to facilitate quick rollbacks.  Maintain a retention policy that balances storage costs with rollback readiness.
- Rollbacks should be treated with the same level of scrutiny as forward deployments; review, monitoring and testing are essential.
- This runbook complements the **Runbook — Deploy** document and the **Cutover Plan**.  Together they define safe operational procedures for TALON lifecycle management.
