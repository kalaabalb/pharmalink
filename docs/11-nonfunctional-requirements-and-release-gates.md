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

## 10. Numeric targets still open

Do not invent service-level objectives. Establish targets after user research, representative device/network measurements, architecture evaluation, and cost analysis. At minimum, set:
- p50/p95/p99 latency for critical journeys;
- event delivery and replay objectives;
- service availability objectives;
- maximum stale availability age;
- offline queue maximum age;
- recovery time and recovery point objectives;
- backup restore verification frequency;
- security incident acknowledgement and response targets.

## 11. Approval record

Every release gate should record scope, evidence links, test results, known limitations, unresolved risks, approver, date, and any time-bounded exception. Exceptions must not silently become permanent requirements.
