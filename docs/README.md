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
| 10-security-privacy-and-threat-model.md | Trust boundaries, threats, access control, data protection | Draft |
| 11-nonfunctional-requirements-and-release-gates.md | Quality attributes, testing, migration, recovery, release gates | Draft |
| 12-functional-product-and-production-deployability-contract.md | Definition of a real functional, deployable, supportable product | Draft |
| 13-inventory-and-batch-lifecycle-specification.md | Inventory, batch/lot traceability, stock movements, transfers, expiry, and recall | Draft |
| 14-sales-pos-and-payment-lifecycle-specification.md | Private POS, sales, payment state, receipts, returns, refunds, and reconciliation | Draft |
| 15-supplier-and-purchasing-lifecycle-specification.md | Supplier records, purchase orders, receipts, invoices, returns, and supplier payments | Draft |
| 16-medicine-catalogue-and-product-identity-specification.md | Canonical medicine identity, presentations, barcodes, provenance, search, and correction | Draft |
| 17-finance-and-reconciliation-specification.md | Financial records, cashbook, receivables/payables, reconciliation, and reporting | Draft |
| 18-proposed-permission-matrix.md | Canonical permission registry, action-specific evidence gates, and proposed first-release role-grant baseline | Draft |
| 19-public-discovery-and-availability-specification.md | Public pharmacy discovery, opt-in profiles, medicine search, availability freshness, moderation, and privacy | Draft |
| 20-organization-branch-and-membership-lifecycle-specification.md | Tenant, legal-entity relationships, branches, invitations, roles, ownership, suspension, and offboarding | Draft |
| 21-external-integrations-and-provider-lifecycle-specification.md | Provider due diligence, webhooks, payment, messaging, maps, storage, outages, and exit | Draft |
| 22-privacy-and-personal-data-lifecycle-specification.md | Data inventory, purpose limitation, notices, rights, retention, deletion, processors, and privacy incidents | Draft |
| 23-user-experience-accessibility-and-localization-specification.md | Navigation, workflow feedback, accessibility, localization, responsive and offline UX | Draft |
| 24-cross-module-transaction-and-implementation-contract.md | Shared transaction boundaries, state-machine rules, identity, authorization, event, and offline contracts | Draft |
| 25-first-release-scope-and-launch-decision-framework.md | Proposed first-release scope, end-to-end journeys, exclusions, pilot model, and launch gates | Draft |
| 26-data-migration-import-export-backup-and-restoration-specification.md | Controlled imports, opening balances, tenant export, backups, restore testing, and disaster recovery | Draft |
| 27-inventory-valuation-stock-reconciliation-and-cost-integrity-specification.md | Cost provenance, valuation policy decision framework, stock reconciliation, write-offs, and cost integrity | Draft |
| 28-production-operations-incident-response-and-service-continuity-specification.md | Service ownership, observability, alerting, incident response, support access, releases, and continuity | Draft |
| 29-application-architecture-api-database-and-deployment-contract.md | Domain boundaries, API/database/deployment contracts, workflow traceability, and command/query schema rules | Draft |
| 30-workflow-state-transition-and-recovery-contract.md | Explicit workflow transitions, guards, cross-module ordering, failure recovery, and acceptance tests | Draft |

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
- Enforceable application architecture, API contracts, database integrity, event delivery, deployment topology, and technology decision records.
- Cross-module state-transition rules, approval guards, transaction ordering, and failure/recovery behavior.

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

## Product-owner clarification — platform access and direct-main workflow

The product owner has explicitly clarified:
- Pharmalink is a platform offering management capabilities and optional public-facing access/discovery.
- A pharmacy retailer may use management functions without selling, listing products, or publishing availability through the app.
- Public participation is opt-in and independently controllable.
- An individual professional licence is not a general account-registration or ordinary platform-use requirement. Credential checks are scoped to specific regulated actions or verification claims.
- Documentation and implementation work should be committed directly to `main` by default. Do not require the product owner to open and merge a pull request for routine work.

These are user-owned product decisions and override earlier draft assumptions that implied a blanket professional credential gate.


## Production-product standard

Document completion is not permission to ship a demo. The selected first release must be a coherent vertical slice that a real pharmacy can deploy and use for its approved scope. Every enabled capability must have authoritative persistence, server-side authorization, real business transactions, failure/recovery behavior, automated acceptance evidence, observability, and an operational owner.

The production deployability contract in [12-functional-product-and-production-deployability-contract.md](12-functional-product-and-production-deployability-contract.md) is mandatory across all modules. Mock services and synthetic fixtures are permitted only in isolated development/test contexts and must never silently substitute for production functionality.

A capability that is not fully implemented and verified must be marked unavailable, internal-only, pilot, paused, or deferred—not presented as working.


## Module-specification rule

Module specifications must define state transitions, transaction boundaries, concurrency, idempotency, permissions, event behavior, offline eligibility, audit, failure recovery, migration, reporting, and negative acceptance tests. Inventory and batch traceability is the first detailed module specification because it underpins sales, purchasing, public availability, and recall handling.
