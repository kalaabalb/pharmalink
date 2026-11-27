# 06 — End-to-end workflows

**Status:** Draft — workflow requirements; regulated workflows require qualified local review  
**Purpose:** Define lifecycle behavior before screen design or implementation.

## 1. Workflow notation and invariants

Each workflow must define its initiating actor, scope, preconditions, state transitions, side effects, failure behavior, audit events, notifications, and acceptance tests.

Global invariants:
- Every operational record belongs to exactly one tenant and, where operationally relevant, one branch.
- A committed transaction has one authoritative outcome; retries must not create duplicate business effects.
- A finalized financial or stock record is not silently overwritten. Corrections are linked and attributable.
- Every stock-affecting event is traceable to a source transaction and a responsible actor or system process.
- A user must not be shown an action they cannot perform, and the server must independently enforce that restriction.
- A network timeout does not prove that a transaction failed. The client must resolve the transaction outcome by idempotency key or server acknowledgement before retrying.
- The product records operational facts; it does not diagnose, prescribe, recommend treatment, or independently authorize regulated professional decisions.

## 2. Organization onboarding and branch setup

1. An applicant creates an organization profile and identifies the establishment type and operating location.
2. The applicant submits the evidence required for that establishment category and identifies the responsible professional where applicable.
3. The platform records each evidence item, source, submission time, review state, and reviewer decision.
4. The organization remains in a restricted onboarding state until required checks are satisfied.
5. An approved owner creates or activates a branch only when the necessary establishment evidence is accepted.
6. The owner invites staff and assigns scoped permissions.
7. The owner confirms the initial operating configuration, including locale, currency, tax configuration status, receipt preferences, and permitted service capabilities.
8. The branch becomes operational only after required setup gates pass.

Failure paths: duplicate organization, incomplete evidence, inconsistent address, expired credential, rejected application, reviewer conflict, appeal, and regulator/source verification outage. A submitted application is not an approved licence.

## 3. Catalogue setup and data correction

1. The operator searches the canonical catalogue before creating a local item.
2. The system presents candidate matches with provenance and distinguishing attributes.
3. A user with permission links the local sellable item to a canonical record or submits a proposed correction/new record.
4. The system preserves the source and review status of the catalogue data.
5. Local commercial attributes (such as local display name or barcode mapping) remain distinct from authoritative product identity.
6. Catalogue changes that affect multiple tenants require controlled publication, versioning, and a correction trail.

The product must not invent product identity, regulatory status, professional guidance, or clinical equivalence when source data is missing. Matching uncertainty must be visible and routed for review.

## 4. Supplier and purchasing workflow

1. An authorized user maintains supplier identity and contact records.
2. A purchasing operator drafts a purchase request or order with line items and expected quantities.
3. Required approval is obtained before the order is issued.
4. The supplier response and any order amendments are recorded as revisions.
5. Goods receipt is recorded separately from order creation and approval.
6. The receiving record captures discrepancies and relevant traceability attributes from the source documentation.
7. Accepted receipt entries update the branch stock ledger through a single committed operation.
8. Supplier invoices and payment status are reconciled separately from physical receipt.

Partial receipt, over/under delivery, damaged goods, duplicate supplier documents, cancelled orders, and offline receipt attempts must have explicit outcomes. The product must not assume that an order is proof that goods were received.

## 5. Inventory operations

Supported workflow categories include opening balances, receipt, transfer, count, adjustment, quarantine/hold, and authorized disposition. Each movement must have a source, destination where relevant, reason, actor, time, and reference.

A count is a measurement, not an automatic overwrite. Variances become reviewable adjustment proposals. Transfers must have explicit in-transit and received states if the product supports multi-step movement; stock must not appear simultaneously available at both locations.

Batch/lot and expiry attributes must remain attached to the stock units they describe. Missing or unreadable traceability information must be represented as unknown, not fabricated. Quarantined, recalled, expired, or otherwise blocked stock must be excluded from ordinary availability and operational selection by server-enforced rules.

## 6. Point-of-sale transaction lifecycle

This document specifies software integrity, not clinical or dispensing instructions.

1. An authorized operator opens a sale transaction within one branch.
2. The system resolves the selected catalogue item to the branch’s permitted stock records and validates current state.
3. The system computes totals using the configured, versioned pricing and tax rules.
4. The operator confirms the transaction and the server atomically commits the sale, financial entries, and corresponding stock movement.
5. The system returns a durable transaction identifier and receipt representation.
6. If payment is external, payment initiation, provider confirmation, reconciliation, and sale finalization are distinct states.
7. If a timeout occurs, the client checks the original transaction identifier/idempotency key before retrying.
8. A correction follows a controlled void/return/reversal workflow; it does not erase the original sale.

The application must never represent an unconfirmed payment as paid. Cash, card, transfer, and other methods are configurable only after legal, provider, and accounting review.

## 7. Returns, voids, and corrections

A return or void references the original transaction and records its reason, actor, time, approval, and financial treatment. Returned goods are not automatically returned to saleable stock: their condition and disposition must be explicitly reviewed under an approved policy. Partial returns, duplicate requests, late returns, and already-reversed transactions require deterministic outcomes.

## 8. Expiry, quality, and recall handling

The product must support review queues and traceable status changes for stock that requires attention. A recall/quality notice must be recorded with source, scope, affected catalogue or batch identifiers where reliably known, and the authority or reviewer responsible for the decision. Matching uncertainty must be visible. The system should identify potentially affected records for authorized review; it must not claim a complete recall match when source identifiers are incomplete.

## 9. Public discovery and availability publication

1. A pharmacy explicitly opts into public listing and selects the fields it will publish.
2. The platform verifies the establishment status according to its documented verification method.
3. The branch publishes only approved public profile information.
4. Availability is derived from eligible branch stock and a successful server-side confirmation.
5. The public view includes a last-confirmed time and freshness state.
6. When the data exceeds the freshness threshold, it becomes stale or unknown rather than remaining “available”.
7. A pharmacy can pause publication or a specific listing, with an auditable reason.
8. Public discovery is informational only in the initial scope; it does not reserve stock, accept prescriptions, authorize dispensing, or guarantee that stock remains available.

## 10. Offline operation and recovery

The application may support only a specifically approved set of offline actions. The exact list remains open. Every offline-capable action must define local durability, encryption, device identity, authorization expiry, conflict handling, retry behavior, and what the user sees while pending.

A local pending transaction is not globally committed. The interface must distinguish locally saved, queued, submitted, accepted, rejected, and needs-review states. The system must never claim that public availability is current based solely on an offline device.

## 11. Subscription and account lifecycle

Trial, subscription, invoice, payment, grace period, suspension, cancellation, and reactivation are separate states. Commercial terms remain unapproved. Subscription suspension must not destroy access to legally or operationally necessary records, audit history, data export, or support/appeal processes. Any restriction on creating new transactions requires a documented policy and legal review.

## 12. Platform support and verification workflow

Support cases are separate from tenant operational records. A case may request limited, time-bound access to specific evidence. Every privileged access session is logged. Verification reviewers must be prevented from reviewing their own submissions, and decisions must preserve evidence, rationale, and appeal history.

## 13. Workflow acceptance checklist

For every workflow, create tests for:
- valid normal path;
- missing or expired permission;
- wrong tenant or branch;
- duplicate submission and retry;
- concurrency conflict;
- partial completion;
- timeout after server commit;
- offline interruption and later replay;
- invalid state transition;
- audit and notification completeness;
- reversal/correction path;
- data export and retention implications.

## 14. Management-only onboarding and optional public participation

1. A retailer creates an account and an organization profile using the minimum information required to establish the tenant and secure its access.
2. The retailer selects management-only use by default; no public listing or selling is enabled implicitly.
3. The retailer configures branches, staff, roles, catalogue, suppliers, inventory, and other permitted management capabilities.
4. The retailer may separately opt into a public business profile, product discovery, and availability publication.
5. Each public capability has its own eligibility checks, disclosure, publication controls, and pause/withdrawal behavior.
6. Any future ordering, reservation, payment, or delivery capability is a separate opt-in workflow and must not be assumed merely because a pharmacy is listed.
7. Professional credentials are requested only when a specific regulated workflow or legal verification requirement needs them. A missing professional credential must not block unrelated management-only use.

A retailer can turn public participation off while retaining its management account and historical records. Platform access does not itself certify that the establishment is licensed, nor does it authorize any regulated professional activity.
