# 12 — Functional product and production deployability contract

**Status:** Draft — mandatory cross-module acceptance standard  
**Purpose:** Prevent Pharmalink from being reduced to a UI demo, mock backend, or incomplete operational tool.

## 1. Product delivery commitment

The target is a deployable, secure, maintainable product that real pharmacy organizations can use for the explicitly approved first-release scope. “The documentation is complete” means the team can implement, test, deploy, operate, support, recover, and evolve that scope without inventing critical rules during coding. The proposed first-release candidate, end-to-end journeys, exclusions, and evidence gates are set out in [Specification 25](25-first-release-scope-and-launch-decision-framework.md); that proposal remains subject to product-owner approval and field, legal, and accounting validation.

It does not mean every possible Pharmalink capability must ship in the first release. It means the selected release is coherent and genuinely functional from end to end. Deferred features must be clearly unavailable, not simulated.

## 2. Definition of a real functional capability

A capability is complete only when it has:

1. an identified user problem, actor, permission, and supported operating context;
2. approved workflow, preconditions, business rules, state transitions, and failure paths;
3. authoritative data ownership, schema, validation, retention, and audit behavior;
4. server-side API/command behavior and transaction boundaries;
5. concurrency, idempotency, and duplicate-submission handling where applicable;
6. event publication, subscription, replay, and reconciliation behavior where applicable;
7. offline classification and synchronization rules where applicable;
8. accessible UI states for loading, success, empty, pending, stale, denied, offline, conflict, and failure;
9. automated tests, including negative and adversarial cases;
10. monitoring, support, and recovery procedures;
11. migration and deployment implications;
12. acceptance evidence from a realistic environment.

A screen or button without the corresponding authoritative behavior is not a completed capability.

## 3. First-release scope must be a coherent vertical slice

The first production scope must be selected from validated pharmacy research, not from the size of the feature list. It should include a complete path through the chosen business problem, such as organization setup, staff access, catalogue/stock setup, and the selected operational transaction(s), plus the necessary reporting, audit, export, support, and recovery.

The exact first-release workflow set remains an open product decision until field research establishes:
- target establishment categories and pharmacy sizes;
- real receiving, inventory, sales, return, and finance practices;
- who performs and approves each task;
- devices, printers, scanners, and connectivity constraints;
- legal and professional boundaries for each included operation;
- the smallest set of workflows that provides sustained operational value.

Do not select a broad collection of disconnected screens and call it an MVP. Do not include high-risk regulated workflows simply to make the product appear complete.

## 4. System-wide functional requirements

The implementation specification must define, at minimum:

### Identity and organization
- account creation, sign-in, recovery, session revocation, and device/session management;
- organization creation, duplicate detection, invitation acceptance, membership changes, and branch assignment;
- management-only operation as a first-class mode;
- permission grants, approval separation, and audit attribution;
- every enabled action maps to the canonical permission registry in [Specification 18](18-proposed-permission-matrix.md), with an approved role/scope grant and tested denial cases; The proposed first-release role-grant baseline in Specification 18 §12 is a review draft only; no grant is production-approved until explicitly accepted.
- independently managed establishment evidence, professional credentials, management entitlements, public participation, and commerce capability states.

### Pharmacy operations
- catalogue identity, local aliases, pack/presentation identity, and source provenance;
- stock by branch, item, batch/lot, expiry, and quality status;
- receiving, counts, transfers, adjustments, holds, release, recall, and disposal;
- supplier records and purchase lifecycle;
- sales, voids, returns, refunds, and payment reconciliation for the approved scope;
- finance records and reports with explicit accounting/tax assumptions;
- audit, export, and correction workflows.

### Public discovery
- explicit opt-in at organization/branch/capability level;
- field-level publication policy and preview;
- public projection isolated from private operational data;
- freshness timestamp and automatic stale/unknown transitions;
- pause, withdrawal, moderation, correction, and appeal;
- no reservation, ordering, payment, or delivery unless separately specified and legally reviewed.

### Platform operations
- onboarding review and evidence verification;
- support case workflow and purpose-bound privileged access;
- subscriptions/entitlements and transparent suspension behavior;
- security monitoring, incident handling, moderation, and operational audit.

## 5. Architecture and implementation specifications required

The module specifications are complemented by [Specification 26](26-data-migration-import-export-backup-and-restoration-specification.md), which defines controlled data migration, opening balances, tenant export, backup, restoration, and disaster recovery. The module specifications are also complemented by [Specification 21](21-external-integrations-and-provider-lifecycle-specification.md), which governs external provider selection, contracts, secrets, webhooks, outage handling, and provider exit, and [Specification 22](22-privacy-and-personal-data-lifecycle-specification.md), which governs personal-data purposes, notices, rights, retention, deletion, and privacy incident handling, and [Specification 23](23-user-experience-accessibility-and-localization-specification.md), which defines the required interaction, accessibility, responsive, and localization behavior. No third-party capability is production-enabled merely because a sandbox integration succeeds.


Before coding a module, its specification must include:
- module responsibility and explicit non-responsibilities;
- authoritative data owner and database transaction boundary;
- API/command/query contracts and validation/error codes;
- event names, schemas, versions, delivery and replay semantics;
- tenant/branch authorization strategy;
- offline and local persistence policy;
- external dependencies and provider failure modes;
- observability signals and support actions;
- test fixtures, acceptance tests, and migration strategy.

The initial architecture may use a modular monolith if validated, but module boundaries must be enforceable in code. Avoid premature microservices and avoid a monolith whose modules freely mutate each other's data. The final architecture decision must follow workflow, consistency, event, security, scale, and operational analysis. The detailed module, API, database, event, deployment, and technology-decision contract is [Specification 29](29-application-architecture-api-database-and-deployment-contract.md).

## 6. Production data and integrity requirements

- The database is authoritative for server-confirmed business state.
- Financial and stock-affecting commands must be atomic at the level required by their invariants.
- Finalized records must be corrected through traceable reversal/adjustment records, not silent edits.
- All money calculations use exact decimal/minor-unit rules and an explicit currency; no binary floating-point arithmetic for financial values.
- Every stock movement identifies item, batch where applicable, source/destination, quantity/unit, actor, time, reason, and originating transaction.
- Every retryable write has an idempotency design.
- Reports must disclose their source, time window, branch scope, and calculation basis.
- Imports must validate, preview, report rejected rows, and support reconciliation before finalization.
- Data export must be usable, documented, and integrity-checkable.
- Production data must never depend on local browser state or manually edited database rows.

## 7. Offline and event-driven behavior

- Use subscriptions/push for normal live updates; do not implement frequent polling as the default refresh mechanism.
- Persist durable event publication intent with the transaction.
- Consumers tolerate duplicates, gaps, reordering, reconnects, and replay.
- Clients show the distinction between local draft, queued, server-submitted, accepted, rejected, and needs-review.
- Offline permissions have bounded validity and are revalidated on synchronization.
- Only explicitly approved operations may be queued offline. Do not assume sensitive stock, money, or regulated actions are safe to finalize offline.
- Public availability requires fresh server confirmation and automatically becomes stale/unknown when its freshness deadline passes.

## 8. Non-functional and operational requirements

The first release must set measurable targets for:
- critical journey latency and throughput;
- event delivery and replay;
- service availability and degraded-mode behavior;
- maximum public availability age;
- offline queue age and recovery;
- backup frequency, recovery point objective, and recovery time objective;
- security response and vulnerability remediation;
- supported device/browser/OS versions;
- localization, accessibility, and low-connectivity usability.

Targets must be justified by research, threat analysis, load testing, and operating cost—not invented to make a checklist look complete.

## 9. Environments and deployment

Define separate development, test/CI, staging, and production environments. Each must have:
- explicit configuration and secret boundaries;
- reproducible provisioning and deployment;
- versioned database migrations;
- controlled access and audit;
- isolated test data;
- health/readiness checks;
- monitoring and alerting appropriate to its purpose.

Production deployment must not require undocumented manual steps. A deployment must identify the application version, migration version, configuration version, and release evidence. Rollback must account for schema compatibility and irreversible data changes; database rollback must not be assumed safe.

## 10. Security, privacy, and legal release gates

Before production:
- tenant/branch isolation and privilege-escalation tests pass;
- account recovery, session revocation, and privileged access are tested;
- secrets and sensitive data are not exposed in source, logs, client bundles, or notifications;
- data purposes, lawful bases, notices, retention, processors, hosting, and transfer arrangements receive required review;
- regulated workflows and public claims have current legal/regulatory review;
- no professional credential or establishment evidence is used as a universal platform-access gate;
- no unapproved clinical decision, advertising, ordering, payment, or delivery capability is enabled.

## 11. Testing and acceptance evidence

Required evidence includes:
- unit and property-based tests for domain invariants;
- integration tests for persistence, transactions, outbox, and provider adapters;
- API and event contract tests;
- end-to-end tests for each supported workflow;
- authorization tests for allowed and denied role/tenant/branch combinations;
- concurrency, duplicate, retry, and failure-injection tests;
- offline/reconnect/replay tests;
- migration, export, backup, and restore tests;
- accessibility, localization, and representative-device tests;
- load/performance tests for agreed targets;
- user acceptance sessions with representative pharmacy staff.

Passing a test suite is necessary but not sufficient. Acceptance evidence must be linked to the release and the tested build.

## 12. Launch, support, and continuity

The operating model must identify:
- who owns production, security, privacy, support, and regulatory escalation;
- support hours and response targets;
- incident severity and escalation;
- customer onboarding and training;
- tenant export and exit;
- subscription failure and service suspension behavior;
- provider outage communication;
- backup restoration and disaster recovery;
- security/privacy incident notification responsibilities;
- release communications and post-deployment verification.

A production product must be supportable by an accountable operator, not only by the developer who wrote it.
The API operation inventory and first-release workflow-to-command/query/permission/event/test traceability matrix are specified in [Specification 29](29-application-architecture-api-database-and-deployment-contract.md), §4.7–§4.9. Common command/query envelopes and critical operation payload/authority requirements are specified in §4.10–§4.11; operation-specific schemas remain mandatory before implementation. Every enabled UI action must map to an implemented API operation; a visual control without an operation contract is not a working feature.

The detailed production operating model—including named ownership, observability, actionable alerts, incident response, scoped support access, provider/event operations, deployment verification, and continuity—is specified in [Specification 28](28-production-operations-incident-response-and-service-continuity-specification.md). Production is not supportable until the required owners, runbooks, and operational acceptance evidence exist.


## 13. Explicit non-demo acceptance rule

A feature fails acceptance if any of the following is true:
- it uses hard-coded or synthetic production values;
- it reports a successful write without a committed authoritative transaction;
- it relies only on client-side permission checks;
- it silently substitutes mock data after a backend failure;
- it has no defined empty, denied, stale, offline, conflict, or error state;
- it cannot be restored from backup or migrated using documented procedures;
- it exposes data across tenant or branch boundaries;
- it claims verification, legal approval, stock availability, or provider integration that has not actually been established;
- support and operational ownership are undefined.

A production release is approved only for the capabilities that pass this standard. Anything else remains explicitly disabled, internal-only, pilot, or deferred.
