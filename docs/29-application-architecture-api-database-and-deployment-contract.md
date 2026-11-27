# 29 — Application architecture, API, database, and deployment contract

**Status:** Draft — architecture contract; technology choices remain provisional  
**Purpose:** Translate the approved domain and cross-module rules into an implementable system architecture without prematurely selecting vendors or splitting the product into distributed services.

## 1. Authority and design posture

This specification complements, and does not override:
- Specification 07 — domain model and data ownership.
- Specification 08 — event-driven updates and offline requirements.
- Specification 10 — security, privacy, and threat model.
- Specification 11 — non-functional requirements and release gates.
- Specification 12 — functional product and production deployability contract.
- Specification 21 — external integrations and provider lifecycle.
- Specification 22 — privacy and personal-data lifecycle.
- Specification 24 — cross-module transaction and implementation contract.
- Specification 25 — first-release scope and launch decision framework.
- Specification 26 — migration, import/export, backup, and restoration.
- Specification 28 — production operations and service continuity.

If a conflict is found, resolve it in the authoritative documents before implementation. This document is a proposed architecture contract, not evidence that any technology has already been selected or deployed.

**Proposed starting posture:** one deployable application organized as enforceable domain modules, one authoritative transactional database for the initial release, and separately managed asynchronous workers only where operationally justified. A modular monolith is a hypothesis to validate against workflows, team capability, reliability, scale, and deployment constraints. Microservices are not a default goal.

## 2. System context and trust boundaries

The system contains these logical actors and boundaries:

1. **Web and mobile clients:** untrusted presentation and local-draft surfaces. They never own authoritative stock, payment, membership, permission, or financial state.
2. **Application/API boundary:** authenticates requests, derives actor identity, validates input, authorizes tenant/branch scope, invokes domain use cases, and returns stable response contracts.
3. **Domain modules:** own business rules and authoritative state transitions. A module may not directly mutate another module's private tables or bypass its command/query interface.
4. **Transactional database:** authoritative persistence for committed server-side facts, constraints, idempotency records, audit references, and transactional outbox entries.
5. **Background processing:** handles outbox publication, notifications, imports, exports, scheduled controls, and other retryable work. Workers are not an alternate authority for business rules.
6. **Realtime delivery:** transports authorized change notifications to clients. It is not a source of truth; clients reconcile against authorized API state or durable cursors.
7. **External providers:** payment, messaging, maps, storage, identity, and other third parties. Treat all callbacks, files, and responses as untrusted inputs until authenticated, validated, correlated, and deduplicated.
8. **Operations boundary:** monitoring, deployment, support, incident response, and backup access are privileged, purpose-limited, and audited.

No client, event channel, worker, support tool, or external provider may bypass server-side authorization and domain validation.

## 3. Domain module boundaries and dependency rules

The initial logical modules are:

| Module | Owns | Must not own |
|---|---|---|
| Identity and access | Account linkage, sessions, device trust, memberships, role/permission grants | Pharmacy stock, sales, or commercial entitlement |
| Organization | Tenant, legal-entity links, establishments, branches, invitations, ownership and lifecycle | Individual authentication secrets or pharmacy transactions |
| Catalogue | Canonical identity, source provenance, presentations, identifiers, correction governance | Tenant stock balances or sale history |
| Tenant catalogue and commercial configuration | Local item enablement, aliases, branch prices and policy snapshots | Canonical catalogue authority or finalized sale edits |
| Inventory | Batch/lot, movements, holds, counts, transfers, stock projections and valuation references | Payment confirmation or independent financial posting |
| Purchasing | Suppliers, purchase orders, receipts, supplier invoices and returns | Payment-provider truth or independent stock balance edits |
| Sales | Sale lifecycle, sale lines, price/tax snapshots, void and return linkage | Provider settlement truth or direct stock projection mutation |
| Payments | Attempts, provider references, confirmation evidence, refunds and reconciliation states | Sale creation or arbitrary sale status mutation |
| Finance | Cash sessions, financial postings, reconciliation, period controls and report definitions | Rewriting source transactions owned by other modules |
| Public discovery | Explicit publication settings, privacy-filtered projections, availability freshness and moderation | Direct access to private operational tables as a public read model |
| Platform operations | Onboarding decisions, service entitlements, support cases, privileged-access grants and platform policy | Tenant business facts |
| Audit and compliance services | Accountability records, policy evidence, retention controls and security events | Replacing domain-owned transaction ledgers |

### 3.1 Dependency direction

- Shared kernel content must remain small and stable: identifiers, money/currency primitives, quantity/unit primitives, time abstractions, tenant/branch scope types, and common result/error types.
- Domain modules expose explicit application interfaces (commands, queries, or published events).
- A module may depend on another module's public interface, never its private persistence implementation.
- Circular module dependencies are prohibited. If two modules require each other's internal details, redesign the ownership boundary or introduce a narrowly defined application-level orchestration.
- Cross-module workflows are coordinated by an application use case or explicit process manager. Do not hide multi-module side effects inside database triggers.
- Database constraints may enforce referential integrity and local invariants, but triggers must not become undocumented business workflows.
- Shared tables are not a shortcut for shared ownership. Every table has one owning module and documented read/write policy.

### 3.2 Modular monolith fitness checks

Before choosing a modular monolith, validate:
- transaction boundaries that must be atomic;
- expected deployment and team size;
- independent scaling needs of realtime, imports, and background work;
- operational ability to run, monitor, and recover multiple services;
- security boundaries and data isolation;
- failure blast radius and provider dependency behavior.

Split a component into a separate service only when a measurable requirement justifies independent deployment, scaling, security, or failure isolation and the cost of distributed transactions, observability, networking, and operations is accepted. A separate worker process may share the application code and database while remaining logically non-authoritative.

## 4. API surface and contract

The exact API style and transport remain open. Whether REST/JSON, typed RPC, or a combination is selected, the following semantics are mandatory.

### 4.1 API design rules

- Use explicit resource and command semantics. Queries do not mutate business state; commands identify the intended state transition.
- Publish an API contract (for example OpenAPI for HTTP endpoints) and validate implementation against it in CI.
- Version externally consumed contracts deliberately. Prefer additive compatible changes within a version; breaking changes require an explicit version, migration window, and consumer plan.
- Do not expose database table names, ORM models, internal exception messages, provider secrets, or unrestricted domain objects as public API contracts.
- Validate request shape, type, length, allowed values, relationships, and business preconditions on the server.
- Reject unknown or unauthorized fields rather than silently accepting privileged mass assignment.
- Derive actor identity from the authenticated session. Never trust client-supplied actor, tenant, branch, role, or permission claims.
- Tenant and branch scope are derived from authorized context and verified against every referenced resource.
- Responses include stable resource identifiers and explicit state. Avoid ambiguous booleans that collapse independent state machines.
- Use timezone-aware timestamps in a documented format. Preserve business-local date/time where the domain requires it.
- Represent money as exact decimal strings or integer minor units with explicit currency; document rounding and scale. Never use binary floating-point.
- Represent quantities with explicit unit identifiers and precision rules; do not assume all quantities are integers.
- Do not return sensitive fields merely because they exist on an internal model.

### 4.2 Command safety and idempotency

Every retryable state-changing command must define:
- a stable idempotency key supplied or generated by the client;
- key scope (authenticated actor, tenant, operation, and relevant resource);
- canonical request fingerprint and behavior if the same key is reused with a different payload;
- retention period at least as long as the maximum supported retry/reconciliation window;
- durable result reference and replay behavior;
- concurrency preconditions and conflict response.

A duplicate command with the same key and same semantic request returns the original outcome or a stable reference to it. Reuse with a different request body is rejected. Idempotency records must be committed atomically with the business transaction they protect.

An HTTP timeout or disconnected client means the outcome may be unknown. The client must query the command/resource by idempotency reference before issuing a new command. It must not assume that a timeout rolled back the transaction.

### 4.3 Concurrency and conditional updates

- Use aggregate versions, conditional updates, or equivalent compare-and-swap semantics for concurrent edits.
- Expose an explicit version or ETag where a client needs to detect stale updates.
- A stale write receives a conflict response with enough safe information to refresh and resolve; it must not overwrite newer authoritative state.
- Critical stock and money invariants are protected by database transactions, uniqueness/foreign-key/check constraints, and appropriate row-level or equivalent locking.
- Define lock ordering for multi-resource transactions to reduce deadlocks.
- Deadlock and serialization failures may be retried only when the operation is safe and idempotent; retries are bounded and observable.

### 4.4 Query behavior

- List endpoints use bounded pagination. Prefer opaque cursor pagination for changing/high-volume datasets; offset pagination may be used for small stable reference lists with explicit limits.
- Every list query requires authorized tenant/branch scope where applicable.
- Filters and sorting are allow-listed and validated. Never interpolate untrusted SQL identifiers.
- Search results disclose their freshness/source limitations where relevant.
- Exports are asynchronous for large datasets, permission-checked at request and execution time, and produce an auditable completion record.
- Avoid N+1 query patterns and unbounded nested response graphs.
- Define maximum page size, request body size, query complexity, and export size based on measured usage.

### 4.5 Standard errors and request tracing

Every API error must provide:
- stable machine-readable error code;
- safe human-readable message suitable for localization;
- correlation/request ID;
- field-level validation details when appropriate;
- retryability classification where meaningful;
- current resource version/state only when the caller is authorized to see it.

Do not expose stack traces, SQL, secrets, personal data, provider payloads, or internal hostnames. Distinguish validation, unauthenticated, forbidden, not found, conflict, rate limit, dependency unavailable, and unexpected server failure. Avoid leaking resource existence across tenant boundaries.

Every request has a server-generated correlation ID propagated to logs, traces, background jobs, outbox delivery, and provider calls. A client-provided trace value may be retained only as untrusted correlation metadata, not used as an authorization input.

### 4.6 Rate limits and abuse controls

- Apply configurable limits by account, tenant, endpoint class, and source-risk signal.
- Use stricter limits for authentication, invitation, search, public enquiry, export, and expensive operations.
- Return a stable rate-limit error and retry guidance without exposing detection rules.
- Limits must not create a bypass around tenant isolation or prevent authorized emergency recovery.
- Record aggregate abuse signals without collecting unnecessary sensitive content.

### 4.7 Required command and query inventory

The following is the minimum application-operation inventory implied by the current first-release candidate and module specifications. Names are logical operation identifiers, not final HTTP routes or a claim that every operation is approved for launch. The API contract/OpenAPI document must define the exact request, response, authorization, idempotency, concurrency, error, and audit behavior for every enabled operation.

| Domain | Commands (state-changing) | Queries (read-only) |
|---|---|---|
| Identity and organization | Register/sign in; create/update organization; create/update branch; invite/revoke membership; assign/revoke role; transfer ownership; register/revoke device; change organization lifecycle state | Current account/session; organizations and memberships; branches; invitation state; effective permissions; device/session state |
| Catalogue | Create/update/disable tenant item; propose/review/approve canonical correction; propose/review barcode mapping; review regulatory classification; publish approved catalogue projection | Canonical search/detail; tenant catalogue search/detail; product presentation and unit conversion; barcode lookup; correction/evidence history |
| Inventory | Create/post goods receipt; open/record/post count; request/approve adjustment; create/dispatch/receive transfer; place/release hold; request/approve disposal; process customer return disposition; process recall decision | Stock position by branch/batch/status; movement history; receipt/transfer/count detail; expiry/recall queue; discrepancy and reconciliation report |
| Purchasing | Create/update/block supplier; create/submit/approve requisition; record quotation; create/submit/approve/issue/cancel purchase order; create/accept receipt; record/match/resolve exception/approve supplier invoice; create/approve supplier return; initiate/confirm/reconcile supplier payment | Supplier list/detail; requisition and quotation status; purchase-order status; receipt/invoice matching; supplier balances; payment/reconciliation status |
| Sales and payments | Create/update sale draft; validate sale; commit sale; request/approve void; initiate payment; confirm permitted manual payment; process authenticated provider callback; create/submit/review customer return; create/submit/review customer return; request/approve refund; reissue receipt; open/count/close cash session | Sale and line detail; sale history; payment attempt/status; receipt; return/refund status; cash session and variance |
| Finance | Create cash movement; create/approve expense; post/reverse journal; reconcile source; resolve reconciliation exception; close accounting period; initiate authorized export | Cashbook; finance summary; journal and posting history; payment/settlement reconciliation; expense list; period status; finance reports |
| Public discovery | Configure/publish/pause profile; configure/publish/pause catalogue; confirm/publish/pause availability; receive enquiry; submit report; triage/suppress content; record moderation decision/appeal; submit/review evidence; publish verification claim | Public pharmacy and catalogue search; public profile; availability state and freshness; enquiry inbox; moderation queue; verification case |
| Platform operations | Create support case; approve/revoke scoped support access; change service entitlement; suspend/reinstate capability under policy; record platform security action | Support case; access-grant history; service health and incident view; tenant entitlement and restriction history |

#### Command rules

- A command is named for one intended business transition. Do not expose a generic `update` endpoint that can mutate arbitrary lifecycle fields.
- Draft creation/editing and final commitment are separate operations wherever the module defines approval or commit states.
- Approval commands carry the approver identity from the authenticated session and enforce separation of duties server-side.
- A command that crosses module boundaries is orchestrated by an application use case; each domain module retains authority over its own facts.
- External callbacks are ingress commands into a provider adapter. They must be authenticated, schema-validated, correlated to a known attempt, deduplicated, and recorded as untrusted provider evidence before any domain transition is accepted.
- Export requests are commands that create an asynchronous export job. Download is a separately authorized query against a completed, unexpired artifact.
- Every command must declare whether it is idempotent, its idempotency-key scope, resource-version precondition, authorization permissions, audit event, emitted domain events, and retry/unknown-outcome behavior.
- Queries must declare tenant/branch scope, sensitive fields, freshness, pagination, sorting/filter allowlists, and whether the result is a source record or a derived projection.

#### Event catalogue requirement

Before implementation, create a versioned event registry with one entry per event type. Each entry must define producer/owning module, triggering committed transition, schema version, stable event ID, aggregate ID/version, tenant/branch scope, payload classification, consumers, ordering assumptions, retention, replay policy, and compatibility/deprecation plan.

The event names already proposed in module specifications—such as `GoodsReceiptAccepted`, `PurchaseOrderApproved`, `PublicAvailabilityUpdated`, and `PublicAvailabilityBecameStale`—are candidate names, not an approved complete catalogue. No consumer may infer a business transition from a generic notification such as `RecordUpdated`.

#### Operation coverage gate

For every enabled first-release workflow, maintain traceability from:
1. user-visible action and workflow step;
2. logical command/query;
3. API contract operation and schema;
4. permission identifier and scope;
5. authoritative module and persistence transaction;
6. emitted event(s) and consumer/projection;
7. failure, retry, and recovery behavior;
8. automated contract, authorization, and end-to-end tests.

An operation without this traceability is not implementation-ready. A UI control must not be built as a working feature until its operation contract and backend behavior exist.

## 4.8 Workflow transition registry alignment

The command/query inventory in this specification is complemented by [Specification 30](30-workflow-state-transition-and-recovery-contract.md), which defines candidate aggregate transitions, guards, transaction ordering, event semantics, and failure recovery. API contracts must represent each approved transition as an explicit command with expected version, server-derived actor/scope, canonical permission, idempotency behavior, and stable result. Do not expose generic status mutation.

## 5. Database architecture and integrity contract

The database technology and hosting provider remain open. A relational transactional database, with PostgreSQL as a candidate, is the leading hypothesis because the domain requires relational integrity, multi-row transactions, uniqueness, concurrency control, and reporting. Validate it against hosting, operations, backup, regional availability, cost, and team expertise before selection.

### 5.1 Authoritative persistence

- One authoritative write owner per business fact, as defined in Specifications 07 and 24.
- No critical business fact may exist only in browser storage, a cache, a search index, an event broker, or an analytics store.
- Caches and read models are disposable projections with explicit invalidation/rebuild/reconciliation behavior.
- Critical transaction records and their outbox events commit in one database transaction.
- Do not use cross-database distributed transactions for first-release workflows unless a documented requirement makes them unavoidable.
- Use database constraints as a final integrity boundary, not only application validation.

### 5.2 Tenant isolation

The tenant security boundary must be enforced in every query and write path, not inferred from UI navigation.

The implementation decision between shared tables with tenant keys, schema-per-tenant, or database-per-tenant remains open pending operational and security analysis. Whichever model is selected:
- every tenant-owned record has an unambiguous tenant ownership path;
- branch-owned records have validated branch-to-tenant consistency;
- foreign keys and unique constraints include tenant scope where needed to prevent cross-tenant references;
- background jobs, exports, imports, event delivery, and support tools enforce the same scope;
- cross-tenant administrative access is a separate privileged workflow with purpose-bound grants and audit;
- automated tests attempt guessed identifiers, swapped tenant IDs, nested-resource attacks, and cross-tenant joins.

Database row-level security may be an additional defense if its operational and connection-pooling behavior is verified. It must not replace application authorization or tests.

### 5.3 Data types and schema rules

- Use exact decimal or integer-minor-unit representation for money, with explicit currency and documented scale.
- Use integer/decimal quantities with explicit unit and precision rules.
- Use timezone-aware instants for events and timestamps; store business dates separately when required.
- Use stable opaque identifiers for externally addressable records; do not expose sequential database keys as authorization.
- Store status as constrained enumerated values or reference records with explicit transition rules, not arbitrary strings.
- Define nullability intentionally. Unknown, not applicable, not collected, and zero are different states.
- Avoid overloaded JSON blobs for core business facts. JSON is appropriate for versioned provider payload evidence, extensible metadata with schema validation, and immutable snapshots where justified.
- Use explicit created/updated/committed timestamps and actor references; finalized records also preserve source and correction links.
- Define deletion behavior per entity: hard delete, soft delete, anonymize, or retain under a documented legal/business basis. Soft delete is not a substitute for retention policy.
- Index tenant/branch scope and high-value query paths; validate indexes using real query plans and representative data. Avoid speculative indexes on every field.

### 5.4 Ledger and projection separation

- Stock movements, finalized sales, payment evidence, and financial postings are durable source records with append-only correction semantics.
- Current stock, availability, dashboards, and report aggregates are projections that can be reconciled or rebuilt.
- A projection update failure never rolls back or rewrites a committed source fact; it creates an observable lag/recovery task.
- Projection rebuilds are versioned, bounded, idempotent, and safe to run without duplicating business effects.
- A projection must declare its source events/tables, transformation version, last processed cursor, and reconciliation method.
- Reports must state whether they are live, delayed, or as-of a defined cutoff.

### 5.5 Migrations and schema evolution

- Every schema change is version-controlled, reviewed, deterministic, and executed through a migration mechanism.
- Production schema changes are never performed manually as an undocumented fix.
- Migrations must declare preconditions, expected duration/locking impact, data transformation, validation, and recovery/forward-fix strategy.
- Prefer expand-migrate-contract for changes that must be deployed across rolling versions; do not retain compatibility code indefinitely. Remove transitional structures after all data is migrated and verified.
- Destructive changes require verified backup, row/count/checksum reconciliation, and explicit approval.
- A migration must be safe to retry or detect partial completion. Record migration version and outcome.
- Application startup must not run unbounded or destructive migrations automatically.
- Large backfills run as observable, resumable jobs with rate limits and checkpoints.

## 6. Transactional outbox, event delivery, and realtime

- The domain transaction writes its authoritative facts and outbox rows atomically.
- Outbox publishers claim work safely across worker instances, publish with bounded retries, and mark delivery only after broker/transport acknowledgement.
- A crash after publish but before marking delivered may cause duplicate delivery; consumers deduplicate by stable event ID.
- Consumers record inbox/deduplication state atomically with their local side effect where possible.
- Event payloads are versioned, minimal, and free of secrets and unnecessary personal/health data.
- Aggregate ID/version supports ordering checks for a single aggregate; do not promise global ordering.
- Poison messages move to a controlled dead-letter state with reason, owner, retry policy, and replay procedure. Never delete them merely to clear an alert.
- Replay is an operational action with authorization, bounded scope, audit, and idempotency safeguards.
- Realtime subscriptions are authenticated, tenant/branch scoped, re-authorized on reconnect, and revocable.
- The transport is replaceable. WebSocket, server-sent events, managed realtime, or another push mechanism may be evaluated; select based on client support, reconnect semantics, scale, cost, and operations.
- Mobile background push is a notification hint, not a guaranteed delivery channel or a source of sensitive data.
- On reconnect or cursor gap, clients resume from a durable cursor or fetch an authoritative snapshot. Do not fall back to frequent short-interval polling.
- Scheduled jobs are appropriate for expiry, retention, reconciliation, reminders, and integrity checks; they are not substitutes for immediate event propagation.

## 7. Offline synchronization and local storage boundary

- Local persistence is limited to approved cached reads, drafts, and explicitly approved queued commands.
- Local data is encrypted using platform-appropriate secure storage, minimized, tenant-partitioned, and cleared on sign-out/device revocation according to the approved policy.
- Local command records include command ID, idempotency key, schema version, tenant/branch, actor/device, creation time, base version, and queue expiry.
- The server revalidates current identity, membership, permission, credential/action gate, resource state, and business rules at synchronization.
- A queued command can be rejected after permissions or business state change; the UI must preserve the user's draft and explain the result.
- A local optimistic update must be visibly provisional until server acknowledgement.
- No offline finalization of sensitive stock, financial, payment, refund, approval, or regulated actions unless a specific capability has a separately approved risk model and reconciliation policy.
- Conflict resolution must preserve both the user's submitted intent and authoritative server state. No silent last-write-wins for stock, money, permissions, or regulated records.

## 8. File and document storage

The object-storage provider remains open. The design must:
- keep file bytes outside relational rows except for small values where justified;
- store metadata in the database: owner tenant, uploader, purpose, classification, MIME type, size, checksum, object key, upload state, retention class, and malware-scan status where relevant;
- use opaque, non-guessable object keys and private-by-default access;
- issue short-lived, narrowly scoped upload/download grants after authorization;
- validate file size, content type, file signature, and allowed extension; do not trust browser MIME declarations;
- scan uploaded files where risk warrants and quarantine untrusted content before use;
- prevent public access to private documents through predictable URLs or misconfigured buckets;
- encrypt in transit and at rest, manage keys/secrets separately, and define deletion/retention behavior;
- make exports and backups access-controlled, integrity-checked, time-bounded, and audited;
- avoid placing prescriptions, patient records, or other high-sensitivity documents in the first release unless separately approved.

## 9. Search, reporting, and read models

- Transactional database records remain authoritative even when search or analytics uses a separate index/store.
- Search indexing is asynchronous and may lag; expose freshness or last-updated context where the lag matters.
- Search results must be tenant-scoped for private queries and privacy-filtered for public discovery.
- Public availability must derive from explicitly publishable source data and an expiry/freshness deadline; stale data transitions to unknown/stale rather than remaining falsely available.
- Search indexing must not leak private aliases, prices, quantities, supplier data, customer information, or internal identifiers.
- Reporting definitions specify source facts, inclusion/exclusion rules, timezone, currency, branch scope, rounding, and as-of semantics.
- Financial and inventory reports must reconcile to their source ledgers. Aggregate caches must be explainable and rebuildable.
- A report export is generated from an authorized, bounded snapshot and records the requester, scope, filters, schema version, and completion outcome.

## 10. Deployment topology and environment separation

The initial topology should be the smallest topology that satisfies reliability, security, recovery, and performance requirements. Candidate components:
- client delivery (web application and, if approved, mobile clients);
- application/API runtime;
- background worker runtime;
- transactional database;
- object storage;
- event/realtime transport;
- monitoring, logs, traces, and alerting;
- external provider connections.

These may be managed services or self-operated components. Provider selection must be documented through Specification 21 and operationally validated under Specification 28.

### 10.1 Environments

Maintain separate development, test/CI, staging, and production environments with:
- isolated databases, object stores, credentials, provider accounts, and callback URLs;
- no production personal or pharmacy data in development/test unless a lawful, minimized, access-controlled dataset is explicitly approved;
- synthetic fixtures clearly labeled and incapable of being mistaken for live data;
- staging configured close enough to production to validate migrations, deployment, auth, event delivery, and recovery;
- production secrets never copied into source control, build logs, client bundles, or test artifacts.

### 10.2 Deployment and release

- Build immutable, versioned artifacts from reviewed source commits.
- CI must run formatting/linting, static analysis, unit tests, integration tests, contract tests, migration checks, dependency/security scanning, and relevant end-to-end acceptance tests.
- Deployments must be traceable to commit, artifact digest, configuration version, migration version, and operator/automation identity.
- Use health/readiness checks that validate actual dependencies without creating real business transactions.
- Deployment should support controlled rollout and a verified rollback or forward-fix strategy.
- Database migrations must be coordinated with application rollout and must not assume application rollback reverses committed data.
- Feature flags may disable incomplete or risky capabilities, but must be server-enforced and audited for regulated/financial behavior. Flags are not a substitute for removing obsolete code.
- No deployment is considered successful until synthetic checks and critical workflow smoke tests pass and monitoring shows stable service behavior.
- A failed deployment must not silently leave a mixed version that corrupts transactions or event contracts.

### 10.3 Secrets and configuration

- Separate secrets from non-secret configuration.
- Use a managed secret store or equivalent controlled mechanism; restrict read access by service identity and environment.
- Rotate credentials and document revocation/rotation procedures.
- Validate required configuration at startup and fail closed for missing security-critical settings.
- Never log secret values. Redact authorization headers, tokens, provider signatures, and sensitive payload fields.
- Client-side configuration contains only values safe for public disclosure.

### 10.4 Capacity and scaling

- Measure real request latency, database utilization, connection pool saturation, queue age, event lag, object storage, and external-provider latency before scaling decisions.
- Bound database connection pools across all application and worker instances.
- Apply backpressure to expensive jobs and provider calls; do not let a queue grow without alerting and capacity policy.
- Scale stateless API and worker processes independently where useful; preserve single-writer/locking guarantees for critical operations.
- Define capacity tests from pilot-derived transaction volumes and realistic data sizes.
- Avoid introducing a cache, broker, search cluster, or microservice without an explicit requirement and operating owner.

## 11. Security, privacy, and operational controls

- Authenticate every non-public API and subscription; authorize each operation and resource at execution time.
- Use least privilege for database roles, workers, service identities, deployment automation, and support personnel.
- Separate migration/admin credentials from normal runtime credentials.
- Apply secure headers, TLS, CSRF protection where cookie-based authentication is used, secure session handling, and appropriate password/session controls.
- Protect against injection, broken object-level authorization, mass assignment, SSRF, unsafe file upload, replay, request forgery, and resource exhaustion.
- Encrypt sensitive data in transit and at rest; document key ownership and rotation.
- Keep personal-data processing purpose-limited and consistent with Specification 22.
- Audit privileged access, exports, permission changes, credential decisions, sensitive corrections, and operational replay.
- Logs, traces, and metrics must not contain passwords, access tokens, payment secrets, unnecessary personal data, or health data.
- Security controls and incidents are managed under Specifications 10 and 28.

## 12. Testing and architecture fitness

Required test layers:
1. **Domain unit tests:** state transitions, invariants, validation, rounding, unit conversion, and permission rules.
2. **Persistence tests:** constraints, transaction rollback, uniqueness, tenant scope, concurrency, and migration behavior using the actual database engine.
3. **API contract tests:** schema validation, stable errors, idempotency, authorization, pagination, and version compatibility.
4. **Module integration tests:** public interfaces, event publication, and cross-module orchestration.
5. **Event tests:** duplicate, delayed, reordered, missed, replayed, malformed, and poison messages.
6. **Security tests:** cross-tenant and cross-branch access, guessed IDs, revoked sessions, privilege escalation, upload access, support grants, and export scope.
7. **End-to-end tests:** every approved first-release journey from Specification 25.
8. **Resilience tests:** provider timeout, database connection loss, worker crash, event transport outage, deploy interruption, and restore.
9. **Performance tests:** representative data volumes, concurrent users, inventory queries, reporting, imports, and event fan-out.
10. **Operational drills:** incident response, secret rotation, dead-letter replay, backup restore, and tenant export.

Architecture fitness checks should run in CI where feasible:
- domain modules do not import another module's persistence internals;
- no direct cross-module table writes outside approved interfaces;
- all state-changing endpoints declare idempotency and authorization behavior;
- every committed domain event has a schema/version and outbox path;
- every tenant-owned query path has scope enforcement and negative tests;
- migrations are ordered and validated;
- no production build includes test fixtures, mock provider defaults, or development credentials.

## 13. Technology decision register

No technology below is approved solely by appearing in this document.

| Decision | Candidate options to evaluate | Required evidence before selection |
|---|---|---|
| Application architecture | Modular monolith; selectively separated services | Transaction boundaries, team capability, failure isolation, scale and operations |
| Backend runtime/framework | Supported mainstream runtime/framework options | Security support, maintainability, ecosystem, hiring, testability and deployment |
| API style | REST/JSON; typed RPC; hybrid | Client needs, contract tooling, streaming needs, versioning and interoperability |
| Database | PostgreSQL candidate; other relational systems | Transaction/concurrency behavior, constraints, hosting, backups, expertise and cost |
| Tenant isolation | Shared tables with tenant keys; schema per tenant; database per tenant | Threat model, operational complexity, restore/export, scale and legal requirements |
| Realtime transport | WebSocket; server-sent events; managed realtime | Web/mobile support, reconnect/replay, authorization, cost and provider exit |
| Job/event transport | Database-backed outbox worker; managed queue/broker | Throughput, delivery semantics, operational ownership and recovery |
| Object storage | S3-compatible or managed object storage | Region, access control, encryption, lifecycle, egress, backup and provider exit |
| Search | Database search initially; dedicated search later | Catalogue size, language support, relevance, privacy, freshness and operating cost |
| Hosting/region | Qualified providers and regions | Data protection, residency/transfer review, availability, latency, support, recovery and cost |
| Client architecture | Responsive web/PWA; native/cross-platform mobile | Device constraints, offline needs, notifications, accessibility and support burden |

A decision record must include context, options, evidence, trade-offs, decision owner, date, consequences, reversibility, and review trigger. Legal/data-location questions require qualified Ethiopian review; vendor marketing claims are not sufficient evidence.

## 14. Implementation sequence after specification approval

Implementation should proceed as a vertical slice with production foundations, not as disconnected UI pages:

1. Repository and CI baseline, dependency policy, secure configuration, environments, and reproducible builds.
2. Identity, tenant/organization/branch scope, membership, permission enforcement, and audit.
3. Database schema/migrations, transaction conventions, idempotency, and outbox.
4. Catalogue identity and tenant-local catalogue setup.
5. Supplier, purchase order, goods receipt, batch capture, and inventory ledger.
6. Inventory queries, counts, holds, corrections, and approved transfer scope.
7. Approved sales/POS and payment state workflow only after legal/accounting gates.
8. Finance/cash reconciliation and traceable operational reports.
9. Event subscriptions, reconnect/replay, notifications, and approved offline drafts.
10. Import/export, backup/restore, support operations, monitoring, and incident drills.
11. Representative user acceptance and controlled pilot deployment.

The sequence may change when dependencies or field evidence require it, but no stage may bypass its security, data, acceptance, and operational gates. Public discovery and commerce remain independently opt-in and are not prerequisites for management-only operation.

## 15. Open decisions and release blockers

The following remain unresolved and must not be silently selected by developers:
- final backend runtime/framework and client stack;
- hosting provider, deployment region, data residency/transfer posture, and service architecture;
- database provider and tenant-isolation strategy;
- API style and authentication/session mechanism;
- realtime transport, durable cursor strategy, and broker/queue need;
- object storage and file scanning provider;
- search/indexing strategy and Ethiopian language requirements;
- pilot workload, availability targets, latency targets, and capacity assumptions;
- monitoring/telemetry provider and data retention;
- actual hosting, messaging, storage, and provider costs;
- migration and deployment automation choices.

These are not all blockers to further product discovery. They are blockers to locking the corresponding architecture or enabling affected production capabilities. Resolve them using workflow evidence, security/privacy review, provider due diligence, operational exercises, and measurable pilot needs.

## 16. Architecture readiness gate

Architecture is ready for implementation only when:
- module ownership and dependency rules are accepted and testable;
- each included workflow maps to commands, queries, authoritative records, transactions, events, and permissions;
- tenant isolation is selected and adversarially tested;
- API contracts and idempotency/concurrency semantics are defined for included capabilities;
- database types, constraints, migration strategy, and projection ownership are defined;
- event delivery, replay, realtime authorization, and offline boundaries are defined;
- deployment environments, secrets, CI/CD, monitoring, backup/restore, and incident ownership are operationally testable;
- technology choices are justified against cost, expertise, data protection, provider exit, and failure modes;
- unresolved items are explicitly excluded, safely deferred, or assigned a decision owner and gate;
- the approved first-release journeys pass end-to-end acceptance in a production-like environment.

**No demo substitution:** a successful local screen render, mocked API, synthetic provider callback, or green unit-test suite alone does not satisfy this gate. Production readiness requires the complete enabled workflow and operational evidence described in Specifications 11, 12, 25, 26, and 28.
