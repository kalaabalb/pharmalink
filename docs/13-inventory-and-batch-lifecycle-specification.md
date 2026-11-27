# 13 — Inventory, batch, and stock-movement specification

**Status:** Draft — foundational domain specification; pharmacy field validation and regulatory review required  
**Priority:** Critical — stock integrity affects operational correctness and public availability  
**Scope:** Catalogue-to-stock identity, batch/lot traceability, receiving, stock movements, counts, transfers, holds, expiry, recall, returns, and reconciliation.

## 1. Purpose and evidence labels

This document defines proposed system behavior for inventory. It is not a legal determination of which records every establishment must maintain.

- **Product invariant:** a rule Pharmalink must preserve to maintain data integrity.
- **Proposed requirement:** recommended behavior requiring validation with pharmacy operators.
- **Open decision:** must be resolved before implementation of the affected workflow.
- **Regulatory dependency:** must be confirmed against current federal/regional requirements and qualified Ethiopian review.

## 2. Core inventory invariants

1. A catalogue identity, tenant catalogue entry, product presentation/pack, physical batch/lot, and branch stock position are distinct concepts.
2. A stock balance is a derived projection of accepted stock movements; it is not an independently editable source of truth.
3. Every accepted movement is attributable to an actor or system process, tenant, branch, time, reason, and originating workflow.
4. A purchase order alone does not create physical stock. Stock is created only by an accepted receipt or an explicitly approved opening-balance process.
5. A sale and its stock deduction must commit atomically, or neither may be reported as completed.
6. Duplicate command submission must not create duplicate stock movements.
7. Concurrent transactions must not oversell or silently overwrite stock.
8. Finalized movements are immutable. Corrections use linked reversals or compensating movements with reasons and approvals.
9. Stock held for quality, recall, expiry, damage, or investigation is not eligible for ordinary sale allocation.
10. A return does not automatically make returned stock saleable.
11. Branch stock is never implicitly shared with another branch. A transfer is an explicit workflow with source, destination, dispatch, and receipt.
12. Public availability is derived from eligible stock and fresh server confirmation; it is never a direct exposure of the private stock ledger.

## 3. Identity hierarchy

The implementation must distinguish:

- **Canonical catalogue item:** source-governed identity, such as the regulated medicine/product identity where a reliable source exists.
- **Presentation/pack:** exact strength, dosage form, pack size, concentration, route, and other identity-defining presentation attributes where applicable.
- **Tenant catalogue entry:** the pharmacy's enabled/local record linked to the best-supported canonical identity; includes local labels and operational settings.
- **Sellable unit:** the unit in which the pharmacy receives, stores, prices, and sells a product. Its conversion to other units must be explicit.
- **Batch/lot:** a physical traceability grouping associated with a product presentation and receipt/source.
- **Stock position:** a derived quantity for a tenant, branch, product, batch, location, and stock-status combination.

Names and barcodes alone are not sufficient identity proof. Ambiguous matches must be reviewable and must not silently merge stock from different presentations.

## 4. Batch and lot record

Candidate fields, subject to product category and source availability:

- immutable batch record ID;
- tenant and branch ownership;
- catalogue item and presentation references;
- batch/lot number exactly as received, with normalized search value stored separately if needed;
- manufacturing and expiry dates when supplied/required;
- supplier and source receipt;
- manufacturer/authorization-holder metadata where known;
- received quantity and unit;
- current derived quantity by stock status;
- purchase cost and cost allocation reference, with accounting policy version;
- storage location and handling metadata where needed;
- quality/inspection status;
- provenance and evidence references;
- created/received/updated timestamps and actor references.

Do not fabricate missing lot numbers, expiry dates, registration identifiers, or manufacturer details. Missing or uncertain data must have an explicit state and must be handled according to the approved product-category policy.

A batch identifier is not globally unique by itself. Uniqueness rules must be scoped by tenant, product/presentation, supplier/source, and other validated identity dimensions.

## 5. Stock status model

At minimum, distinguish quantities that are:

- **Available:** eligible for allocation under the configured product and branch rules.
- **Held/quarantined:** unavailable pending inspection, investigation, recall review, or authorized release.
- **Damaged:** not eligible for ordinary sale.
- **Expired:** not eligible for ordinary sale.
- **Recalled/blocked:** not eligible for ordinary sale.
- **In transit:** dispatched from a source branch but not yet accepted at the destination.
- **Disposed:** permanently removed from usable stock through an approved disposition.
- **Unknown/unreconciled:** quantity or status is uncertain and must not be represented as safely available.

The model must define whether each state is represented as a separate balance bucket, a batch disposition, or both. It must prevent the same physical quantity from being counted in two states.

## 6. Stock movement ledger

A movement is an immutable business fact. Candidate movement types:

- opening balance;
- goods receipt;
- sale issue;
- customer return receipt;
- supplier return;
- branch transfer dispatch;
- branch transfer receipt;
- count gain/loss;
- authorized adjustment;
- hold/quarantine;
- release from hold;
- damage;
- expiry;
- recall block/release;
- disposal;
- correction/reversal.

Each movement records:
- movement ID and type;
- tenant and branch scope;
- product, presentation, batch, and location references where applicable;
- quantity and unit, including conversion version;
- source and destination stock status/location;
- effective business time and server-recorded time;
- actor, device, and authorization context;
- reason code and explanatory note where required;
- originating transaction/document;
- idempotency key and command reference;
- approval and separation-of-duties references where required;
- reversal/compensation link where applicable;
- aggregate version and correlation ID.

The ledger must retain both effective time and recorded time so delayed synchronization or correction does not rewrite history.

## 7. Receiving workflow

1. An authorized user selects or creates the supplier and references an approved purchase order where the business process uses one.
2. The user records received lines, actual quantities, units, product presentation, batch/lot details, expiry where available/required, cost, and source documents.
3. The system validates product identity, unit conversion, duplicate receipt, branch scope, supplier, and relevant quality/expiry rules.
4. Discrepancies against the purchase order are recorded explicitly: short, excess, substituted, damaged, missing evidence, or rejected quantity.
5. Any required inspection/approval is completed before affected stock becomes available.
6. Posting creates the receipt and stock movement(s) in one authoritative transaction, or leaves the receipt in a clearly pending state.
7. The resulting stock projection and audit history update from the committed movement.
8. A receipt cannot be silently edited after posting. Corrections use a linked reversal or adjustment workflow.

**Failure paths:** duplicate supplier document; same receipt submitted twice; unknown catalogue item; invalid unit conversion; missing batch data where required; expired or damaged goods; partial receipt; server timeout after commit; concurrent receipt; and supplier invoice mismatch.

A timed-out client must query the authoritative transaction outcome using its idempotency/command reference before retrying.

## 8. Sale allocation and stock deduction

- Sale allocation must use only stock positions eligible for sale under product, branch, quality, expiry, and regulatory rules.
- The allocation policy (including whether and how FEFO applies) must be configurable only within approved rules and must be visible in the transaction record.
- The server validates current stock and permission at commit time; a client-side balance is informational only.
- A sale cannot commit if the required eligible quantity is unavailable under the approved policy.
- Sale, sale lines, stock issue movements, and audit references must commit atomically within the internal sale transaction. Payment initiation, provider confirmation, settlement, refund, and reconciliation are separate payment lifecycle transitions; they must not be treated as part of the same atomic database transaction as sale commitment. Apply the shared rules in [Specification 24](24-cross-module-transaction-and-implementation-contract.md) and the payment lifecycle in [Specification 14](14-sales-pos-and-payment-lifecycle-specification.md).
- A sale must not be marked paid merely because the client received a success screen or a provider request was sent.
- A sale reversal must restore stock only through a separately validated return/disposition process; it must not automatically restore all goods as available.

**Open decisions:** whether negative stock is categorically prohibited in the first release; how substitutions are represented (no clinical substitution decisions); how partial fulfillment works; how sale allocation is handled when multiple batches are eligible; and which product categories require additional approval.

## 9. Physical counts and adjustments

A count session records:
- branch and scope;
- count start/end and status;
- counted items/batches/locations;
- counter identity and device;
- expected snapshot/version;
- observed quantity and unit;
- variance and reason;
- reviewer/approver where required;
- posted movement references.

A count must not overwrite a live balance. Posting reconciles the observed count against a defined authoritative snapshot and creates explicit gain/loss movements. If material stock changes occur during counting, the system must use a documented freeze, movement-cutoff, or version-conflict policy; it must not silently apply a stale count over newer sales/receipts.

Adjustments require a reason and permission. High-risk adjustments may require a second approver; thresholds and exceptions are open decisions requiring field validation.

## 10. Branch transfer lifecycle

A transfer is a multi-step workflow, not a direct balance edit:

1. Draft transfer identifies source, destination, products/batches, quantities, and initiator.
2. Source branch validates permission and available quantity.
3. Dispatch posts an immutable source movement and places the quantity in transit.
4. Destination branch records receipt, including actual quantity, condition, and discrepancies.
5. Accepted quantities become destination stock; shortages, damage, or disputes enter a separate exception workflow.
6. The transfer closes only when all lines are received, rejected, cancelled, or explicitly resolved.

The same physical quantity must not be simultaneously available at source and destination. Cross-tenant transfers are not allowed unless a separate commercial/legal model is approved.

## 11. Returns, holds, expiry, recall, and disposal

- A customer return references the original sale where possible and records condition, time, actor, and disposition.
- Returned stock enters a non-saleable/inspection state until an authorized disposition decision is made.
- Supplier returns reference the source receipt/batch and record dispatch and supplier acknowledgement where available.
- Hold/quarantine is a traceable state transition with reason, scope, evidence, and authorized releaser.
- Expiry checks operate on stored dates and configured policy; they must not silently change or fabricate expiry data.
- Recall notices retain source, authority, issue date, affected product/batch identifiers, matching confidence, and review outcome.
- Uncertain recall matches are flagged for authorized review; the system must not claim complete affected-stock detection when identifiers are incomplete.
- Disposal records quantity, method/category, authorization, witness where required, and evidence according to the approved policy.
- Release from hold requires explicit authorization and must not bypass expiry, recall, or other blocking states.

## 12. Offline and event behavior

- Receiving, sale finalization, transfer dispatch/receipt, stock adjustment, return disposition, hold release, and disposal are online-only by default until a specific offline policy is approved.
- Offline users may view only an authorized encrypted snapshot with a last-confirmed time and visible offline state.
- Local drafts may be prepared where approved, but are not authoritative stock movements.
- Every committed stock transaction writes its domain event to the transactional outbox in the same database transaction.
- Event consumers are idempotent and version-aware.
- Clients subscribe to relevant branch/product changes; do not poll continuously for unchanged balances.
- On reconnect, the client replays from a durable cursor or fetches an authoritative snapshot, then reconciles local drafts/commands.
- Public availability is refreshed only from server-confirmed eligible stock and expires to stale/unknown when the freshness deadline passes.

## 13. Permissions and separation of duties

The canonical identifiers are registered in [Specification 18](18-proposed-permission-matrix.md); its role grants remain a release blocker until approved. This module requires at least:
- inventory.read;
- inventory.receive;
- inventory.count;
- inventory.adjust.request;
- inventory.adjust.approve;
- inventory.transfer.create;
- inventory.transfer.dispatch;
- inventory.transfer.receive;
- inventory.hold;
- inventory.release;
- inventory.disposal.request;
- inventory.disposal.approve;
- inventory.return.disposition;
- inventory.export.

A role template may group permissions, but the server evaluates explicit grants and scope. A non-professional user may perform permitted inventory-management tasks without a professional licence. A regulated professional credential is checked only when the specific action requires it under the reviewed rules.

## 14. Reports, reconciliation, and export

Inventory reports must identify tenant, branch, as-of time, filters, stock status, unit, and calculation basis. Provide:
- on-hand by product and batch;
- available versus held/expired/damaged/recalled/in-transit quantities;
- movement history and adjustment reasons;
- expiry horizon and recall review queues;
- receiving and transfer discrepancies;
- count variance and reconciliation;
- stock valuation only after the accounting valuation method is approved.

Exports must preserve identifiers, units, timestamps, batch links, source documents, and movement references. Exported totals must reconcile to the authoritative system for the same scope and as-of time.

## 15. Required test scenarios

At minimum, automated tests must prove:
- catalogue items with similar names but different strength/form/pack are not silently merged;
- duplicate receipt requests create one receipt/movement set;
- a purchase order without a receipt creates no stock;
- a receipt cannot create negative, duplicated, or double-counted quantity;
- concurrent sales cannot oversell the same eligible stock;
- expired, recalled, damaged, and held stock cannot be allocated to an ordinary sale;
- a return is not automatically saleable;
- a count cannot overwrite newer movements;
- transfer dispatch removes source availability and receipt adds destination availability exactly once;
- duplicate, delayed, reordered, and replayed events do not duplicate stock effects;
- revoked membership or permission blocks queued commands at synchronization;
- tenant and branch boundaries are enforced in APIs, events, exports, and background jobs;
- reversal preserves original history and creates a linked compensating record;
- public availability never exposes private quantity or stale stock as freshly confirmed.

## 16. Valuation and cost-integrity boundary

The detailed cost provenance, valuation-method decision framework, landed-cost allocation, stock-count reconciliation, returns/write-off treatment, and valuation acceptance tests are defined in [Specification 27](27-inventory-valuation-stock-reconciliation-and-cost-integrity-specification.md). Until the accounting method and source-data quality are approved, inventory quantity operations must remain distinct from valuation, gross-margin, and cost-of-goods-sold claims.

## 17. Open decisions before implementation

1. Which establishment categories and product categories are in the first release?
2. Which batch/expiry fields are mandatory for each supported product category, based on current law and actual pharmacy practice?
3. What are the permitted units and conversion rules, and who can define/approve them?
4. What stock valuation method is required for the initial accounting/reporting scope?
5. Is negative stock prohibited without exception in the first release?
6. Which count adjustments require a second approver, and what is the single-person-site exception?
7. What is the approved FEFO/FIFO/manual allocation policy by product category?
8. Which actions, if any, can be queued offline, and what is the maximum queue age?
9. Which barcode/catalogue data sources are authoritative and licensed for use?
10. What evidence and retention rules apply to receipt, transfer, recall, and disposal records?
11. Which public availability states and freshness thresholds are acceptable for each product category?
12. What are the expected peak transaction volumes, branch sizes, and device constraints from field research?

No implementation should invent answers to these decisions. Each must be resolved, scoped as not applicable, or explicitly deferred with a safe default before the affected capability is enabled.


## 18. Cross-module transition alignment

The shared state-transition, transaction ordering, event, and recovery contract is defined in [Specification 30](30-workflow-state-transition-and-recovery-contract.md). Inventory-specific stock status and movement rules in this document remain authoritative; the shared contract does not make a batch saleable without the inventory eligibility checks defined here.
