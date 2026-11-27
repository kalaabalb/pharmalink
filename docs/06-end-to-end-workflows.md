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

**Invariant:** account access, organization membership, private management setup, establishment evidence review, professional credential verification, and public participation are separate workflows. A person does not need an individual professional licence merely to register, accept an invitation, sign in, or use ordinary permitted management functions.

1. A person creates an account or accepts an invitation using the platform's identity and anti-abuse controls. No professional credential is requested as a general account prerequisite.
2. The person creates an organization workspace or accepts an organization invitation. The system checks for likely duplicates and records the applicant's authority to create or administer the workspace without treating that assertion as proof of establishment licensing.
3. The organization selects its intended operating configuration. Management-only is the default; public participation and commerce-related capabilities remain off unless explicitly selected and separately approved.
4. The organization records its business identity and adds one or more draft branches. Branch setup can proceed for private management configuration without claiming that the branch is legally authorized to operate.
5. The organization may submit establishment evidence when required for a specific operation, establishment category, public trust claim, or legal review. The evidence record identifies the relevant organization/branch, issuing authority, jurisdiction, evidence type, validity dates, source, and verification status.
6. The owner or authorized administrator invites staff and assigns tenant- and branch-scoped permissions. Ordinary staff membership does not require an individual professional licence.
7. The system requests professional credential evidence only when a defined role, action, or public verification claim requires it. The credential is linked to the person and its verified scope, not treated as an organization-wide credential.
8. The organization configures private operational settings such as locale, currency, opening balances, stock units, tax configuration status, receipt preferences, and branch controls. Unresolved tax or regulatory configuration must be visible and must not be represented as legally approved.
9. The system evaluates each requested capability independently. Missing or pending establishment evidence may block a particular legally dependent operation or public claim; it must not silently disable unrelated management features.
10. The organization reviews and confirms its membership, role, branch, and configuration setup. Every material change is audited.

**Separate public-participation path:** the organization explicitly opts into a public profile, product discovery, or availability publication. Each capability has its own eligibility rules, data disclosure, review, publication state, pause/withdrawal control, and audit trail. Commerce is not enabled by public listing and requires its own legal, provider, payment, consumer-protection, and operational approval.

**Failure and recovery paths:** duplicate organization; invitation sent to the wrong identity; applicant authority disputed; incomplete or inconsistent evidence; establishment evidence expired or rejected; professional credential not verified or outside scope; source registry unavailable; reviewer conflict; appeal; branch address correction; public listing rejected or paused; and account recovery. These failures must produce a scoped outcome and clear next action. They must not convert one failed evidence check into an unrelated account-wide lock.

**Important boundary:** Pharmalink workspace creation is not an establishment licence, and a management account is not permission to conduct a regulated professional act. This workflow does not waive any legal obligation to lawfully establish or operate a physical pharmacy.

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

1. An authorized organization representative explicitly opts the organization/branch into a specific public capability and selects the fields to publish.
2. The platform evaluates eligibility and the establishment evidence required for that exact capability under the reviewed rules for the relevant jurisdiction and establishment category. Do not infer that every public field requires the same evidence.
3. A public verification claim is shown only when the exact subject, evidence type, issuer, verification method, scope, and current status support that claim. If verification cannot be completed, do not imply that it has been completed. Whether a non-verified profile may publish at all remains a legal/product decision for the launch jurisdiction.
4. The branch publishes only approved, intentionally public profile information. Private operational records remain inaccessible through public endpoints.
5. Availability is derived from eligible branch stock and a successful server-side confirmation. The public view includes a last-confirmed time and freshness state.
6. When the data exceeds the freshness threshold, it becomes stale or unknown rather than remaining “available”.
7. A pharmacy can pause publication or a specific listing, with an auditable reason. Withdrawal removes the public projection but preserves private source records and publication history.
8. Public discovery is informational only in the initial scope; it does not reserve stock, accept prescriptions, authorize dispensing, or guarantee that stock remains available. See [Specification 19](19-public-discovery-and-availability-specification.md) for publication consent, public projection, privacy, freshness expiry, moderation, and appeal requirements.

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

## 14. Management-only mode and workflow linkage

The authoritative organization onboarding and branch-setup sequence is defined in [Workflow 2](#2-organization-onboarding-and-branch-setup). This section is retained as a cross-workflow invariant:

- Management-only is the default operating mode and a complete supported product mode.
- Account creation, organization membership, and ordinary permitted management use do not require an individual's professional licence.
- Establishment evidence and professional credentials are separate evidence records. They are requested only for a specific operation or verification claim that requires them.
- Public profile, product discovery, availability publication, and any future commerce capability are independent opt-ins with separate eligibility and lifecycle controls.
- Turning off public participation must not disable private management or erase private operational history.
- Platform access is not proof of establishment licensing and does not authorize a regulated professional act.

These boundaries apply to all workflows in this document. A capability-specific legal review may restrict a particular operation, but must not be converted into an unrelated account-wide gate.
