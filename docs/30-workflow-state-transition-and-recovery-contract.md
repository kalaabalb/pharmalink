# 30 — Workflow state-transition and recovery contract

**Status:** Draft — cross-module implementation contract  
**Purpose:** Define how authoritative workflow states change, what guards each transition, which records/events are produced, and how failures are recovered. This document makes candidate lifecycle descriptions implementable without silently deciding unresolved commercial, legal, or accounting policy.

## 1. Authority and scope

This specification operationalizes:
- Specification 06 — end-to-end workflows;
- Specifications 13–17 — inventory, sales/payments, purchasing, catalogue, and finance;
- Specification 18 — canonical permission identifiers and role-grant register;
- Specification 20 — organization, branch, and membership lifecycle;
- Specification 21 — external provider lifecycle;
- Specification 24 — cross-module transaction and implementation contract;
- Specification 25 — first-release scope and launch gates;
- Specification 27 — inventory valuation and cost integrity;
- Specification 28 — production operations and recovery;
- Specification 29 — API, database, event, and deployment contract.

Where this document conflicts with a module's business rule, the owning module's business rule is authoritative. This document defines the shared transition shape and minimum controls. It does not approve a capability for first release, define a legal permission, approve a payment provider, or replace product-owner/accounting/regulatory decisions.

**State vocabulary rule:** names such as Draft, Pending approval, Active, and Closed are domain states, not UI labels. Persist stable machine identifiers; localize labels separately. Do not reuse the same state identifier for different aggregates.

## 2. Universal transition contract

Every authoritative state transition must be a server-side command with the following attributes:

1. **Aggregate:** aggregate type and stable ID.
2. **Expected version:** current version or equivalent concurrency precondition.
3. **Actor:** authenticated user or identified system service; client-supplied actor identity is never authoritative.
4. **Scope:** tenant, branch, and resource scope revalidated at execution.
5. **Permission:** one or more canonical permission identifiers from Specification 18, including scope and any required approval separation.
6. **Guard:** all business, lifecycle, legal/credential, entitlement, and evidence conditions required before transition.
7. **Atomic effects:** authoritative records and required outbox events committed together in one database transaction where they share the same transactional boundary.
8. **Audit:** prior state, new state, actor, time, reason, policy/configuration version, correlation ID, and source command.
9. **Result:** durable aggregate version and command outcome, safe to retrieve after a timeout.
10. **Failure behavior:** no partial authoritative transition; return a stable error or a durable pending/reconciliation state where an external side effect makes the outcome uncertain.

### 2.1 Transition classes

- **Ordinary transition:** allowed under current state, permission, and business guards.
- **Approval transition:** requires an independently authorized approver; requester/approver separation is enforced server-side where policy requires it.
- **External-evidence transition:** depends on verified provider/authority evidence. Client assertion alone cannot satisfy the guard.
- **Compensating transition:** creates a linked correction/reversal; it does not erase or rewrite the original fact.
- **Administrative restriction:** limits a defined capability with reason, scope, effective time, owner, and review/appeal path. It must not silently alter unrelated aggregates.
- **Projection transition:** updates a derived view only. It cannot change the authoritative aggregate state.

### 2.2 Forbidden transition patterns

- Direct client update of a status field.
- Skipping approval or validation states through a generic update endpoint.
- Reopening a finalized/closed aggregate by overwriting its status.
- Deleting a committed transaction to simulate a correction.
- Treating a notification, event delivery, or projection update as proof that the source transition committed.
- Treating a timeout as proof of failure or success.
- Replaying an event as a new business command.
- Granting a permission or professional credential because a user has a particular job title.
- Treating establishment evidence, individual professional credentials, account identity, and platform entitlement as interchangeable.

## 3. First-release workflow transition register

The tables below define the minimum candidate transitions for the first-release journeys. “Candidate” means the transition remains subject to the scope and policy gates in the referenced module. An omitted transition is not implicitly allowed.

### 3.1 Organization and branch

| Aggregate | From | Command / to | Required guards and effects |
|---|---|---|---|
| Organization | absent | Create → Draft | organization.manage; authenticated account; duplicate check; record creator and claimed authority; create private workspace only |
| Organization | Draft | Activate permitted management → Active | organization.manage; required workspace configuration and membership/owner invariants pass; does not assert legal establishment authorization |
| Organization | Active | Restrict capability → Restricted | platform.organization.restrict; defined capability, evidence/policy basis, reason, scope, review date; unrelated management capabilities remain available |
| Organization | Restricted | Restore capability → Active or Restricted | platform.organization.restrict; restriction-specific review passes; only reviewed capability restored |
| Organization | Active/Restricted | Suspend service scope → Suspended | platform.organization.restrict and, where applicable, platform.entitlement.manage; approved policy, scope, reason, notification and appeal path; retain records/export/legal access |
| Organization | Active/Restricted/Suspended | Begin offboarding → Closing | organization.manage; authorized request and identity/authority check; export/retention plan created |
| Organization | Closing | Complete closure → Closed | organization.manage; export, retention, legal hold, and deletion obligations completed or explicitly tracked; irreversible actions evidenced |
| Branch | absent | Create → Draft | Organization membership and branch.manage; private setup does not imply legal operating authorization |
| Branch | Draft | Activate internal operations → Active | branch.manage; branch configuration and required internal controls pass; legal/evidence gates apply only to dependent capabilities |
| Branch | Active | Restrict / temporarily close → Restricted / Temporarily closed | branch.manage; reason, scope, effective period, open-workflow handling, notifications |
| Branch | Active/Restricted/Temporarily closed | Permanently close → Permanently closed | branch.manage; authorized decision; unresolved stock, cash, orders, records, and retention handled |
| Branch | Permanently closed | Archive → Archived | branch.manage; no open operational workflow; history remains immutable and queryable under retention rules |

Branch status and public profile publication are separate aggregates. Pausing a public profile never closes a branch; a private branch remaining active does not authorize an unlawful physical operation.

### 3.2 Membership, role, and professional evidence

| Aggregate | From | Command / to | Required guards and effects |
|---|---|---|---|
| Invitation | absent | Invite → Invited | membership.invite; inviter may delegate proposed scope; single-use token, expiry, intended identity |
| Invitation | Invited | Accept → Accepted/pending activation | membership.accept; authenticated intended invitee; show tenant/branch/role scope; acceptance is not credential verification |
| Invitation | Invited | Revoke / expire → Revoked / Expired | membership.revoke or expiry process; token unusable |
| Membership | Accepted/pending activation | Activate → Active | membership.accept; membership and minimum required scope checks; no blanket professional credential requirement |
| Membership | Active | Restrict / suspend → Restricted / Suspended | membership.restrict; defined permission/scope, reason, effective time, review path; revoke affected sessions/subscriptions promptly |
| Membership | Active/Restricted/Suspended | Revoke → Revoked | membership.revoke; authorized actor; revoke sessions, subscriptions, queued command eligibility; preserve historical attribution |
| Role grant | absent | Assign → Active grant | role.assign; assigner delegation authority; tenant/branch/resource scope; approved grant mapping |
| Role grant | Active grant | Revoke / expire → Revoked / Expired | Authorized revocation or expiry; authorization re-evaluated on every command and reconnect |
| Credential evidence | absent | Submit → Submitted | verification.evidence.submit; person, credential type, issuer, scope, dates, source/evidence reference |
| Credential evidence | Submitted | Review → Under review | Reviewer independence and verification.evidence.review |
| Credential evidence | Under review | Verify / reject / request more evidence → Verified / Rejected / More information required | Source and scope validated; rationale, reviewer, time, validity and appeal path recorded |
| Credential evidence | Verified | Expire / suspend / revoke → Expired / Suspended / Revoked | Expiry or verified source/decision; invalidate only dependent actions, not ordinary account access |

A credential transition changes eligibility only for actions whose approved policy explicitly depends on that credential. It does not automatically suspend account sign-in or unrelated management permissions.

### 3.3 Supplier and purchase order

| Aggregate | From | Command / to | Required guards and effects |
|---|---|---|---|
| Supplier | absent | Create → Draft | supplier.create; tenant scope; duplicate candidate check |
| Supplier | Draft | Activate → Active | supplier.update; required operational fields and review pass |
| Supplier | Active | Deactivate / block / review → Inactive / Blocked / Under review | supplier.update for deactivation/review or supplier.block for block; reason; prohibit configured new operations while preserving history |
| Supplier | Inactive/Under review | Reactivate → Active | supplier.update; review and authorization pass; blocked supplier requires explicit unblock decision |
| Requisition (if included) | absent | Create → Draft | requisition.create; valid organization/branch scope and product references |
| Requisition (if included) | Draft | Submit → Pending approval | requisition.submit; valid items, units, branch, quantities |
| Requisition | Pending approval | Approve / reject → Approved / Rejected | requisition.approve; threshold and separation rules; reason |
| Quotation | absent | Record → Recorded | quotation.record; supplier, source, currency, validity, terms, and quoted unit snapshot |
| Purchase order | Draft | Create → Draft | purchase_order.create; valid supplier, branch, lines, units and expected cost snapshot |
| Purchase order | Draft | Submit → Pending approval | purchase_order.submit; supplier and line validation; expected cost/unit snapshot |
| Purchase order | Pending approval | Approve / reject → Approved / Rejected | purchase_order.approve; approval threshold and requester/approver separation |
| Purchase order | Approved | Issue → Issued | purchase_order.issue; supplier channel/address and immutable issued version captured |
| Purchase order | Issued | Record acknowledgement → Acknowledged | Evidence of supplier response; does not create stock |
| Purchase order | Issued/Acknowledged/Partially received | Accept receipt effect → Partially received / Fully received | Goods receipt accepted by inventory workflow; accepted/rejected/outstanding line quantities reconciled |
| Purchase order | Draft/Pending approval/Approved/Issued/Acknowledged/Partially received | Cancel → Cancelled | purchase_order.cancel; resolve outstanding quantities and supplier communication; preserve receipts/invoices already committed |
| Purchase order | Fully received/Cancelled | Close → Closed | purchase_order.close; all line quantities resolved and required invoice/financial workflow handled under explicit closure policy |

**Important separation:** “Fully received” means physical line quantities are resolved under the receipt policy. “Closed” means the order's remaining operational and financial follow-up has been resolved under the closure policy. Receipt, invoice acceptance, payable recognition, supplier payment, and settlement are distinct aggregates/transitions.

The precise conditions for rejection, over-delivery, substitutions, backorders, invoice completion, and closure remain product/accounting decisions. Until resolved, the affected path must be disabled or routed to a controlled exception state.

### 3.4 Goods receipt and inventory

| Aggregate | From | Command / to | Required guards and effects |
|---|---|---|---|
| Goods receipt | Draft | Submit for acceptance → Pending inspection/acceptance | goods_receipt.create; source order/reference if required; item/unit and branch validation |
| Goods receipt line | Pending inspection/acceptance | Accept / reject / quarantine → Accepted / Rejected / Quarantined | goods_receipt.accept; quantity, presentation, batch/expiry, condition, evidence and category policy checked |
| Goods receipt | All lines resolved | Commit receipt → Accepted/Partially accepted | goods_receipt.accept and inventory.receive; one authoritative receipt commit; accepted quantities create stock movements atomically with outbox; rejected/quarantined lines excluded from available stock |
| Stock batch | Eligible | Place hold → Held | inventory.hold; reason, scope, source, actor; available quantity reduced immediately |
| Stock batch | Held | Release hold → Eligible or another restricted status | inventory.release; release authority and evidence; recalculate eligibility, do not assume saleable |
| Stock count | Draft | Record count → Count recorded | inventory.count; count scope/time/device and counted quantities; count is evidence, not stock overwrite |
| Stock count | Count recorded | Review variance → Adjustment requested | inventory.adjust.request; compare count to ledger at defined cutoff; preserve intervening movements |
| Stock adjustment | Requested | Approve/reject → Approved/Rejected | inventory.adjust.approve; reason/evidence; separation/threshold policy |
| Stock adjustment | Approved | Commit adjustment → Posted | Append compensating movement; unique command; no direct balance overwrite |
| Branch transfer | Draft | Create → Requested | inventory.transfer.create; source/destination tenant consistency; item/unit and available quantity |
| Branch transfer | Requested | Dispatch → In transit | inventory.transfer.dispatch; atomic source availability reduction and in-transit movement |
| Branch transfer | In transit | Receive → Received / Partially received | inventory.transfer.receive; destination accepted quantity; discrepancy recorded; never duplicate availability |
| Branch transfer | Requested | Cancel → Cancelled | inventory.transfer.cancel; no dispatch committed; audit cancellation |
| Branch transfer | In transit | Resolve discrepancy → Received with exception / Disputed | Reconcile dispatched vs received; do not silently recreate missing quantity |

Batch expiry, recall, damage, disposal, returns, and valuation follow Specifications 13 and 27. No state label alone makes a batch saleable: sale eligibility is a server-side predicate over product category, batch status, dates, holds, recall, quantity, and any required authorization.

### 3.5 Sale, payment, return, and refund

| Aggregate | From | Command / to | Required guards and effects |
|---|---|---|---|
| Sale | absent | Create draft → Draft | sale.create; branch/session context; no stock/financial commitment |
| Sale | Draft | Submit validation → Pending validation | Server revalidates identity, price/tax snapshot, eligible stock, permissions, and action-specific regulated gates |
| Sale | Pending validation | Validation failure → Draft or Rejected | Return line-specific safe errors; no stock/financial commitment |
| Sale | Pending validation | Commit → Committed | sale.commit; atomic sale, line snapshots, stock issue, audit, and outbox; this does not imply payment confirmation or an unapproved accounting posting |
| Sale | Committed | Request void → Correction pending | sale.void.request; original reference, reason, affected lines; no immediate erase/reversal |
| Sale correction | Correction pending | Approve/reject → Approved/Rejected | sale.void.approve or applicable approval; separation and policy checks |
| Sale | Committed | Reflect accepted return → Partially returned / Returned | Derived from an accepted customer-return aggregate; no direct sale status mutation; stock disposition remains separate |
| Sale | Committed | Complete approved void → Voided | Linked reversal/correction; stock restoration only through separate validated inventory movement |
| Payment | absent | Initiate → Initiated | payment.initiate; sale reference, amount/currency, method, idempotency key |
| Payment | Initiated | Await provider/customer → Awaiting action / Pending confirmation | Provider request/reference captured; no paid status |
| Payment | Pending confirmation | Confirm → Confirmed | Verified authoritative evidence; provider callback or approved manual confirmation; deduplicate and reconcile |
| Payment | Pending confirmation | Fail/cancel/expire → Failed / Cancelled / Expired | Authoritative failure/cancel/expiry evidence or reviewed manual outcome |
| Payment | Confirmed | Reverse/dispute → Reversed / Disputed | Provider/bank evidence; linked financial correction and reconciliation case |
| Return | absent | Create → Draft | customer_return.create; references original sale and line/quantity; no stock or payment effect |
| Return | Draft | Submit → Pending inspection | customer_return.submit; return reason, actor, time, source sale and eligibility window recorded |
| Return | Pending inspection | Accept / reject → Accepted / Rejected | customer_return.review; authorized review; accepted quantity and condition recorded; no automatic restock/refund |
| Return | Accepted | Complete disposition → Disposition complete | inventory.return.disposition; separate inventory decision records restock, quarantine, disposal, or other approved outcome |
| Supplier invoice | absent | Record → Recorded | supplier_invoice.record; supplier, document identity, amount/currency, lines and source evidence |
| Supplier invoice | Recorded | Match → Matching / Exception | supplier_invoice.match; compare to purchase order and accepted receipts; preserve line-level differences |
| Supplier invoice | Matching | Resolve exception → Matching or Exception resolved | supplier_invoice.exception.resolve; evidence and rationale recorded; unresolved material discrepancy blocks approval |
| Supplier invoice | Matching / Exception resolved | Approve → Approved | supplier_invoice.approve; accounting policy, authority, duplicate check, approval separation |
| Supplier invoice | Matching / Exception resolved | Dispute → Disputed | supplier_invoice.dispute; preserve dispute reason/evidence; block payable settlement pending resolution |
| Supplier invoice | Approved | Record settlement allocation → Partially paid / Paid | Only after independently confirmed supplier payment and reconciliation; never from invoice approval alone |
| Refund | absent | Request → Pending approval or Pending provider | payment.refund.request; original payment/sale, eligible amount, reason |
| Refund | Pending approval | Approve/reject → Approved/Rejected | payment.refund.approve; authority, amount and separation checks |
| Refund | Approved | Initiate → Pending provider/cash execution | Provider/cash reference and idempotency key |
| Refund | Pending execution | Confirm/fail/dispute → Refunded / Failed / Disputed | Authoritative confirmation/reconciliation; no completed label on initiation alone |

**Sale/payment separation:** the sale may be committed while its payment is pending only if the approved sales policy permits it. If policy requires payment first, the method-specific orchestration must reserve/confirm payment safely before the sale commit and handle uncertain provider outcomes without duplicate collection. The product owner and accounting/legal reviewers must approve the ordering for each enabled payment method.

A refund does not automatically reverse inventory. A returned item does not automatically produce a refund. Each transition is linked but independently authorized and reconciled.

### 3.6 Cash session and financial reconciliation

| Aggregate | From | Command / to | Required guards and effects |
|---|---|---|---|
| Cash session | absent | Open → Active | finance.cash_session.open; branch/cashier/till scope and no conflicting open session under chosen policy |
| Cash session | Active | Begin close/count → Closing count | finance.cash_session.count; stop or explicitly control new cash transactions under the selected policy |
| Cash session | Closing count | Variance found → Review required | Expected vs counted cash, movements and discrepancy recorded; no source sale rewritten |
| Cash session | Closing count | Close → Closed | finance.cash_session.close; count/review policy passes; immutable close snapshot |
| Cash session | Review required | Resolve variance → Closed or Review required | finance.reconciliation_exception.resolve; reason/evidence/approval; linked correction movement if needed |
| Reconciliation batch | absent | Create/import → Imported | finance.reconcile; source, hash, covered period, currency, actor/system and file validation |
| Reconciliation batch | Imported | Match → Matched with exceptions | Deterministic matching; duplicates/unmatched/disputed remain explicit |
| Reconciliation exception | Open | Resolve → Resolved | finance.reconciliation_exception.resolve; evidence, rationale, reviewer and linked correction where required |
| Reconciliation batch | All required exceptions resolved | Finalize → Finalized | Totals reconcile; approval and immutable source snapshot; replay is idempotent |

Whether cash sessions, expenses, supplier payments, and reconciliation are first-release capabilities remains subject to Specification 25 and accounting review. Do not expose a “close” action before its concurrency, late-arriving cash, shared till, and variance policy is defined.

## 4. Cross-module transaction ordering

The following ordering is mandatory unless a separately approved method-specific design proves a safer alternative:

### 4.1 Purchase to stock

1. Purchase order authorization does not create stock.
2. Physical goods are inspected and accepted through a goods receipt.
3. Receipt commit creates inventory movement(s) and receipt source facts atomically.
4. Invoice recording/matching and payable treatment are separate.
5. Supplier payment initiation and settlement are separate from invoice approval.
6. Discrepancies remain explicit; no silent quantity/cost adjustment.

### 4.2 Sale to payment

1. Sale draft and validation do not commit stock.
2. Sale commit and stock issue are atomic.
3. Payment state is independently authoritative.
4. A payment callback is verified, deduplicated, correlated, and reconciled before confirmation.
5. A timeout triggers lookup by original command/idempotency reference; it never justifies a second collection.
6. Refund, sale correction, and stock disposition are separate linked workflows.

### 4.3 Count to adjustment

1. Count records measured facts at a declared cutoff.
2. The system compares the count with ledger movements through that cutoff.
3. Variance becomes an adjustment request, not an overwrite.
4. Approval and posting create a linked compensating movement.
5. Concurrent movement after the count is preserved and included in reconciliation.

## 5. API, permission, event, and audit traceability

Specification 29 defines the API operation inventory. Before an operation is implemented, its contract must additionally identify:

| Traceability element | Required record |
|---|---|
| UI/workflow action | Workflow and step identifier |
| API command/query | Stable operation ID and request/response schema |
| Aggregate transition | Aggregate type, from-state, to-state, expected version |
| Permission | Canonical identifier(s), role grant, tenant/branch/resource scope |
| Guard | Business and evidence checks; credential dependency only for the specific regulated action |
| Transaction | Authoritative writes, atomic boundary, unique constraints and idempotency key |
| Event | Versioned event type, producer, aggregate version, payload classification and consumers |
| Audit | Actor, time, reason, policy version, before/after, correlation and source command |
| Failure/recovery | Stable error, pending/unknown outcome, retry, reconciliation, compensation and operator action |
| Tests | Allowed, denied, stale version, duplicate, concurrency, timeout, replay, cross-tenant and recovery |

### 5.1 Event names are not state transitions

Events describe committed facts, not requested intentions. For example:
- `PurchaseOrderApproved` is emitted only after approval commits.
- `GoodsReceiptAccepted` is emitted only after accepted receipt quantities and stock movements commit.
- `SaleCommitted` is emitted only after the sale/stock transaction commits.
- `PaymentConfirmed` is emitted only after authoritative payment evidence passes validation.
- `CashSessionClosed` is emitted only after close and required variance handling commit.

Commands such as “approve”, “receive”, and “confirm” are not events. Rejected commands may produce security/audit records, but must not publish a success domain event.

## 6. Failure, timeout, and recovery contract

| Failure point | Required behavior |
|---|---|
| Validation fails before transaction | No authoritative state change; return field/resource-specific safe errors |
| Permission/credential revoked before execution | Deny at execution; preserve draft; record denied attempt where policy requires |
| Concurrent update wins | Reject stale expected version; caller refreshes and explicitly resolves |
| Database transaction rolls back | No partial authoritative records or outbox event; retry only with same idempotency identity |
| Commit succeeds but response is lost | Resolve original command outcome; do not submit a new command |
| Outbox publish fails | Source transaction remains committed; retry publication and alert on lag |
| Consumer receives duplicate | Deduplicate by stable event ID and aggregate version |
| Consumer receives out of order | Detect version gap; defer/reconcile; do not apply stale projection |
| Provider request times out | Mark outcome unknown/pending; query provider/reconciliation path before retrying side effect |
| Provider callback is invalid | Reject/quarantine; no payment state change; security/operations alert as appropriate |
| Provider confirms but internal processing fails | Preserve provider evidence; retry idempotent internal processing; reconcile before customer-facing completion |
| Projection/search update fails | Source fact remains authoritative; mark projection lag and rebuild/reconcile |
| Worker crashes during processing | Resume from durable checkpoint/inbox/outbox; no repeated business effect |
| Restore/replay follows disaster | Reconcile external side effects and revalidate authorization/idempotency; never blindly replay all commands |

Unknown outcomes must be represented explicitly. Do not map unknown to failed, paid, received, refunded, or completed merely to simplify the UI.

## 7. Approval and separation-of-duties policy

The canonical permission identifiers in Specification 18 are necessary but not sufficient. For every enabled approval transition, record:
- requester permission and approver permission;
- whether the same person may request and approve;
- thresholds by amount, quantity, category, or risk;
- single-person branch exception, if any, and compensating review;
- re-authentication requirements;
- required reason and evidence;
- delegation authority and expiry;
- action-specific professional credential or establishment evidence, if legally required;
- audit and appeal path.

No generic “manager can approve everything” policy. No role may self-grant higher authority. An owner may manage ordinary platform configuration without automatically receiving regulated professional authority.

## 8. Decisions still open — do not invent

| Decision | Owner/evidence | Safe interim behavior |
|---|---|---|
| First-release sale/payment ordering per method | Product owner, payment provider, accountant, legal review | Do not enable the method until specified |
| Whether unpaid/part-paid sales are allowed | Product owner and accountant | No assumed receivable or silent paid status |
| PO rejection, revision, substitution, and closure rules | Pharmacy field research and product owner | Keep unsupported transition disabled |
| Partial receipt and over-delivery acceptance policy | Pharmacy operators and accounting | Quarantine/exception; no silent acceptance |
| Cash session concurrency/shared till/late cash policy | Pilot workflow research and accountant | Do not claim reliable cash close until resolved |
| Adjustment thresholds and single-person approval exception | Pharmacy research and product owner | Require conservative approval; no self-approval |
| Return/refund eligibility and timing | Product owner, accounting and regulatory review | Do not auto-refund or restock |
| Credential and establishment gates by product/action | Qualified Ethiopian legal/regulatory review | Gate only the exact unreviewed regulated action |
| Reconciliation matching tolerances and final approval | Accountant and provider/bank evidence | Leave unmatched/disputed |
| Organization suspension and service restriction policy | Product owner, contract and legal review | Preserve required access, export and records |

An unresolved decision may remain open during discovery. It must not be implemented as an undocumented default. The corresponding capability is excluded, disabled, or restricted to a safe exception path until the decision is approved.

## 9. Acceptance-test matrix

For every transition in Section 3, automated tests must include the applicable cases:

1. Valid transition from each permitted source state.
2. Invalid transition from every other state.
3. Missing, expired, revoked, or out-of-scope permission.
4. Cross-tenant and cross-branch resource reference.
5. Required professional credential absent, expired, suspended, or out of scope—only for the dependent action.
6. Required establishment evidence absent/expired—only for the dependent capability.
7. Missing required reason, evidence, or approval.
8. Requester attempts prohibited self-approval.
9. Duplicate command with same key and same payload.
10. Same key reused with a different payload.
11. Concurrent commands using the same aggregate version.
12. Timeout before commit and timeout after commit.
13. Outbox publish failure, duplicate delivery, ordering gap, replay.
14. Provider timeout, duplicate callback, forged callback, late reversal, and disputed settlement where applicable.
15. Correction/reversal preserves original records and links all compensating effects.
16. Audit and notifications reflect the committed result, not the requested action.
17. Projection failure does not corrupt source facts.
18. Offline queued command is rejected safely after permission, membership, credential, or state changes.
19. Export and retention preserve required history and respect privacy scope.
20. Restore/recovery does not duplicate external or internal side effects.

Property-based/stateful tests should generate valid and invalid command sequences and assert invariants after every step. At minimum, invariants include: no negative/duplicated stock effect; no unconfirmed payment shown as paid; no stock from an unaccepted receipt; no double transfer availability; no finalized fact silently rewritten; no cross-tenant reference; no unauthorized transition; and no event replay repeating a business effect.

## 10. Implementation readiness rule

A workflow is implementation-ready only when:
- its aggregate and state identifiers are stable;
- every allowed transition and forbidden transition is defined;
- all guards and permission identifiers are mapped to approved role/scope grants;
- transaction ordering and atomic writes are explicit;
- API command/query contracts exist;
- event and audit behavior is specified;
- failure, unknown outcome, retry, compensation, and operator recovery are defined;
- all material policy decisions are resolved or the affected transition is safely excluded;
- automated transition and cross-module tests are specified and pass in the implemented system.

This contract does not mark any module Approved. Specifications remain Draft until evidence, owner review, and acceptance are complete. A UI that displays a state or button without a valid server-side transition is not a functioning feature.
