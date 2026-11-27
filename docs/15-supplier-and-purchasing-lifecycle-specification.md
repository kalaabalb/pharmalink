# 15 — Supplier and purchasing lifecycle specification

**Status:** Draft — pharmacy field validation, accounting review, and regulatory review required  
**Priority:** High — purchasing connects supplier obligations, goods receipts, batch traceability, and finance  
**Scope:** Supplier records, purchasing requests, quotations, purchase orders, approvals, goods receipt references, invoices, supplier returns, payments, and reconciliation.

## 1. Scope and boundaries

This module supports a pharmacy's private purchasing and supplier-management operations. It is independent of public product listings and public commerce participation.

The system records business relationships and transactions. It must not imply that a supplier, distributor, product, or document is regulator-approved merely because a user created a record or uploaded a file. Any regulated supplier/product verification claim requires an identified authoritative source and a separate review.

A purchase order is an instruction/request to supply goods. It is not proof that goods were dispatched, received, inspected, invoiced, or paid.

## 2. Core invariants

1. Supplier, quotation, purchase order, goods receipt, supplier invoice, supplier return, and supplier payment are distinct business records.
2. A purchase order alone creates no stock and no payable balance unless an explicitly approved accounting policy says otherwise.
3. Only an accepted goods receipt creates stock movements, using the inventory rules in [Specification 13](13-inventory-and-batch-lifecycle-specification.md).
4. A supplier invoice does not prove goods were received; a goods receipt does not prove an invoice was accepted; an invoice does not prove payment.
5. Partial deliveries, backorders, substitutions, shortages, excess quantities, and damaged goods must be represented explicitly.
6. Supplier and product identities must not be silently merged on similar names, phone numbers, or identifiers.
7. Approval, submission, receipt, invoice acceptance, and payment are separate permissions and state transitions.
8. All financial obligations and payments are traceable, idempotent, and reconcilable.
9. Finalized purchasing and financial records are not silently edited or deleted; corrections use linked records.
10. Branch, tenant, and supplier data access is enforced server-side.
11. Private purchasing works independently of public listing or public commerce.
12. Ordinary purchasing and inventory staff do not need an individual professional licence merely to use their permitted management functions.

## 3. Supplier record

Candidate supplier fields:
- stable supplier ID and tenant ownership;
- legal/business name and trading name where applicable;
- contact persons, phone, email, and business address;
- supplier category and supported product categories;
- identifiers and licensing/authorization claims only where required and source-backed;
- status (active, inactive, blocked, under review);
- terms and currency;
- delivery locations and lead-time estimates;
- payment instructions/reference data subject to security and privacy controls;
- documents and source provenance;
- notes and internal risk flags;
- created/updated actor and timestamps.

A supplier can be tenant-private. Do not publish supplier details to public discovery. Do not store unnecessary personal or banking data. Supplier bank/payment data must have restricted access, change audit, and appropriate re-authentication.

A supplier's status is an internal business decision unless the product explicitly verifies a legal status from a reliable source. Do not use an internal “approved supplier” flag as a public regulatory claim.

## 4. Supplier lifecycle and duplicate handling

Candidate states:
- draft;
- active;
- inactive;
- blocked;
- under review;
- archived.

The system must define which states permit new purchase orders, receipts, invoices, or payments. Blocking a supplier must not erase historical transactions or prevent lawful export.

Duplicate detection may use name, address, contact, identifier, and user review. It must not automatically merge records or historical transactions. A merge requires an authorized, auditable process preserving old IDs and reference links.

## 5. Requisition and quotation workflow

A requisition may be used to capture an internal need before purchasing. If included:
1. An authorized user creates a draft requisition with branch, item/presentation, quantity, unit, need date, and reason.
2. The system validates product identity, unit conversion, and branch scope.
3. The requisition is submitted and approved under the configured threshold/separation policy.
4. Authorized purchasing staff request or record supplier quotations.
5. Quotations preserve supplier, quoted item, unit, price, currency, taxes/fees as stated, validity period, delivery terms, and source document.
6. The selected quotation and decision rationale are retained.

A quotation is not a binding order unless the relevant process explicitly makes it one. Expired quotations must not be silently used as current prices.

## 6. Purchase order lifecycle

Candidate states:
- **Draft:** editable; no supplier commitment.
- **Pending approval:** submitted for review.
- **Approved:** internally authorized; not yet necessarily sent.
- **Issued:** sent to supplier through a recorded channel.
- **Acknowledged:** supplier confirmation recorded, where available.
- **Partially received:** some accepted goods have been received.
- **Fully received:** all expected lines have been resolved through receipt, rejection, or approved closure.
- **Cancelled:** cancelled through an authorized workflow before or after issue, with supplier communication and outstanding quantity resolution recorded.
- **Closed:** no outstanding quantity/financial workflow remains under the defined policy.

The exact transitions and meaning of “fully received” versus “closed” must be finalized. A purchase order can have multiple receipt records and partial invoices.

Each order and line should retain:
- order ID, version, tenant, branch, supplier;
- creator, approver, and issued-by identities;
- item/presentation, quantity, unit, unit conversion version;
- expected unit cost, currency, discounts, taxes/fees where known;
- expected delivery date and delivery location;
- supplier quotation and source document references;
- line status, received quantity, rejected quantity, outstanding quantity;
- revision history and reason for changes;
- idempotency/command reference;
- audit and event references.

A material change after approval (supplier, item, quantity, price, currency, destination branch, or terms) must create a new revision and may require re-approval. Do not overwrite the approved snapshot.

## 7. Goods receipt integration

Goods receipt is specified in [Specification 13](13-inventory-and-batch-lifecycle-specification.md). This module must:
- link each receipt line to the purchase-order line where one exists;
- allow approved non-PO receipts only if the business policy explicitly permits them;
- preserve ordered, received, accepted, rejected, and outstanding quantities;
- record partial deliveries and multiple receipts;
- record batch/lot, expiry, unit, actual quantity, condition, and evidence;
- route discrepancies for review;
- prevent the same receipt document or command from creating duplicate stock;
- update the purchase order's received/outstanding projection only after the receipt transaction commits.

A receipt may be accepted in part. Damaged, expired, recalled, or otherwise ineligible goods must enter the appropriate hold/rejection workflow rather than available stock.

## 8. Supplier invoice and matching

An invoice record should capture supplier, invoice number, invoice date, currency, line references, amounts, taxes/fees as stated, due date, document evidence, and review state.

Invoice number uniqueness must be scoped appropriately (at least supplier and tenant, subject to supplier practices). Duplicate invoices must be detected and sent for review, not silently discarded.

Define matching policy:
- **Two-way match:** purchase order to invoice, where goods receipt matching is not required by the approved workflow.
- **Three-way match:** purchase order, accepted goods receipt, and invoice.
- **Exception review:** quantity/price/tax/identity mismatch, missing evidence, duplicate invoice, or supplier dispute.

The system must retain original supplier-stated values separately from internal calculated or approved values. Do not silently rewrite invoice totals based on the purchase order.

Invoice approval and payable recognition require an accounting policy reviewed for the launch jurisdiction. Do not infer accounting treatment from a document upload.

## 9. Supplier returns and disputes

A supplier return references the original receipt/batch and records:
- reason and evidence;
- product, batch, quantity, and unit;
- requested/approved/dispatch/acknowledgement states;
- source branch and supplier;
- stock movement and disposition;
- credit note or financial adjustment reference where supplied;
- responsible actor and approval.

Returned stock must leave eligible branch availability through an accepted stock movement. A supplier's acknowledgement or credit note must not be fabricated when absent. Disputes remain linked to the source order, receipt, invoice, or return.

## 10. Supplier payments and reconciliation

Supplier payment is a separate financial transaction. It must reference the payable/invoice or approved obligation and include amount, currency, method, payment date, external reference, initiator, confirmation source, and reconciliation status.

- Sending a payment instruction is not proof of settlement.
- Duplicate payment submission must be idempotent.
- Partial payments and allocations across invoices require explicit policy.
- Overpayments, underpayments, fees, withholding, credit notes, and currency differences require defined treatment.
- Bank/provider imports must preserve source files and row provenance and produce a reviewable reconciliation result.
- Payment credentials and sensitive bank details require least-privilege access and audit.

No payment integration is production-enabled until provider access, security, settlement semantics, reconciliation, and legal/accounting review are complete.

## 11. Permissions and separation of duties

Candidate permissions:
- supplier.read;
- supplier.create;
- supplier.update;
- supplier.block;
- supplier.sensitive_data.read;
- requisition.create;
- requisition.submit;
- requisition.approve;
- quotation.record;
- purchase_order.create;
- purchase_order.submit;
- purchase_order.approve;
- purchase_order.issue;
- purchase_order.cancel;
- goods_receipt.create;
- goods_receipt.accept;
- supplier_invoice.record;
- supplier_invoice.match;
- supplier_invoice.exception.resolve;
- supplier_invoice.dispute;
- supplier_invoice.approve;
- supplier_return.create;
- supplier_return.approve;
- supplier_payment.initiate;
- supplier_payment.confirm;
- supplier_payment.reconcile;
- purchasing.export.

Canonical identifiers and the unresolved role-grant register are maintained in [Specification 18](18-proposed-permission-matrix.md). The final grant mapping must define tenant/branch scope, thresholds, approval separation, re-authentication, sensitive-data access, audit events, and offline eligibility. Owners do not automatically bypass approval or regulated-action constraints.

## 12. Offline and event-driven behavior

- Supplier, quotation, and purchase-order drafts may be prepared offline only if local data protection and conflict behavior are approved.
- Issuing an order, accepting a receipt, approving an invoice, initiating/confirming a payment, and closing a financial reconciliation are online-only by default.
- Offline drafts must show their local/pending state and cannot create stock or payables until accepted by the server.
- Commands carry idempotency keys and base versions.
- Server-side authorization and business rules are revalidated on synchronization.
- Candidate events include SupplierChanged, PurchaseOrderApproved, PurchaseOrderIssued, GoodsReceiptAccepted, SupplierInvoiceRecorded, SupplierInvoiceApproved, SupplierReturnDispatched, SupplierPaymentStatusChanged, and SupplierBalanceReconciled.
- Events are written through the transactional outbox and consumers tolerate duplicate delivery and replay.
- Relevant screens subscribe to branch/tenant events; do not poll continuously for unchanged supplier or order state.

## 13. Reports and operational controls

Provide, as approved:
- open purchase orders and outstanding quantities;
- delivery lead time and supplier fulfillment history;
- price history by product/presentation, unit, currency, and effective date;
- receipt discrepancy and damaged/rejected goods;
- invoice aging and payable balance;
- supplier returns and credit-note status;
- supplier payment and reconciliation status.

Reports must state their as-of time, tenant/branch scope, currency, and calculation basis. Historical price comparisons must not combine incompatible units or currencies without explicit normalization.

## 14. Required test scenarios

Automated tests must prove:
- creating or approving a purchase order does not create stock;
- a supplier invoice does not create stock or prove payment;
- duplicate supplier and invoice records are detected without destructive auto-merge;
- a purchase-order revision after approval preserves history and enforces re-approval where required;
- partial and multiple receipts reconcile to ordered, accepted, rejected, and outstanding quantities;
- duplicate receipt commands cannot duplicate stock;
- supplier substitution cannot silently change product identity;
- expired/damaged/ineligible receipt lines do not enter available stock;
- invoice mismatches enter an exception workflow;
- invoice dispute and exception resolution require their distinct permissions; approval cannot silently clear a dispute;
- payment initiation is not treated as settlement;
- duplicate/late payment callbacks do not duplicate payments or allocations;
- blocked suppliers cannot perform prohibited new operations but historical records remain available;
- branch/tenant isolation is enforced in reads, writes, reports, events, and exports;
- revoked permissions block queued commands on synchronization.

## 15. Open decisions before implementation

1. Which supplier types and purchasing workflows are needed by the first pharmacy cohort?
2. Are internal requisitions and quotation comparison in the first release?
3. Are purchase orders mandatory for receiving, optional, or configurable by organization?
4. Which approval thresholds and separation-of-duties rules are required?
5. How are supplier identifiers, documents, and any regulated authorization claims verified?
6. What are the accepted partial delivery, substitution, backorder, and cancellation policies?
7. Which invoice matching policy is required for the first release?
8. What accounting treatment applies to invoice approval, payables, credit notes, taxes, and withholding?
9. Which local payment and bank data formats are used by pilot pharmacies?
10. Which supplier/contact/banking fields are necessary, and what retention/security controls apply?
11. What import/export formats are needed to migrate existing supplier and purchasing records?
12. Which supplier, purchasing, and financial actions may be drafted offline, if any?

Resolve these decisions through pharmacy field research and qualified accounting/legal review. Do not make an unverified supplier or payment process appear production-ready.


## 16. Cross-module transition alignment

The shared supplier, purchase-order, goods-receipt, and recovery transition contract is defined in [Specification 30](30-workflow-state-transition-and-recovery-contract.md). “Fully received” and “closed” remain distinct; unresolved PO closure and partial receipt policy must not be invented during implementation.
