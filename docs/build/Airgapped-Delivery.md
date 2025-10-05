# Air‑gapped Delivery Guide

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑10‑03

---

## Purpose

This guide describes how to deliver, verify, and install the TALON product in an air‑gapped environment at Blue Eagle Robotics. It ensures that both the offline bundle and the resulting deployment are secure, auditable, and fully functional.

## Overview

TALON is delivered as an offline bundle containing signed container images, configuration files, infrastructure‑as‑code (IaC) modules, and runbooks. Because the target environment is air‑gapped, all dependencies and artifacts must be packaged in advance and shipped to the site. This document explains how to generate the bundle, transfer it securely, verify its integrity and authenticity, and perform the installation.

## Prerequisites

Before beginning, ensure the following:

- A secure workstation with access to the TALON release artifacts.
- GPG (or equivalent) installed for signature verification.
- A SHA256 checksum utility.
- Physical access to the target air‑gapped environment with appropriate permissions.

## 1. Bundle Creation

1. **Gather artifacts**
   - Build and sign all TALON container images.  
   - Export the IaC modules, policy packs, and configuration files.  
   - Include all runbooks (`deploy.md`, `triage.md`, `rollback.md`) and documentation (SOW, HLD, LLD, 30‑60‑90 plan, etc.).

2. **Generate checksums**
   - For each file and container image tarball, compute a SHA256 hash and record the results in a file named `SHA256SUMS`.  
   - Use a standardized format (e.g., `sha256sum <file> >> SHA256SUMS`).

3. **Sign the bundle**
   - Create a detached GPG signature of the `SHA256SUMS` file:  
     `gpg --armor --detach-sign --output SHA256SUMS.sig SHA256SUMS`  
   - Optionally sign individual container images or manifest files using a tool such as Cosign.

4. **Package the bundle**
   - Compress all artifacts, the `SHA256SUMS` file, and its signature into a single archive (e.g., `talon‑ber‑v0.1‑offline.tar.gz`).  
   - Include a top‑level `README_OFFLINE.md` that lists the contents and any special instructions for the site engineer.

## 2. Transfer and Chain of Custody

1. **Record chain of custody**
   - Document who created the bundle, when it was created, and who has handled it during transport. Use a secure, tamper‑evident medium for transport (e.g., a sealed USB drive with a unique serial number).  
   - Maintain a signed log or spreadsheet recording each hand‑off.

2. **Verify during transit**
   - Upon receipt, compare the physical media’s ID against the shipping record to ensure it matches.  
   - Store the media in a secure location until installation.

## 3. Verification

Prior to installation, perform the following verification steps in the staging area:

1. **Integrity check**
   - Unpack the archive into a staging directory.  
   - Run `sha256sum -c SHA256SUMS` to verify that all files match their recorded checksums.  
   - Investigate any mismatch immediately; do not proceed until resolved.

2. **Signature verification**
   - Verify the detached signature of the checksum file using GPG:  
     `gpg --verify SHA256SUMS.sig SHA256SUMS`  
   - Ensure the signature is valid and matches the trusted public key from Reliable AI Network.

3. **Image verification**
   - If container images are individually signed, verify each signature using your chosen signing solution (e.g., Cosign):  
     `cosign verify --key <public-key> <image.tar>`  
   - Confirm that the images correspond to the expected versions and have not been altered.

## 4. Installation

1. **Prepare the environment**
   - Confirm that the air‑gapped environment meets the minimum resource and network prerequisites (sufficient GPU nodes, storage capacity, correct OS versions).  
   - Ensure that a container runtime and orchestrator (e.g., Docker, containerd, Kubernetes/OpenShift) are installed and configured.

2. **Load container images**
   - Use `ctr images import` or `docker load` to import each container image tarball into the local registry.  
   - Tag images appropriately for the internal registry or runtime.

3. **Apply IaC**
   - Navigate to the `terraform/` directory included in the bundle.  
   - Copy `terraform.tfvars.example` to `terraform.tfvars` and set the appropriate variables (prefix, location, etc.).  
   - Initialize and plan the configuration in dry‑run mode:  
     ```sh
     terraform init
     terraform plan
     ```
   - Review the output and ensure it aligns with the intended infrastructure. If approved, apply the configuration:  
     `terraform apply`

4. **Deploy TALON services**
   - Follow the `deploy.md` runbook to deploy TALON using GitOps or manual manifests.  
   - Confirm that the FastAPI service is running and accessible on the expected port within the air‑gapped network.  
   - Perform smoke tests using the provided `curl` commands to submit perception and infrastructure jobs.

## 5. Post‑Installation Verification

1. **Functionality tests**
   - Submit a perception job with a valid frame ID and verify that a response is returned within the latency thresholds (e.g., ≤2 seconds for 50th percentile).  
   - Submit an infrastructure job and verify that the plan is generated and policy evaluation returns the expected outcome.

2. **Provenance check**
   - Confirm that provenance records are being written to the designated directory (by default `/mnt/data/talon‑prov`).  
   - Inspect a sample record to ensure it includes job ID, timestamps, input parameters, output values, and policy decisions.

3. **Policy enforcement**
   - Intentionally submit jobs that should fail the policy (e.g., a perception job with a forced low confidence or an infra plan using a forbidden prefix) to verify that the system marks them for review or fails the job.  
   - Confirm that the Operator agent and alerting mechanisms notify the appropriate users.

## 6. Maintenance and Updates

Updates will be provided periodically as new offline bundles. For each update:

1. Repeat the bundle creation, transfer, and verification steps outlined above.  
2. Follow the runbooks for applying updates to the environment.  
3. Retain all provenance and audit records for compliance purposes.

For support or questions, contact Reliable AI Network support at **support@reliableAInetwork.com**. Include the bundle version, environment details, and relevant logs or provenance records.

## 7. Security Considerations

- Always handle offline media securely and track its custody from creation through installation.  
- Do not connect the air‑gapped environment to any external network during the installation process.  
- Rotate signing keys periodically and verify signatures using trusted key management practices.  
- Ensure that all personnel involved in the installation have been trained in secure handling and verification procedures.  
- Maintain a record of all changes applied through TALON and any overrides performed during emergency situations.

---

© 2025 Reliable AI Network, Inc. All rights reserved.
