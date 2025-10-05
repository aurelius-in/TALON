# Image Signing & SBOM SOP

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑10‑05  

---

## Purpose and Scope

This standard operating procedure (SOP) describes how RAIN creates, signs and distributes software bills of materials (SBOMs) and container images for the TALON platform delivered to BER.  A signed SBOM and image provide verifiable evidence of what is inside the software, who produced it and whether it has been tampered with.  The process outlined here ensures that every image built by RAIN is accompanied by a trustworthy SBOM and that downstream consumers can verify both the software contents and provenance.

## Roles and Responsibilities

* **Build/Release Engineer** — Executes the build pipeline, generates SBOMs, signs images and SBOMs, and publishes them to the container registry.
* **Security/Compliance Officer** — Defines signing policies, manages keys or coordinates keyless signing, verifies signatures and audits compliance.  Maintains integration with transparency logs and monitors for unsigned artifacts.
* **Developer** — Ensures the code and dependencies are accurately represented in the SBOM and remediates vulnerabilities that surface during SBOM review.

## Definitions

* **Software Bill of Materials (SBOM)** — A structured list of the libraries, modules, licences and version information that compose a software artifact.  An SBOM helps identify vulnerabilities, licence compliance obligations and supply‑chain risks.
* **Cosign** — A command‑line tool for signing and verifying software artifacts.  Cosign supports “keyless” signing using OpenID Connect (OIDC) identities and short‑lived certificates, storing signatures and attestations in an append‑only transparency log for auditability.  It can sign container images and other artifacts including SBOMs.
* **Attestation** — A signed statement about an artifact—such as an SBOM or vulnerability scan—that can be associated with the artifact and verified independently.  Storing the SBOM as an attestation ties it to the image and proves its provenance.
* **Rekor Transparency Log** — A public, tamper‑evident log used by Cosign to record signatures and attestations.  Signing events recorded in the log provide an audit trail and enable independent verification.

## Tools

* **Syft** — Generates SBOMs in SPDX or CycloneDX format from container images.
* **Cosign** — Signs container images, creates attestations and verifies signatures.  Supports both keyless (OIDC) and key‑based signing.  Integrates with Rekor for transparency.
* **Container Registry** — The registry (e.g. Harbor, Artifactory, ECR) where images, signatures and attestations are stored and from which consumers pull images.
* **Rekor** — The transparency log service where signing events are recorded for auditing and verification.

## Process Overview

### 1. SBOM Generation

1. Pull or build the container image to be released.
2. Use Syft to generate an SBOM in the desired format.  For example, generate CycloneDX JSON:

   ```bash
   syft <image‑reference> --output cyclonedx‑json --file sbom.json
   ```

3. Inspect the SBOM to ensure it lists all expected components, versions and licences.  Address any omissions or unexpected dependencies.

### 2. Image Signing

1. Authenticate with the appropriate identity provider or load the signing key.  Keyless signing is preferred because it avoids managing long‑lived keys.
2. Sign the container image using Cosign.  For OIDC‑based signing:

   ```bash
   cosign sign --yes <image‑reference>
   ```

   For key‑based signing using a private key stored in an HSM or KMS:

   ```bash
   cosign sign --key <private‑key‑file> <image‑reference>
   ```

3. Cosign creates a signature object, pushes it to the registry alongside the image and records the signing event in the transparency log.

### 3. SBOM Attestation

1. Associate the generated SBOM with the image as an attestation.  Use Cosign to create and sign the attestation:

   ```bash
   cosign attest --yes \
     --predicate sbom.json \
     --type cyclone‑dx \
     <image‑reference>
   ```

   The `--type` flag denotes the SBOM format.  Cosign signs the attestation, stores it in the registry and records the event in Rekor.

2. Verify that the attestation is discoverable in the registry and that a corresponding entry exists in the transparency log.

### 4. Signature Verification

1. To verify an image signature, run:

   ```bash
   cosign verify <image‑reference>
   ```

2. To verify the SBOM attestation, supply the expected type and (if applicable) certificate identity and issuer:

   ```bash
   cosign verify‑attestation \
     --type cyclone‑dx \
     --certificate‑identity <OIDC‑subject> \
     --certificate‑oidc‑issuer <OIDC‑issuer‑url> \
     <image‑reference>
   ```

3. Verification steps should be integrated into deployment gates so that unsigned images or invalid attestations block promotion to downstream environments.

### 5. Publish

1. Push the signed image, signature and SBOM attestation to the designated container registry.
2. Make the SBOM file available to internal stakeholders and, where required, attach it to compliance filings.  Export the SBOM into vulnerability management or licence‑compliance tools for scanning.

### 6. Documentation and Record Keeping

1. Archive the generated SBOM files, signature objects, attestation files and verification logs for each release.  Store these artefacts in a secure location with restricted access.
2. Maintain release notes documenting the SBOM hash, signature digest and attestation identity for traceability.

## Security and Compliance Considerations

* **Key Management** — If key‑based signing is used, private keys must be stored in a hardware security module (HSM) or cloud key‑management service (KMS).  Limit key access to authorised personnel and rotate keys regularly.  Consider short‑lived signing certificates and automated key revocation to reduce the attack surface.
* **Keyless Signing** — When possible, use Cosign’s identity‑based signing.  Cosign obtains a short‑lived certificate from a trusted certificate authority after authenticating via OIDC.  This eliminates the need for managing long‑term private keys.
* **Transparency and Provenance** — Recording signatures and attestations in the Rekor transparency log creates an immutable, tamper‑evident audit trail.  Consumers can verify who signed the artifact and when, independent of the registry.
* **Tamper Resistance** — Without signatures, consumers must trust only the transport layer security (e.g. TLS) and registry access controls.  Signing the image and SBOM ensures they have not been modified since release and ties the SBOM to the exact image digest.
* **CI/CD Integration** — Integrate SBOM generation and signing into the build pipeline.  Automate policy checks so that any step fails if the image or attestation is missing or invalid.  Signing should happen automatically in each build, and verification should gate promotion to higher environments.
* **SBOM Use Cases** — A signed SBOM provides the basis for vulnerability scanning, licence compliance and dependency analysis.  Use the SBOM to trigger alerting on newly disclosed vulnerabilities and to verify open‑source dependencies before deployment.

## Sample Pipeline Snippets

```bash
## Build the image
docker build -t registry.internal/ber/talon:1.0.0 .

## Generate SBOM in CycloneDX format
syft registry.internal/ber/talon:1.0.0 \
     --output cyclonedx‑json \
     --file sbom.json

## Sign the image using OIDC
cosign sign --yes registry.internal/ber/talon:1.0.0

## Attach the SBOM as an attestation
cosign attest --yes \
  --predicate sbom.json \
  --type cyclone‑dx \
  registry.internal/ber/talon:1.0.0

## Verify the image signature
cosign verify registry.internal/ber/talon:1.0.0

## Verify the SBOM attestation
cosign verify‑attestation \
  --type cyclone‑dx \
  --certificate‑identity <OIDC‑subject> \
  --certificate‑oidc‑issuer <OIDC‑issuer‑url> \
  registry.internal/ber/talon:1.0.0
```

## Distribution and Air‑Gapped Considerations

* If BER operates in an air‑gapped environment, mirror the container registry so that it includes the image, signature and attestation.  Export the transparency log entries associated with the signatures and store them alongside the artefacts.
* Provide the SBOM files and verification metadata as part of the delivery package.  Instruct operators to use Cosign in offline verification mode, supplying the public certificate or key and the signature and attestation files to validate artefacts without network access.

## Maintenance and Future Improvements

* Update SBOM generation and signing tools (Syft, Cosign) as new versions are released to benefit from security improvements and new features.
* Extend the attestation process to include additional provenance data, such as vulnerability scan results, test coverage reports or build metadata.
* Integrate SBOM ingestion into risk‑management platforms and track remediation of vulnerabilities uncovered by SBOM analysis.

## References

This SOP draws on industry guidance around SBOMs and signing.  An SBOM is used to catalogue components, licences and version information, making it possible to identify vulnerabilities and compliance issues.  Signing an SBOM ensures it cannot be tampered with and proves who created it.  Cosign provides the tooling to sign container images and other artefacts, store signatures in a transparency log and attach SBOMs as attestations.
