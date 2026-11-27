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
| Identity and organization | Register/sign in; create/update organization; create/update branch; invite/accept/restrict/revoke membership; assign/revoke role; transfer ownership; register/revoke device; change organization lifecycle state | Current account/session; organizations and memberships; branches; invitation state; effective permissions; device/session state |
| Catalogue | Create/update/disable tenant item; propose/review/approve canonical correction; propose/review barcode mapping; review regulatory classification; publish approved catalogue projection | Canonical search/detail; tenant catalogue search/detail; product presentation and unit conversion; barcode lookup; correction/evidence history |
| Inventory | Create/post goods receipt; open/record/post count; request/approve adjustment and internally commit the approved movement; create/dispatch/receive/cancel transfer; place/release hold; request/approve disposal; process customer return disposition; process recall decision | Stock position by branch/batch/status; movement history; receipt/transfer/count detail; expiry/recall queue; discrepancy and reconciliation report |
| Purchasing | Create/update/block supplier; create/submit/approve requisition; record quotation; create/submit/approve/issue/cancel/close purchase order; create/accept receipt; record/match/resolve exception/approve supplier invoice; create/approve supplier return; initiate/confirm/reconcile supplier payment | Supplier list/detail; requisition and quotation status; purchase-order status; receipt/invoice matching; supplier balances; payment/reconciliation status |
| Sales and payments | Create/update sale draft; validate sale; commit sale; request/approve void (conditional on approved void thresholds, separation, and accounting treatment); initiate payment; confirm permitted manual payment; process authenticated provider callback; create/submit/review customer return; request/approve refund; reissue receipt; open/count/close cash session | Sale and line detail; sale history; payment attempt/status; receipt; return/refund status; cash session and variance |
| Finance | Create cash movement; create/approve expense; post/reverse journal; reconcile source; resolve reconciliation exception; close accounting period; initiate authorized export | Cashbook; finance summary; journal and posting history; payment/settlement reconciliation; expense list; period status; finance reports |
| Public discovery | Configure/publish/pause profile; configure/publish/pause catalogue; confirm/publish/pause availability; receive enquiry; submit report; triage/suppress content; record moderation decision/appeal; submit/review evidence; publish verification claim | Public pharmacy and catalogue search; public profile; availability state and freshness; enquiry inbox; moderation queue; verification case |
| Platform operations | Create support case; approve/revoke scoped support access; manage service entitlement; restrict/reinstate organization capability under policy; record platform security action | Support case; access-grant history; service health and incident view; tenant entitlement and restriction history |

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

The candidate event taxonomy is maintained in [Specification 08](08-event-driven-and-offline-requirements.md), Section 3. Before implementation, create a versioned event registry with one entry per event type. Each entry must define producer/owning module, triggering committed transition, schema version, stable event ID, aggregate ID/version, tenant/branch scope, payload classification, consumers, ordering assumptions, retention, replay policy, and compatibility/deprecation plan.

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



## 4.9 First-release workflow-to-operation traceability matrix

This is a logical traceability baseline for the proposed private-operations release in Specification 25. It is **not an approved launch scope, final route list, or final event registry**. Operation IDs are stable design identifiers; HTTP paths and schemas are defined later in the API contract. Candidate event names must be registered and versioned before implementation. A workflow not approved for the release must not be exposed merely because an operation appears here.

| Workflow / step | Logical command or query ID | Required permission(s) | Authoritative owner and transaction boundary | Candidate committed event(s) | Minimum acceptance-test IDs |
|---|---|---|---|---|---|
| Create private organization workspace | `organization.create` | `organization.manage` | Identity/Organization; create tenant, initial owner membership, and bootstrap grants atomically | `OrganizationCreated`, `MembershipActivated` | ORG-001, ORG-002 |
| Create branch | `branch.create` | `branch.manage` | Organization; branch belongs to tenant; no legal operating claim inferred | `BranchCreated` | ORG-003, TENANT-001 |
| Read organization, branch, and effective access | `organization.detail`, `branch.list`, `branch.detail`, `membership.effective_permissions` | `organization.read`, `branch.read` | Identity/Organization; tenant and assigned-branch scope; effective permissions are calculated server-side and expose no secret policy data | No business event; access/projection telemetry only | ORG-QRY-001 through ORG-QRY-004 |
| Register or revoke tenant device | `device.register`, `device.revoke` | `device.manage` | Identity/Organization; device ownership, session revocation, and lost-device recovery are audited | `DeviceRegistered`, `DeviceRevoked` | DEVICE-001 through DEVICE-004 |
| Invite and accept member | `membership.invitation.create`, `membership.invitation.accept` | `membership.invite`, `membership.accept` | Identity/Organization; invitation token single-use; acceptance bound to intended account | `MembershipInvitationCreated`, `MembershipActivated` | ORG-004, AUTH-001, AUTH-002 |
| Assign and revoke member roles | `role.assign`, `role.revoke` | `role.assign`, `role.revoke` | Identity/Organization; explicit bounded grant allowlist, assigner delegation, tenant/branch scope, no self-escalation, and immediate effective-permission re-evaluation | `RoleGrantChanged` | AUTH-004 through AUTH-008 |
| Restrict or revoke membership | `membership.restrict`, `membership.revoke` | `membership.restrict`, `membership.revoke` | Identity/Organization; session, subscription, and queued-command invalidation is coordinated and versioned | `MembershipRestricted`, `MembershipRevoked` | AUTH-003, OFFLINE-001 |
| Read branch stock position and movement history | `inventory.stock_position`, `inventory.movement_history` | `inventory.read` | Inventory projections; branch/batch/status scope and as-of/freshness metadata | No business event; projection freshness/lag telemetry only | INV-QRY-001 through INV-QRY-005 |
| Read catalogue | `catalogue.search`, `catalogue.item.detail` | `catalogue.search`, `catalogue.read` | Catalogue; tenant and shared catalogue visibility filtered by grants and provenance | No business event; query/projection telemetry only | CAT-QRY-001 through CAT-QRY-003 |
| Propose canonical catalogue correction | `catalogue.correction.propose` | `catalogue_correction.propose` | Catalogue; proposal preserves source evidence and cannot mutate canonical data or approve its own proposal | `CatalogueCorrectionProposed` | CAT-CORR-001 through CAT-CORR-004 |
| Read supplier and purchasing records | `supplier.list`, `purchase_order.detail`, `goods_receipt.detail`, `supplier_invoice.detail`, `supplier_payment.status` | `supplier.read`, `purchase_order.read`, `goods_receipt.read`, `supplier_invoice.read`, `supplier_payment.read` | Purchasing/Finance; invoice/payment fields separately classified and authorized | No business event; query/projection telemetry only | PUR-QRY-001 through PUR-QRY-006 |
| Read requisition and quotation records (conditional) | `requisition.detail`, `quotation.detail` | `requisition.read`, `quotation.read` | Purchasing; enable only if the pilot includes requisition/quotation workflows; otherwise disable both queries | No business event; query/projection telemetry only | PUR-REQ-QRY-001 through PUR-REQ-QRY-003 |
| Read sales, payment, return, and receipt records | `sale.detail`, `payment.status`, `customer_return.status`, `receipt.detail` | `sale.read`, `payment.read`, `customer_return.read`, `receipt.read` | Sales/Payments; payment and return visibility follows original sale scope and sensitive-field rules | No business event; query/projection telemetry only | POS-QRY-001 through POS-QRY-006 |
| Create and maintain supplier records | `supplier.create`, `supplier.update`, `supplier.block` | `supplier.create`, `supplier.update`, `supplier.block` | Purchasing; duplicate detection and historical reference preservation | `SupplierChanged` | SUP-001 through SUP-005 |
| Create requisitions and record quotations (conditional) | `requisition.create`, `requisition.submit`, `requisition.approve`, `quotation.record` | `requisition.create`, `requisition.submit`, `requisition.approve`, `quotation.record` | Purchasing; include only if the pilot requires internal requisition/quote comparison; otherwise disable | `RequisitionSubmitted`, `RequisitionApproved`, `QuotationRecorded` | PUR-REQ-001 through PUR-REQ-006 |
| Record and review supplier invoice (conditional) | `supplier_invoice.record`, `supplier_invoice.match`, `supplier_invoice.exception.resolve`, `supplier_invoice.dispute`, `supplier_invoice.approve` | `supplier_invoice.record`, `supplier_invoice.match`, `supplier_invoice.exception.resolve`, `supplier_invoice.dispute`, `supplier_invoice.approve` | Purchasing + Finance; enable only after invoice matching, payable treatment, and accountant review are approved | `SupplierInvoiceRecorded`, `SupplierInvoiceMatched`, `SupplierInvoiceExceptionResolved`, `SupplierInvoiceDisputed`, `SupplierInvoiceApproved` | INV-AP-001 through INV-AP-008 |
| Maintain tenant catalogue | `catalogue.item.create`, `catalogue.item.update`, `catalogue.item.disable` | `tenant_catalogue.create`, `tenant_catalogue.update`, `tenant_catalogue.disable` | Catalogue; tenant-local identity, source provenance, and versioned presentation/unit mapping | `TenantCatalogueItemCreated`, `TenantCatalogueItemUpdated`, `TenantCatalogueItemDisabled` | CAT-001 through CAT-006 |
| Create and approve purchase order | `purchase_order.create`, `purchase_order.submit`, `purchase_order.approve`, `purchase_order.issue`, `purchase_order.cancel`, `purchase_order.close` | `purchase_order.create`, `purchase_order.submit`, `purchase_order.approve`, `purchase_order.issue`, `purchase_order.cancel`, `purchase_order.close` | Purchasing; each approved/issued version is immutable; order approval never creates stock | `PurchaseOrderSubmitted`, `PurchaseOrderApproved`, `PurchaseOrderIssued`, `PurchaseOrderCancelled`, `PurchaseOrderClosed` | PUR-001 through PUR-008 |
| Record and accept goods receipt | `goods_receipt.create`, `goods_receipt.accept`, `goods_receipt.commit` (internal continuation after all lines are resolved; not a client-invoked or separately role-granted action) | `goods_receipt.create` for draft creation; `goods_receipt.accept` plus `inventory.receive` for authorized acceptance/receipt posting | Purchasing + Inventory orchestration; the server invokes the internal commit only after accepted quantities, batch evidence, and policy checks are complete; receipt result, stock movements, audit, and outbox commit atomically | `GoodsReceiptAccepted`, `StockMovementCommitted` | REC-001 through REC-008, INV-001 |
| Count stock and post adjustment | `inventory.count.record`, `inventory.adjustment.request`, `inventory.adjustment.approve`, `inventory.adjustment.commit` (internal continuation only after approval; not a client-invoked or separately role-granted action) | `inventory.count`, `inventory.adjust.request`, `inventory.adjust.approve` | Inventory; count snapshot/cutoff preserved; adjustment is a linked movement, not a balance overwrite | `StockCountRecorded`, `StockAdjustmentRequested`, `StockMovementCommitted` | INV-002 through INV-009 |
| Transfer stock between branches (conditional on multi-branch pilot approval) | `inventory.transfer.create`, `inventory.transfer.dispatch`, `inventory.transfer.receive`, `inventory.transfer.cancel` | `inventory.transfer.create`, `inventory.transfer.dispatch`, `inventory.transfer.receive`, `inventory.transfer.cancel` | Inventory; dispatch creates in-transit quantity; destination receipt is a separate leg | `StockTransferDispatched`, `StockTransferReceived`, `StockTransferCancelled`, `StockMovementCommitted` | XFER-001 through XFER-007 |
| Hold and release stock; disposal only if separately included | `inventory.hold.apply`, `inventory.hold.release`, `inventory.disposal.request`, `inventory.disposal.approve` | `inventory.hold`, `inventory.release`, `inventory.disposal.request`, `inventory.disposal.approve` | Inventory; eligibility recalculated against expiry, recall, quality, and evidence rules; disposal actions remain disabled unless separately approved in Specification 25 | `StockHoldApplied`, `StockHoldReleased`, `StockMovementCommitted` | INV-010 through INV-015 |
| Request and approve sale discount (conditional on discount policy) | `sale.discount.request`, `sale.discount.approve` | `sale.discount.request`, `sale.discount.approve` | Sales; threshold, reason, and requester/approver separation; disable if discount policy is not approved | No event until approved discount is committed with sale; candidate `SaleDiscountApproved` | POS-DISC-001 through POS-DISC-005 |
| Request and approve sale void (conditional on approved void thresholds, reason capture, separation, and accounting treatment) | `sale.void.request`, `sale.void.approve`, `sale.void.complete` (internal continuation after approval and validated correction/disposition; not a client-set status) | `sale.void.request`, `sale.void.approve` | Sales + Inventory + Finance orchestration; linked correction, no deletion, stock restoration only by validated disposition | `SaleVoidRequested`, `SaleVoided`, `StockMovementCommitted` | POS-VOID-001 through POS-VOID-007 |
| Create and validate sale | `sale.draft.create`, `sale.validate`, `sale.commit` | `sale.create`, `sale.commit` | Sales orchestrates Inventory; sale lines, price/tax snapshots, stock issue, audit, and outbox commit atomically | `SaleCommitted`, `StockMovementCommitted` | POS-001 through POS-009 |
| Reissue transaction receipt | `receipt.reissue` | `receipt.reissue` | Sales; reissue references the original finalized sale; receipt detail is covered by the sales/receipt read query above and reissue never creates a new sale | `ReceiptReissued` only when reissue record commits | RECPT-001 through RECPT-004 |
| Initiate and confirm payment | `payment.initiate`, `payment.confirm.manual`, `payment.provider.callback` | `payment.initiate`; `payment.confirm_manual` for manual confirmation; provider callback uses a dedicated authenticated service identity | Payment adapter + Payments; external confirmation is verified and persisted independently from sale commit | `PaymentInitiated`, `PaymentConfirmed`, `PaymentFailed`, `PaymentDisputed` | PAY-001 through PAY-010 |
| Initiate and reconcile supplier payment (only if approved for the release) | `supplier_payment.initiate`, `supplier_payment.confirm`, `supplier_payment.reconcile` | `supplier_payment.initiate`, `supplier_payment.confirm`, `supplier_payment.reconcile` | Purchasing + Finance; supplier obligation and payment evidence remain separate; confirmation requires authoritative provider/bank evidence | `SupplierPaymentInitiated`, `SupplierPaymentConfirmed`, `SupplierPaymentReconciled` | SUPPAY-001 through SUPPAY-008 |
| Request and complete refund | `payment.refund.request`, `payment.refund.approve`, `payment.refund.execute` (internal execution continuation after approval; not a client-invoked or separately role-granted action) | `payment.refund.request`, `payment.refund.approve` | Payments; refund references original payment; the server-side execution continuation starts only after committed approval and policy/provider revalidation; provider/cash completion requires authoritative evidence | `RefundRequested`, `RefundApproved`, `RefundConfirmed`, `RefundFailed` | PAY-011 through PAY-016 |
| Process customer return and stock disposition | `customer_return.create`, `customer_return.submit`, `customer_return.review`, `inventory.return.disposition` | `customer_return.create`, `customer_return.submit`, `customer_return.review`, then `inventory.return.disposition` for stock action | Sales owns customer return; Inventory owns stock disposition; refund remains a separate Payments aggregate | `CustomerReturnSubmitted`, `CustomerReturnAccepted`, `CustomerReturnRejected`, `CustomerReturnDispositionRecorded`, `StockMovementCommitted` | RET-001 through RET-010 |
| Open, count, and close cash session | `cash_session.open`, `cash_session.count`, `cash_session.close` | `finance.cash_session.open`, `finance.cash_session.count`, `finance.cash_session.close` | Finance; session snapshot and cash movements are authoritative; close records variance without rewriting sales | `CashSessionOpened`, `CashSessionCounted`, `CashSessionClosed` | CASH-001 through CASH-009 |
| Record operational cash movement | `cash_movement.create` | `finance.cash_movement.create` | Finance; requires assigned session/context, movement type, source reference where applicable, reason, actor and idempotency; never edits source sale/payment facts | `CashMovementRecorded` | CASH-MOVE-001 through CASH-MOVE-005 |
| Reconcile payment/cash evidence | `finance.reconciliation.import`, `finance.reconciliation.match`, `finance.reconciliation_exception.resolve` | `finance.reconcile`, `finance.reconciliation_exception.resolve` | Finance; source file/event provenance, idempotent import, unmatched and disputed rows retained | `ReconciliationImported`, `ReconciliationMatched`, `ReconciliationExceptionResolved` | FIN-001 through FIN-008 |
| Read cash-session and cashbook detail | `cash_session.detail`, `cashbook.detail` | `finance.read`, `finance.cashbook.read` | Finance; cashier sees assigned session only; finance/auditor visibility follows explicit scope | No business event; projection freshness/lag telemetry only | CASH-QRY-001 through CASH-QRY-004 |
| Read operational reports | `inventory.report.read`, `sales.report.read`, `purchasing.report.read`, `finance.report.read` | Domain read permissions such as `inventory.read`, `sale.read`, `supplier.read`, `finance.read` | Read models are projections; each report states tenant/branch, as-of time, filters, freshness, and calculation basis | No business event; projection freshness/lag telemetry only | REPORT-001 through REPORT-008 |
| Export tenant records | `tenant.export.request`, `tenant.export.download` | Requested dataset's explicit export permissions: `catalogue.export`, `inventory.export`, `sales.export`, `purchasing.export`, and/or `finance.export`; tenant/branch scope | Export service; asynchronous job, immutable artifact manifest, expiry, access audit and integrity check | `TenantExportRequested`, `TenantExportCompleted`, `TenantExportExpired` | EXP-001 through EXP-006 |

### 4.9.1 Operation contract completion requirements

Operation IDs are not automatically client commands. Each operation must be classified as **client-invoked**, **authenticated provider callback**, or **internal continuation**. Internal continuations are invoked only by their owning server module after a committed prerequisite and policy revalidation; they have no public/client endpoint, no independent tenant-role grant, and no client-controlled state transition. The prerequisite human action carries the relevant permission(s), while the continuation enforces a service/module trust boundary, committed approval or acceptance reference, expected aggregate version, and stable idempotency identity. Provider callbacks use a dedicated authenticated service identity and are not tenant-user permissions.

Before implementation, each logical command/query above must have a versioned API schema that specifies:
- request fields, validation, canonical identifiers, and prohibited client-controlled fields;
- response shape, aggregate version, stable operation ID, and error codes;
- authentication, tenant/branch/resource scope, required permissions, and action-specific evidence gates;
- idempotency key and payload binding for writes;
- optimistic concurrency/version preconditions;
- transaction boundary, authoritative writes, unique constraints, and outbox event;
- audit record, actor, reason, policy version, and correlation ID;
- timeout/unknown outcome, retry, compensation, and reconciliation behavior;
- rate limits, data classification, retention, and observability;
- contract, authorization, concurrency, failure-injection, and end-to-end tests.

Queries must define field-level visibility, source versus projection semantics, pagination/filter allowlists, freshness, cache behavior, and export restrictions. A report query must not be implemented by giving the client unrestricted database access.

### 4.9.2 Test identifier convention

The IDs in the matrix are planning identifiers, not evidence that tests already exist. The implementation test registry must map each ID to an automated test, test owner, build, result, and defect link. Tests must include successful, denied, duplicate, stale-version, concurrent, timeout-after-commit, event replay, cross-tenant, cross-branch, revoked-permission, and recovery cases wherever applicable.

### 4.9.3 First-release exclusions

Public profile/catalogue/availability publication, public enquiries, moderation, verification claims, platform support data access, full general-ledger posting, statutory tax filing, fiscal-device integrations, and regulated product workflows remain outside this matrix unless Specification 25 is formally amended and the relevant legal, operational, security, and product gates pass.



## 4.10 Common command, query, and response envelope

The transport is not selected yet. The following is a transport-neutral semantic contract. REST/JSON, typed RPC, or another transport must preserve these meanings. It is not permission to expose a generic command endpoint that accepts arbitrary domain mutations.

### 4.10.1 Command request envelope

Every state-changing application command has:
- **operation ID:** stable logical operation identifier from the operation registry;
- **command ID:** unique client-generated identifier for this attempted business intent;
- **idempotency key:** stable across retries of the same semantic request; scoped to authenticated actor, tenant, operation, and resource as appropriate;
- **resource reference:** aggregate type and ID for existing resources; creation commands omit an existing ID;
- **expected version:** required for updates/transitions against an existing aggregate, except where a create command has no prior version;
- **payload:** only operation-specific business fields, validated against an explicit schema;
- **reason/evidence references:** mandatory for corrections, exceptions, disputes, restrictions, adjustments, and other policy-defined actions;
- **client context:** optional client version, locale, timezone, and device reference for diagnostics and presentation only.

The server derives authenticated actor, tenant membership, branch scope, effective permissions, service identity, and trusted device/session state. Client-supplied actor IDs, role names, permission lists, tenant IDs, branch IDs, professional status, approval identity, and authoritative timestamps must be rejected or ignored as authorization inputs. Where tenant/branch identifiers are needed for routing, the server verifies them against the authenticated membership and resource ownership before processing.

The request schema for each operation must state whether each field is required, optional, nullable, server-generated, immutable after submission, or prohibited. Unknown fields are rejected. A client cannot set aggregate status directly.

### 4.10.2 Command result envelope

A successful command response identifies:
- operation ID and command ID;
- authoritative aggregate type, ID, and new version;
- committed domain state and a safe result summary;
- server commit timestamp;
- durable result reference for later retrieval after timeout/reconnect;
- whether an external side effect remains pending;
- correlation ID.

A response may report a committed internal sale while payment remains pending, or an approved refund while provider execution remains pending. It must never collapse these into a generic “success” boolean that implies every downstream effect is complete.

A duplicate request with the same idempotency key and semantic payload returns the original durable result/reference. The same key with a different semantic payload returns a stable conflict error. A client timeout triggers outcome lookup by the original command/idempotency reference; it never justifies a new business command.

### 4.10.3 Query request and response envelope

Queries declare:
- logical query ID and API contract version;
- server-derived tenant/branch/resource scope;
- explicit field-level permission requirements;
- allow-listed filters, sort fields, and maximum page size;
- opaque cursor where cursor pagination is used;
- source-of-truth versus projection/read-model designation;
- freshness/as-of metadata and stale/unknown state where relevant;
- data classification and permitted export behavior.

A query response includes the requested resource/list, stable IDs, resource versions where relevant, server as-of time, projection freshness when applicable, and a next cursor when another page exists. It does not include fields that the caller is not authorized to see. The server must not disclose whether a foreign-tenant resource exists through different error messages or timing behavior.

### 4.10.4 Standard error code vocabulary

The following stable codes are the minimum semantic set; exact HTTP/RPC status mapping depends on the transport:

| Error code | Meaning | Client behavior |
|---|---|---|
| **UNAUTHENTICATED** | No valid authenticated identity/session | Re-authenticate through the approved flow |
| **FORBIDDEN** | Authenticated but lacks the required permission/scope/evidence | Do not retry unchanged; show a safe denial |
| **NOT_FOUND_OR_NOT_VISIBLE** | Resource absent or intentionally hidden by scope policy | Do not infer cross-tenant existence |
| **VALIDATION_FAILED** | Field or business-rule validation failed before commit | Correct the identified fields; no authoritative change |
| **RESOURCE_STATE_INVALID** | Command is not allowed from the current aggregate state | Refresh authorized state and use a valid workflow |
| **VERSION_CONFLICT** | Expected version is stale | Refresh and explicitly resolve; never overwrite silently |
| **IDEMPOTENCY_KEY_REUSED** | Same key was used with a different semantic request | Stop; create a new key only for a genuinely new intent |
| **APPROVAL_REQUIRED** | An independent approval transition is required | Create/continue the approval workflow; do not retry as the same actor |
| **EVIDENCE_REQUIRED** | Required source/credential/establishment evidence is missing or invalid | Provide/verify evidence for this exact capability only |
| **OUTCOME_PENDING** | An external side effect or durable command result is not yet resolved | Query/reconcile the original attempt; do not repeat collection or execution |
| **DEPENDENCY_UNAVAILABLE** | Required provider/service is unavailable | Preserve pending state and use bounded retry/recovery guidance |
| **RATE_LIMITED** | Configured request limit reached | Honor retry guidance; do not create duplicate commands |
| **INTERNAL_ERROR** | Unexpected server failure | Show correlation ID; do not expose internal details |

Error responses include a server-generated correlation ID and localized safe message. They must not expose SQL, stack traces, provider secrets, raw provider payloads, hidden resource identifiers, or sensitive evidence.

### 4.10.5 External callback and asynchronous job envelopes

Provider callbacks and asynchronous job completions are not ordinary user commands:
- callbacks authenticate a provider/service identity, verify provider signature/authenticity, validate schema, correlate to a known attempt, deduplicate, and persist evidence before a domain transition;
- background jobs run under narrowly scoped service identities and revalidate job ownership, tenant scope, resource version, and idempotency;
- job status is distinct from the business aggregate state;
- retries are bounded and observable; unknown external outcomes enter reconciliation rather than being guessed;
- the UI subscribes to authorized state-change events and may query a job status as a bounded recovery action, not poll continuously.


The API contract/OpenAPI or equivalent schema must instantiate these envelopes per operation before coding. This common contract does not replace operation-specific request/response schemas, authorization rules, or acceptance tests.

### 4.10.6 Action-specific regulatory evidence evaluation

The authorization service must evaluate professional and establishment evidence through a versioned **action-policy registry**, not a global tenant/person verification boolean.

Each policy entry must contain:
- a distinct evidence subject/type model: establishment licence (establishment), professional competency licence (person), and establishment-linked certificate of competence/technical-manager designation (person-to-establishment relationship). These types are not interchangeable; the exact certificate issuer, legal subject, and scope require jurisdiction-specific confirmation.
- stable policy ID/version and exact logical action(s);
- jurisdiction and effective-from/effective-to dates;
- establishment category and product/regulatory classification predicates, where relevant;
- evidence mode: `no_evidence_required`, `evidence_required`, or `unresolved`, plus an explicit `required_evidence_types` list containing zero or more of `establishment_licence`, `professional_competency_licence`, and `establishment_linked_certificate_of_competence`;
- establishment evidence type, issuer/source, legal entity/branch binding, licence scope, validity and verification method, if required;
- professional credential type, issuer/source, person identity match, professional scope, jurisdiction, validity and current status, if required;
- permitted actor/role and any delegation, approval, witness, re-authentication, or separation constraints;
- source citations, accountable legal/regulatory reviewer, decision rationale, approval date, and regression-test references.

The server derives the applicable policy from trusted resource and tenant context, including jurisdiction, establishment category, product classification, action, and effective date. The client cannot choose a policy version, claim an exemption, set a verification result, or substitute a different establishment/person. If classification or policy resolution is ambiguous, the affected regulated action fails closed with a safe `EVIDENCE_REQUIRED` or policy-review denial; unrelated management operations remain available.

Evidence evaluation returns an internal decision reference containing policy ID/version, evidence record references, verification method/result, checked-at time, and expiry/recheck boundary. The decision reference is auditable and revalidated for delayed approvals/execution. Do not expose raw credential or establishment documents in API responses, events, logs, analytics, or push payloads. Only the minimum decision and reference metadata needed by authorized consumers may cross module boundaries.

No regulated action may be enabled in production while its policy is `unresolved`, lacks an authoritative/manual verification method, or lacks accountable legal/regulatory approval and positive/negative tests. A change in law, issuer source, establishment category, product classification, or policy version must trigger impact analysis and controlled reauthorization; it must not silently rewrite historical decisions.



### 4.10.7 Regulatory action granularity and decision lifecycle

An action-policy key must be narrower than a screen, module, or broad permission whenever the legal responsibility differs within that workflow. Do not treat “POS”, “sale”, “goods receipt”, “inventory”, “purchase”, or “return” as a single regulatory action.

| Operational stage | Meaning | Credential/evidence rule |
|---|---|---|
| Administrative preparation | Drafting a purchase, receipt, stock count, sale basket, return, or other operational record | No professional credential solely for data entry; normal membership, permission, scope, and management-entitlement checks still apply |
| Physical observation/data capture | Recording observed quantities, batch identifiers, expiry, condition, or discrepancy | No blanket professional credential for ordinary capture; any legally reserved inspection or accountable attestation is a separate action |
| Operational acceptance/posting | Accepting a receipt, committing a stock movement, or finalizing an administrative record | Evaluate the exact action and establishment/product context; do not infer a credential requirement or exemption from the word “accept” |
| Professional inspection/sign-off | Attesting to quality, suitability, procurement, storage, dispensing, or another matter assigned by applicable law/standard to a qualified professional or designated technical manager | Require the exact verified person credential and/or establishment-linked designation only where the approved jurisdiction/category/action policy says so |
| Sale recording/payment | Recording a permitted in-branch transaction and processing its financial lifecycle | Separate ordinary transaction recording from medicine dispensing, professional counselling, prescription validation, or handover |
| Regulated dispensing/handover | Performing the legally reserved professional act associated with a medicine or restricted product | Disabled unless the action-specific legal matrix identifies the allowed actor, product class, establishment eligibility, evidence types, scope, and verification source and receives accountable legal/regulatory approval |
| Disposal/recall execution | Recording and carrying out regulated stock destruction, return, quarantine, or recall steps | Separate ordinary record preparation from professional authorization, witness, and execution requirements; unresolved category-specific rules keep the affected action disabled |

Mandatory rules:

1. **No permission implies professional authority.** A grant such as `goods_receipt.accept`, `inventory.receive`, or `sale.commit` authorizes only the software operation described by that permission. It cannot substitute for legally required inspection, professional sign-off, dispensing, or handover authorization.
2. **Split mixed operations.** If one interaction combines ordinary data entry with a regulated attestation, define separate logical operations and permission/evidence decisions. The server must not treat a cashier, purchasing operator, or inventory operator as the professional approver.
3. **Do not over-gate.** Where legal review confirms that an administrative step needs no professional credential, encode an explicit `no_evidence_required` policy for that precise action, jurisdiction, establishment category, and product class. Do not inherit a credential requirement from an adjacent regulated action.
4. **Do not infer exemptions.** If law, standards, regional rules, or product classification remain unresolved, mark only the affected action policy `unresolved` and disable that action. Keep unrelated private management and independently lawful record preparation available.
5. **Represent delegation explicitly.** Where a qualified professional may delegate supporting tasks, record the delegated task, delegator, delegate identity, scope, validity, limits, and source policy. Delegation does not transfer credential ownership or exceed the delegator's lawful scope.
6. **Separate decision from execution.** Revalidate identity, credential status, establishment relationship, policy version, resource state, and witness/separation conditions at the legally accountable action boundary, including delayed execution.
7. **Preserve provenance.** Store policy decision reference, evidence references, actor, exact operation, resource/product classification, jurisdiction, checked-at time, and policy version. Never copy raw credential documents into domain events or operational logs.

Before enabling a regulated workflow, its action decomposition must be reflected consistently in the operation inventory, canonical permission registry, role-grant baseline, transition contract, API schema, audit event, and positive/negative tests. A broad UI action must not conceal multiple actors or legal responsibilities.

## 4.11 Critical command schema minimums

The following are minimum semantic request/response requirements for high-impact operations. They are not final field names or wire schemas. The approved API contract must turn them into exact versioned schemas and operation tests.

| Logical operation | Minimum request information | Server-controlled or prohibited client claims | Commit/result semantics |
|---|---|---|---|
| goods_receipt.commit (internal) | Receipt ID, expected receipt version, committed accepted-line references, stable internal command/idempotency identity | Client invocation, client-selected accepted quantities, client-supplied actor identity, direct stock balance, or bypass of receipt acceptance | Purchasing/Inventory server-side continuation only after all lines are resolved and acceptance/receipt authorization succeeds; atomically persist receipt result, accepted stock movements, audit, and outbox exactly once |
| sale.void.complete (internal) | Committed void approval reference, linked correction ID, expected sale/correction versions, stable internal command/idempotency identity | Client invocation, client-selected sale status, client-supplied approval, direct stock restoration, refund completion, or altered original sale | Sales server-side continuation only after committed approval and policy revalidation; coordinate separately authorized Inventory disposition and Payments refund workflows; complete the sale correction once without deleting original facts |
| sale.void.approve | Void/correction request ID, expected correction and sale versions, approve/reject decision, reason/evidence, idempotency key | Client-supplied approval identity, direct sale status, automatic stock restoration, refund completion, or altered original sale | Commit only the authorized approval decision; the internal `sale.void.complete` continuation may proceed only after approval and policy revalidation; inventory restoration and payment refund remain separately validated workflows |
| sale.commit | Sale ID, expected sale version, idempotency key, optional explicit client confirmation of the reviewed draft | Final total, tax result, payment status, stock balance, batch eligibility, cashier identity, tenant/branch authorization | Revalidate draft and current batch eligibility; atomically commit sale lines, price/tax snapshots, stock issue, audit, and outbox |
| goods_receipt.commit | Receipt ID, expected receipt version, idempotency key | Accepted quantity, batch validity, available stock balance, supplier acceptance, actor identity | Revalidate inspected lines; atomically post accepted stock movements and receipt result; rejected/quarantined quantities stay unavailable |
| payment.initiate | Sale ID, amount/currency intent, enabled method, idempotency key | Provider transaction ID before provider response, paid/confirmed status, settlement status | Create durable payment attempt; provider request outcome may remain pending/unknown |
| payment.confirm.manual | Payment ID, expected payment version, amount/currency evidence, source/evidence reference, reason where required, idempotency key | Client assertion that payment is confirmed; fabricated bank/provider reference; initiator/approver identity | Independently authorize and verify evidence; commit confirmation or leave pending/disputed; audit reviewer |
| payment.provider.callback | Authenticated provider identity, provider event/reference, signed payload, correlation to known attempt | User-supplied tenant/actor, unsigned status, arbitrary payment ID without correlation | Verify signature/schema/correlation; persist untrusted evidence; deduplicate; only then commit a valid payment transition |
| payment.refund.execute (internal) | Committed refund approval reference, expected refund/payment versions, stable internal command/idempotency identity, approved execution method | Client invocation, client-supplied approval, fabricated provider/cash completion, altered original payment, or client-selected settlement status | Payments service-side continuation only after approval and policy revalidation; initiate the approved provider/cash action once, preserve pending/unknown outcomes, and mark complete only after authoritative confirmation/reconciliation |
| payment.refund.approve | Refund request ID, expected version, approve/reject decision, reason/evidence where required | Refund-completed status, provider settlement, altered original payment | Commit approval decision only; execution remains asynchronous and separately confirmed |
| customer_return.review | Return ID, expected version, accept/reject decision, reason, evidence and inspected line/condition facts | Client-created original sale relationship, automatic restock, automatic refund | Commit customer-return decision; inventory disposition and payment refund remain separate |
| inventory.return.disposition | Accepted return ID, return-line IDs, actual quantity/unit, batch/condition, disposition decision, evidence, idempotency key | Client assertion that returned stock is saleable; fabricated batch/expiry; refund status | Inventory validates physical and regulatory eligibility; any stock movement is committed through the inventory ledger |
| inventory.adjustment.commit (internal) | Committed approval reference, expected adjustment version, stable internal command/idempotency identity | Client invocation, client-supplied approval, direct on-hand balance, or bypass of approval policy | Inventory-only continuation; verify committed approval and unchanged source facts, then atomically append exactly one linked stock movement and outbox event |
| inventory.adjustment.approve | Adjustment request ID, expected version, approve/reject decision, reason/evidence | Direct on-hand balance, fabricated count, requester identity | Commit approval; on approval, the Inventory module invokes the internal `inventory.adjustment.commit` continuation with the approved decision reference and stable idempotency identity; posting creates a linked stock movement and cannot overwrite the ledger |
| inventory.transfer.dispatch / receive | Transfer ID, expected version, dispatched/received line quantities, batch/condition, discrepancy evidence, idempotency key | Client-computed source/destination balances; simultaneous availability at both branches | Dispatch moves quantity to in-transit; receipt posts accepted destination movement; discrepancy remains explicit |
| supplier_invoice.approve | Invoice ID, expected version, match result reference, approve/reject decision, reason/evidence | Client assertion that a mismatch is resolved or payable has been paid | Commit invoice approval/dispute state under accounting policy; payment and settlement remain separate |
| finance.cash_session.close | Session ID, expected version, counted amounts by currency/tender as supported, variance reason/evidence | Rewritten sale/payment totals, expected cash supplied by client as authority, silent variance suppression | Commit immutable close/count snapshot; variance is a linked exception, not a rewrite of source transactions |

### 4.11.1 Server-side authority and derived values

For the operations above and all other state-changing commands:
- the server calculates authoritative totals, stock availability, eligibility, and state transitions from current records and approved policy;
- clients may submit observed facts (for example, a physical count or received quantity), but the server validates scope, version, unit, evidence, and allowed transition;
- provider/bank evidence is verified independently and remains linked to the source;
- no command may set a status field directly;
- the response identifies which aggregate committed and which dependent aggregates remain pending;
- outbox events are emitted only for committed facts and are not proof of external settlement.

### 4.11.2 Required operation-specific negative tests

At minimum:
- sale commit rejects stale price/tax configuration, expired/recalled/held stock, duplicate command, and concurrent oversell;
- goods receipt rejects duplicate receipt, invalid batch/expiry, unit mismatch, and unaccepted/rejected lines; the internal `goods_receipt.commit` continuation rejects client invocation, missing accepted-line references, stale receipt version, and duplicate posting; it runs only after the initiating acceptance action passes `goods_receipt.accept` and `inventory.receive` authorization;
- manual payment confirmation rejects insufficient/invalid evidence, wrong amount/currency, wrong scope, and prohibited self-confirmation;
- provider callback rejects invalid signatures, unknown attempts, replay, duplicate, and out-of-order events without corrupting state;
- refund approval does not mark refund complete and cannot be replayed into duplicate execution; the internal `payment.refund.execute` continuation rejects client invocation or missing committed approval, is idempotent, and cannot mark a refund complete without authoritative provider/cash confirmation;
- return acceptance cannot directly make stock saleable or trigger refund;
- stock disposition rejects mismatched batch/quantity and requires inventory permission independently of return review;
- stock adjustment approval cannot be replayed into a second posting; the internal commit continuation is idempotent, version-checked, and cannot run without a committed approval; transfer enforces expected versions and prevents duplicated or negative movements;
- invoice approval cannot clear unresolved dispute or mark supplier payment settled;
- cash close cannot rewrite source sales/payments or conceal variance;
- sale void approval cannot directly set a sale to voided; the internal completion step requires the committed approval, linked correction, policy checks, and validated inventory disposition. The internal `sale.void.complete` continuation rejects client invocation and duplicate completion, and cannot complete a refund or stock restoration as an implicit side effect.

The operation-specific schema and test set must be approved with the relevant domain specification. A generic JSON schema alone does not establish business correctness.

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


### 5.2.1 Trusted request context and resource resolution

Every authenticated request must resolve a server-side authorization context containing the authenticated person or service identity, active tenant membership, effective permission set, permitted branch scope, session/device state, and applicable policy versions. The client may request a tenant or branch context when a person belongs to multiple organizations, but the server must verify active membership and scope before accepting the context. A resource ID, URL path, filter, header, token claim supplied by an untrusted client, or previously cached selection is never proof of ownership or authorization.

For every resource operation, resolve the resource from its authoritative owner and verify that its tenant and (where applicable) branch match the trusted context. Nested-resource operations must validate the complete ownership chain, not merely the parent ID supplied by the client. Bulk operations must independently constrain every selected record; one authorized record must not authorize neighboring IDs. When a resource is outside scope, return the same externally observable not-found/not-visible behavior used for absent resources, without revealing its existence through response detail, count, or materially different timing.

### 5.2.2 Branch scope and organization-wide records

Every operational record must be classified as tenant-wide, branch-owned, platform-owned, person-owned, or a deliberately public projection. Branch-owned records must carry a branch reference or an equivalent enforceable ownership path. A branch reference must be proven to belong to the same tenant as the record; a valid branch ID from another tenant is invalid even if both IDs exist. Organization-wide access is a separate explicit scope, not a wildcard inferred from missing branch ID. A member assigned to one branch receives no access to another branch unless an explicit organization-wide or multi-branch grant permits the requested action. Cross-branch reports and exports must apply the same explicit scope and disclose the included branches to the requester.

### 5.2.3 Persistence constraints and transaction enforcement

The physical ownership registry must identify the authoritative tenant and branch path for every tenant-owned table and projection. Where records reference other tenant-owned records, use tenant-scoped composite foreign keys or an equivalent database-enforced constraint so that a child in tenant A cannot reference a parent in tenant B. Uniqueness constraints must include tenant scope where values are only unique within a tenant. Branch-scoped references must enforce branch-to-tenant consistency at the database boundary where supported. Application checks remain necessary but are not a substitute for these constraints.

If row-level security is selected, define the trusted session context, transaction-local setup/reset behavior, connection-pool hygiene, migration/maintenance bypass, worker identity, and test coverage. Context must be set from authenticated server state within the transaction and reliably cleared; pooled connections must never carry one request's tenant context into another. Table-owner, superuser, migration, and service roles that can bypass row policies require separate controls and adversarial tests. RLS is defense-in-depth, not the sole authorization layer.

### 5.2.4 Secondary data paths and derived systems

The same authorization boundary applies to every derivative or transport of tenant data:
- **Search and reports:** filter before aggregation and pagination; never compute global totals and then filter visible rows if the result can reveal hidden counts or values. Every projection records source ownership, tenant/branch scope, transformation version, and freshness/cursor.
- **Caches:** include tenant, branch scope, effective authorization/version, resource identity, and relevant query parameters in cache identity. Never cache an authorization decision or private response under a key that can collide across tenants or memberships. Permission, membership, branch, and resource visibility changes invalidate or version affected entries; cache misses must reauthorize.
- **Realtime and event delivery:** authorize subscription creation, every channel/topic/resource binding, and reconnect. Derive destinations from committed event ownership, not client-selected topic strings. Filter payloads for the subscriber's current branch/tenant scope; re-check after membership or permission changes and on durable replay. A notification is not authority to fetch a resource.
- **Files and exports:** resolve metadata and ownership before issuing a short-lived grant. Bind download grants to a specific object, tenant, intended purpose, and expiry; re-check authorization when feasible at download time. Opaque object keys do not replace authorization. Temporary artifacts and generated previews inherit source sensitivity and scope.
- **Background jobs and queues:** each job carries an immutable initiating tenant/scope reference, initiating actor or service identity, operation, resource references, and correlation/idempotency identity. Workers load current authorization/policy as required, verify every resource's ownership, and use narrowly scoped service credentials. Queue routing, retries, dead letters, scheduled jobs, and replay must not broaden scope. A global worker credential is not a license for unrestricted data access.
- **Analytics and telemetry:** tenant data is excluded or minimized by default. Any tenant-specific operational view is separately authorized; metric labels must not expose confidential identifiers or permit reconstruction of private activity.

### 5.2.5 Platform operations and exceptional access

Platform support, verification, billing, security, and operational functions must access only the platform-owned records required for their function. A platform role does not imply a standing right to browse pharmacy operational records. Exceptional tenant-data access requires a recorded case or incident, stated purpose, exact tenant and resource scope, named requester and approver, minimum necessary permission, read-only default, re-authentication, short expiry, and tamper-evident audit. Mutating access requires a separate explicit grant and approved change procedure. Emergency break-glass access must be time-limited, immediately alerted, restricted to the incident scope, and independently reviewed after use. Impersonation must never be silent: the system must preserve both the human operator identity and the tenant context, visibly label the session, and record all reads and writes. No support workflow may ask for or use a customer's password or OTP.

### 5.2.6 Tenant-safe import, export, backup, and restore

Import jobs must bind to one authorized tenant and explicitly declared branch scope; every row is validated against that scope before commit. Export manifests must identify the requesting tenant, included branches, dataset, filters, schema version, record counts, checksums, creation time, expiry, and authorized requester. An export must not include another tenant's rows through joins, attachments, shared identifiers, or aggregate queries. Download authorization is re-evaluated and artifacts expire and are deleted according to policy.

Backup and restore procedures must preserve tenant/branch ownership, stable identifiers, uniqueness, foreign-key integrity, audit provenance, and event/outbox relationships. A tenant-level restore or export/import must not copy platform secrets, unrelated tenants, or global records the tenant does not own. If a full-database restore is used, the restored environment remains isolated from production until reconciliation, tenant-boundary checks, and duplicate/out-of-order event controls pass. Selective tenant restore must be treated as a dependency-aware migration, not a raw row copy. See Specification 26 for the operational procedure and evidence.

### 5.2.7 Required negative and isolation test matrix

The architecture test registry must include automated adversarial cases for each applicable interface and data path:
1. Tenant A user substitutes Tenant B IDs in path, query, body, header, nested reference, pagination cursor, or bulk list; read and write are denied without existence disclosure.
2. A branch-A-only member attempts branch B resources, reports, search, exports, files, and subscriptions within the same tenant; denied unless a current explicit grant covers branch B.
3. A child record in tenant A attempts to reference a parent, branch, attachment, or aggregate in tenant B; database constraint or transaction rejects it.
4. Search, dashboard, aggregate, count, autocomplete, and pagination cannot leak foreign tenant or branch data.
5. Cache keys cannot collide across tenants, branches, memberships, permission versions, or query scopes; revocation invalidates or makes stale entries unusable.
6. Realtime topic guessing, reconnect, replay, cursor reuse, and membership revocation cannot expose out-of-scope events.
7. A user cannot download another tenant's file by guessing an object key, reusing an expired grant, changing an export ID, or following an attachment reference.
8. A queued job cannot be reassigned to another tenant by editing payload metadata; retries, dead-letter redrive, and scheduled execution revalidate scope and idempotency.
9. Support staff cannot browse tenant data without an active case-bound grant; grant expiry/revocation stops access, and every exceptional read/write is attributable.
10. Import, export, backup, full restore, and tenant-selective restore preserve tenant boundaries and reconcile record counts, hashes, relationships, and event state.
11. RLS, if used, is tested under pooled connections, transaction rollback, worker/service roles, migration roles, and attempted policy bypass.
12. Revoking membership, role, branch assignment, or support grant takes effect across API, cache, subscriptions, files, and queued work within the approved revocation objective.

A test is not satisfied by a mocked authorization function alone. Persistence tests use the selected database engine and production-equivalent roles; integration tests exercise actual query, queue, storage, and realtime adapters. Record test ID, tenant/branch fixture identities, execution role, expected denial, observed result, and build evidence. Any confirmed cross-tenant disclosure or mutation is a critical security incident and release blocker.

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


### 5.6 Physical ownership and cross-module write discipline

The conceptual ownership model in Specifications 07 and 24 must be translated into a physical schema ownership registry before the first production migration. The registry is a design artifact reviewed with the schema; it is not a requirement to create one database schema per module.

For every table or durable record family, the registry must state:
- owning domain module and accountable code owner;
- tenant/organization/branch ownership rule, including explicitly platform-global records;
- authoritative versus projection/cache/archive classification;
- aggregate root and version/concurrency strategy, where applicable;
- permitted write entry point and permitted readers;
- foreign-key and uniqueness scope;
- retention, deletion, export, and legal-hold class;
- event(s) emitted after commit and projection consumers, if any.

A module owns the meaning and mutation rules of its records. Another module must not bypass that owner by issuing direct writes to its tables, even when both modules share one relational database. Cross-module orchestration must call the owning module's application command/interface and participate in a documented local transaction where atomicity is required. Shared database access is not shared domain ownership.

The following minimum transaction map is required for the first-release workflow set. It identifies the minimum atomic boundary; provider/bank actions remain external and cannot be made atomic with the local database.

| Business operation | Authoritative write owners | Minimum local atomic boundary | Explicitly outside that boundary |
|---|---|---|---|
| Sale commit | Sales; Inventory; Audit/Outbox | Sale and immutable lines, exact stock issue movement(s), aggregate versions, audit reference, and outbox intent commit together | Payment-provider collection, settlement, analytics/search projection |
| Goods receipt acceptance | Purchasing/Receiving; Inventory; Audit/Outbox | Accepted receipt quantities and corresponding stock movements, source references, audit reference, and outbox intent commit together | Supplier invoice approval, payable recognition, supplier payment |
| Transfer dispatch | Inventory transfer workflow and stock ledger | Transfer state/version, source stock reduction, in-transit custody movement, audit, and outbox intent commit together | Physical transport and destination acceptance |
| Transfer receipt | Inventory transfer workflow and stock ledger | Accepted destination quantity, in-transit reduction/closure, discrepancy state, audit, and outbox intent commit together | Any later discrepancy settlement or supplier/financial claim |
| Customer-return disposition | Returns workflow; Inventory; Audit/Outbox | Approved disposition decision and exactly the resulting accepted inventory movement(s), linked to return line and audit/outbox | Customer refund or payment-provider reversal |
| Payment confirmation | Payment; Audit/Outbox | Verified, deduplicated payment evidence, payment state/version, correlation to the intended attempt, audit, and outbox intent commit together | Provider-side capture/settlement and later bank reconciliation |
| Refund confirmation | Payment; Audit/Outbox | Verified refund evidence, refund state/version, original payment linkage, audit, and outbox intent commit together | Physical returned-goods disposition |
| Supplier invoice approval | Purchasing/Finance under approved accounting policy | Invoice decision, duplicate/matching evidence, payable recognition or explicit no-posting outcome, audit, and outbox intent commit together | Supplier payment initiation and bank settlement |
| Supplier payment confirmation | Finance/Payment integration | Verified settlement evidence, allocation to approved obligations, payment state, reconciliation references, audit, and outbox intent commit together | Bank/provider settlement itself |
| Cash-session close | Finance/Cash; Audit/Outbox | Final counted snapshot, accepted cash movements/cutoff, variance decision, close state/version, audit, and outbox intent commit together | Later bank deposit and external statement reconciliation |

If the chosen workflow is not included in the approved first release, its row is a future design constraint, not permission to implement it. If an operation cannot meet its stated atomic boundary in the selected architecture, the architecture or workflow must be redesigned before implementation; do not weaken the invariant by silently splitting authoritative writes.

### 5.7 Database constraint and concurrency checklist

For each physical aggregate and relationship, schema review must identify the concrete database enforcement mechanism for the following applicable invariants:

- **Tenant and branch consistency:** tenant-owned rows carry or reach one unambiguous tenant; branch references prove that the branch belongs to the same tenant. Composite foreign keys or an equivalent database-enforced design prevent cross-tenant parent/child links.
- **Scoped uniqueness:** business identifiers are unique in their actual scope (for example, tenant, branch, provider, or document issuer), not accidentally global. The scope and treatment of reused external identifiers are documented.
- **Idempotency:** command idempotency keys are unique within a documented actor/tenant/operation scope; the stored request fingerprint prevents reuse with a materially different payload; outcome and retention behavior are defined.
- **Aggregate concurrency:** state-changing commands use a version or equivalent conditional-write guard. A stale write cannot silently overwrite a newer committed transition.
- **Stock integrity:** movement identity and source-command identity prevent duplicate posting; stock allocation and availability checks are concurrency-safe; a cached position is never the only negative-stock guard.
- **Financial integrity:** currency, precision, sign, allocation, and balancing constraints are explicit; finalized financial records cannot be updated through ordinary application paths.
- **Outbox/inbox integrity:** outbox identity is unique to its source fact/transaction; consumer deduplication identity is stable and scoped to consumer plus event; retry does not create a second business effect.
- **Reference integrity:** finalized records retain valid links to their source, correction, reversal, and evidence records; deletion behavior cannot orphan legally or operationally required history.
- **State integrity:** persisted states are constrained to registered values; legal transitions are enforced by the owning application service and protected by version checks, not by arbitrary client-supplied status values.
- **Temporal integrity:** effective periods and expiry windows are represented consistently; overlapping or impossible periods are rejected where the domain requires it.

The schema review must distinguish constraints that can be expressed directly in the selected database from invariants that require a transaction, application service, or controlled workflow. Every invariant must have one named enforcement point and a negative test. “Validated in the UI” is never an enforcement point.

Do not introduce a generic cross-module foreign-key or shared writable table merely to simplify reporting. Read models may join authorized source data or consume events, but their rebuild and reconciliation procedure must remain explicit. Reporting access must not become a hidden write path into an owning module.

### 5.8 Schema-lock and migration evidence gate

Before the first production schema is approved, attach or link:
1. the physical table/record ownership registry required by §5.6;
2. an entity-relationship diagram showing tenant, legal-entity, organization, branch, and global-reference boundaries;
3. the constraint catalogue required by §5.7, including enforcement point and test ID;
4. the transaction map for every included workflow, with proof that all local atomic writes and outbox rows share the intended transaction;
5. representative query plans and index rationale for high-volume operational paths;
6. migration, backup, restore, and data-reconciliation procedures for the selected database and hosting topology;
7. a threat-test report covering cross-tenant references, guessed identifiers, duplicate commands, stale versions, and unauthorized background jobs.

This evidence gate is a prerequisite to schema lock and production migration design. It does not require choosing a database vendor prematurely, and it does not approve any unresolved first-release workflow or accounting/legal policy.

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
