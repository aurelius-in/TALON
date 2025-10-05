# IP & Open Source Annex

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑09-17

---

## Purpose

This annex documents the intellectual property (IP) ownership and open‑source software (OSS) components used within TALON.  It clarifies the rights retained by RAIN, the rights granted to BER, and outlines the licence obligations associated with third‑party software.  The goal is to promote transparency while ensuring compliance with permissive licences and protecting proprietary work.

## Ownership of TALON

TALON is an original work of Reliable AI Network, Inc.  RAIN owns all IP rights in the source code, designs, diagrams, documentation, and associated tooling developed for TALON.  Under the Statement of Work, BER receives a non‑exclusive, perpetual licence to use TALON internally for its business operations.  No ownership of RAIN’s proprietary code or trade secrets is transferred.  BER may not redistribute TALON or its source code outside its organisation without RAIN’s written consent.

## Open Source Components

TALON is built using several widely adopted OSS packages.  These components are distributed under permissive licences that allow commercial use, modification, and redistribution subject to simple obligations such as preserving copyright notices and licence texts.  Table 1 lists the major OSS dependencies and their licences.

| Component                | Version | Licence | Notes |
|-------------------------|:------:|:-------:|-------|
| **Python**              | 3.11   | PSF     | Core programming language. |
| **FastAPI**             | 0.100 | MIT     | Web framework for APIs. |
| **Uvicorn**             | 0.23  | BSD     | ASGI server used to run FastAPI. |
| **Pydantic**            | 2.x    | MIT     | Data validation and settings management. |
| **Terraform**           | 1.x    | MPL‑2.0 | Infrastructure‑as‑Code engine. |
| **OPA** (Open Policy Agent) | v1.x | Apache‑2.0 | Policy evaluation engine. |
| **Syft/Syft‑based SBOM generators** | latest | Apache‑2.0 | Used to generate SBOMs. |
| **Other Python packages** | various | MIT/BSD/Apache | Transitive dependencies used by the above packages. |

*Table 1 – Major open‑source components and their licences*

## Licence Obligations

### MIT‑licensed software

Many of TALON’s dependencies (e.g., FastAPI and Pydantic) are released under the MIT licence.  The MIT licence is highly permissive: users may use, copy, modify, merge, publish, distribute, sublicense and sell copies of the software.  The only condition is that the original copyright notice and licence text must be included in all copies or substantial portions of the software【233672151105559†L64-L71】.  The licence also disclaims warranties and liabilities【233672151105559†L64-L71】.  TALON satisfies this obligation by embedding the required notices in the source tree and in the Notice file supplied with releases.

### Apache License 2.0

Some third‑party components (e.g., OPA and Syft) are licensed under the Apache License 2.0.  The Apache licence grants copyright and patent rights to reproduce, prepare derivative works, display and distribute the software【492272653103369†L67-L73】.  Redistribution is permitted provided that:

1. A copy of the licence accompanies the distribution【492272653103369†L90-L97】.
2. Modified files include prominent notices stating that changes have been made【492272653103369†L94-L100】.
3. All copyright, patent, trademark and attribution notices from the source form are retained in derivative works【492272653103369†L101-L105】.
4. If the original distribution includes a `NOTICE` file, a readable copy of its attribution notices is included in derivative works【492272653103369†L107-L116】.
5. Trademark rights are not granted; trademarks of the licensor may only be used for descriptive purposes【492272653103369†L139-L142】.

TALON complies with these conditions by distributing each Apache‑licensed component with its licence text and by retaining the relevant notices in the SBOM and Notice file.  RAIN does not remove or alter any required notices and does not use third‑party trademarks without permission.

### MPL 2.0 (Mozilla Public License)

Terraform is licensed under the Mozilla Public License 2.0.  This licence permits use, modification, and distribution (including in proprietary software) provided that any files containing MPL‑licensed code remain subject to the MPL.  If we modify Terraform’s source code, we must make those modifications available under the MPL.  TALON uses Terraform as a dependency without modifying its source code; therefore, no special obligations apply beyond retaining the MPL notice in our distribution.

### BSD and other permissive licences

Uvicorn and several other dependencies are licensed under two‑ or three‑clause BSD licences.  These licences permit redistribution in source and binary forms, with or without modification, provided that copyright notices, licence text, and disclaimer statements are retained in redistributions.  TALON includes these notices in the Notice file.

## Proprietary Content and Third‑Party IP

Any proprietary libraries, models, scripts, diagrams, or templates created by RAIN for TALON remain RAIN’s confidential IP and are not subject to open‑source licences.  These assets may include:

- Custom machine‑learning models or inference logic.
- Proprietary orchestration logic in the Planner, Policy or Provenance agents.
- Configuration templates, runbooks and documentation created for BER.

BER must treat these proprietary materials as confidential and may not share them outside its organisation without RAIN’s consent.  RAIN warrants that it has the right to licence these assets to BER and that the assets do not infringe third‑party rights.

## Software Bill of Materials (SBOM) and Compliance

RAIN produces a complete SBOM for each TALON release.  The SBOM lists all third‑party packages, versions, checksums, and their respective licences.  As part of the CI/CD pipeline, RAIN uses tools (e.g., Syft) to generate the SBOM and signs it using Cosign.  This ensures that any user of TALON can verify that they are receiving the same components that were originally built and that the SBOM has not been tampered with.  RAIN also scans dependencies for known vulnerabilities and licence issues using automated tools.  Any identified risks are tracked and addressed prior to release.

## Attribution and Notice File

TALON’s source distribution includes a `NOTICE` file.  This file aggregates copyright statements and licence texts for all OSS dependencies, along with a brief attribution statement for each.  The Notice file clearly separates RAIN’s proprietary IP from third‑party components and specifies the applicable licence for each component.  When distributing derivative works, recipients must maintain the Notice file in accordance with the obligations described above.

## Trademark Usage

TALON and Reliable AI Network are trademarks of RAIN.  No rights to use these trademarks are granted except for identifying the software.  Third‑party component trademarks (e.g., Python™, FastAPI™) remain the property of their respective owners.  BER and any downstream recipients may reference these trademarks solely to describe compatibility or provenance but may not imply endorsement or affiliation.

## Reporting and Contact

Questions related to IP or OSS compliance for TALON should be directed to RAIN’s legal or compliance contact at **legal@reliableAInetwork.com**.  RAIN will respond promptly to any notices of non‑compliance or suspected infringement.

---
