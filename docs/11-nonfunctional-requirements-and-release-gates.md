# 11 — Non-functional requirements, verification, and release gates

**Status:** Draft — measurable targets remain to be validated.

## 1. Purpose

A feature is not complete because its happy path works. Every release must meet defined requirements for correctness, security, performance, accessibility, reliability, recoverability, and supportability.

## 2. Correctness and integrity

- Financial calculations use exact arithmetic and explicit currency.
- Stock-affecting operations have a defined atomicity boundary.
- Every command that may be retried has an idempotency strategy.
- State transitions are validated server-side.
- Historical finalized records are corrected through linked records, not silent mutation.
- Derived projections can be rebuilt from authoritative data.
- Time-sensitive assertions carry source and freshness metadata.
- No unverified data is presented as verified.

## 3. Performance and responsiveness

Define targets by user journey and device/network class rather than one global number. Measure:
- initial app load and return-to-app time;
- common read and write latency;
- search latency;
- time to receive subscribed updates;
- reconnect and replay completion;
- large catalogue and report behavior;
- low-memory and low-end-device performance.

Targets must be selected from field research and measured under representative Ethiopian connectivity and device conditions. Avoid expensive client-side refresh loops. Event subscriptions should update only affected views and must not cause unnecessary rendering or battery use.

## 4. Reliability and resilience

- Define availability objectives for core operations and public discovery separately.
- Define behavior during database, event transport, identity, payment, messaging, and map-provider outages.
- Make external dependencies non-authoritative unless explicitly designated as the source of truth.
- Use bounded retries with backoff and idempotency.
- Provide operational visibility into queues, failed events, stale projections, and reconciliation mismatches.
- Avoid a single provider failure causing silent corruption or false success.
- Degraded mode must be explicit to the user and must not bypass authorization or transaction invariants.

## 5. Accessibility and localization

- Support keyboard navigation and screen-reader semantics for primary workflows.
- Use readable contrast, scalable text, clear focus states, and non-color-only status indicators.
- Plan for English and Amharic at minimum as a discovery requirement; launch language scope remains to be validated.
- Separate translated labels from canonical data and legal text.
- Define locale, currency, number formatting, date/time, and calendar behavior explicitly.
- Do not assume all users use the same calendar or date-entry convention.
- Ensure errors and offline states are understandable and actionable.

## 6. Backup, recovery, and continuity

Before launch:
- define recovery time and recovery point objectives by service/data class;
- document backup frequency, retention, encryption, and access controls;
- verify restore procedures in an isolated environment;
- validate restored data integrity and event replay consistency;
- test loss of a device and loss of a service component;
- define tenant export and exit procedures;
- assign operational ownership and escalation contacts.

A backup is not considered valid merely because a backup job succeeded. A restore test must prove recoverability.

## 7. Migration and clean replacement

When replacing an existing implementation:
1. Inventory current schemas, data, integrations, user-visible behavior, and legal retention obligations.
2. Identify authoritative data and obsolete/duplicate records.
3. Define mapping, transformation, rejected-record handling, and provenance.
4. Create a verified backup and test restoration.
5. Run migration rehearsal against representative data.
6. Compare counts, totals, relationships, constraints, and business invariants.
7. Reconcile exceptions with an accountable owner.
8. Define a controlled cutover and rollback/recovery decision.
9. Remove obsolete code/contracts only after the new implementation is verified and required records are preserved.
10. Do not retain compatibility layers merely to avoid making a clean transition.

## 8. Test strategy

Required test layers:
- unit tests for domain rules and state transitions;
- property-based tests for arithmetic, inventory invariants, and idempotency where appropriate;
- integration tests for transaction boundaries, outbox publication, and external-provider adapters;
- contract tests for event schemas and API authorization;
- end-to-end tests for normal, alternate, and failure workflows;
- adversarial security tests for tenant isolation and privilege escalation;
- offline/reconnect tests for duplicates, ordering, gaps, expiry, and conflict outcomes;
- migration and restore tests;
- accessibility and localization checks;
- performance tests on representative low-end devices and constrained networks.

Negative tests are mandatory. Tests must prove that prohibited actions fail, not only that permitted actions succeed.

## 9. Release gates

A release cannot be approved until:
- scope and acceptance criteria are reviewed;
- all high-severity security findings are resolved or formally accepted by an accountable owner;
- tenant and branch isolation tests pass;
- regulated workflows have required legal/regulatory review;
- data collection, privacy notices, retention, and processor arrangements are approved;
- event delivery and missed-event recovery are tested;
- offline behavior is documented and tested;
- backup restore and migration verification pass;
- support, monitoring, incident response, and rollback/recovery are ready;
- public claims match actual verification and freshness capabilities.

## 10. Migration, backup, and recovery evidence

The detailed control model is defined in [Specification 26](26-data-migration-import-export-backup-and-restoration-specification.md). Release evidence must demonstrate controlled import and opening-balance reconciliation, usable tenant export, protected backups, and an end-to-end restore exercise. A configured backup job without a successful restore test does not satisfy this gate.

## 11. Operational readiness and incident response

The operating model, ownership register, alerting, severity model, support-access controls, event/provider runbooks, and post-deployment verification are specified in [Specification 28](28-production-operations-incident-response-and-service-continuity-specification.md). This is a blocking production gate: the team must demonstrate that named owners can execute the runbooks, not merely that documents exist.

## 12. Numeric targets still open

Do not invent service-level objectives. Establish targets after user research, representative device/network measurements, architecture evaluation, and cost analysis. At minimum, set:
- p50/p95/p99 latency for critical journeys;
- event delivery and replay objectives;
- service availability objectives;
- maximum stale availability age;
- offline queue maximum age;
- recovery time and recovery point objectives;
- backup restore verification frequency;
- security incident acknowledgement and response targets.


## 13. Approval record

Every release gate should record scope, evidence links, test results, known limitations, unresolved risks, approver, date, and any time-bounded exception. Exceptions must not silently become permanent requirements.

## 14. Production product means deployable and operational—not a demo

The release target is a functional product that a real pharmacy organization can safely deploy and use for the approved scope. A prototype, static interface, scripted walkthrough, seeded showcase, or mocked API is not a production release.

For every capability declared available, all of the following must be true:

- **Real persistence:** data is stored in a production-grade database with defined constraints, transactions, migrations, backups, and restore procedures. In-memory state or browser-only storage is not the authoritative record.
- **Real authorization:** every read and write is enforced server-side using tenant, branch, membership, permission, entitlement, resource-state, and applicable regulated-action checks.
- **Real business rules:** stock, purchasing, sales, returns, finance, and other enabled workflows execute actual domain transactions. Totals, stock balances, and reports are derived from authoritative records, not hard-coded examples.
- **Real failure behavior:** network loss, duplicate requests, concurrent edits, provider timeouts, event gaps, expired sessions, and partial failures produce defined recoverable states. The interface must not report success before the authoritative operation succeeds.
- **Real integrations or honest exclusion:** enabled payment, messaging, identity, maps, printing, or external-data features use configured providers and tested failure handling. If an integration is not implemented, the feature is clearly unavailable; it must not simulate success.
- **Real operational controls:** production configuration, secret management, deployment process, health checks, structured logs, metrics, alerting, incident response, support ownership, and rollback/recovery procedures exist.
- **Real data lifecycle:** import, export, correction, retention, deletion, tenant exit, and migration behavior are specified and tested.
- **Real security assurance:** tenant isolation, permission boundaries, credential handling, rate limits, abuse controls, dependency review, and security tests pass.
- **Real user acceptance:** representative pharmacy staff complete end-to-end tasks using realistic devices and network conditions, with observed usability and correctness evidence.

### Demo and test data rules

- Demo fixtures and synthetic records must be isolated from production and visibly identified.
- Production must not silently fall back to mock data when an API, database, or provider is unavailable.
- A failed request must never be replaced by a fabricated success response or fake stock/financial values.
- Test accounts and sample organizations must not be mixed with real tenant data.
- No public listing, verification badge, payment status, or availability claim may be fabricated for presentation purposes.

### Minimum deployment and operations package

Before a production release, the repository must contain or reference:
1. reproducible environment setup and deployment instructions;
2. environment-variable and secret inventory with no committed secret values;
3. database schema and forward migration strategy;
4. backup, restore, and disaster-recovery runbooks with successful restore evidence;
5. production/staging separation and safe test-data policy;
6. health/readiness checks and dependency-failure behavior;
7. structured logging, metrics, alert thresholds, and dashboards;
8. event/outbox monitoring, replay, dead-letter, and reconciliation runbooks;
9. security incident, privacy incident, and service outage procedures;
10. tenant onboarding, support, export, suspension, and offboarding procedures;
11. release checklist, rollback decision criteria, and post-release verification;
12. named operational ownership and support escalation path.

A release is not production-ready merely because it deploys successfully. The deployed build must pass end-to-end acceptance, security, migration, backup-restore, observability, and operational-readiness checks.

## 15. Capability-level release status

Every feature must be assigned one explicit status: **not built**, **internal-only**, **pilot**, **production-enabled**, **paused**, or **retired**. The status must be enforced by the backend, reflected accurately in the UI, documented for support, and included in release evidence. A feature cannot be described as production-enabled while it relies on mock services, manual database edits, undocumented operator intervention, or unverified legal assumptions.


## 16. Detailed UX and accessibility acceptance

The interaction, accessibility, responsive behavior, localization, date/time/number conventions, and low-bandwidth interface requirements are detailed in [Specification 23](23-user-experience-accessibility-and-localization-specification.md). WCAG 2.2 AA is the proposed design/test baseline; the supported device matrix and language/calendar decisions remain open until user research and accessibility review.


## Architecture verification linkage

The API, database, event-delivery, deployment, and architecture-fitness requirements are specified in [Specification 29](29-application-architecture-api-database-and-deployment-contract.md). Release evidence must show that the selected architecture meets the measurable non-functional targets defined here; merely selecting a technology or drawing a topology does not satisfy a release gate.
