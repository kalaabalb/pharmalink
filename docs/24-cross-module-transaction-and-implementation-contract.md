# 24 — Cross-module transaction and implementation contract

**Status:** Draft — mandatory consistency contract; must be reviewed with Specifications 07–18 before implementation  
**Purpose:** Resolve shared transaction, state, identity, authorization, and event rules that otherwise risk being implemented differently across modules.

## 1. Authority and relationship to other specifications

This document does not replace module specifications. It defines cross-module invariants that every module must obey. If a module appears to conflict with this contract, the conflict must be resolved explicitly and the affected documents updated before implementation. Do not silently choose whichever interpretation is easiest to code.

Primary dependencies:
- Specification 07 — domain model and data ownership.
- Specification 08 — event-driven and offline requirements.
- Specification 12 — functional product and production deployability contract.
- Specification 13 — inventory, batch, and stock movements.
- Specification 14 — sales, POS, and payment lifecycle.
- Specification 15 — supplier and purchasing lifecycle.
- Specification 16 — medicine catalogue and product identity.
- Specification 17 — finance and reconciliation.
- Specification 18 — role and permission matrix.
- Specifications 20–23 — organization lifecycle, integrations, privacy, and UX.
- Specification 29 — application architecture, API, database, and deployment contract.

## 2. Canonical business facts and ownership

Each business fact has one authoritative owner. Other modules may consume it through a documented command, query, or event, but must not independently create or mutate a competing version of the same fact.

| Business fact | Authoritative owner | Other modules must not |
|---|---|---|
| Canonical medicine/product identity | Catalogue domain | Infer or silently merge canonical identities |
| Tenant-local item enablement and local alias | Tenant catalogue | Mutate platform canonical identity |
| Branch price and price policy | Commercial configuration owned by tenant/branch | Rewrite finalized sale prices |
| Batch identity and physical stock movement | Inventory domain | Change stock by editing a balance |
| Current stock position | Inventory projection from accepted movements | Treat a cached/client balance as authoritative |
| Sale and sale-line commitment | Sales domain | Recreate a sale from a receipt or payment callback |
| Payment attempt, confirmation, refund, settlement evidence | Payment domain/provider adapter with tenant transaction references | Treat initiation or UI success as confirmation |
| Supplier obligation and invoice acceptance | Purchasing/finance under approved accounting policy | Infer a payable from a purchase order alone |
| Financial posting and reconciliation outcome | Finance domain | Recompute or overwrite finalized entries silently |
| Membership and permission grant | Organization/access domain | Trust client-supplied role or branch identifiers |
| Public listing and availability projection | Public discovery domain | Read or expose private operational tables directly |

Derived projections must be rebuildable or reconcilable from their authoritative source. A projection failure must not mutate the underlying business fact.

## 3. Transaction boundaries: internal sale versus external payment

### 3.1 Internal sale commitment

The sales command must validate current server-side authorization, branch scope, item eligibility, pricing snapshot, required regulated-action authorization, and stock allocation. The sale commit and its stock issue movement(s) must succeed or fail together within the agreed database transaction boundary.

The commit must also persist:
- sale and line identifiers;
- applied price, discount, tax/configuration snapshots where applicable;
- exact stock allocation and movement references;
- actor, branch, command/idempotency key, and audit references;
- a durable outbox record for committed domain events.

A client timeout after commit is an unknown client outcome, not proof of failure. The client must query by command/idempotency reference before attempting another commit.

### 3.2 Payment initiation and confirmation

Payment is a separate state machine. Initiating a provider request does not make a sale paid. A provider callback or reconciliation result is an external input that must be authenticated, validated, deduplicated, correlated to the intended payment attempt, and processed idempotently.

The payment state transition and its own outbox event commit atomically in Pharmalink's database. Provider-side effects cannot be included in a local database transaction; use explicit pending/unknown states and reconciliation for ambiguous outcomes.

A sale may be:
- committed and unpaid;
- committed and partially paid, if the approved policy permits split/partial payment;
- committed and paid after authoritative evidence;
- committed with a payment dispute, reversal, or refund represented separately.

The exact permitted combinations and whether unpaid sales are allowed are first-release policy decisions. Never infer “paid” from a successful API request, redirect, client notification, or unverified webhook.

### 3.3 Refunds and returns

A refund is a financial/payment fact. A product return is an inventory disposition workflow. They may be related but are not interchangeable and need not occur at the same instant.

A refund must reference the original payment/sale and provider or cash evidence. Returned goods must enter the inventory return/inspection process and must not automatically become available stock. A refund does not itself create a stock movement; a return does not itself prove a refund.

## 4. Purchasing and finance transaction boundaries

- Purchase order approval/issue is not receipt, invoice acceptance, payable recognition, or payment.
- Accepted goods receipt creates inventory movements. Receipt acceptance and stock creation must be atomic for the accepted quantities.
- Supplier invoice recording is not invoice approval. Invoice approval and payable recognition follow the approved accounting policy.
- Supplier payment initiation, confirmation, bank settlement, and reconciliation are distinct states.
- A provider/bank import must be idempotent and preserve source-file identity, row identity, import version, and rejected/ambiguous matches.
- A purchase receipt, supplier invoice, and payment must retain explicit references without forcing one-to-one relationships; partial receipts, multiple invoices, credit notes, and partial payments may exist where the approved workflow supports them.

The initial release must decide whether it provides operational cashbook/summary reporting, journal export, or a full general ledger. These are different product commitments. Do not label one as another.

## 5. Identity, unit, and quantity consistency

The following hierarchy is mandatory:
1. Canonical catalogue identity.
2. Exact product presentation.
3. Pack configuration and unit conversion.
4. Tenant-local catalogue entry.
5. Batch/lot identity.
6. Branch/location stock position derived from movements.

Every quantity-bearing transaction must record the unit used and the exact conversion/version applied. Conversion changes must not rewrite historical transactions. Ambiguous product identity or unit conversion blocks finalization; it must not be guessed from a display name or barcode alone.

A branch transfer is a custody workflow with dispatch and receipt legs. It is not a direct edit to two branch balances. In-transit quantity must not be counted as available at either branch.

## 6. State-machine and correction rules

Every durable lifecycle must define:
- named states and their meaning;
- allowed transitions and forbidden transitions;
- initiating and approving permissions;
- required evidence and reason codes;
- whether a transition is reversible;
- linked compensating action;
- event emitted after commit;
- timeout, retry, and recovery behavior.

Finalized stock, sale, payment, supplier, and financial facts are not silently edited or deleted. Corrections append linked reversals, adjustments, returns, credit/debit records, or other approved compensating facts. A correction must preserve the original record and identify who authorized it and why.

A UI may present a combined status for convenience, but the underlying independent state machines remain distinct. For example, “sale complete” must not collapse sale commitment, payment confirmation, and settlement into one unqualified boolean.

## 7. Authorization is evaluated at execution time

- The server derives the actor from the authenticated session, not a client-supplied actor ID.
- Tenant and branch scope are checked for every command, query, export, subscription, background job, and event delivery.
- Permission is re-evaluated when a queued command is synchronized; a formerly valid grant does not guarantee current authorization.
- Role templates are convenience groupings. Effective access is explicit, scoped, time-bounded where appropriate, and constrained by separation of duties.
- Professional credential checks apply only to the exact action that requires a verified credential under reviewed rules. They do not become a blanket account, membership, or ordinary management gate.
- Establishment evidence, individual professional credentials, platform entitlements, and public verification claims remain distinct.
- Support access is denied by default and requires a purpose-bound, approved, time-limited grant with audit.

## 8. Event and projection contract

For each committed authoritative state change:
1. Persist the domain fact and outbox event in the same local database transaction.
2. Publish asynchronously with bounded retry and observable delivery state.
3. Deduplicate consumers by stable event ID and enforce aggregate version/order rules.
4. Treat replay as re-delivery of an existing fact, never as permission to repeat its business side effect.
5. Keep payloads minimal; do not publish personal/health data, secrets, credentials, or private quantities to consumers that do not need them.
6. Make public projections consume only explicitly publishable, privacy-reviewed fields.
7. Provide cursor-based recovery or authoritative snapshot reconciliation after missed events.

A notification, UI refresh, search index update, or analytics update is a downstream projection. It is not the authoritative transaction. Normal live updates use subscriptions/push rather than frequent polling.

## 9. Offline command contract

Offline drafts are not committed business facts. Before an offline command capability is approved, its module must define:
- allowed operation and risk rationale;
- encrypted local data and device trust requirements;
- actor/tenant/branch scope and authorization expiry;
- unique command/idempotency identity;
- maximum queue age;
- server revalidation and conflict handling;
- duplicate/replay behavior;
- user-visible pending, accepted, rejected, and needs-review states;
- device loss, revocation, and local data deletion behavior.

Until a specific operation has an approved offline policy, it is online-only. This is particularly important for sale finalization, stock movement, payment, refund, regulated actions, and approval decisions.

## 10. Required cross-module negative tests

The implementation must prove at least:
- a payment initiation or unverified callback cannot mark a sale paid;
- duplicate sale commit requests cannot create duplicate sales or stock issues;
- a sale cannot commit against expired, recalled, held, damaged, or insufficient stock;
- a refund cannot silently restore stock;
- a returned item cannot become available without an authorized inventory disposition;
- a purchase order or invoice upload alone cannot create physical stock;
- a goods receipt cannot silently create a payable unless the approved accounting policy explicitly defines that posting;
- a payment cannot be allocated to another tenant's sale by guessing an identifier;
- unit conversion changes do not alter historical sale/receipt quantities;
- a branch transfer cannot make the same quantity available at both ends;
- a role or credential revoked before synchronization blocks the queued command where required;
- a replayed event cannot repeat stock, payment, refund, or financial effects;
- a public projection cannot reveal private stock, customer, supplier, staff, or financial data;
- a support operator cannot access tenant records without a valid purpose-bound grant;
- subscription suspension cannot erase tenant data or block lawful export/required access.

## 11. Decisions required before affected capabilities are implemented

| Decision | Required owner/evidence | Safe interim rule |
|---|---|---|
| First-release establishment and product categories | Product owner + pharmacy field research + regulatory review | Do not enable unreviewed categories |
| Whether unpaid/partially paid sales are permitted | Product owner + accounting review | Do not assume; define explicit policy |
| Supported payment methods and provider | Product owner + provider due diligence + payment/legal review | No live payment claim until production provider is verified |
| Refund and return policy | Pharmacy research + accounting/legal review | Keep refund and inventory disposition separate |
| Unit hierarchy and conversion governance | Pharmacy operators + catalogue governance | Block ambiguous conversions |
| Stock allocation policy and negative-stock rule | Pharmacy research + regulatory review | Do not oversell; do not silently create negative stock |
| Initial finance scope (cashbook, export, or ledger) | Product owner + qualified accountant | Do not claim full accounting compliance |
| Receipt/invoice numbering and statutory fields | Qualified Ethiopian accountant/counsel and applicable authority | Do not claim statutory compliance before validation |
| Professional credential gates by action | Qualified Ethiopian legal/regulatory review | No blanket platform gate; block only unreviewed regulated action |
| Offline-enabled operations | Product owner + threat/risk analysis + field research | Online-only for finalization and sensitive writes |
| Public availability freshness thresholds | Pharmacy research + product owner | Unknown/stale when freshness cannot be established |

## 12. Architecture implementation alignment

The module and transaction rules in this contract must be realized through the enforceable boundaries, API semantics, database constraints, transactional outbox, and deployment controls in [Specification 29](29-application-architecture-api-database-and-deployment-contract.md). The architecture may choose different technologies, but it may not weaken these business invariants or introduce undocumented cross-module writes.

## 13. Readiness rule

A module is not implementation-ready merely because its own document has a workflow diagram or a list of states. Its state transitions, transaction boundaries, permissions, events, data ownership, privacy rules, failure recovery, and acceptance tests must agree with this cross-module contract and with every upstream/downstream module it touches.

Any unresolved decision must be either:
- resolved with evidence and recorded in the relevant specification;
- explicitly excluded from the approved first-release scope; or
- assigned a safe, visible interim rule that prevents unsafe or misleading behavior.

Do not turn unresolved decisions into undocumented developer assumptions.
