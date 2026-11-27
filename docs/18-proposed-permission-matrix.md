# 18 — Proposed role and permission matrix

**Status:** Draft — must be validated against first-release workflows and Ethiopian regulatory review  
**Purpose:** Turn the role model into an implementable authorization contract without making professional credentials a blanket platform-access condition.

## 1. How to read this matrix

This is a proposed starting matrix, not a final legal role prescription. It defines ordinary management permissions separately from professional/regulatory authorization.

- **Owner:** business owner or authorized organization administrator; may be a non-professional.
- **Manager:** branch/operations manager with assigned branch scope.
- **Professional:** person whose professional identity and credential have been verified for the relevant scope where required.
- **Cashier:** staff member assigned to permitted in-branch sales/cash tasks.
- **Inventory:** staff member assigned to receiving, counting, transfer, and stock tasks.
- **Purchasing:** staff member assigned to supplier and purchasing tasks.
- **Accountant:** staff member assigned to financial review/reconciliation.
- **Auditor:** read-only reviewer with explicitly granted scope.
- **Platform support:** Pharmalink support staff; no default tenant-data access.

A person may hold multiple roles. The effective permission set is the union of explicitly assigned grants, constrained by tenant/branch scope, resource state, entitlement, separation of duties, and any action-specific legal/professional requirement.

## 2. Platform and organization access

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| account.register / account.sign_in | Self | Self | Self | Self | Self | Self | Self | Self | Self | Authentication only; no professional credential prerequisite |
| organization.read | Yes | Assigned | Assigned | No by default | No by default | No by default | Limited | Read-only grant | No default | Tenant-level |
| organization.manage | Yes | Limited delegated | No by default | No | No | No | No | No | No default | Owner/admin permission; no regulated authority |
| branch.read | Yes | Assigned | Assigned | Assigned | Assigned | Assigned | Assigned | Read-only grant | No default | Branch scope |
| branch.manage | Yes | Assigned if delegated | No by default | No | No | No | No | No | No default | Owner or delegated manager; scope-limited |
| membership.invite | Yes | If delegated | No by default | No | No | No | No | No | No default | Invitation scope and intended role/branch required |
| membership.revoke | Yes | If delegated | No | No | No | No | No | No | No default | Revocation invalidates access within defined objective |
| role.assign / role.revoke | Yes | If delegated | No | No | No | No | No | No | No default | Cannot grant permissions beyond assigner's authority |
| device.manage | Yes | If delegated | No by default | No | No | No | No | No | No default | Register/revoke devices for tenant/branch |

## 3. Catalogue and inventory

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| catalogue.read / catalogue.search | Yes | Yes | Yes | Yes | Yes | Yes | Read if needed | Read-only grant | No default | Tenant data and permitted shared catalogue |
| tenant_catalogue.create / update | Yes | Yes | If granted | No by default | If granted | If granted | No | No | No default | Tenant-local data only |
| catalogue_correction.propose | Yes | Yes | Yes | If granted | If granted | If granted | No | If granted | No default | Proposal does not mutate canonical data |
| catalogue_correction.approve | No self-approval | No self-approval | Only if designated reviewer | No | No | No | No | No | Platform reviewer only | Platform-global governance, separated from proposal |
| inventory.read | Yes | Yes | Yes | Limited | Yes | Limited | Limited report | Read-only grant | No default | Branch scope |
| inventory.receive | Yes | Yes | If granted | No | Yes | If granted | No | No | No default | Accepted receipt creates stock movement |
| inventory.count | Yes | Yes | If granted | No | Yes | No | No | No | No default | Count session and snapshot/version required |
| inventory.adjust.request | Yes | Yes | If granted | No | Yes | No | No | No | No default | Reason and evidence required |
| inventory.adjust.approve | Yes if not requester | If delegated and not requester | If designated and legally permitted | No | No self-approval | No | No | No | No default | Threshold and separation policy apply |
| inventory.transfer.create / dispatch | Yes | Yes | If granted | No | Yes | If granted | No | No | No default | Source branch scope |
| inventory.transfer.receive | Yes | Yes | If granted | No | Yes | If granted | No | No | No default | Destination branch scope |
| inventory.hold | Yes | Yes | If granted | No | Yes | No | No | No | No default | Any staff may report an issue; authorized role applies hold |
| inventory.release | Yes if authorized | If delegated | If action requires professional review, verified professional only | No | No by default | No | No | No | No default | Release criteria depend on hold reason and law |
| inventory.disposal.request / approve | Yes / separate approver | If delegated | Where required by reviewed policy | No | Request if granted | No | No | No | No default | Evidence and approval rules required |
| inventory.export | Yes | If delegated | No by default | No | If granted | If granted | If granted | Read-only grant | No default | Scope-filtered export and audit |

## 4. Supplier and purchasing

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| supplier.read | Yes | Yes | If granted | No | If granted | Yes | Limited | Read-only grant | No default | Tenant scope |
| supplier.create / update | Yes | If delegated | No by default | No | No by default | Yes | No | No | No default | Sensitive supplier data separately protected |
| supplier.block | Yes | If delegated | No | No | No | If delegated | No | No | No default | Reason and audit required |
| purchase_order.create | Yes | If delegated | No by default | No | If granted | Yes | No | No | No default | Branch and supplier scope |
| purchase_order.submit | Yes | If delegated | No | No | If granted | Yes | No | No | No default | Submission does not equal approval |
| purchase_order.approve | Yes if not requester | If delegated and not requester | No by default | No | No by default | No self-approval | No | No | No default | Threshold and separation rules |
| purchase_order.issue / cancel | Yes | If delegated | No | No | No | Yes if approved | No | No | No default | External supplier communication logged |
| goods_receipt.create | Yes | Yes | If granted | No | Yes | If granted | No | No | No default | Actual quantity and batch evidence |
| goods_receipt.accept | Yes if authorized | If delegated | If required by policy | No | If granted | If granted | No | No | No default | Discrepancy/inspection policy applies |
| supplier_invoice.record | Yes | If delegated | No | No | No | Yes | Yes | No | No default | Recording does not approve payable |
| supplier_invoice.match | Yes if delegated | If delegated | No | No | No | Yes if granted | Yes if granted | No | No default | Matching is distinct from approval; preserve exceptions |\n| supplier_invoice.exception.resolve | Yes if authorized | If delegated | No | No | No | If granted | Yes if granted | No | No default | Evidence and reason required; approval separation applies |\n| supplier_invoice.approve | Yes if authorized | If delegated | No | No | No | No self-approval | Yes if delegated | No | No default | Accounting policy and match exceptions |
| supplier_payment.initiate / confirm | Yes by policy | If delegated | No | No | No | Initiate if granted | Confirm/reconcile if granted | No | No default | Payment confirmation must be authoritative |
| purchasing.export | Yes | If delegated | No | No | If granted | Yes | If granted | Read-only grant | No default | Tenant/branch scope |

## 5. Sales, payments, and finance

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| sale.create / commit | Yes if granted | Yes if granted | Yes if granted | Yes if granted | No by default | No | No | No | No default | Item eligibility and any regulated-action gate evaluated separately |
| sale.discount.request | Yes | Yes | If granted | If granted | No | No | No | No | No default | Threshold and reason |
| sale.discount.approve | Not self-approve where required | If delegated | If delegated | No self-approval | No | No | No | No | No default | Amount/role policy |
| sale.void.request | Yes | Yes | If granted | If granted | No | No | No | No | No default | Linked to original sale |
| sale.void.approve | Yes if not requester | If delegated and not requester | If delegated | No self-approval | No | No | If granted | No | No default | Separate approval policy |\n| customer_return.create / submit | Yes | If delegated | If granted | If granted | No | No | No | No | No default | Return is linked to original sale; no automatic refund/restock |\n| customer_return.review | Yes if not requester | If delegated and not requester | If designated | No self-approval | No | No | If granted | No | No default | Accept/reject based on policy and evidence |\n| inventory.return.disposition | Yes if authorized | If delegated | If required by policy | No | If granted | No | No | No | No default | Stock disposition is separate from return acceptance/refund |
| payment.initiate | Yes if granted | Yes if granted | If granted | Yes if granted | No | No | If granted | No | No default | Method/provider must be enabled |
| payment.confirm_manual | Yes if delegated | If delegated | No | No self-approval | No | No | Yes if granted | No | No default | Evidence and separation required |
| payment.refund.request / approve | Request | Approve if not requester | If granted | Request only if granted | No | No | Approve/reconcile if granted | No | No default | Provider/cash policy |
| finance.read | Yes | Limited | No by default | No | No | Limited | Yes | Read-only grant | No default | Branch/tenant scope |
| finance.reconcile | Yes if delegated | No by default | No | No | No | No | Yes | No | No default | Source evidence and exception handling |
| finance.export | Yes | If delegated | No | No | No | No | Yes | Read-only grant | No default | Sensitive export audit |
| cash_session.open / close | Yes | Yes | If granted | Open; close if policy permits | No | No | Review | No | No default | Cashier/session attribution and count |

## 6. Public participation and platform operations

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| listing.configure | Yes | If delegated | If granted | No | No | No | No | No | No default | Opt-in and field-level publication |
| listing.publish / pause | Yes | If delegated | If granted | No | No | No | No | No | No default | Capability-specific eligibility and review |
| availability.publish | Yes | If delegated | If granted | No | If granted | No | No | No | No default | Server-confirmed eligible stock only |
| verification.submit | Yes | If delegated | Self-submit credential evidence | No | No | No | No | No | No | Evidence remains unverified until review |
| verification.review / approve | No self-review | No | No self-review | No | No | No | No | No | Designated reviewer | Platform role; purpose-bound evidence access |
| support.case.create | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Case-scoped |
| support.tenant_data.access | No default | No | No | No | No | No | No | No | No default | Requires approved, time-limited, case-scoped grant |
| platform.security.admin | No | No | No | No | No | No | No | No | Designated security admin | Separate privileged platform role |

## 7. Professional and establishment checks

The following are separate checks and must never be represented by a single global verification flag:

- **Professional credential check:** person identity, credential type, issuer, validity, scope, and jurisdiction. Apply only to a specific regulated professional action or a public claim that depends on that credential.
- **Establishment evidence check:** organization/branch, establishment category, issuer, jurisdiction, validity, and capability. Apply only to an operation or claim that requires it.
- **Management entitlement check:** whether the tenant is entitled to use a software capability under the service relationship. This is not legal authorization.
- **Public participation check:** whether the organization/branch explicitly opted into a particular public capability and passed its required checks.

If a professional credential expires, only credential-dependent actions are blocked. If establishment evidence expires, affected operations/public claims are evaluated individually. Neither event automatically removes ordinary staff memberships or unrelated management permissions.

The exact credential and establishment dependencies remain unresolved until the relevant Ethiopian federal/regional rules are reviewed. No role table in this draft overrides applicable law.

## 8. Server-side authorization algorithm

For every command and protected read:
1. authenticate the person/service identity;
2. load active account and session state;
3. resolve tenant membership from trusted server-side identity context;
4. verify branch/resource scope;
5. evaluate explicit permission grant and role constraints;
6. evaluate management entitlement for the requested software capability;
7. validate current resource state and business invariants;
8. evaluate only the establishment/professional evidence checks required by this exact action;
9. enforce approval, separation-of-duties, and re-authentication;
10. record the authorization result and relevant audit reference.

The client must not decide whether an action is authorized. Hidden UI controls are usability behavior, not security.

## 9. Required negative authorization tests

- An owner without a professional credential can use permitted management functions but cannot perform a regulated act reserved to a qualified professional.
- A cashier can use granted POS functions without a professional credential where the operation does not require one.
- An inventory operator can receive/count stock without a professional credential where permitted.
- A purchasing operator can create and submit purchase orders without a professional credential.
- An accountant can read/reconcile only granted financial scopes and cannot silently alter stock.
- A professional credential does not grant organization ownership, finance access, or branch access by itself.
- An owner cannot bypass a required professional credential for a regulated action.
- A manager cannot access an unassigned branch by changing a request parameter.
- Revoked membership/permission blocks API requests and queued offline commands after the documented revocation objective.
- Public listing pause does not remove private management access.
- A platform support agent cannot access tenant data without an approved case-scoped grant.
- A reviewer cannot approve their own evidence submission.
- A credential or establishment-evidence expiry blocks only the capability that depends on it.

## 10. Approval and maintenance

This matrix must be reconciled with every module's permission vocabulary before implementation. Any new permission requires:
- a unique stable identifier;
- purpose and affected data;
- allowed roles and scopes;
- evidence dependencies;
- approval/re-authentication requirements;
- offline eligibility;
- audit event;
- positive and negative tests;
- owner and review date.

Do not create broad permissions such as `admin.all`, `pharmacy.verified`, or `professional.user` as authorization shortcuts. Emergency access must be separate, time-limited, approved, alerted, and reviewed.


## 11. Canonical permission vocabulary and reconciliation status

The existing role tables above are a high-level draft. Module specifications define more granular actions. A broad row such as `inventory.receive` or `sale.create / commit` must not be interpreted as an automatic grant of every related approval, export, correction, or reconciliation action.

The following identifiers are the canonical draft vocabulary. The module specification is the source of the action's business meaning; this register is the source of the stable identifier. **Role grants for identifiers marked “grant review required” are not approved and must not be enabled in production by inference.** The role-by-role grant table must be completed against the approved first-release workflows before implementation.

| Domain | Canonical permission identifiers | Source |
|---|---|---|
| Identity and organization | `account.register`, `account.sign_in`, `organization.read`, `organization.manage`, `branch.read`, `branch.manage`, `membership.invite`, `membership.revoke`, `role.assign`, `role.revoke`, `device.manage` | This specification; Spec 20 |
| Catalogue | `catalogue.read`, `catalogue.search`, `catalogue.export`, `tenant_catalogue.create`, `tenant_catalogue.update`, `tenant_catalogue.disable`, `catalogue_correction.propose`, `catalogue_correction.review`, `catalogue_correction.approve`, `barcode_mapping.propose`, `barcode_mapping.review`, `regulatory_classification.read`, `regulatory_classification.review` | Spec 16 |
| Inventory | `inventory.read`, `inventory.receive`, `inventory.count`, `inventory.adjust.request`, `inventory.adjust.approve`, `inventory.transfer.create`, `inventory.transfer.dispatch`, `inventory.transfer.receive`, `inventory.hold`, `inventory.release`, `inventory.disposal.request`, `inventory.disposal.approve`, `inventory.export` | Spec 13 |
| Supplier and purchasing | `supplier.read`, `supplier.create`, `supplier.update`, `supplier.block`, `supplier.sensitive_data.read`, `requisition.create`, `requisition.submit`, `requisition.approve`, `quotation.record`, `purchase_order.create`, `purchase_order.submit`, `purchase_order.approve`, `purchase_order.issue`, `purchase_order.cancel`, `goods_receipt.create`, `goods_receipt.accept`, `supplier_invoice.record`, `supplier_invoice.match`, `supplier_invoice.exception.resolve`, `supplier_invoice.approve`, `supplier_return.create`, `supplier_return.approve`, `supplier_payment.initiate`, `supplier_payment.confirm`, `supplier_payment.reconcile`, `purchasing.export` | Spec 15 |
| Sales and payments | `sale.create`, `sale.read`, `sale.commit`, `sale.discount.request`, `sale.discount.approve`, `sale.void.request`, `sale.void.approve`, `customer_return.create`, `customer_return.submit`, `customer_return.review`, `inventory.return.disposition`, `payment.initiate`, `payment.confirm_manual`, `payment.refund.request`, `payment.refund.approve`, `receipt.read`, `receipt.reissue`, `sales.export` | Spec 14 |
| Cash and finance | `finance.read`, `finance.cashbook.read`, `finance.cash_movement.create`, `finance.cash_session.open`, `finance.cash_session.count`, `finance.cash_session.close`, `finance.reconcile`, `finance.reconciliation_exception.resolve`, `finance.expense.create`, `finance.expense.approve`, `finance.journal.read`, `finance.journal.post`, `finance.journal.reverse`, `finance.period.close`, `finance.export`, `finance.sensitive_payment_data.read` | Spec 17; Spec 14 |
| Public participation | `public_profile.configure`, `public_profile.publish`, `public_profile.pause`, `public_profile.update_contact`, `public_profile.update_location`, `public_catalogue.configure`, `public_catalogue.publish`, `public_catalogue.pause`, `public_availability.confirm`, `public_availability.publish`, `public_availability.pause`, `public_enquiry.read`, `public_report.submit` | Spec 19 |
| Moderation and verification | `moderation.case.read`, `moderation.case.triage`, `moderation.content.suppress`, `moderation.decision.record`, `moderation.appeal.review`, `verification.evidence.submit`, `verification.evidence.review`, `verification.claim.publish` | Spec 19; Spec 20 |
| Platform operations and support | `support.case.create`, `support.tenant_data.access`, `platform.security.admin` | This specification; Specs 20, 28 |

### 11.1 Naming normalization

- Customer returns use `customer_return.*` permissions; stock disposition uses `inventory.return.disposition`; neither implies a payment refund. Supplier invoice permissions use the `supplier_invoice.*` namespace so invoice actions cannot be confused with generic invoices.\n- Cash-session actions use the `finance.cash_session.*` namespace because the session creates and reconciles cash-control records. Specification 14 must use these same identifiers when describing POS permissions.
- Cash and financial reconciliation actions use `finance.reconcile` and `finance.reconciliation_exception.resolve`; generic `reconciliation.read/resolve` must not be used as a separate ambiguous grant.
- Public profile/catalogue/availability identifiers in Specification 19 are canonical. Generic `listing.configure` and `listing.publish` are descriptive labels only and are not permission identifiers.
- `catalogue_correction.review` and `catalogue_correction.approve` are distinct actions. A reviewer may prepare a recommendation without having final approval authority.
- Compound labels in the earlier role tables (for example, `sale.create / commit`) are explanatory groupings, not a single permission that bypasses the separate draft and commit checks.

### 11.2 Required grant-resolution work

For every identifier above, the approved matrix must record:
- allowed role(s) and whether the grant is default, delegated, or individually assigned;
- tenant, branch, resource, and platform-global scope;
- requester/approver separation and amount or risk thresholds;
- professional credential or establishment-evidence dependency, if any, for this exact action;
- entitlement and public opt-in dependency, if any;
- re-authentication and audit requirements;
- online/offline eligibility and revocation behavior;
- positive, negative, cross-tenant, cross-branch, and revoked-grant tests.

A role title never grants a permission by implication. In particular, an owner, manager, or professional does not receive every action in the domain automatically. No blanket professional credential gate may be introduced for account access or ordinary management.

### 11.3 Implementation blocker

The current role tables do not yet assign every canonical identifier to roles with sufficient precision. This is an explicit unresolved authorization gap, not permission to let developers guess. Before implementing a capability, complete its role/scope/approval rows and tests. Capabilities without an approved grant mapping remain disabled or unavailable in production.
