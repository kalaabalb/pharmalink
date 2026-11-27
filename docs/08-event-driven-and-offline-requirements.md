# 08 — Event-driven updates and offline requirements

**Status:** Draft — architecture constraints and product behavior  
**Priority:** High; these requirements are foundational.

## 1. Objective

Clients should receive relevant state changes through subscriptions rather than repeatedly asking the server whether unchanged data has changed. Routine short-interval polling is prohibited for normal live-update paths. Scheduled jobs remain valid for bounded tasks such as expiry checks, reminders, reconciliation, retention processing, and integrity verification.

The user experience must remain correct when devices disconnect, events are delayed, duplicated, reordered, or missed.

## 2. Event design principles

1. Publish events only after the underlying business transaction is durably committed.
2. Use a transactional outbox or equivalent atomic publication mechanism so a committed change cannot be lost between database commit and event publication.
3. Assume at-least-once delivery. Consumers must be idempotent.
4. Include stable event IDs, aggregate IDs and versions, tenant/branch scope, schema version, occurred-at time, correlation ID, and causation ID where applicable.
5. Do not rely on delivery order across unrelated aggregates. Enforce version checks for aggregate-specific ordering.
6. Keep payloads minimal. Send identifiers and safe change metadata; fetch authorized current state when necessary.
7. Authorize subscription establishment and every data fetch independently. A channel name is not an access-control mechanism.
8. Never broadcast one tenant's private event to another tenant or an unauthorized branch.
9. Use bounded retries, dead-letter handling, observable failure states, and replay/reconciliation.
10. Event schemas are contracts. Breaking changes require explicit versioning and coordinated consumer updates; do not preserve obsolete contracts indefinitely.

## 3. Candidate event catalogue

This is an initial taxonomy; each event requires a schema and ownership review.

| Domain | Candidate committed events |
|---|---|
| Organization | OrganizationUpdated, BranchCreated, BranchStatusChanged |
| Membership | InvitationIssued, MembershipActivated, MembershipRevoked, PermissionChanged |
| Catalogue governance | CanonicalCatalogueVersionPublished, CatalogueEntryLinked, TenantCatalogueEntryChanged, CatalogueCorrectionProposed, CatalogueCorrectionReviewed, BarcodeMappingReviewed |
| Inventory ledger | StockMovementCommitted, StockCountPosted, StockHeld, StockReleased, StockDisposed, RecallNoticeRecorded |
| Branch transfer | StockTransferDispatched, StockTransferReceived |
| Purchasing | SupplierChanged, PurchaseOrderSubmitted, PurchaseOrderApproved, PurchaseOrderIssued, PurchaseOrderCancelled, GoodsReceiptAccepted, SupplierInvoiceRecorded, SupplierInvoiceApproved, SupplierReturnDispatched, SupplierPaymentStatusChanged |
| Sales/POS | SaleCommitted, SaleVoided, ReturnAccepted, RefundStatusChanged, CashSessionOpened, CashSessionClosed |
| Payments | PaymentAttemptInitiated, PaymentStatusChanged, PaymentReconciled |
| Finance | FinancialEntryPosted, FinancialEntryReversed, ReconciliationBatchImported, ReconciliationExceptionResolved, FinancialPeriodClosed |
| Quality | AffectedStockFlagged, QualityDispositionRecorded |
| Public projection | PublicProfilePublicationConfigured, PublicProfilePublished, PublicProfilePaused, PublicProfileChanged, PublicCatalogueEntryPublished, PublicCatalogueEntryPaused, PublicAvailabilityConfirmed, PublicAvailabilityUpdated, PublicAvailabilityBecameStale, PublicEnquiryReceived, PublicReportSubmitted, ModerationCaseOpened, PublicContentSuppressed, ModerationDecisionRecorded, ModerationAppealFiled, VerificationClaimChanged |
| Subscription | SubscriptionChanged, EntitlementChanged, InvoiceStatusChanged |
| Platform | VerificationDecisionRecorded, SupportCaseUpdated, PrivilegedAccessGranted, SecurityIncidentRecorded |

Events must describe facts, not commands. “SaleCommitted” means the transaction is committed; “TryToCommitSale” is a command, not an event.

## 4. Delivery topology and client behavior

Candidate channels include:
- authenticated server push for web/mobile foreground sessions;
- platform push notifications for user-visible alerts when an app is backgrounded;
- a durable server-side event log or change cursor for reconnect/replay;
- local application event dispatch for UI state changes within one client.

The transport choice is deferred. The product requirements are not: subscription authorization, tenant isolation, resumability, duplicate handling, event versioning, and a full-state reconciliation path are mandatory.

A client should:
1. authenticate and establish authorized subscriptions;
2. receive events and update only the relevant state;
3. persist the last safely processed cursor/version;
4. detect a gap, invalid version, or reconnect after an unknown interval;
5. request a bounded change replay or authoritative snapshot;
6. reconcile local state and resume the subscription.

Do not use frequent polling as a substitute for missing event infrastructure. A bounded health/recovery check may exist only where justified, documented, and rate-limited.

## 5. Consistency and concurrency

- Critical business writes are validated and committed by the server.
- Stock and financial updates use transactional consistency appropriate to the invariant being protected.
- Optimistic concurrency uses explicit aggregate versions or equivalent compare-and-swap semantics.
- Duplicate commands carry idempotency keys scoped to actor, tenant, and operation.
- Repeated event delivery must not repeat stock movements, payment capture, notifications, or other non-idempotent side effects.
- A stale client cannot overwrite newer authoritative state without a conflict response.
- The UI distinguishes optimistic local presentation from server-confirmed state.

## 6. Offline product contract

Offline is not a blanket promise. Each operation must be classified:

- **Read cached:** may display a previously authorized snapshot with its last-updated time and offline label.
- **Local draft:** may be prepared offline but is not an authoritative business record.
- **Queue for submission:** may be stored locally and submitted later only if the operation is explicitly approved for offline queuing.
- **Online-only:** must be blocked when the server cannot validate the required state or authority.
- **Needs review:** cannot be safely auto-merged and requires an authorized resolution.

The exact operations allowed offline are not yet approved. Do not assume that a sale, stock adjustment, transfer, return, payment, or regulated workflow may be finalized offline.

## 7. Offline security and device trust

- Local data is encrypted at rest using platform-appropriate secure storage.
- Cache only the minimum data needed for the approved offline workflow.
- Device registration and revocation are server-controlled.
- Local authorization has a bounded validity period and must not silently outlive revoked membership or expired credentials.
- Shared-device use requires clear account switching and local data isolation.
- Lost-device response includes remote revocation and defined cache invalidation behavior when the device reconnects.
- Sensitive data is not placed in push-notification payloads or diagnostic logs.

## 8. Conflict and replay policy

Never silently use last-write-wins for stock, money, permissions, credential status, or regulated records.

For each queued command define:
- immutable command ID and idempotency key;
- originating user, device, tenant, and branch;
- creation time and local sequence;
- base aggregate version;
- payload schema version;
- authorization context and its validity constraints;
- expiry/maximum queue age;
- server acknowledgement and final result.

On synchronization, the server revalidates current authorization, business rules, resource state, and idempotency. Outcomes are accepted, rejected, duplicate-already-processed, or needs-review. Conflicts must preserve the submitted intent and authoritative server state for authorized resolution; they must not silently overwrite either.

## 9. Freshness and public availability

Availability is a derived, time-bounded assertion. It must include:
- source branch and item;
- last successful server confirmation time;
- source revision/version;
- freshness deadline;
- status and reason category;
- publication state.

When freshness expires, the public status transitions to stale or unknown. A device being offline cannot extend freshness. A missed event must be recoverable through replay or reconciliation; it must not leave a stale public assertion indefinitely.

## 10. Observability and service objectives

Measure:
- event creation-to-publication latency;
- publication-to-client delivery latency;
- reconnect and replay success;
- duplicate and out-of-order event rates;
- dead-letter count and age;
- consumer lag;
- stale availability duration;
- offline queue age and rejection rate;
- reconciliation mismatch rate.

Set numeric service objectives only after user research and load modeling. Alert on broken delivery paths and prolonged stale states. Do not claim “real time” without a measured objective and user-visible freshness semantics.

## 11. Acceptance criteria

- A committed domain transaction always has a recoverable publication record.
- Duplicate delivery produces no duplicate business effect.
- A reconnecting client can detect and recover from missed events.
- Subscriptions are authorized and tenant/branch scoped.
- Normal live UI updates do not depend on frequent polling.
- Offline data is visibly distinguished from server-confirmed data.
- Sensitive operations cannot be finalized offline unless a separately approved policy explicitly allows them.
- Public availability expires to stale/unknown without fresh server confirmation.
- Conflict tests cover duplicate, reordered, delayed, missed, expired-authorization, and concurrent-update scenarios.

## 12. Open decisions

- Which specific actions, if any, are permitted as offline queued commands in the first release?
- What are the maximum offline queue age and authorization validity windows?
- Which events should reach foreground sessions only versus background push notifications?
- What freshness thresholds are appropriate for public availability by connection quality and operational risk?
- What are the replay retention window and full-resynchronization procedure?
- Which event transport best meets cost, device battery, latency, and reliability requirements?


## 13. Event coverage for detailed business modules

The canonical candidate event catalogue is maintained in Section 3 and aligned with the inventory, POS, purchasing, finance, and catalogue specifications. Final schemas, event ownership, and consumer behavior must be reviewed before implementation.

### Event ownership and transaction alignment

- A business event is emitted only after its owning aggregate transaction commits.
- Inventory movements, sale commits, receipt acceptance, financial postings, and verification decisions must persist their outbox record in the same transaction as the authoritative state change.
- A payment-provider callback is an external input. It is verified and processed idempotently; the resulting PaymentStatusChanged event is emitted only after the payment state transition commits.
- A report refresh, UI toast, or push notification is a downstream reaction, not a replacement for the durable domain event.
- Do not publish raw prescription/customer health information, full payment credentials, supplier banking data, or verification evidence in event payloads. Use minimal identifiers and safe change metadata.
- Consumers must be idempotent and version-aware. Event replay must not re-run stock, money, payment, or notification side effects.
- Public projection events must not expose private stock quantities or tenant-only details.
- Schema changes must be versioned and tested against all consumers. Breaking event contracts require coordinated migration; do not keep obsolete contracts indefinitely merely for compatibility.

### Inventory event naming rule

`StockMovementCommitted` is the canonical event for each accepted stock-ledger movement; its versioned payload identifies the movement type (receipt, sale issue, adjustment, hold, release, disposal, transfer leg, and so on). Do not emit parallel `StockReceived` or `StockAdjusted` events for the same ledger fact. Workflow-level events such as `StockTransferDispatched` and `StockTransferReceived` may be emitted in addition when they represent distinct transfer aggregate state transitions. Consumers must use movement IDs and aggregate versions to prevent double application.
