# 17 — Finance, financial records, and reconciliation specification

**Status:** Draft — Ethiopian accounting/tax review and pharmacy field validation required  
**Priority:** High — financial records must be accurate, traceable, and recoverable  
**Scope:** Operational financial records, cash movements, receivables/payables references, expenses, payment reconciliation, financial reporting, and controlled exports. This is not yet a complete statutory accounting or tax-compliance specification.

## 1. Purpose and boundary

Pharmalink must provide reliable financial visibility for the approved pharmacy workflows without pretending that an unreviewed reporting module is a compliant accounting system.

Separate:
- operational transaction facts (sales, returns, purchases, receipts, expenses);
- payment/settlement facts (cash received, provider confirmation, bank settlement);
- accounting entries and balances;
- management reports and derived metrics;
- statutory tax/invoice records and filings.

A sale total is not necessarily cash received. A purchase order is not a payable. A supplier invoice is not proof of payment. Revenue, gross profit, cash flow, receivables, payables, and net profit are different measures.

The first release must state which finance capabilities are operational reports and which, if any, are reviewed accounting functions. Do not market unreviewed reports as audited, statutory, or tax-compliant accounts.

## 2. Financial integrity invariants

1. Store every amount with explicit currency and exact decimal/minor-unit semantics. Never use binary floating-point for money calculations.
2. Preserve the original transaction amount, currency, calculation inputs, rounding rule, and applied configuration version.
3. Finalized financial entries are immutable. Corrections use linked reversals, adjustments, credit/debit notes, or other approved records.
4. Every financial record has tenant/branch scope, actor/system attribution, effective time, recorded time, source transaction, and audit trail.
5. Payment initiation, payment confirmation, settlement, allocation, and reconciliation are distinct states.
6. No financial report may imply cash received from an unpaid or unconfirmed sale.
7. No payable may be inferred solely from a purchase order or unaccepted goods receipt.
8. Duplicate commands, imports, provider callbacks, and reconciliation files must not duplicate financial effects.
9. Financial totals must reconcile to their underlying source records for the same scope, period, currency, and as-of time.
10. Tenant and branch financial data is isolated server-side and exports are permission-controlled.
11. Subscription state must not erase tenant financial history or prevent lawful export.
12. Accounting/tax treatment must be configuration- and jurisdiction-aware and approved before production claims are made.

## 3. Financial record taxonomy

Candidate record types:

- **Operational source transaction:** sale, return, purchase receipt, supplier invoice, expense, cash movement.
- **Payment record:** cash receipt, card/provider payment, bank transfer, supplier payment, refund, reversal.
- **Receivable/payable record:** obligation recognized under an approved accounting policy.
- **Journal/ledger entry:** debit/credit or equivalent accounting record with account mapping and source reference.
- **Reconciliation record:** match or discrepancy between two authoritative records or data sources.
- **Tax/invoice record:** legally relevant document and applied tax configuration, subject to local review.
- **Report snapshot/export:** derived view with filters, source versions, and generation time.

Do not use one generic “transaction” record to represent all of these different facts.

## 4. Accounting model and chart of accounts

Before implementing a general ledger, decide whether the first release requires:
- cashbook and operational financial summaries only;
- configurable account mapping and journal export;
- a full double-entry general ledger;
- integration with an existing external accounting system.

If a ledger is implemented, specify:
- account identifiers, types, and hierarchy;
- debit/credit convention;
- journal and period lifecycle;
- source transaction mapping;
- posting and reversal rules;
- opening balances;
- closing and locked periods;
- multi-branch consolidation;
- currency and foreign-exchange treatment if applicable;
- audit and export requirements.

Do not silently label a cashbook or summary report as a full accounting ledger. The chart of accounts and posting rules require qualified accounting review.

## 5. Revenue, sales, returns, and cost of goods sold

Sales and payments are specified in [Specification 14](14-sales-pos-and-payment-lifecycle-specification.md). Inventory cost and valuation depend on [Specification 13](13-inventory-and-batch-lifecycle-specification.md).

The finance model must distinguish:
- gross sales;
- discounts;
- returns and refunds;
- net sales;
- tax collected/owed where applicable;
- payment amounts initiated, confirmed, and settled;
- cash received;
- receivables;
- cost of goods sold;
- gross profit;
- operating expenses;
- net profit only where the accounting model supports it.

Cost of goods sold requires an approved stock valuation method and cost allocation rules. Do not calculate margin from current purchase price if historical cost differs. Store cost snapshots or ledger references so later supplier-price changes do not rewrite past results.

## 6. Supplier liabilities and purchasing

Purchasing is specified in [Specification 15](15-supplier-and-purchasing-lifecycle-specification.md).

The finance workflow must define when an accepted supplier invoice becomes an approved payable, how credit notes and supplier returns affect it, and how partial/combined payments are allocated.

A purchase order, quotation, or unaccepted receipt must not silently create a payable. Invoice matching exceptions must remain visible until resolved. Supplier payments require authoritative confirmation and reconciliation before they are shown as settled.

## 7. Cashbook and cash sessions

If included in the first release, cashbook records should distinguish:
- cash sale receipt;
- customer cash refund;
- opening float;
- paid-in and paid-out;
- supplier cash payment if allowed;
- owner/manager cash movement;
- bank deposit/withdrawal;
- cash count and discrepancy.

Each cash movement records actor, session, branch, amount, currency, direction, reason, source transaction, effective/recorded time, and approval where required.

A cash session may have opening, active, closing-count, review-required, and closed states. Closing compares expected cash with counted cash and records variance; it does not silently rewrite sale or payment records. Corrections are linked movements. The system must define whether one cashier may hold multiple sessions and how shared tills are attributed.

## 8. Payment reconciliation

Reconciliation compares Pharmalink's payment records with external provider/bank/cash evidence.

Each reconciliation batch should record:
- source/provider and account reference;
- imported file/API reference and hash where appropriate;
- covered period and currency;
- source row/transaction identifiers;
- import time and actor/system;
- matching rules/version;
- matched, unmatched, duplicate, reversed, and disputed totals;
- exception owner, status, and resolution;
- final reviewer and approval.

Matching must use stable references and amounts/currency/date within an explicitly defined tolerance. Fuzzy matches may suggest candidates but must not automatically settle material discrepancies.

Imports must be idempotent. Reprocessing the same file or provider event cannot duplicate payments, refunds, fees, or ledger postings. Preserve source provenance and report rejected rows.

## 9. Expenses and other operating costs

If expense tracking is included:
- define permitted expense categories and account mappings;
- record payee, date, amount, currency, tax/receipt evidence, branch, payment status, and creator;
- define approval thresholds and attachment access;
- distinguish expense recognition from payment;
- support correction/reversal without deleting history;
- avoid collecting unnecessary personal data from payees.

Whether expenses and budgets are in the first release remains an open scope decision.

## 10. Multi-branch and multi-currency behavior

Every financial fact must have explicit tenant and branch ownership or an explicitly defined organization-wide scope. Consolidated reports must:
- state included branches and exclusions;
- avoid double-counting transfers between branches;
- distinguish branch-local cash from centralized payments;
- preserve source currency;
- use an approved exchange-rate source and date if multi-currency is supported;
- never combine amounts in different currencies without explicit conversion metadata.

Whether the first release supports one currency only or multi-currency is an open decision.

## 11. Tax, invoices, and statutory reporting

Tax rates, invoice formats, fiscal devices, numbering, retention, and statutory exports require current Ethiopian federal/regional review and qualified accounting/legal advice.

The system must:
- version tax configuration and preserve the version applied to each finalized transaction;
- prevent unauthorized backdating or rewriting of issued documents;
- record correction documents and their references;
- distinguish estimates from approved statutory calculations;
- identify unsupported tax scenarios rather than silently applying a default;
- retain required records and export evidence.

Do not hard-code a tax rate, generate a statutory compliance claim, or mark a fiscal integration as complete until verified for the launch jurisdiction and establishment category.

## 12. Reporting and definitions

Every report must specify:
- metric definition and calculation version;
- tenant/branch scope;
- date/time range and timezone;
- currency and any conversion basis;
- source records and status inclusion rules;
- generated-at/as-of time;
- filters and excluded records;
- whether the result is provisional, operational, reconciled, or accounting-approved.

Required candidate reports:
- sales and returns by period, branch, and cashier;
- payment method and settlement status;
- cash session and variance;
- supplier payable and aging;
- supplier payment and reconciliation exceptions;
- expenses by category;
- inventory valuation (only after method approval);
- gross margin (only with valid cost basis);
- tax summaries (only after review);
- audit and correction history.

Users must be able to trace a reported total to its source records subject to permission.

## 13. Permissions and separation of duties

Candidate permissions:
- finance.read;
- finance.cashbook.read;
- finance.cash_movement.create;
- finance.cash_session.open;
- finance.cash_session.close;
- finance.reconcile;
- finance.reconciliation_exception.resolve;
- finance.expense.create;
- finance.expense.approve;
- finance.journal.read;
- finance.journal.post;
- finance.journal.reverse;
- finance.period.close;
- finance.export;
- finance.sensitive_payment_data.read.

The final matrix must define branch/tenant scope, approval thresholds, self-approval restrictions, re-authentication, audit events, and export controls. A cashier does not automatically gain access to profit, supplier bank, payroll, or owner financial data. A business owner does not bypass professional/legal controls on regulated workflows.

## 14. Offline and event-driven behavior

- Finalizing financial transactions, confirming payments, posting journals, closing periods, and resolving reconciliation exceptions are online-only by default.
- Local report snapshots may be cached only with scope, permission, generation time, and stale/offline labels.
- Financial commands use idempotency keys and server-side authorization.
- Financial events are published through the transactional outbox only after commit.
- Consumers are idempotent and recover missed events through replay/reconciliation.
- Relevant dashboards subscribe to changes; do not poll continuously for unchanged financial totals.
- Push notifications must not expose sensitive amounts or account details unless a specific reviewed notification policy permits it.

## 15. Required test scenarios

Automated tests must prove:
- money arithmetic and rounding are exact and deterministic;
- currency mismatches cannot be silently combined;
- a sale is not counted as cash received before confirmed payment;
- purchase orders and unaccepted receipts do not create payables;
- finalized entries cannot be edited or deleted through ordinary APIs;
- reversals and refunds link to the original source and do not duplicate effects;
- provider callback and reconciliation-file replays are idempotent;
- reconciliation totals and exceptions are reproducible;
- multi-branch reports do not double-count internal transfers;
- reports identify their scope, period, currency, and calculation basis;
- unauthorized users cannot read/export restricted finance data;
- tenant/branch isolation holds for dashboards, exports, jobs, and events;
- closed periods and issued documents cannot be silently rewritten;
- subscription suspension does not erase history or block lawful export.

## 16. Open decisions before implementation

1. Is the first release a cashbook/operational reporting tool, a journal-export tool, or a full general ledger?
2. Which accounting standards and local statutory requirements apply to target customers?
3. Which chart of accounts and account mappings are appropriate?
4. Which inventory valuation and cost-of-goods-sold method will be supported?
5. Are expenses, budgets, receivables, and payables in the first release?
6. Which local payment/banking providers and statement formats must be supported?
7. What are the rules for partial payments, fees, withholding, refunds, and credit notes?
8. Which tax configurations, invoice formats, fiscal devices, and statutory exports apply to the launch region?
9. Is multi-currency required at launch?
10. What financial records must remain accessible during subscription suspension and after tenant exit?
11. Which actions require two-person approval or re-authentication?
12. What is the expected reporting scale and export format required by pilot pharmacies?

Until these decisions are validated, label reports accurately as operational/provisional and do not claim full accounting or tax compliance.
