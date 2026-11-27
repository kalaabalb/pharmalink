# 27 — Inventory valuation, stock reconciliation, and cost integrity specification

**Status:** Draft — accounting policy and pilot evidence required  
**Priority:** Critical for any valuation, margin, cost-of-goods-sold, or financial reporting capability  
**Scope:** Cost provenance, inventory valuation methods, landed cost, stock-count reconciliation, cost corrections, write-downs, returns, transfers, and financial reporting boundaries.

## 1. Purpose and boundary

This specification defines how Pharmalink may calculate and explain inventory cost. It does not select an accounting method or establish Ethiopian statutory treatment. Those decisions require the applicable accounting framework, launch-jurisdiction review, and a qualified Ethiopian accountant.

Stock quantity and stock value are different measures. The stock movement ledger remains the authoritative record of physical quantity. Cost layers and valuation calculations are separate, versioned financial projections derived from source documents and approved policy.

No inventory valuation, gross margin, or cost-of-goods-sold figure may be labelled accounting-approved until its method, configuration, data quality, and accounting treatment have been reviewed and approved.

## 2. Non-negotiable cost invariants

1. Every cost amount has an explicit currency, precision, source, effective date, recorded date, and tenant/branch scope.
2. Original supplier-document values and accepted receipt quantities are preserved; later corrections do not overwrite source evidence.
3. Quantity and cost are never inferred from product names, retail prices, or unrelated transactions.
4. No binary floating-point arithmetic is used for monetary calculations.
5. A cost calculation is reproducible from source records, valuation policy version, conversion rules, and calculation timestamp.
6. A stock quantity correction does not silently change unit cost; a cost correction does not silently change physical quantity.
7. A return, supplier credit, refund, or payment does not automatically imply the corresponding inventory or payable effect. Each domain transition is explicit and linked.
8. Internal branch transfers must not create organization-wide profit or duplicate inventory value.
9. Negative stock, missing cost, ambiguous unit conversion, and unresolved receipt discrepancies must be surfaced as exceptions, not hidden by estimates.
10. Historical reports retain the policy version and source snapshot used when generated.

## 3. Cost source hierarchy and provenance

Potential cost inputs include:
- accepted supplier invoice line;
- accepted goods receipt and its purchase-order allocation;
- approved freight, insurance, handling, or other landed-cost evidence;
- supplier credit note or rebate;
- approved opening-balance valuation evidence;
- authorized cost correction with supporting reason and approval.

A purchase order, quotation, draft invoice, unaccepted receipt, or payment request is not by itself proof of final inventory cost. The implementation must define which source becomes authoritative for each supported purchasing workflow.

For every cost component preserve:
- source document and line identifier;
- supplier and receipt references;
- original amount and currency;
- allocated amount and allocation method/version;
- tax treatment status (verified, excluded, included, or unresolved);
- quantity basis and unit-conversion version;
- effective and recorded timestamps;
- approval and correction links.

Where source evidence is missing, the system records an unknown/unverified cost state. It must not silently substitute selling price, catalogue reference price, latest price, or zero.

## 4. Valuation method decision framework

The product must evaluate, rather than silently choose among, applicable methods such as:
- specific identification where individually traceable cost is appropriate;
- FIFO cost layers;
- weighted-average cost, with the averaging interval and recalculation behavior explicitly defined;
- another method only where the applicable accounting framework and qualified accountant support it.

For each candidate method, document:
- accounting acceptability for the target establishment and reporting purpose;
- whether it is periodic or perpetual;
- cost-layer creation and consumption rules;
- treatment of partial receipts, invoice timing, discounts, rebates, freight, and other landed costs;
- treatment of returns, supplier credits, stock losses, transfers, expiry, damage, and write-downs;
- handling of backdated source documents and late corrections;
- branch-level versus organization-level cost pools;
- reproducibility, audit trail, and migration requirements;
- impact on cost of goods sold, gross margin, and financial statements.

The selected policy must be approved, versioned, effective-dated, and applied consistently. A policy change must not silently rewrite historical posted transactions. If retrospective restatement is legally/accountingly required, it must be a separately approved, traceable process with before/after reports.

## 5. Cost allocation and unit conversions

Landed costs may be allocated only under a documented approved rule, such as a justified quantity, value, weight, or volume basis. The system must retain the source amount, allocation base, formula/version, rounding rule, and residual allocation.

The total allocated amount must reconcile exactly to the source amount at the permitted currency precision. Any rounding remainder must be assigned deterministically and disclosed; it must not disappear.

Pack, box, strip, tablet, bottle, vial, and other unit relationships must use an approved, versioned conversion. Cost per unit is calculated from the receipt quantity and conversion snapshot, not from a current editable catalogue conversion. A later conversion correction does not mutate historic receipt quantities or costs.

## 6. Receipt and invoice matching

Receipt cost may initially be provisional if supplier invoice evidence is not yet available. The UI and reports must distinguish provisional receipt cost from accepted/approved cost.

When an invoice arrives:
1. match supplier, purchase order, receipt, product, presentation, unit, quantity, currency, and price;
2. identify quantity, price, tax, discount, freight, and other differences;
3. route material differences for review;
4. apply approved cost adjustments using a traceable adjustment record;
5. preserve original receipt and invoice values;
6. recalculate only the projections permitted by the selected valuation policy;
7. produce a reconciliation report.

An invoice mismatch must not silently alter stock quantity or mark a supplier payable as settled. Payment remains a separate lifecycle.

## 7. Stock-count and ledger reconciliation

A stock count compares observed physical quantity with the system's expected quantity at a defined cutoff. It must preserve:
- count scope and snapshot/version;
- cutoff time and treatment of concurrent movements;
- counters, reviewers, and approval;
- observed quantity, unit, batch, location, and stock status;
- variance quantity and reason;
- posted gain/loss movements and references.

The system must not replace the ledger balance with a count value. A posted variance creates explicit movements, and the original expected quantity remains reconstructible.

Reconciliation reports must distinguish:
- quantity variance;
- unit or pack-conversion mismatch;
- batch identity/expiry mismatch;
- missing or duplicate receipt;
- unposted or delayed transaction;
- held, damaged, expired, recalled, or in-transit stock;
- unexplained or untraceable stock;
- cost-only variance.

A count taken while transactions continue must use an approved movement cutoff or version-conflict procedure. A stale count must be rejected or reviewed; it cannot erase intervening receipts, sales, returns, or transfers.

## 8. Returns, supplier credits, and cost treatment

Customer return disposition is independent of refund/payment state. Returned items remain non-saleable until the approved quality/disposition workflow allows otherwise. The cost effect must follow the approved accounting method and return policy; it must not assume that every returned item re-enters available inventory at its original cost.

Supplier returns reference the original receipt/batch and record quantity, condition, dispatch, supplier acknowledgement, credit note, and financial settlement separately. The system must avoid reducing inventory value twice when both a supplier credit and a stock movement are processed.

Refunds, credit notes, and supplier payments are financial events. They do not automatically create or reverse physical stock movements.

## 9. Transfers, expiry, damage, and write-downs

A branch transfer moves inventory within the same organization and must preserve cost provenance. It must not create revenue or organization-wide gross margin. Any transfer pricing or branch cost allocation must be explicitly approved and must not be confused with an external sale.

Expiry, damage, recall, theft/loss, and other write-offs require:
- affected batch and quantity;
- status and reason;
- evidence and approval where required;
- physical stock movement/disposition;
- separate financial valuation/write-down treatment;
- effective and recorded timestamps;
- links to recall, incident, or count records where applicable.

The quantity ledger and value calculation must reconcile independently. A write-down cannot silently dispose of physical stock, and a disposal cannot silently fabricate an accounting expense.

## 10. Reports and disclosure

Reports must disclose:
- tenant and branch scope;
- as-of timestamp and timezone;
- included stock statuses;
- currency;
- valuation method and policy version;
- cost source coverage and provisional/unknown portions;
- treatment of tax and landed costs;
- whether the result is operational, provisional, reconciled, or accounting-approved.

Candidate reports include on-hand quantity by batch, inventory value by batch/product/branch, cost-layer history, receipt-to-invoice variance, stock-count variance, write-off history, cost corrections, and cost-of-goods-sold/gross-margin reports only when the method and input quality are approved.

If any material portion of the inventory lacks reliable cost, the report must disclose coverage and avoid presenting an apparently precise total as complete.

## 11. Permissions and controls

Candidate permissions:
- inventory.valuation.read;
- inventory.valuation.policy.read;
- inventory.valuation.policy.propose;
- inventory.valuation.policy.approve;
- inventory.cost.adjust.request;
- inventory.cost.adjust.approve;
- inventory.reconciliation.read;
- inventory.reconciliation.resolve;
- inventory.writeoff.request;
- inventory.writeoff.approve;
- finance.cost_report.export.

Policy approval, cost adjustment, write-off approval, and export may require separation of duties, re-authentication, or a second approver. Thresholds must be established from field research and accounting review. Ordinary inventory management access does not itself imply professional credential authorization; regulated acts remain separately gated.

## 12. Required tests

Automated and acceptance tests must prove:
- all monetary calculations are exact and deterministic;
- valuation output is reproducible from source evidence and policy version;
- a purchase order or draft invoice alone does not create finalized inventory cost;
- partial receipt and invoice price variance are visible and correctly reconciled;
- landed-cost allocation sums exactly to the source amount;
- unit conversion changes do not rewrite historic costs;
- unknown cost is reported as unknown rather than zero or retail price;
- negative stock and unresolved cost layers produce explicit exceptions;
- count variance creates movements without overwriting the ledger;
- concurrent movements cannot be erased by a stale count;
- returns and refunds do not duplicate or incorrectly reverse stock/value effects;
- supplier credit and supplier-return processing do not double-count reductions;
- branch transfers do not create organization-wide sales or profit;
- expiry, damage, recall, and disposal have separate quantity and value effects;
- policy changes do not silently restate historical reports;
- unauthorized users cannot read, alter, approve, or export cost data.

## 13. Decisions required before enabling valuation

1. Which accounting framework and reporting purpose apply to the first-release customer cohort?
2. Which valuation method is acceptable and operationally practical for those customers?
3. Is valuation perpetual or periodic, and at what organizational/branch scope?
4. How are freight, discounts, taxes, rebates, and other landed costs treated?
5. Which receipt/invoice evidence establishes approved cost?
6. How are late invoices and backdated corrections handled?
7. How are returns, supplier credits, expiry, damage, recall, and write-offs valued?
8. Is branch-level cost pooling required, and how are internal transfers represented?
9. What cost precision and rounding rules apply by currency?
10. Which reports may be labelled operational versus accounting-approved?
11. What data quality is sufficient to calculate gross margin or cost of goods sold?
12. Who is the qualified accounting approver and how is approval evidence retained?

Until these questions are resolved, inventory quantity operations may proceed within their approved scope, but valuation, margin, and accounting claims remain disabled or explicitly provisional.

## 14. Cross-document dependencies

- [Specification 13](13-inventory-and-batch-lifecycle-specification.md): quantity ledger, count, batch, transfer, and disposition.
- [Specification 14](14-sales-pos-and-payment-lifecycle-specification.md): sale, return, refund, and payment separation.
- [Specification 15](15-supplier-and-purchasing-lifecycle-specification.md): purchase, receipt, invoice, credit note, and supplier return.
- [Specification 17](17-finance-and-reconciliation-specification.md): finance records, reconciliation, and reporting.
- [Specification 24](24-cross-module-transaction-and-implementation-contract.md): shared transaction, correction, and event contracts.
- [Specification 26](26-data-migration-import-export-backup-and-restoration-specification.md): opening balances, imported costs, and migration reconciliation.
