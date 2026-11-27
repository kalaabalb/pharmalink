# 03 — Principles and decision log

**Status:** Draft — principles proposed; technology choices intentionally deferred

## 1. Project principles

**P-01 — One source of truth.** Every business fact has a defined owner. Inventory, sales, payment, catalogue and verification facts must not have competing authoritative copies.

**P-02 — Medicine identity is not stock.** A catalogue product describes what a product is. A batch/lot represents a physical supply with its own batch number, expiry, supplier, cost and quality state.

**P-03 — Stock is traceable.** Every stock change has a reason, actor, timestamp and source document where applicable. No unexplained quantity edits.

**P-04 — Financial history is immutable.** Correct completed sales, refunds and payments through traceable reversals or adjustments, not silent edits or deletion.

**P-05 — Tenant and branch boundaries are server-enforced.** Client-side filtering is not a security boundary.

**P-06 — Least privilege.** Role templates are presets; explicit permissions and scope govern authorization.

**P-07 — Event-driven live updates.** Clients subscribe to relevant changes. Avoid routine polling for unchanged state. Scheduled work remains appropriate for reminders, reconciliation, cleanup and periodic checks.

**P-08 — Reliable event publication.** Business changes and event-outbox records should commit consistently. Consumers must tolerate duplicate delivery and retries.

**P-09 — Explicit offline behavior.** Define allowed actions, local durability, idempotency, conflict rules, sync status and recovery. Offline clients cannot claim globally current stock.

**P-10 — No unnecessary legacy compatibility.** Replace obsolete contracts and implementations after required data migration and verification. Do not retain broken behavior through permanent fallbacks, duplicate APIs or parallel V1/V2 systems. This does not authorize data loss or removal of legally required records.

**P-11 — Permission-aware AI.** AI uses authorized data, identifies sources and reporting periods, distinguishes fact from inference, and does not invent figures or make clinical decisions. Actions require permission, confirmation and audit.

**P-12 — Availability is time-bounded.** Public availability must include freshness and degrade to stale/unknown when confirmation is no longer reliable.

**P-13 — Regulatory and clinical boundaries.** The platform is not a pharmacy, prescriber or regulator. Regulated capabilities require jurisdiction-specific review.

**P-14 — Localization is foundational.** Language, script, calendar, currency, low bandwidth and device constraints belong in domain and UX design.

**P-15 — Modular monolith is the initial architecture hypothesis.** Keep clear domain boundaries in a coherent initial backend. Extract services only for demonstrated reasons.

## 2. Initial decisions

### D-001 — Pharmalink working name
- Status: Provisional
- Decision: Use Pharmalink in project documents.
- Consequence: Not trademark, domain or legal-name clearance.
- Revisit after brand and name checks.

### D-002 — Three connected product surfaces
- Status: Proposed
- Decision: Specify Pharmacy Operations, Public Discovery and Platform Operations separately over shared domain services and security controls.
- Rationale: Different actors and permissions; shared pharmacy, branch, medicine and availability facts.

### D-003 — Modular monolith first
- Status: Proposed
- Decision: Begin with modular domain boundaries rather than premature microservices.
- Revisit if workload, team topology or isolation requirements justify extraction.

### D-004 — Event subscriptions over routine polling
- Status: Proposed
- Decision: Use server-pushed events where useful; choose WebSockets, SSE, push or another mechanism per interaction.
- Qualification: Events do not replace initial loads, reconnect reconciliation, scheduled jobs or integrity checks.

### D-005 — Clean replacement
- Status: Proposed
- Decision: Replace obsolete features and contracts after migration and verification.
- Qualification: Preserve required records; retain safe recovery and audit history.

### D-006 — Discovery is not a reservation
- Status: Proposed
- Decision: Initial public discovery communicates freshness-qualified availability, not guaranteed reservation or sale.
- Revisit only after regulatory and operational assessment.

## 3. Technology decisions intentionally deferred

Backend/framework, frontend and app packaging, database, event broker, WebSocket/SSE topology, offline database/encryption, search engine, object storage, hosting/data residency, SMS/email/push, payment providers, maps/geocoding, AI provider/data boundaries, monitoring and tracing.

Select these from requirements and evidence, not habit or preference.

## 4. Decision quality

A decision must record context, options, rationale, consequences, evidence and revisit conditions. If evidence is missing, label the item a hypothesis or open question.
