# Low‑Level Design (LLD)

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑10‑05  

---

## 1. Context and Scope

TALON is a control plane with agentic components—Planner, Vision, Infra, Policy, Provenance and Operator—that orchestrate perception and provisioning jobs.  
This low‑level design builds upon the high‑level architecture to define technical details for APIs, messaging patterns, storage schema, networking topology and observability.  
The system runs in containerised microservices on Kubernetes at the factory edge and in the central control plane.  
Assumptions include the use of open‑source technologies, compliance with regulatory requirements and the need to operate in air‑gapped environments.

## 2. API Design

### 2.1 REST Principles

TALON exposes a small set of RESTful endpoints.  
Resources are named with nouns (e.g., `job`, `plan`); URIs use plural forms for collections and avoid verbs.  
Standard HTTP methods convey intent (`POST` to create jobs, `GET` to fetch results).  
Requests are stateless and contain all information needed to process them.  
Responses use JSON for content negotiation and include appropriate status codes.

### 2.2 Endpoints

- **POST `/job/perception`** — Submit a perception job.  
  *Request body:* `{ "image_id": "frame‑001", "roi": [x1, y1, x2, y2] }` where `image_id` references an image in object storage and `roi` defines a rectangular region of interest.  
  *Response:* `{ "job_id": "…", "value": "72.4 psi", "confidence": 0.96, "latency_ms": 680, "model_version": "gaugelens‑1.2.3", "policy": {…}, "time": <timestamp> }`.  
  Errors return 400 for validation issues or 500 for internal failures, with an `error_code` and human‑readable message.

- **POST `/job/infra`** — Request an infrastructure change plan.  
  *Request body:* `{ "change_request": { "add": ["rg", "vnet"], "modify": [], "destroy": [] } }`.  
  *Response:* `{ "job_id": "…", "plan_id": "plan‑1234", "summary": { … }, "artifact_ref": "artifacts/plan‑1234.bin", "policy": {…}, "time": <timestamp> }`.  
  The plan is generated with Terraform and stored in object storage; the `policy` field indicates whether it passed or requires review.

- **GET `/job/{job_id}`** — Retrieve job status and results.  
  Returns the original request, agent outputs, policy decision and provenance metadata for the given `job_id`.

### 2.3 API Conventions

Authentication uses OAuth 2.0/OIDC bearer tokens for client calls; service‑to‑service calls employ mutual TLS with short‑lived certificates.  
Endpoints are versioned via a prefix (e.g., `/v1/job/perception`) to maintain backward compatibility.  
Idempotent operations accept a client‑generated idempotency key; the same key returns the same `job_id` on retries.  
Rate limits protect shared resources.

## 3. Messaging & Integration

### 3.1 Asynchronous Communication

TALON uses an event‑driven architecture to decouple producers and consumers.  
Agents emit events when a job completes or when a policy violation occurs.  
Subscribers such as dashboards, notification services or downstream systems consume these events.  
Decoupling enhances scalability and allows near‑real‑time delivery without coupling producers to specific consumers.

### 3.2 Event Schema

Events adopt the CloudEvents 1.0 specification.  
Each event contains an `id`, `source`, `type`, `subject`, `time` and `data` field.  
For perception events, `data` includes `job_id`, `image_id`, `value`, `confidence`, `latency_ms` and `policy_status`.  
For infra events, `data` includes `job_id`, `plan_id`, `summary`, `policy_status` and any `violations`.  
Producers must provide complete context; consumers must be idempotent and handle duplicates.

### 3.3 Delivery Topology

A broker (for example, NATS or Azure Service Bus) routes messages from producers to consumers.  
Use publish‑subscribe topics for broadcast notifications and point‑to‑point queues for work that should be processed by a single consumer.  
Dead‑letter queues capture messages that cannot be processed and require manual intervention.

### 3.4 Idempotency & Retries

Consumers deduplicate based on `job_id` or the event `id`.  
Retry policies employ exponential backoff and limited attempts to prevent message storms.

## 4. Data Storage

### 4.1 Metadata Store

A relational database (PostgreSQL) stores job metadata and results.  
Tables include `jobs` (`job_id`, `type`, `request_payload`, `status`, `result_payload`, `created_at`, `updated_at`), `plans` (`plan_id`, `job_id`, `summary`, `artifact_ref`) and `inferences` (`job_id`, `value`, `confidence`, `latency_ms`, `model_version`).  
JSONB columns store flexible payloads; indexes on `job_id` and `created_at` optimise queries.

### 4.2 Object Storage

Large binary artefacts (camera frames, plan files, signed provenance records) are stored in object storage (S3, Azure Blob).  
The database stores only metadata such as file name, storage URI, size, checksum and timestamps.  
This pattern keeps the database lean and reduces backup complexity.  
Object storage uses server‑side encryption and lifecycle policies for automatic expiration.

### 4.3 Cache

An in‑memory cache (Redis) stores frequently accessed job statuses to reduce database load.  
TTL values ensure stale data is evicted automatically.

## 5. Networking

TALON components run in a Kubernetes cluster; internal communication uses a service mesh (Istio) that provides mutual TLS, traffic routing and observability.  
An API gateway or ingress controller exposes the REST API; network policies restrict ingress/egress and enforce least‑privilege segmentation.  
Communication uses HTTP/2 or gRPC over TLS 1.2+ with modern cipher suites.  
Outbound traffic to object storage and identity providers is permitted via controlled VPC endpoints.  
Namespaces separate the control plane, data plane and observability tools to limit blast radius.

## 6. Observability

### 6.1 Logging

Logs are emitted in structured JSON format, making them easy to parse and correlate.  
Combine related events into canonical log entries instead of emitting multiple tiny logs to reduce noise.  
Each log includes unique identifiers such as `job_id`, request ID or trace ID to enable correlation across services.  
Field names and types are standardised across services (timestamp, level, message, context fields).  
Sensitive data like passwords or personal identifiers must never be logged; mask or omit such fields.  
Logs are treated as data: they can be filtered by fields and aggregated into metrics such as count of jobs processed or error rates.  
All logs are forwarded via the OpenTelemetry Collector to a central store (e.g., Elasticsearch, Loki).  
Retention policies define how long logs are kept; compliance rules may require deletion after a fixed period.  
Alerting rules trigger notifications on critical log events (e.g., repeated failures or policy violations).  
Documentation of log formats and logging practices is maintained to support onboarding and consistency.

### 6.2 Metrics

OpenTelemetry metrics instrumentation captures system and application metrics.  
Key metrics include request throughput, latency percentiles, error rates, CPU and memory usage, and queue depths.  
Custom metrics track domain events such as number of perception jobs processed per minute, plan generation time and policy pass/fail counts.  
Metrics are scraped by Prometheus and visualised in Grafana.  
Service‑level objectives (SLOs) define acceptable thresholds (e.g., p50 latency ≤ 2 s, p95 latency ≤ 5 s).  
Alerts fire when metrics exceed thresholds.

### 6.3 Tracing

Distributed tracing is implemented across all microservices using OpenTelemetry.  
Each incoming request generates a trace that propagates through the Planner, Vision/Infra, Policy and Provenance agents.  
Trace context is passed via standard headers (e.g., `traceparent`).  
Traces are exported to a backend such as Jaeger or Tempo.  
Combining traces with logs and metrics provides end‑to‑end observability and simplifies root‑cause analysis.

### 6.4 Provenance and Audit

The Provenance agent records an immutable audit record for every job.  
Records include the job type, request parameters, outputs from each agent, policy decision, timestamps and references to artefacts.  
Records are stored in append‑only object storage to ensure tamper‑evident evidence for audits.

## 7. Security Considerations

All APIs enforce authentication and authorisation.  
Service‑to‑service communication uses mutual TLS and short‑lived certificates.  
Images are scanned for vulnerabilities; signed images and SBOMs provide supply chain integrity.  
Secrets (keys, tokens, passwords) are stored in a secret manager and injected at runtime; they are not stored in source control or environment variables.  
Network policies limit which services can communicate; a Web Application Firewall (WAF) protects the API gateway.  
Input is validated and sanitised to prevent injection attacks.  
Regular penetration tests and vulnerability scans are performed.

## 8. Success Measures & Quality Attributes

- **Reliability:** 99.9 % availability; mean time between failures (MTBF) greater than six months; mean time to recover (MTTR) less than 10 minutes.  
- **Performance:** Perception job p50 latency ≤ 2 seconds and p95 latency ≤ 5 seconds; infra plan generation ≤ 5 seconds.  
- **Scalability:** Horizontal scaling supports increased job load; the message broker and storage systems scale by adding partitions or replicas.  
- **Security:** Zero critical vulnerabilities in deployed containers; 100 % of images signed; compliance with applicable security standards.  
- **Cost Efficiency:** Monitor infrastructure costs and apply auto‑scaling, rightsizing and spot instances where possible.  
- **Developer Productivity:** Onboard new developers in less than three days; deliver new features within one sprint.

## 9. Deployment and Evolution

Infrastructure‑as‑Code (Terraform) modules provision all resources.  
Modules are versioned and tagged; each module resides in a dedicated repository and documents its required and optional inputs and outputs.  
CI/CD pipelines include stages for building and testing code, scanning for vulnerabilities, signing images, generating SBOMs, planning Terraform changes, running policy checks, requiring manual approval and applying changes.  
Deployments use GitOps (e.g., Argo CD); merging to the main branch triggers automated rollouts.  
Blue‑green or canary deployments minimise risk; rollback procedures are documented.  
The LLD should be updated whenever new endpoints or agents are introduced or architectural changes occur.
