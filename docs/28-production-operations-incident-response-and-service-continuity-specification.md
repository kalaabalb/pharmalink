# 28 — Production operations, incident response, and service continuity specification

**Status:** Draft — operational ownership and measurable targets required before production  
**Priority:** Critical — a deployable application is not a supportable service without accountable operations  
**Scope:** Service ownership, observability, alerting, incident management, security/privacy response coordination, service requests, release operations, change control, and continuity.

## 1. Operating principles

1. Production has named human owners and an escalation path; responsibility cannot exist only in code or with an unavailable developer.
2. User-visible service status must reflect observed system state, not optimistic assumptions.
3. Alerts are actionable, routed to an owner, severity-classified, and linked to a runbook.
4. Incidents are managed to restore safe service first, preserve evidence, communicate clearly, and then learn and prevent recurrence.
5. No operator may bypass tenant isolation, authorization, audit, or legal controls to resolve a support ticket.
6. Manual intervention is controlled, time-limited, attributable, reviewed, and followed by reconciliation.
7. Deployment, migration, rollback, provider change, and recovery are planned operational changes with evidence.
8. The service must degrade safely. When correctness cannot be established, affected capabilities are paused rather than reporting false success.
9. Support access and customer communication must respect privacy, data minimization, and contractual/legal duties.
10. Operational readiness is a release gate, not a post-launch task.

## 2. Service ownership and responsibility model

Before production, maintain a service ownership register naming primary and backup owners for product/release approval, application/infrastructure operations, database recovery, security, privacy, regulatory escalation, customer support/onboarding, provider relationships, and finance/payment reconciliation where enabled.

For each role define authority, coverage hours, contact/escalation route, access level, succession/absence plan, and conflict-of-interest controls. One person may hold multiple roles in an early pilot, but high-impact approvals should remain independently reviewable where practical. A third-party provider's support commitment does not replace Pharmalink's own incident ownership.

## 3. Service inventory and dependency map

Maintain a versioned service register covering clients, API/authentication, database/migrations, object storage, event broker/outbox/workers, scheduled jobs, search/public projections, external providers, observability, secrets, backup, and deployment systems. For each component record owner, environment, criticality, data classification, dependencies, failure mode, health signal, recovery method, provider status, and runbook. Identify common failure domains and which private pharmacy operations can continue during each outage.

## 4. Observability requirements

Use structured, correlated logs, metrics, traces, and business audit records. Capture timestamp, severity, service/version, environment, correlation ID, outcome, and safe tenant/branch context. Monitor request volume/latency/errors, saturation, database health, queue depth/age, event lag, retries/dead letters, scheduled jobs, provider errors, and critical workflow outcomes.

Never place passwords, access tokens, secret keys, full payment credentials, unnecessary personal data, or sensitive health-related content in logs, traces, metric labels, or alert payloads. Avoid high-cardinality tenant/user identifiers in metric labels. Telemetry must be time-synchronized, access-controlled, retained under a defined schedule, and protected against unauthorized alteration. Production and staging telemetry must be distinguishable.

## 5. Health, readiness, and synthetic checks

Define separate liveness, readiness, dependency-health, and workflow-health signals. Liveness should not restart a healthy process solely because an external dependency is temporarily unavailable. Readiness must account for required dependencies and migration/startup state. A green process check does not prove public discovery, private operations, event delivery, or payment integration is healthy.

Synthetic checks must use isolated synthetic tenants and test providers. They must never create real sales, payments, stock movements, or customer messages, and must never appear as production business activity.

## 6. Alerting and service objectives

Before launch, define measurable objectives and alert thresholds for critical journey availability/latency, API errors, database saturation, event/outbox age and dead letters, scheduled jobs, backup age and restore-test age, payment/provider uncertainty, reconciliation backlog, suspicious authentication/privilege/export/support access, storage capacity, certificates, quotas, and unexpected spend.

Each alert must state severity, trigger, owner, acknowledgement expectation, escalation, user impact, and runbook. Tune alerts to avoid alert fatigue. Set service objectives only after representative load, network, device, cost, and pilot evidence. Do not claim an SLA unless contractually approved and measurable.

## 7. Incident severity and command

Define a severity model before production:
- **Critical:** confirmed or strongly suspected cross-tenant exposure, privileged credential compromise, material data corruption/loss, unsafe transaction integrity, broad inability to perform critical operations, or serious active privacy/security event.
- **High:** major workflow unavailable or materially incorrect for multiple customers, payment/stock state uncertain at scale, prolonged event backlog affecting correctness, or provider outage without safe workaround.
- **Moderate:** limited workflow degradation, isolated customer impact, recoverable queue/job failure, or non-critical integration outage.
- **Low:** cosmetic defect, documentation issue, or minor degradation without material data/security impact.

For each incident create a record and assign commander, technical lead, communications owner, and scribe as needed. Establish scope, affected tenants/branches, start time, impact, and confidence. Restrict unsafe capability if correctness is uncertain. Preserve logs, audit events, snapshots, provider evidence, and timeline. Contain, restore using approved runbooks, validate business invariants before re-enabling writes, communicate status and next update, and conduct a proportionate post-incident review. The exact thresholds and response targets remain launch decisions.

## 8. Security and privacy incident coordination

Cover suspected account/credential compromise, unauthorized tenant access, exposed exports/backups/attachments, malicious uploads, compromised dependencies, payment credential compromise, data loss/ransomware, misdirected notifications, and lost devices with cached data.

Preserve evidence and chain of custody; restrict incident-record access; avoid unnecessary copying of sensitive data. Determine affected data subjects, categories, purposes, processors, geography, and likely impact. Qualified legal review determines notification obligations, recipients, content, and deadlines; do not hard-code unverified legal time limits. Coordinate credential rotation, session revocation, provider notification, containment, customer communication, and recovery without destroying evidence.

## 9. Customer support and privileged access

Categorize requests (account/access, organization setup, data correction, inventory/sales, integration/payment, billing, privacy, security, defect). Authenticate the requester and verify tenant authority; collect minimum information; classify impact; use documented diagnostics; escalate regulated, financial, privacy, and security issues; record actions and communication.

Support staff receive no standing unrestricted tenant access. Exceptional access must be ticket-bound, purpose-limited, independently approved where feasible, tenant/resource-scoped, time-limited, re-authenticated, logged, read-only by default, and reviewed. Do not request passwords, OTPs, payment credentials, or unredacted personal/health records through support channels.

## 10. Change, release, and deployment operations

Every production change records owner, reason, risk, affected services/tenants, build/dependency versions, migration behavior, configuration changes, tests/security evidence, deployment window, monitoring plan, rollback/forward-fix/disable criteria, approval, and completion evidence.

Prefer small reversible application releases. Use expand/migrate/contract only where needed for a safe transition; remove obsolete schema/code after verified migration rather than indefinite compatibility layers. Never deploy destructive migrations without tested recovery and an explicit plan for concurrent writes. Code rollback does not reverse committed transactions. Feature flags must be server-enforced and must not expose incomplete functionality.

After deployment verify version, readiness, critical journeys, database health, event delivery, errors, and provider callbacks. Pause rollout or disable affected capability if acceptance checks fail.

## 11. Event pipeline and background job operations

Monitor outbox age, publish failures, consumer lag, retries, dead letters, poison messages, scheduled-job lateness, and replay progress. Runbooks define safe retry/backoff, idempotency verification, dead-letter inspection/redrive, schema mismatch handling, poison-message quarantine, checkpoint recovery, event-gap detection, snapshot reconciliation, and tenant-scoped replay.

Never delete dead letters merely to clear a dashboard. Redrive must revalidate event identity/schema, authorization context, idempotency, and whether the side effect already occurred. Delivery status is not business transaction status.

## 12. Provider outage and degraded-mode operations

For each enabled provider define owner, status source, support route, contract limits, quota/cost thresholds, timeout/retry policy, circuit breaker, queue policy, reconciliation path, and customer message.

Preserve independent private workflows during outage. Mark dependent workflows pending, unavailable, or needing reconciliation. Do not fabricate confirmation or settlement. Stop retries when they risk duplicate external effects. Manual workarounds require approval, audit, and reconciliation. Payment uncertainty must not trigger another collection attempt or a paid status until authoritative state is reconciled.

## 13. Continuity and capacity

Maintain continuity plans for database/storage/event broker/cloud/DNS/certificate/provider outages, operator-access loss, and compromised deployment credentials. Monitor database connections/storage, CPU/memory, worker saturation, queue growth, file storage, API quotas, and egress. Define safe throttling and admission controls. Prioritize transaction correctness and tenant isolation over nonessential features; public discovery or analytics may be paused while private authoritative operations recover.

## 14. Operational runbooks required before launch

Maintain tested runbooks for deployment/verification/rollback, migrations, backup/restore, incident severity/escalation/communications, security/privacy incidents, tenant isolation alerts, payment reconciliation, webhook validation/backlog/dead-letter/replay, event-gap recovery, scheduled-job failures, storage outages, tenant onboarding/support/export/suspension/exit, vulnerability response, emergency patching, certificates/DNS/quotas, and secret rotation.

Each runbook states prerequisites, permissions, read-only versus mutating actions, expected outcomes, verification, abort criteria, recovery, evidence, and escalation. Commands must be safe to copy and must not rely on undocumented developer knowledge.

## 15. Operational acceptance tests

Demonstrate that alerts reach owners; dependency failures are detected and communicated; critical workflows fail safely; support access is scoped and expires; privileged access is audited; dead letters can be safely investigated and replayed; payment timeout remains unresolved until reconciled; failed deployment can be paused without assuming data rollback; compromised credentials/sessions can be revoked; restore is completed and business records reconciled; and owners can execute runbooks without the original developer.

## 16. Decisions required before launch

1. Who are named primary and backup owners for production, security, privacy, support, data, and regulatory escalation?
2. What support coverage hours and response commitments are actually supportable?
3. What severity thresholds and acknowledgement/response objectives can the team meet?
4. Which telemetry platform, retention, region, and access controls are acceptable?
5. Which alerts page an on-call person versus create a ticket?
6. Who has emergency authority to pause sales, inventory writes, discovery, or integrations?
7. What support access model and independent approval are feasible?
8. What provider support contracts and escalation routes exist?
9. Which customer status channels are appropriate?
10. What operational capacity and cost can the initial team sustain?

Until owners, runbooks, alerting, recovery evidence, and support boundaries exist, production release is blocked. Do not invent on-call coverage or service commitments.

## 17. Cross-document dependencies

- [Specification 10](10-security-privacy-and-threat-model.md): security monitoring and privileged access.
- [Specification 11](11-nonfunctional-requirements-and-release-gates.md): service objectives and release gates.
- [Specification 12](12-functional-product-and-production-deployability-contract.md): deployability and supportability.
- [Specification 21](21-external-integrations-and-provider-lifecycle-specification.md): provider lifecycle and outages.
- [Specification 22](22-privacy-and-personal-data-lifecycle-specification.md): privacy incident handling.
- [Specification 24](24-cross-module-transaction-and-implementation-contract.md): transaction integrity and event replay.
- [Specification 26](26-data-migration-import-export-backup-and-restoration-specification.md): backup and disaster recovery.

## 18. Tenant-data access operations and isolation incident runbook

### 18.1 Normal support diagnostics

Support should begin with customer-provided request IDs, timestamps, error codes, and non-sensitive screenshots or descriptions. Prefer diagnostics that return service health, job state, projection freshness, and safe validation outcomes without exposing pharmacy records. Confirm the requester’s identity and authority for the relevant tenant before discussing tenant-specific status. A support case does not itself grant access to the tenant's database, files, reports, or event stream.

### 18.2 Exceptional tenant-data access procedure

Where direct tenant-data access is genuinely necessary, the support case or incident record must state the business purpose, customer/tenant, exact resources or bounded query, requested actions, data classification, duration, requester, approver, and expected outcome. Grant the minimum permission, read-only by default, with re-authentication and automatic expiry. Mutating access requires a separate approved change instruction, explicit scope, before/after evidence, and customer communication where appropriate. The access system must preserve the support operator's identity and must not replace it with the customer's identity. All reads, downloads, writes, queries, and failed attempts within the privileged session are audited. The grant must be revoked immediately when the task ends or the case is closed. An independent reviewer examines the grant and activity afterward.

Emergency break-glass access is limited to containment or restoration of an active incident when normal approval is unavailable and delay would materially increase harm. It requires a named incident commander, recorded justification, smallest possible tenant/resource scope, shortest feasible expiry, immediate alert to security/operations ownership, and retrospective independent review. Break-glass does not authorize bypassing data-integrity controls, making unapproved business decisions, or hiding operator identity. If the system cannot enforce or audit the required scope, use a safer recovery method rather than granting broad database access.

### 18.3 Isolation alert triage and containment

Treat any credible indication of cross-tenant disclosure, cross-branch access outside an explicit grant, tenant-context confusion in a worker/cache/realtime path, foreign-tenant mutation, or unlogged privileged access as Critical until disproven. The incident commander must:
1. preserve relevant audit, access, application, database, queue, cache, object-storage, and provider evidence with timestamps and integrity metadata;
2. identify the suspected access path, identities/service roles, affected tenant and branch scope, data classes, time window, and whether reads, exports, or mutations occurred;
3. revoke suspect sessions, grants, credentials, tokens, subscriptions, download links, and queued work as appropriate;
4. pause the affected endpoint, worker, export, subscription, cache path, or integration if isolation cannot be established; do not disable unrelated private management capabilities without evidence that they are affected;
5. prevent replay or retry from repeating an unauthorized or uncertain side effect;
6. involve security, privacy, legal/regulatory, and customer communication owners according to the incident facts; qualified review determines notification duties and timing;
7. restore only after the ownership defect is fixed, affected data is assessed, integrity is reconciled, and regression tests cover the exploit path.

Do not clear caches, queues, logs, or database state before evidence is preserved. Do not assume that the absence of a successful write proves there was no disclosure; read access, aggregate inference, and artifact download must also be assessed.

### 18.4 Operational acceptance evidence

Before production, demonstrate that support grants can be created, narrowed, expired, revoked, and independently reviewed; that a support user cannot enumerate tenant records without a grant; that impersonation is visibly labeled and preserves operator attribution; and that a simulated cross-tenant alert reaches an owner with an executable containment runbook. The test must include API, database, object storage, exports, event subscriptions, caches, background jobs, and restore operations. Record test owner, identities, scope, timestamps, expected/observed access, audit evidence, and corrective actions.
