# 26 — Data migration, import/export, backup, and restoration specification

**Status:** Draft — mandatory before production data onboarding or launch  
**Priority:** Critical — data loss, corrupted opening balances, or an untested restore can make the platform unsafe to operate  
**Scope:** Legacy data assessment, import, migration, export, backup, restore, disaster recovery, tenant exit, verification, and operational evidence.

## 1. Purpose and governing rules

This specification defines how Pharmalink acquires, transforms, protects, restores, and exports operational data. It applies to tenant onboarding, bulk catalogue and stock setup, upgrades, provider changes, disaster recovery, and customer exit.

The database is authoritative only after a controlled import or transaction has been validated and committed. A spreadsheet upload, successful file parse, browser preview, or migration script completion is not proof that business records are correct.

Non-negotiable rules:

1. No production import, migration, destructive cleanup, or restore without a verified recovery point and an approved execution plan.
2. Never silently discard source rows, identifiers, units, dates, batch details, financial history, or provenance.
3. Preserve source evidence and transformation decisions sufficiently to explain every accepted, rejected, merged, split, or corrected record.
4. Do not overwrite live transactional history to make imported totals match. Use explicit opening balances or controlled correction movements with traceable approvals.
5. A restore must be tested, not merely reported as successful by a backup provider.
6. Backup copies, exports, staging datasets, logs, and temporary files receive the same tenant isolation and confidentiality controls as their source data.
7. Migration and restore procedures must be repeatable, observable, interruption-safe, and recoverable.
8. Retention and deletion must follow the approved legal, privacy, contractual, and business-record policy; a tenant deletion request does not authorize destruction of records that must lawfully be retained.

## 2. Data inventory and classification

Before migration or backup design is approved, maintain a versioned inventory of:

- accounts, authentication references, memberships, roles, invitations, and organization/branch relationships;
- establishment evidence and professional credentials, including access restrictions and expiry/review status;
- catalogue identities, presentations, local aliases, barcodes, units, and source provenance;
- suppliers, purchase orders, receipts, invoices, credit notes, and supplier payments;
- batches/lots, stock positions, movements, counts, transfers, holds, recalls, and dispositions;
- sales, sale lines, price/tax snapshots, receipts, voids, returns, refunds, and payment references;
- cash sessions, cashbook movements, reconciliation imports, and operational reports;
- public profile and publication preferences, availability projections, moderation history;
- audit events, domain events, outbox/inbox records, idempotency records, and integration references;
- support cases, consent/notice records, privacy requests, and operational metadata.

For each data class record its owner, tenant/branch scope, sensitivity, purpose, authoritative source, retention rule, export eligibility, backup requirement, encryption requirement, and restore dependency. Credentials, secrets, payment authentication data, and unnecessary health-related data must not be copied into migration datasets.

## 3. Source assessment and migration planning

Every migration begins with a source assessment that records:

- source system, version, owner, extraction method, extraction timestamp, and source timezone;
- source tables/files and row counts;
- known data-quality issues, missing fields, duplicate patterns, and unsupported formats;
- units, currency, date/calendar conventions, locale, and encoding;
- whether source records are authoritative, provisional, duplicated, or manually maintained;
- legal/contractual rights to access, transform, retain, and import the data;
- expected downtime, dual-entry period, cutover window, and rollback boundary.

Create a mapping specification from each source entity/field to its Pharmalink destination, including transformation, normalization, default policy, validation, rejection reason, and provenance. A default must never fabricate a fact such as batch number, expiry date, professional identity, supplier invoice, payment confirmation, or product registration.

If a source field cannot be represented safely, quarantine the row for human resolution or exclude it with an explicit report and owner approval. Do not coerce ambiguous data into a plausible-looking value.

## 4. Import pipeline and controls

The import pipeline must use explicit stages:

1. **Upload/extract:** accept only permitted formats and sizes; validate encoding and malware policy; store in an isolated staging area.
2. **Parse:** retain source row number and stable source key; report malformed rows.
3. **Normalize:** apply versioned transformations for whitespace, identifiers, dates, units, currency, and locale without losing original values.
4. **Match:** resolve catalogue and organization identities using explainable rules and confidence. Ambiguous matches require human review; no silent merging by name or barcode alone.
5. **Validate:** enforce required fields, referential integrity, tenant/branch ownership, permitted states, and domain invariants.
6. **Preview:** show accepted, rejected, warning, duplicate, and unresolved counts plus representative row-level explanations and aggregate impact.
7. **Approve:** an authorized operator confirms scope, source, transformation version, exception policy, and resulting balances/totals. High-risk financial or stock opening data may require a second approver.
8. **Commit:** execute in bounded, idempotent batches with a durable import job ID and checkpoint. Each committed batch is atomic; retries cannot duplicate records.
9. **Reconcile:** compare source, accepted, rejected, and destination counts and relevant control totals. Investigate every unexplained difference.
10. **Close:** retain the immutable import report, mapping version, approvals, hashes, job logs, and links to created records according to retention policy.

The UI must distinguish draft, validating, awaiting review, ready, committing, partially committed, completed, failed, and cancelled. A timeout must be resolved by querying the import job and its checkpoints, not by blindly re-uploading.

Partial completion must be explicit. The operator must be able to resume safely or abandon only uncommitted work; already committed batches require a compensating plan, not destructive rollback.

## 5. Opening balances and historical data

Opening stock must be introduced through a dedicated, approved opening-balance workflow, not by directly editing stock projections. The opening record must include:

- effective date/time and recorded date/time;
- tenant, branch, product presentation, batch/lot, unit, quantity, and stock status;
- source system and source row/document reference;
- expiry/manufacturing data when known and applicable;
- cost basis and valuation policy/version if cost is imported;
- uncertainty/missing-data flags and review disposition;
- preparer, reviewer, and approval evidence.

Opening balances must reconcile to a signed-off physical count or other explicitly accepted evidence. Unknown batch or expiry data must remain unknown and must not be invented. Product categories whose traceability fields are required cannot be enabled for sale until the required evidence is resolved.

Historical sales/payments should be imported only when source integrity, legal retention, and mapping can be established. Otherwise preserve a read-only archive/export and begin authoritative live transactions from a documented cutover point. Do not fabricate detailed historical lines from aggregate totals.

## 6. Cutover, dual running, and rollback

A cutover plan defines:

- source freeze or extraction cutoff;
- treatment of transactions created during extraction;
- final delta extraction and reconciliation;
- user communication and permissions during cutover;
- opening balance approval;
- production activation and health checks;
- criteria to stop, rollback, or proceed;
- ownership and escalation during the window.

Dual entry is not the default because it creates competing sources of truth. If unavoidable, define one authoritative system per data class, a strict end date, reconciliation frequency, and a procedure to resolve divergence.

Rollback must distinguish application rollback from data rollback. Reverting application code does not reverse committed business transactions. Destructive database rollback is prohibited unless the recovery plan proves it safe and approved. Prefer forward fixes or restore to a controlled recovery environment, then reconcile any transactions that occurred after the recovery point.

## 7. Export and tenant portability

An authorized tenant export must be complete for the agreed scope, machine-readable, documented, and integrity-checkable. It should include:

- data dictionary and schema/export version;
- tenant/branch identifiers and stable record IDs;
- catalogue links, batches, stock movements, sales, returns, supplier and finance records within permitted scope;
- source references, timestamps, currencies, units, conversion versions, and state;
- audit history and correction/reversal links where export is lawful and appropriate;
- attachments or a manifest identifying excluded/unavailable attachments;
- checksums, record counts, and reconciliation summaries.

Exports must preserve relationships and distinguish current projections from immutable source facts. They must not expose other tenants' data, internal security secrets, unrelated platform data, or personal data beyond the authorized purpose.

Provide a documented export request, authorization, preparation, expiry/download, audit, and deletion lifecycle. Candidate export lifecycle events are TenantExportRequested, TenantExportCompleted, and TenantExportExpired. They must contain artifact identifiers and scope metadata, not exported personal/financial payloads. Subscription suspension must not silently erase tenant records or prevent access to records that must remain available under law or contract. Tenant exit must define export, retention, legal hold, deletion, and proof-of-completion responsibilities.

## 8. Backup architecture and policy

The production design must specify:

- database backup method, frequency, and point-in-time recovery capability;
- object/file/attachment backup and consistency relationship to database records;
- event/outbox and integration state needed for coherent recovery;
- encryption in transit and at rest, key ownership, rotation, and recovery access;
- separate credentials and access paths for backup administration;
- geographically and administratively separated copies where risk analysis requires;
- immutability or deletion resistance for selected recovery copies;
- backup retention tiers and secure expiry;
- monitoring for failed, delayed, incomplete, or unexpectedly small backups;
- documented provider dependencies and exit procedure.

Backup frequency and retention must be derived from approved recovery point objectives (RPO), legal retention, data growth, cost, and risk. Do not invent numeric RPO/RTO targets. Set them before launch and test whether the chosen architecture can meet them.

A backup job is not considered successful solely because the provider returned HTTP success. Verify completion metadata, expected coverage, integrity checks, and recoverability.

## 9. Restore and disaster recovery procedure

A restore runbook must identify the incident commander, technical executor, approver, communication owner, and decision authority. It must cover:

1. declare incident and classify scope, data-loss window, security/privacy implications;
2. stop or isolate writes when continued operation could worsen corruption or compromise;
3. preserve logs, provider evidence, and affected snapshots;
4. select a recovery point using approved RPO and incident facts;
5. restore into an isolated environment first;
6. verify database integrity, schema/migration version, object references, encryption keys, tenant boundaries, and event/outbox consistency;
7. run domain reconciliation for stock movements/balances, sale/payment states, supplier obligations, and financial totals;
8. identify transactions after the recovery point and determine whether they can be replayed from durable sources without duplication;
9. obtain explicit approval before promoting the recovered environment;
10. communicate service status and known gaps;
11. monitor post-recovery behavior and reconcile delayed provider callbacks/events;
12. document root cause, data loss, recovery duration, corrective actions, and customer impact.

Do not replay all queued commands or webhooks blindly after restore. Revalidate authorization, idempotency, source status, event sequence, and whether the external side effect already occurred. Payment provider reconciliation is mandatory before assuming payment state is correct.

## 10. Restore testing and evidence

Before production and at a defined recurring interval, perform a restore exercise that tests more than database startup:

- restore database and required object data;
- apply or verify schema/migration version;
- confirm tenant isolation and authorization;
- validate representative organization, catalogue, batch, movement, sale, return, supplier, payment, and audit records;
- reconcile stock projections against the movement ledger;
- reconcile sale/payment/refund states against provider or source evidence where applicable;
- verify outbox/inbox replay does not duplicate side effects;
- test application startup, health/readiness, critical workflows, export, and event delivery;
- measure actual restore duration and recoverable data point;
- record failures, corrective actions, approver, and evidence location.

A restore test that only proves a database can be opened is insufficient. Production release evidence must include a successful end-to-end restore exercise against a representative, protected dataset.

## 11. Migration and restore security

- Enforce tenant and role authorization on import, export, backup access, restore, and recovery tools.
- Require strong re-authentication and, for high-impact operations, independent approval.
- Encrypt staging files and temporary artifacts; expire them automatically after the approved retention window.
- Never put secrets, access tokens, or unnecessary personal data in import logs or exported diagnostics.
- Audit who requested, approved, executed, downloaded, restored, or deleted each artifact.
- Protect backup credentials and encryption keys separately from the application runtime.
- Treat support/vendor access as purpose-bound, time-limited, and auditable.
- Test access revocation and emergency credential rotation.

## 12. Acceptance tests and failure injection

Automated and operational tests must prove:

- malformed, oversized, duplicate, and maliciously structured files are rejected safely;
- an ambiguous medicine match is not silently accepted;
- source row counts reconcile to accepted, rejected, and explicitly skipped rows;
- unit/date/currency transformations are deterministic and versioned;
- retries and timeouts do not duplicate committed import batches;
- a partial import resumes without duplicating or losing committed records;
- opening balances create traceable stock movements and never directly overwrite projections;
- a source row cannot cross tenant or branch boundaries;
- unauthorized export and backup access are denied and audited;
- an export preserves relationships, identifiers, units, currencies, timestamps, and correction links;
- a restore recovers database and required objects to a coherent point;
- post-restore replay does not duplicate stock or financial effects;
- corrupted, missing, expired, or inaccessible backups trigger alerts and escalation;
- key loss, provider outage, and failed restore are covered by documented recovery paths;
- tenant exit applies approved retention/legal-hold rules and produces an auditable completion record.

## 13. Operational metrics and evidence

Track at minimum: backup completion and age, restore-test age/result, restore duration, estimated recoverable point, import job duration, row acceptance/rejection rates, unresolved migration exceptions, export job status, artifact expiry, and recovery incidents.

Every migration, export, backup policy change, restore, and recovery exercise must produce an evidence record with scope, actor, approver, timestamps, software/schema versions, source/backup identifiers, checksums, reconciliation results, exceptions, and outcome.

## 14. Decisions required before production

1. Which legacy systems and data formats will pilot pharmacies actually provide?
2. Which data classes must be imported versus retained as a read-only archive?
3. What source evidence is sufficient to approve opening stock and batch details?
4. What RPO/RTO, backup frequency, retention, and geographic separation are justified by the pilot risk and cost?
5. Which cloud/database/object-storage providers and contractual terms are acceptable?
6. Which records must be retained after tenant closure, and for how long under applicable law?
7. Who may approve opening balances, high-risk imports, exports, restores, and tenant deletion?
8. What is the cutover and dual-running policy for each pilot?
9. What restore-test frequency and evidence retention will be required?
10. What is the incident-time authority to stop writes, restore, replay, or delay service recovery?

Resolve each decision, or explicitly defer the affected capability with a safe operational restriction, before production onboarding. The product must not present backup, migration, export, or restore as available until its end-to-end acceptance evidence exists.

## 15. Cross-document dependencies

- [Specification 07](07-domain-model-and-data-ownership.md): entity ownership and data lifecycle.
- [Specification 10](10-security-privacy-and-threat-model.md): trust boundaries, privileged access, and incident handling.
- [Specification 11](11-nonfunctional-requirements-and-release-gates.md): recovery objectives and release gates.
- [Specification 12](12-functional-product-and-production-deployability-contract.md): production deployment and non-demo acceptance.
- [Specification 13](13-inventory-and-batch-lifecycle-specification.md): stock ledger and opening-balance invariants.
- [Specification 17](17-finance-and-reconciliation-specification.md): financial source records and reconciliation.
- [Specification 20](20-organization-branch-and-membership-lifecycle-specification.md): tenant exit and membership lifecycle.
- [Specification 21](21-external-integrations-and-provider-lifecycle-specification.md): provider exit, callbacks, and external reconciliation.
- [Specification 22](22-privacy-and-personal-data-lifecycle-specification.md): retention, rights, deletion, and processors.
- [Specification 24](24-cross-module-transaction-and-implementation-contract.md): transaction, idempotency, and event replay rules.
- [Specification 28](28-production-operations-incident-response-and-service-continuity-specification.md): accountable recovery roles, incident command, operational runbooks, and restore/recovery acceptance.
- [Specification 29](29-application-architecture-api-database-and-deployment-contract.md): physical data ownership registry, schema/migration controls, tenant constraints, and schema-lock evidence gate (§§5.5–5.8).
- [Specification 30](30-workflow-state-transition-and-recovery-contract.md): aggregate state, unknown outcomes, idempotent replay, and post-restore transition/reconciliation rules.

Before migration scripts or production backup procedures are approved, the selected source/destination schema and recovery process must be reconciled against the physical ownership registry and database constraints required by Specification 29. The migration must preserve aggregate identity, tenant/branch ownership, immutable source facts, correction links, and event/outbox consistency; a structurally successful import is not sufficient if those invariants fail.

## 12. Tenant and branch isolation during portability and recovery

Tenant isolation remains mandatory throughout import, export, backup, restore, and tenant exit. A data portability operation is not a privileged shortcut around the normal ownership model.

### 12.1 Import scope

Every import job is bound at creation to one tenant, an explicit set of branches or organization-wide scope, an initiating actor, an approved dataset, and a mapping/schema version. The scope is immutable after approval; changing tenant or branch scope requires cancelling the uncommitted job and creating a newly authorized job. Each source row and every resolved reference is checked against the target tenant and permitted branch before commit. A row that references another tenant's supplier, branch, catalogue-private record, attachment, or aggregate is rejected or quarantined; it is never silently rebound by matching a name. Worker retries reload the persisted job scope and current authorization/policy rather than trusting mutable queue payload fields.

### 12.2 Export completeness and non-disclosure

An export manifest must record tenant ID, included branch IDs or explicit organization-wide scope, requesting identity, authorization evidence, dataset and filters, schema version, source cutoff/as-of time, record counts by entity, attachment counts, checksums, warnings/exclusions, creation and expiry times, and artifact status. The export query must enforce scope before joins and aggregation. Shared canonical reference data may be represented by stable references or an approved portable snapshot; it must not expose platform-only review notes or another tenant's private aliases, prices, or configuration. Excluded data and attachments must be listed with reasons so that “complete” is not misrepresented.

Download authorization is checked against the requesting tenant and current grant at download time. A copied artifact URL or object key must not provide access after expiry or revocation. Temporary archives, manifests, staging files, and generated attachment bundles inherit the source classification and must be encrypted, access-controlled, logged, and deleted after the defined expiry. Support staff cannot download an export merely because they can see its support case.

### 12.3 Backup and restore boundaries

Backups must preserve the relationships needed to prove ownership: tenant and branch IDs, composite references, membership/audit attribution, immutable source records, correction links, and the outbox/inbox/idempotency state needed to prevent duplicate effects after recovery. Backup metadata identifies environment, source database/schema version, time range or recovery point, encryption/key reference, integrity result, and custody/access history without exposing secrets.

A full-system restore is performed into an isolated recovery environment first. Before any restored system can serve tenants or publish events, verify schema/migration compatibility, database constraints, tenant/branch ownership invariants, record counts and hashes, object-to-metadata links, event/outbox cursors, provider callback deduplication, and absence of cross-environment credentials or active production side effects. Production cutover requires an explicit recovery decision and reconciliation of transactions/events that occurred after the recovery point.

A selective tenant or branch restore is a dependency-aware migration, not a raw row copy. It must enumerate required owned records, shared reference dependencies, attachments, audit/provenance, and event state; preserve stable IDs where safe; detect collisions; and prove that no unrelated tenant records are copied or exposed. If safe selective restoration cannot be demonstrated, restore the full environment into isolation and use a reviewed export/re-import procedure rather than improvising SQL row copying.

### 12.4 Restore and export acceptance tests

Before launch and after material changes to schema, storage, or export logic, demonstrate:
- tenant A exports only tenant A and its explicitly authorized branches, including when related rows or attachments contain foreign tenant IDs;
- branch-limited exports exclude other branches and do not leak their totals through aggregate summaries;
- an expired/revoked export grant and guessed object key cannot retrieve an artifact;
- import rejects cross-tenant and unauthorized cross-branch references at both validation and persistence boundaries;
- a full restore preserves all tenant boundaries, constraints, stable identifiers, audit attribution, and event/idempotency state;
- selective tenant restore detects dependency/collision cases and cannot introduce foreign tenant records;
- restored files match metadata, checksums, ownership, retention, and access policy;
- restored outbox/provider state cannot republish a completed external side effect or process an event under the wrong tenant;
- counts, hashes, financial/inventory control totals, and exception reports reconcile to the selected recovery point;
- every operator action and artifact access is attributable and recorded.

Record the tested dataset shape, source and destination versions, execution identity, recovery point, expected and observed counts/hashes, exceptions, duration, and reviewer. A provider “backup succeeded” status or a successful database start is not evidence that tenant isolation and business integrity survived restoration.
