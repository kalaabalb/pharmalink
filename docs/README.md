# Product specification workspace

This directory is the authoritative working specification for Pharmalink. Documents are drafts unless explicitly marked approved. A document being committed does not mean its requirements are validated.

## Document status

- **Draft:** written, not yet fully challenged or validated.
- **Under review:** actively being checked against user needs, workflows, evidence, dependencies, safety, and operational constraints.
- **Rework required:** material gaps or contradictions remain.
- **Approved:** reviewed against the readiness gate and accepted for the stated scope.
- **Superseded:** replaced by a newer decision or document; retained for traceability.

## Current document set

| File | Purpose | Current status |
|---|---|---|
| 01-project-charter.md | Vision, outcomes, boundaries, success measures | Draft |
| 02-product-scope.md | Capability map and exclusions | Draft |
| 03-principles-and-decisions.md | Non-negotiable principles and provisional decisions | Draft |
| 04-risks-and-open-questions.md | Risk register and unresolved questions | Draft |
| 05-actors-personas-and-role-model.md | Actors, roles, permission scopes, and sensitive actions | Draft |
| 06-end-to-end-workflows.md | Lifecycle workflows and failure paths | Draft |
| 07-domain-model-and-data-ownership.md | Domain boundaries, entities, ownership, and data lifecycle | Draft |
| 08-event-driven-and-offline-requirements.md | Push/event requirements, offline behavior, synchronization, freshness | Draft |
| 09-ethiopia-regulatory-evidence-register.md | Primary-source legal/regulatory evidence and unresolved interpretation | Draft |

## Review dimensions

Every module must be reviewed for:

1. Actors, responsibilities, roles, and authorization boundaries.
2. Scope, preconditions, state transitions, and invariants.
3. Normal, alternate, failure, retry, and recovery flows.
4. Data ownership, provenance, lifecycle, retention, and export.
5. Transaction boundaries, consistency, concurrency, and idempotency.
6. Events, subscriptions, delivery, replay, and missed-event recovery.
7. Offline behavior, local security, conflicts, and synchronization.
8. Security, privacy, regulatory boundaries, and abuse cases.
9. Notifications, auditability, reporting, and observability.
10. Accessibility, localization, and low-connectivity usability.
11. Performance, resilience, cost, and service objectives.
12. Acceptance criteria, negative tests, and operational readiness.
13. Dependencies, migration, backup, recovery, and support.

## Required artifacts before implementation

- Product charter, validated personas, role matrix, and operating assumptions.
- End-to-end workflow catalogue and functional requirements.
- Domain glossary, canonical medicine-data model, and source/provenance policy.
- Data ownership, privacy classification, audit, retention, export, and deletion policy.
- Stock, receiving, sales, returns, purchasing, and finance specifications.
- Public discovery and availability freshness specification.
- Offline capability matrix, conflict policy, and recovery design.
- Event catalogue, schema/version policy, delivery guarantees, and observability.
- Threat model, tenant isolation model, and security test plan.
- Ethiopian regulatory evidence register and qualified local review.
- UX specifications, accessibility, localization, and error-state behavior.
- Integration/provider evaluation and failure handling.
- Non-functional requirements and measurable service objectives.
- Test strategy, migration plan, backup/restore verification, release and incident plans.

## Evidence discipline

Label claims as one of:
- **User intent:** explicitly requested by the product owner.
- **Proposed requirement:** recommended design that still needs validation.
- **Verified external fact:** supported by a cited primary source.
- **Assumption:** plausible but not yet evidenced.
- **Open question:** cannot be safely decided yet.

Legal/regulatory claims must cite primary sources and be reviewed by qualified Ethiopian counsel or the relevant regulator. A product requirement must not be described as legally mandatory unless the source supports that conclusion.

## Change control

When changing a requirement:
1. Identify affected workflows, entities, events, permissions, tests, and documents.
2. State the reason and evidence.
3. Check migration and data-loss implications.
4. Update the authoritative document and decision log.
5. Mark replaced decisions as superseded; do not preserve obsolete behavior merely for compatibility.
6. Record any required migration, validation, backup, and recovery procedure.

## Implementation gate

No production implementation begins until the relevant scope has approved workflows, state transitions, data ownership, role/permission rules, failure behavior, event/offline policy, security review, regulatory review where applicable, acceptance tests, and migration/recovery requirements. Approval is scoped; unresolved questions must remain visible rather than being silently guessed.
