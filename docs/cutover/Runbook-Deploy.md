# Runbook — Deploy

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑09-28 

---

## Purpose

This runbook provides step‑by‑step instructions for building, signing, and deploying the **TALON** platform into a target environment using a GitOps workflow.  It is intended for DevOps engineers and release managers responsible for CI/CD operations.  The document is non‑confidential and may be shared publicly as an example of effective deployment procedures.

## Overview

Deploying TALON involves three main stages:

1. **Build & Sign** – compile code, run tests, produce container images, generate and sign a software bill of materials (SBOM) and the image itself.
2. **GitOps Push** – update the GitOps repository with the new image version and create a pull request (PR) for review and merge.
3. **Smoke Tests** – verify basic functionality after the GitOps agent applies the changes to the target environment.

Each stage has associated prerequisites and verification steps to ensure a reliable release.

## Pre‑requisites

- Access to the source code repository and CI pipeline.  Ensure build secrets (e.g. registry credentials, signing keys) are available in the CI environment.
- Access to the GitOps repository (e.g. Argo CD or Flux) and permissions to open pull requests.
- Cosign and syft installed in the build environment for generating SBOMs and signing images.
- A set of smoke tests (curl scripts, API test suites or end‑to‑end checks) defined and stored in the repository.
- Notification channels (e.g. Slack/Teams) set up for deployment status updates.

## Step 1: Build & Sign

1. **Fetch code** – Check out the desired branch or tag in a clean workspace.  Use shallow clones to minimise network usage.
2. **Run unit and integration tests** – Execute test suites (`pytest`, `go test`, etc.) to validate functionality.  Fail the pipeline if any test fails.
3. **Build container image** – Use a reproducible build (e.g. multi‑stage Dockerfile) to produce an OCI image.  Tag the image with the release version and commit SHA, e.g. `ghcr.io/blueeagle/talon:<version>-<sha>`.
4. **Generate SBOM** – Run `syft` against the image to produce an SBOM (e.g. `sbom.json`).  Validate that it contains all dependencies and license data.
5. **Sign the image and SBOM** – Use **Cosign** to sign the container image and attach the SBOM as an attestation.  For example:

   ```bash
   cosign sign --yes ghcr.io/blueeagle/talon:<tag> \
     --bundle talon_bundle.json \
     --attestation sbom.json
   ```

   The `talon_bundle.json` will contain the signature, certificate and Rekor log entry.  Store both the signed image and bundle in the container registry.
6. **Publish artifacts** – Push the signed image, SBOM and attestation bundle to the registry.  Push any compiled binaries or other artifacts to the appropriate storage (e.g. S3, Nexus).
7. **Tag the release** – Create a signed tag in source control (e.g. `v1.2.0`) referencing the commit used for the build.  This ensures traceability between code, image and SBOM.

## Step 2: GitOps Push

1. **Update manifest** – Locate the Kubernetes/Helm manifest or Kustomize file in the GitOps repository that references the TALON image.  Update the image tag to the new version.
2. **Create a pull request (PR)** – Open a PR with a descriptive title (e.g. “Release TALON v1.2.0”) and include the image digest, SBOM hash and relevant change log.  Request reviews from peer engineers or the security team.
3. **Run static checks** – Automated checks (linting, policy validation) should run on the PR.  For example, verify that the manifest meets security policies (e.g. non‑root user, read‑only filesystem) and that the SBOM attestation is attached.
4. **Approve & merge** – After passing checks, reviewers approve the PR.  Merging triggers the GitOps agent (Argo CD/Flux) to reconcile the new manifest with the target environment.
5. **Monitor sync** – Watch the GitOps dashboard to ensure the new version is deployed.  Confirm that the agent reports “Synced” or “Healthy” status for all relevant resources.  Investigate any errors immediately.

## Step 3: Smoke Tests

After the deployment completes, execute smoke tests to confirm TALON is operating as expected.

1. **Check service endpoints** – Use `curl` or a script to call the key API endpoints, such as `/job/perception` and `/job/infra`, ensuring they return successful responses within acceptable latency.
2. **Verify background processes** – Confirm that scheduler pods (e.g. the policy engine, planner, and worker agents) start and register as ready.  Use `kubectl get pods` or the GitOps UI to view pod health.
3. **Review logs** – Tail the logs of newly deployed pods.  Look for error messages, stack traces or authentication failures.  Confirm that the SBOM and signature verification log entries are present.
4. **Check monitoring dashboards** – Ensure metrics (CPU/memory usage, request rate, error rate, response times) remain within normal thresholds.  Alerts should remain clear.
5. **Announce success** – Post a deployment summary in the notification channel, including the release tag, image digest and link to the GitOps commit.  Note any anomalies or follow‑up actions required.

## Rollback

If any smoke test fails or if serious issues are detected, roll back quickly:

1. **Revert manifest** – In the GitOps repository, change the image tag back to the previously stable version and open a new PR.  Merge it as soon as possible to trigger a rollback deployment.
2. **Monitor rollback deployment** – Ensure the environment returns to a healthy state.  Confirm the old version is running and tests pass.
3. **Investigate** – Collect logs and failure evidence from the failed version.  Open tickets to track root causes and corrective actions before attempting another deployment.

## Post‑Deployment Tasks

1. **Update release notes** – Document the features, bug fixes and known issues addressed by the release.  Include hashes of artifacts and references to the SBOM and signature bundle.
2. **Audit attestation** – Verify the Cosign entries in the transparency log.  Store attestation metadata in the compliance management system.
3. **Retrospective** – Schedule a brief post‑mortem or retrospective to capture lessons learned from the deployment process.  Identify improvements for build speed, verification coverage or automation.

## Notes

- Always maintain the principle of immutability: once an image is signed and pushed, it should never be modified.  New versions require a new tag and signature.
- Do not expose signing keys in logs or to untrusted environments.  Use short‑lived signing certificates where possible.
- This runbook complements the separate runbooks for rollback and triage.  It assumes the environment is prepared using the **Cutover Plan** and that infrastructure has been codified following the IaC Module Specifications.
