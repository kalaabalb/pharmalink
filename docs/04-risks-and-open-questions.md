# 04 — Risks and open questions

**Status:** Draft — living risk register  
**Rule:** A risk is not resolved merely because a design proposal exists.

## 1. Product and operational risks

| Risk | Consequence | Required treatment |
|---|---|---|
| Public availability becomes stale | Public users are misled | Timestamped, expiring availability; stale/unknown state; pause control |
| Batch/expiry data is missing or incorrect | Unsafe or noncompliant stock handling | Provenance, validation, quarantine, review, traceable corrections |
| Concurrent sales or adjustments | Duplicate or inconsistent stock | Server-side atomic operations, version checks, idempotency |
| Offline writes conflict | Data loss or false confirmation | Explicit offline capability matrix; queued/pending state; deterministic conflict handling |
| Tenant isolation failure | Confidential business data exposure | Server-enforced tenant/branch checks; adversarial tests |
| Unverified establishment or credential | False trust signal | Evidence-based verification states and re-verification |
| Catalogue matching error | Wrong item association | Source provenance, confidence, human review, correction workflow |
| Payment provider outage or duplicate callback | Incorrect payment/sale status | Idempotency, reconciliation, distinct payment states |
| Lost or shared device | Unauthorized access or local data exposure | Device registration, local encryption, revocation, minimal cache |
| Backup cannot be restored | Irrecoverable business records | Restore drills, integrity checks, documented recovery objectives |
| Subscription suspension blocks necessary records | Business disruption or unlawful retention failure | Preserve read/export/audit/support access; review restrictions |
| AI output is wrong or overreaches | Unsafe or misleading operational decisions | Optional, permission-aware assistance; no clinical decision-making; human review |
| Platform support has excessive access | Privacy breach and abuse risk | Case-scoped, time-limited, logged access |
| Event delivery failure | Stale clients and inconsistent projections | Transactional outbox, replay, reconciliation, delivery monitoring |
| Regulatory interpretation is wrong | Product behavior may breach law | Primary-source register, qualified Ethiopian review, controlled release gates |

## 2. Regulatory questions requiring primary-source review

- Which federal and regional authorities regulate each establishment category and professional credential?
- Which establishment licence and professional credential evidence must be collected, verified, renewed, suspended, or rechecked?
- Which medicine/product categories may be listed or handled by each supported establishment type, and which rules govern regulated workflows?
- What are the requirements for records, prescription-related records, retention, reporting, and inspection access?
- What restrictions apply to medicine advertising, public catalogue display, pricing, and availability publication?
- What rules apply to online ordering, reservations, delivery, and cross-region activity?
- What personal-data roles, lawful bases, notices, rights, security controls, breach duties, and cross-border transfer requirements apply?
- What tax, invoicing, fiscal-device, accounting, and payment-provider obligations apply?
- Which national and regional rules differ, and how are conflicts resolved?

## 3. Market and workflow questions

- Which pharmacy/establishment types are the initial target, and which are explicitly out of scope?
- What systems and paper processes are used today for receiving, inventory, sales, returns, cash, and reporting?
- What devices, printers, scanners, connectivity, and power conditions are common?
- What is the minimum useful offline capability without compromising record integrity?
- Which languages and calendar/date conventions are needed at launch?
- What evidence do pharmacy owners need before trusting public availability information?
- What subscription model and support expectations are viable? Pricing and trial terms are not approved.
- Which operational pain point should define the first release rather than attempting to ship every capability at once?

## 4. Catalogue and traceability questions

- Which authoritative or licensed data sources can seed the catalogue?
- Who owns correction decisions, and how are conflicting sources resolved?
- How should pack, presentation, barcode, brand/generic aliases, and local display names relate?
- What minimum batch/lot and expiry data is required for each supported workflow?
- How are recalls, quality notices, and uncertain catalogue matches handled?

## 5. Offline and event questions

- Which actions may be read offline, drafted offline, queued, or must remain online-only?
- What is the maximum allowed queue age and local authorization lifetime?
- How are device loss, revocation, shared devices, and local cache deletion handled?
- What is the replay retention period and full-resynchronization procedure?
- Which updates require immediate foreground delivery versus a user-visible background notification?
- What measured freshness threshold is acceptable for public availability?

## 6. Operational questions

- Who approves onboarding and verification, and how are appeals handled?
- What is the support escalation and privileged-access process?
- What are the backup, restore, incident response, and service recovery objectives?
- How can a tenant export records and leave the service?
- Which actions remain available during subscription suspension or platform outage?

## 7. Research gates

Do not lock the architecture or regulated workflows until:
1. The first establishment types and user roles are selected.
2. Current federal and relevant regional requirements are reviewed from primary sources.
3. Data protection roles, lawful basis, retention, and cross-border processing are assessed.
4. The public discovery boundary is validated with regulators or qualified counsel.
5. The offline action matrix is approved.
6. A first-release workflow set is selected and validated with real pharmacy operators.

## 8. Cross-cutting access and continuity risks

| Risk | Consequence | Required treatment |
|---|---|---|
| Establishment verification is incorrectly used as a universal platform gate | Legitimate management-only users are blocked; product policy is contradicted | Separate account, membership, management entitlement, establishment evidence, professional credential, and public-participation states |
| A professional credential is required for ordinary staff accounts | Owners, cashiers, inventory staff, accountants, and other non-professional users cannot perform permitted work | Gate only the specific regulated action or claim that requires the credential |
| A pending public review blocks private operations | A public-listing dispute or missing document interrupts pharmacy management | Separate public publication lifecycle from management-service lifecycle |
| Disabling a listing deletes or hides operational source records | Loss of business history or inability to reconcile past activity | Public projections are reversible views; preserve authoritative private records and audit history |
| Platform verification is presented as legal authorization | Users or the public misunderstand what Pharmalink has verified | Name the exact subject, evidence type, issuer, method, scope, and date of every verification claim |
| Subscription or service suspension destroys access to records | Business continuity, export, and retention failures | Define proportionate restrictions; preserve lawful read/export, audit, support, and exit paths |

**Open design question:** Which management operations, if any, must be restricted for a specific establishment category when required establishment evidence is absent, expired, or suspended? Resolve this per operation and applicable law; do not turn it into a blanket rule for all accounts or all management functions.
