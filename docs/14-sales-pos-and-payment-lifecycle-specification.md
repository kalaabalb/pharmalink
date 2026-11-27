# 14 — Sales, point-of-sale, and payment lifecycle specification

**Status:** Draft — operational validation, accounting review, provider evaluation, and regulatory review required  
**Priority:** Critical — sales affect money, stock, receipts, and legally relevant records  
**Scope:** Private in-pharmacy sales/POS operations, payment state, receipt records, voids, returns, refunds, cash sessions, and reconciliation.

## 1. Scope boundary: private POS is not public commerce

This specification concerns a pharmacy's own in-branch/private point-of-sale operations as part of its management system. It does not mean that the pharmacy is selling through Pharmalink's public discovery experience.

A pharmacy may use internal POS and management functions while:
- keeping its public profile unpublished;
- keeping products and stock private;
- not accepting public enquiries or orders;
- not using Pharmalink as an online marketplace, reservation, delivery, or payment intermediary.

Public ordering, payment collection, reservation, and delivery remain separate capabilities with separate opt-in, legal, provider, consumer-protection, and operational approval. Do not conflate a pharmacy's ordinary in-store sale with an online sale through Pharmalink.

## 2. Core transaction invariants

1. A sale is a durable business transaction, not a UI cart or receipt preview.
2. A sale may commit only after the server validates identity, tenant/branch permission, item eligibility, current stock, price rules, and required workflow approvals.
3. Sale header, lines, pricing/tax snapshots, stock issue movements, and audit references must commit atomically within the defined database transaction boundary.
4. Payment state is separate from sale state. A sale may be committed with payment pending only if the approved business policy permits it; it must never be marked paid without authoritative payment evidence.
5. A provider request being sent is not proof of payment. A client success screen is not proof of payment.
6. Every retryable sale/payment command has an idempotency strategy. Duplicate requests cannot create duplicate sales, stock deductions, receipts, or captures.
7. Concurrent checkout cannot oversell eligible stock.
8. Finalized sale and payment records are immutable; corrections create linked void, return, refund, reversal, or adjustment records.
9. A return does not automatically restore returned items to saleable stock.
10. Every sale is attributable to the actual staff account and branch; shared generic cashier identities are not an acceptable substitute for actor attribution.
11. Prescription-related or otherwise regulated workflows are not bypassed by an ordinary POS permission.
12. Internal POS availability is independent of public listing or public commerce participation.

The cross-module transaction, idempotency, event, and correction rules in [Specification 24](24-cross-module-transaction-and-implementation-contract.md) are mandatory. In particular, sale commitment and stock issue are atomic; external payment confirmation and settlement are separate, independently verified state transitions.

## 3. Sale lifecycle

Candidate sale states:

- **Draft:** editable cart; no stock or financial commitment.
- **Pending validation:** submitted for server-side validation.
- **Committed:** sale transaction and stock issue movements are durably recorded.
- **Voided:** sale has been voided through an authorized linked correction workflow.
- **Partially returned / returned:** one or more sale lines have an accepted return.
- **Correction pending:** a linked void, return, or other correction is awaiting required review.

Payment states belong to the payment record, not the sale state. A committed sale may have one or more payment records that are pending, partially settled, confirmed, failed, reversed, refunded, or disputed according to the supported policy. The UI may display a derived combined summary, but the database must not collapse sale lifecycle and payment lifecycle into one state machine.

The final sale state machine must define allowed transitions, who may initiate/approve them, whether they are reversible, and their effect on stock and financial records. A sale must not be silently deleted.

## 4. Sale draft and validation

A draft captures:
- tenant and branch;
- cashier identity and active session;
- product/presentation and quantity/unit;
- requested batch/allocation constraints where applicable;
- price source and proposed discount/override;
- tax configuration reference and provisional calculation;
- customer reference only if necessary and legally approved;
- prescription/authorization workflow reference only where the product category requires it;
- client-generated command ID and draft version.

On submission, the server revalidates:
- membership, branch assignment, permission, and session;
- product identity and sale eligibility;
- batch status, expiry, recall/hold status, and available quantity;
- price and discount authority;
- tax/pricing configuration version and effective date;
- any required professional or other regulated authorization for the exact item/action;
- duplicate command and concurrent version;
- branch business state and configured sales policy.

The server returns a structured validation result. The UI must show which line failed and why. It must not silently substitute a different product, strength, presentation, batch, price, or tax treatment.

## 5. Pricing, discounts, tax, and rounding

- Store the applied unit price and relevant price-source/version snapshot on each finalized sale line.
- Store the discount rule, amount, authorizer, reason, and policy version when a discount or override is used.
- Store the tax configuration/version and applied calculation snapshot with the finalized transaction.
- Use exact decimal or integer minor-unit arithmetic with explicit currency and rounding policy.
- Define line-level versus invoice-level rounding and prove totals reconcile.
- Later changes to a product price, discount policy, or tax configuration must not rewrite finalized sales.
- Do not hard-code a tax rate or claim tax compliance before qualified local accounting/legal review.
- A zero-price, free, promotional, or comped line must have an explicit reason and permission policy.

Open decisions include whether prices are tax-inclusive or tax-exclusive, how discounts are allocated across lines, how rounding is applied, and what invoice/receipt fields are legally required for each supported establishment type.

## 6. Stock allocation and sale commit

The sale service uses the inventory rules in [Specification 13](13-inventory-and-batch-lifecycle-specification.md).

- Allocate only eligible stock at the correct branch.
- Apply the approved batch-selection policy and preserve the exact allocation in sale lines and stock movements.
- Use a transaction or equivalent concurrency-control strategy to ensure the stock issue cannot race another sale or adjustment.
- A failed payment must not cause the application to invent a successful sale or silently restore stock without a linked policy-driven correction.
- If the business policy allows an unpaid committed sale, the receivable/payment obligation and stock issue must be explicit and auditable.
- If policy requires payment before commit, stock must not be issued until authoritative payment confirmation and the commit sequence is complete.
- The exact commit/payment ordering must be selected per supported payment method and tested under timeout, duplicate callback, and provider outage.

## 7. Payment lifecycle

Payment methods may include cash, bank transfer, card, or approved local provider methods, but each must have its own state and reconciliation policy. A method is not production-enabled merely because it appears in a dropdown.

Candidate payment states:
- initiated;
- awaiting customer/provider action;
- pending confirmation;
- confirmed;
- failed;
- cancelled;
- expired;
- reversed;
- partially refunded;
- refunded;
- disputed/reconciliation required.

Each payment record should include:
- payment ID and sale reference;
- method and provider, if applicable;
- amount and currency;
- provider transaction/reference ID;
- idempotency key;
- initiation and confirmation timestamps;
- authoritative confirmation source;
- status transitions and reason;
- fees/settlement details where available and relevant;
- reconciliation state and discrepancy reference.

For external providers:
- verify callbacks/webhooks cryptographically according to provider requirements;
- do not trust client-submitted payment status;
- make callbacks idempotent and tolerate duplicate/out-of-order delivery;
- reconcile provider transactions against Pharmalink records;
- handle delayed, missing, reversed, and disputed settlements;
- never store provider secrets in client code or logs.

For cash:
- record the cashier/session, amount tendered, change, time, and any permitted correction;
- define cash drawer opening/closing and discrepancy handling if cash sessions are in scope;
- do not equate a cash entry with a reconciled bank deposit.

For bank transfer/manual confirmation:
- define acceptable evidence and who may confirm receipt;
- prevent the person initiating a high-risk payment record from approving their own confirmation where separation of duties is required;
- record pending/unverified status until evidence is reviewed.

## 8. Receipt and invoice records

A receipt/document is a durable representation of a finalized transaction and must reference its source sale and numbering policy.

Define:
- document type and legal purpose;
- unique numbering scope and concurrency strategy;
- required seller/branch identity;
- transaction date/time and timezone;
- line descriptions, units, quantities, prices, discounts, taxes, and totals;
- payment status and method as known at issuance;
- correction/void reference where applicable;
- language and localization behavior;
- print/download/email delivery state.

A failed printer or email must not undo a committed sale. Reprinting must be marked or tracked according to the approved document policy and must not create a second sale. Receipt numbering gaps and cancelled documents need an explicit audit treatment.

## 9. Voids, corrections, and refunds

- A void request references the original sale and identifies the reason, actor, time, and affected lines.
- Approval requirements depend on amount, timing, role, and policy; the final matrix must define them.
- A void creates linked correction records. It does not erase the original sale.
- Stock restoration is handled through a validated inventory movement and only when the physical goods are confirmed eligible.
- A refund is a payment transaction with its own provider/cash reference and lifecycle. It is not merely a negative sale amount.
- Partial refunds must reference the relevant sale lines and quantities/amounts.
- A refund can be pending, failed, reversed, or disputed. The UI must not show it as completed before authoritative confirmation.
- Duplicate void/refund requests must be idempotent.
- Late corrections and closed accounting periods require an explicit approval and accounting treatment.

## 10. Cash sessions and reconciliation

If cash register/shift functionality is included in the first release, define:
- session opening float and authorized opener;
- cash sales, refunds, paid-in/paid-out movements;
- cash count and expected balance;
- variance and explanation;
- reviewer/closer and approval;
- lock/close state and correction process;
- reconciliation to deposits and accounting records.

Do not infer a cash drawer balance solely from sale totals if other cash movements are possible. A closed session must not be silently reopened or rewritten; corrections require linked records.

## 11. Offline and event-driven behavior

- Sale finalization and payment confirmation are online-only by default.
- A local cart/draft may be saved offline only if the local data policy permits it; it is not a committed sale and must be visibly labeled.
- Do not queue a sale for automatic finalization later unless a separate offline sales design proves stock allocation, price validity, authorization expiry, payment, receipt numbering, and duplicate handling safe.
- A client timeout after submit must query the command outcome before retrying.
- SaleCommitted, PaymentStatusChanged, SaleVoided, ReturnAccepted, and RefundStatusChanged are candidate domain events.
- Publish events through the transactional outbox after the authoritative transaction commits.
- Clients subscribe to relevant branch/session changes; do not poll continuously for unchanged sales/payment state.
- Payment-provider callbacks are external events and must be verified, deduplicated, and reconciled.

## 12. Permissions and separation of duties

Candidate permissions:
- sale.create;
- sale.read;
- sale.discount.request;
- sale.discount.approve;
- sale.commit;
- sale.void.request;
- sale.void.approve;
- customer_return.create;
- customer_return.submit;
- customer_return.review;
- payment.initiate;
- payment.confirm_manual;
- payment.refund.request;
- payment.refund.approve;
- receipt.read;
- receipt.reissue;
- finance.cash_session.open;
- finance.cash_session.count;
- finance.cash_session.close;
- finance.read;
- finance.reconciliation_exception.resolve;
- sales.export.

Canonical identifiers and the unresolved role-grant register are maintained in [Specification 18](18-proposed-permission-matrix.md). Role grants, branch scope, amount thresholds, approval separation, re-authentication, audit events, and offline eligibility must be approved before enabling each action. Ordinary in-store POS permissions do not imply professional authority for regulated dispensing. Professional credential checks apply only to the regulated action that requires them.

## 13. Required test scenarios

Automated tests must prove:
- duplicate sale submission produces one committed sale and one set of stock movements;
- two concurrent sales cannot oversell the same eligible batch;
- stale price/tax configuration cannot silently rewrite or override server policy;
- unauthorized discount, override, void, refund, or manual payment confirmation is denied;
- payment initiation is not treated as payment confirmation;
- forged, invalid, duplicate, delayed, and out-of-order provider callbacks cannot corrupt payment state;
- a timeout after commit can be resolved without duplicate sale/payment;
- a failed print/email does not undo the sale;
- receipt reissue does not create a new sale;
- return stock is not automatically available for sale;
- customer-return acceptance requires the distinct customer_return.review permission and cannot be performed by an unauthorized requester;
- refund approval is separate from return acceptance, and neither action implies the other;
- return stock disposition requires inventory.return.disposition and cannot be performed by a customer-return reviewer solely by virtue of that review permission;
- partial return/refund totals reconcile to the original sale;
- cash session variance is traceable;
- expired/revoked membership and professional credentials block only the actions that depend on them;
- private POS works when public listing is disabled;
- public APIs do not expose private sale, payment, customer, or cash-session data.

## 14. Open decisions before implementation

1. Which sale and payment methods are included in the first production release?
2. Is payment required before sale commit, or can an explicitly recorded receivable be created?
3. Which local payment providers are technically and contractually available, and what are their webhook/reconciliation guarantees?
4. What receipt/invoice numbering and content requirements apply to each launch establishment category?
5. What tax calculation and rounding policy is approved by Ethiopian accounting/tax reviewers?
6. Which discount, void, refund, and manual payment actions require second approval?
7. Are cash sessions/shift reconciliation part of the first release?
8. Which product categories require prescription/regulated authorization checks, and which workflows remain excluded pending legal review?
9. What is the return eligibility and stock disposition policy by product category?
10. What are the supported currency, language, calendar, and timezone behaviors for transaction documents?
11. What accounting exports and reconciliation formats do pilot pharmacies actually need?
12. What transaction volume, cashier concurrency, and receipt-printing hardware must be supported?

No payment provider, tax treatment, regulated dispensing workflow, or public commerce flow should be marked production-enabled until the relevant evidence and approval are recorded.


## 15. Cross-module transition alignment

The shared sale, payment, return, refund, cash-session, and recovery transition contract is defined in [Specification 30](30-workflow-state-transition-and-recovery-contract.md). The sale and payment state machines remain separate. Method-specific payment ordering is not approved until the open decisions in this document are resolved.
