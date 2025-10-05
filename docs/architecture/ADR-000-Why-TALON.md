# ADR‑000: Choose TALON

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑10‑02  

---

## Status

Accepted – The decision has been made to adopt the TALON platform for Blue Eagle Robotics’ factory automation and infrastructure orchestration needs.

## Context

Blue Eagle Robotics operates multiple manufacturing lines and must manage two critical workflows:

1. **Perception reliability** – Factory sensors and PTZ cameras must produce fast, trustworthy readings of gauges and parts.  Operators need automated assurance that readings are accurate, and compliance officers require verifiable evidence that the data has not been tampered with.
2. **Provisioning trust** – Infrastructure changes (e.g. network slices, compute provisioning, configuration updates) must be managed as code.  They require approvals, policy checks and an audit trail.  Without a standardised process, environment drift occurs and operators hesitate to trust automated changes.

Existing tooling in the market addresses perception and provisioning separately.  Blue Eagle Robotics sought a unified solution that could coordinate both workflows, deliver provable integrity and reduce tool fragmentation.  The solution needed to integrate with the company’s CI/CD pipelines, support air‑gapped deployments, and provide a clear audit trail for each action.

## Decision

Blue Eagle Robotics will adopt **TALON**, Reliable AI Network’s Tactical Agentic Layer for Orchestrated eNvironments, as the unified control plane for both perception and provisioning workflows.  TALON coordinates specialised agents for vision, infrastructure, policy enforcement and provenance under a single orchestrator.  It exposes simple HTTP APIs for “perception” and “infra” jobs, generating signed artefacts and provenance for each action.  TALON packages a modular stack with examples, runbooks and Terraform modules tailored for air‑gapped manufacturing environments.

Alternative solutions were evaluated, including bespoke integration of separate perception and infrastructure tools and commercial orchestration platforms.  These alternatives lacked unified provenance, required significant custom glue code and carried higher licensing and maintenance costs.  TALON’s integrated approach, open architecture and alignment with Blue Eagle Robotics’ desire for extensibility and automation made it the preferred choice.

## Consequences

By choosing TALON, Blue Eagle Robotics will:

* **Unify operations** – A single platform manages perception and infrastructure tasks, reducing tool sprawl and simplifying operator training.
* **Improve traceability** – Every job submitted through TALON generates a signed record.  This enhances auditability and supports compliance requirements without bolting on additional logging systems.
* **Enhance security** – TALON integrates image signing, SBOM attestations and policy‑as‑code checks.  It enforces that only validated artefacts and approved plans progress through the pipeline, reducing supply‑chain risk.
* **Accelerate delivery** – Built‑in examples, Terraform modules and runbooks allow rapid iteration.  Blue Eagle Robotics can deliver measurable value in the first weeks while laying a foundation for future expansions.
* **Increase complexity** – Adopting a unified agentic platform requires a learning curve for development and operations teams.  New roles (e.g. Planner, Policy author) must be defined, and existing CI pipelines must be adapted to call TALON’s APIs.
* **Vendor alignment** – RAIN becomes a strategic vendor for Blue Eagle Robotics.  This requires clear service‑level agreements, support channels and ongoing collaboration to evolve TALON as factory requirements change.

## Rationale

TALON aligns with Blue Eagle Robotics’ strategic goals to automate, secure and document factory operations.  The platform’s agentic architecture emphasises separation of concerns: a Planner coordinates tasks, Vision and Infra agents perform specialised work, a Policy agent enforces guardrails and a Provenance agent records immutable history.  This modularity allows Blue Eagle Robotics to extend individual agents (e.g. swap out the Vision agent for a new model server) without rewriting the entire system.

TALON is designed with supply‑chain security in mind.  It integrates Cosign for signing container images and SBOMs, uses policy engines to evaluate infrastructure plans and generates transparent provenance for each action.  These features satisfy Blue Eagle Robotics’ compliance requirements and provide confidence that no unauthorised changes occur in air‑gapped environments.

Finally, TALON’s open specification and declarative configuration encourage reusability and community contributions.  RAIN has committed to provide a long‑term support roadmap and to engage with Blue Eagle Robotics in shaping future enhancements.  Compared with custom integration or proprietary platforms, TALON offers greater flexibility and lower total cost of ownership.

## Implementation Plan

* **Pilot Deployment** – Deploy TALON in a non‑production environment.  Generate SBOMs for existing images, sign them and ingest them into the platform.  Execute a perception job and an infrastructure plan through the API to prove end‑to‑end functionality.
* **CI/CD Integration** – Modify Blue Eagle Robotics’ pipelines to call TALON’s endpoints for signing and attestation.  Implement automatic policy checks and promote artefacts only when verification passes.
* **Training** – Conduct workshops with engineering, operations and compliance teams to explain agent roles, how to submit jobs and how to interpret provenance records.
* **Evaluation** – After an initial 60‑day period, assess performance, usability and security outcomes.  Iterate on any gaps and plan for a broader rollout across additional manufacturing lines.

## Related Decisions

This ADR links to related documents:

* ADR‑001: Adopt air‑gapped delivery model for TALON
* ADR‑002: Integrate Cosign and Syft into CI/CD pipeline
* ADR‑003: Define Policy and Provenance Schema for TALON Jobs
