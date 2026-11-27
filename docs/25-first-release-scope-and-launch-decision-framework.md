# 25 — First-release scope and launch decision framework

**Status:** Draft — proposed release candidate; not approved for implementation  
**Purpose:** Convert the broad Pharmalink capability map into a testable first production release without pretending unresolved market, legal, accounting, or operational questions are settled.

## 1. Release objective

The first release must solve a recurring, high-value pharmacy operations problem for a defined cohort and remain safe to operate in production. It must not attempt to launch every capability in the platform at once.

The release must be a real deployable product with:
- authoritative server-side persistence;
- enforced tenant and branch isolation;
- complete business workflows rather than disconnected screens;
- durable stock and financial records;
- event-driven updates;
- audit and recovery;
- support, monitoring, backups, tested restoration, and controlled deployment;
- evidence from real pharmacy users.

Management-only is a complete product mode. Public discovery and commerce are not prerequisites for the first production release.

## 2. Proposed first-release candidate (for validation)

The following is a recommended candidate, not an approved scope decision. It deliberately prioritizes a private operational workflow and excludes public commerce.

| Capability | Candidate decision | Reason / dependency |
|---|---|---|
| Account, organization, membership, branch, role access | Include foundationally | Required for every tenant-owned workflow; no blanket professional credential gate |
| Tenant-private medicine/product catalogue | Include | Required to identify what is purchased and stocked; canonical source licensing remains open |
| Batch inventory and stock ledger | Include | Core source of truth for quantity, expiry, hold, and traceability |
| Supplier records and purchase orders | Include a minimal coherent flow | Needed to connect supplier, order, receipt, and batch evidence |
| Goods receiving and batch capture | Include | Accepted receipt must create stock movements atomically |
| In-branch POS | Include only for legally reviewed supported product categories and workflows | Depends on stock allocation, price policy, tax/accounting review, and action-specific professional controls |
| Payment tracking | Include only verified supported methods | Payment initiation, confirmation, settlement, and reconciliation must remain distinct |
| Cash sessions and operational cashbook | Candidate include | Useful for reconciling actual cash to sales; needs field/accounting validation |
| Returns, refunds, and stock disposition | Include a narrowly defined, tested workflow | Required to prevent incorrect stock and financial state; do not equate refund with restock |
| Basic operational reports | Include | Sales, stock, expiry, cash variance, and supplier activity with traceable definitions |
| Audit, notifications, outbox, monitoring, backup/restore | Include as production foundations | Not optional polish; required for operational trust |
| Public pharmacy profile and product availability | Defer from first production release unless separately justified | Opt-in public surface adds freshness, moderation, privacy, and legal dependencies |
| Public enquiries, reservation, ordering, payment collection, delivery | Exclude | Separate legal, consumer, provider, and operational approval required |
| Patient/customer health records and prescription workflow | Exclude pending dedicated legal, privacy, security, and professional review | Avoid creating high-sensitivity data before purpose and controls are validated |
| Full general ledger, statutory tax filing, fiscal-device integration | Exclude unless separately specified and approved | Requires qualified accounting/legal and jurisdiction-specific validation |
| AI clinical recommendations or autonomous financial/stock actions | Exclude | Outside product safety boundary |
| Multi-currency | Candidate defer; validate with pilot cohort | Avoid unsupported conversion and reconciliation complexity |
| Offline finalization of sales, payments, stock movements, refunds, and approvals | Exclude by default | These operations require current server-side state and conflict controls |

A candidate may be narrowed further after field research. It must not be expanded merely because a screen or API can be built.

### Conditional sub-scope rules

The capability rows above do not authorize every related workflow. Apply these boundaries when turning the candidate into an approved release:

- **Branch transfers:** enable only if the pilot includes multiple operating branches and has tested dispatch, in-transit custody, receipt, discrepancy, consolidated reporting, and branch-scoped authorization. Otherwise keep transfer commands and UI disabled.
- **Requisitions and quotations:** include only if field research shows the pharmacy uses them; a direct purchase-order flow may be sufficient for the first cohort.
- **Supplier invoices, disputes, and supplier payments:** enable only after the invoice matching/payable policy, accountant review, and supported payment evidence/provider method are approved. Recording an invoice is not proof of a payable settlement.
- **Discounts and voids:** enable only with explicit thresholds, reason capture, approval separation, and accounting treatment.
- **Disposal:** enable only after the product-category-specific disposal method, evidence, authorization, and any witness/legal requirements are reviewed.
- **Supplier returns and expense tracking:** remain excluded unless a concrete pilot requirement and complete end-to-end policy are approved.
- **Catalogue corrections:** tenant-local catalogue maintenance may be included; platform-global canonical correction approval remains a separate governance workflow.

An operation may appear in Specification 29's traceability matrix as a conditional candidate. That does not make it part of the launch build. The product-owner-approved capability list must explicitly mark each one Included, Conditional, or Deferred before implementation.

## 3. First cohort and pilot model

The launch cohort must be defined by actual research, not assumptions about all Ethiopian pharmacies.

Research should distinguish at least:
- establishment category and applicable regional authority;
- independent versus chain/branch operations;
- daily transaction volume and staffing;
- medicine/product category mix;
- current stock, purchasing, sales, and accounting methods;
- device, barcode, printer, power, and connectivity conditions;
- language and training requirements;
- data migration quality and willingness to use a new system.

**Proposed pilot posture:** begin with a small, directly supported cohort of establishments whose operating model and legal category have been reviewed. Keep the data model capable of multiple branches, but do not claim multi-branch operational readiness until transfer, consolidated reporting, access isolation, and recovery have been exercised with representative users.

The number and identity of pilot customers, launch geography, and establishment categories remain open product-owner decisions informed by field research.

## 4. Explicit first-release exclusions and safe states

A deferred capability must be genuinely unavailable, not a hidden partially working feature. It must not appear enabled in navigation, public pages, APIs, marketing claims, or onboarding where it cannot be safely completed.

Examples:
- Public listing disabled: no public profile or product projection is emitted.
- Payment provider not production-approved: do not display a live payment option or claim payment success.
- Product category not legally reviewed: block sale/publication for that category and explain the restriction to authorized staff.
- No approved tax configuration: do not generate a document represented as a compliant statutory invoice; preserve the required source transaction and provide only reviewed operational evidence.
- No patient-record approval: do not collect patient identity or prescription documents as an informal workaround.
- Offline command not approved: save a clearly labeled local draft only; do not represent it as a committed sale, receipt, transfer, or payment.
- Integration unavailable: show the actual unavailable/degraded state; never silently substitute synthetic data.

## 5. Minimum end-to-end release journeys

The candidate release is not coherent until these journeys work end-to-end:

1. **Organization setup:** account creation, organization creation, branch setup, staff invitation, membership acceptance, scoped role assignment, and access revocation.
2. **Catalogue setup:** create/select a product identity, define exact presentation and units, preserve source/provenance, and enable a tenant-local item without mutating canonical identity.
3. **Purchasing:** create supplier, prepare and approve purchase order, issue it, record partial/full delivery, inspect and accept quantities, capture batch/expiry, and create stock movements.
4. **Inventory operations:** view stock by branch/batch, hold stock, count, request/approve adjustments, and record transfer only if multi-branch pilot workflows are in scope.
5. **In-branch sale:** identify exact product, validate batch eligibility and quantity, apply approved price/tax rules, authorize the action, commit sale and stock issue atomically, and issue a traceable receipt.
6. **Payment:** initiate supported method, process authoritative confirmation, handle unknown/timeout/retry, and reconcile settlement without duplicate financial effects.
7. **Correction:** handle void/return/refund as linked records; inspect returned stock and determine disposition separately.
8. **Cash close:** open session, attribute cash movements, count, record variance, review, and close without rewriting sales.
9. **Reporting:** trace a reported total to source transactions and show scope, period, currency, calculation basis, and as-of time.
10. **Continuity:** recover after service interruption, restore from backup in a test, export tenant records, and revoke a lost/stolen device or staff session.

If any included journey cannot be completed without a manual database edit, developer intervention, fabricated provider response, or untracked spreadsheet correction, the release is not ready.

## 6. Release decision gates

| Gate | Evidence required | Failure outcome |
|---|---|---|
| Cohort and workflow validation | Observed workflows, representative user sessions, signed-off problem statement | Rework scope |
| Legal/regulatory boundary | Current source register and qualified review for establishment/product/action scope | Keep affected capability disabled |
| Data and catalogue readiness | Provenance, identity rules, import rights, correction process | Do not import or publish unverified catalogue |
| Transaction integrity | Concurrency, idempotency, reversal, reconciliation, and negative tests | Block release |
| Authorization and privacy | Tenant/branch isolation, action-specific credential gates, privacy assessment, export tests, and approved role grants for every canonical permission used by the release | Block release |
| Operational readiness | Monitoring, alert ownership, incident process, support path, provider outage plan | Block production launch |
| Backup and recovery | Restore drill, integrity comparison, recovery time evidence | Block production launch |
| Deployment and migration | Reproducible environment, versioned migrations, rollback/forward-fix plan, release verification | Block production launch |
| User acceptance | Representative staff complete real workflows using production-like data | Rework before launch |
| Launch approval | Named accountable owner accepts known residual risks and capability boundaries | No production enablement |

## 7. Proposed default decisions to validate

These defaults reduce scope and avoid premature claims. They are recommendations for product-owner review, not settled facts.

1. Start with management-only private operations; keep public discovery and commerce disabled.
2. Pilot a narrowly defined establishment cohort and geography after local regulatory review.
3. Keep multi-branch concepts in the domain model, but limit pilot operations to workflows actually tested.
4. Use one configured operating currency per tenant for the first release unless pilot evidence proves multi-currency is essential; do not perform implicit currency conversion.
5. Deliver operational finance and cash reconciliation before attempting a full general ledger.
6. Do not collect patient/customer health records in the first release.
7. Do not enable prescription-controlled or otherwise restricted product workflows until the exact categories and professional action gates are approved.
8. Do not finalize sensitive financial or stock-affecting actions offline.
9. Do not enable public availability until the freshness and publication controls have been field-tested.
10. Treat production provider selection, payment methods, and statutory receipt/invoice obligations as release gates, not post-launch cleanup.

## 8. Product-owner decisions that cannot be silently delegated to developers

The following require explicit product-owner acceptance after evidence is gathered:
- initial establishment category and launch geography;
- exact first-release capabilities and excluded workflows;
- whether in-branch POS is included at launch and for which product categories;
- supported payment methods;
- whether cashbook and expense tracking are included;
- whether the first release supports one or multiple branches operationally;
- operating currency and any foreign-currency requirements;
- whether public discovery is included in the first release;
- pricing, trial, subscription, and suspension policy;
- pilot customers and launch approval authority.

Developers may propose options and document trade-offs, but must not invent commercial policy, legal interpretation, or customer commitments.

## 9. Definition of first-release readiness

The release candidate is ready for implementation only when:
- its scope is approved and every excluded feature is explicitly disabled;
- each included journey has complete, cross-consistent specifications;
- no critical unresolved question is hidden in code assumptions;
- authoritative data ownership, state machines, transaction boundaries, permissions, events, privacy, retention, and recovery are defined;
- every enabled action maps to a canonical permission identifier, approved role/scope grant, approval rule, audit event, and positive/negative authorization tests; Specification 18 §12 provides a proposed grant baseline for review, not automatic or approved production grants.
- testable acceptance criteria exist for normal, alternate, denied, failure, retry, concurrency, offline, and recovery cases;
- architecture and provider choices are justified by the actual workflows;
- legal/accounting review is complete for the enabled capabilities;
- a production operating owner and support process are identified.

A document set can be extensive and still fail this gate. Approval is based on evidence and completeness of the selected scope, not the number of documents.
