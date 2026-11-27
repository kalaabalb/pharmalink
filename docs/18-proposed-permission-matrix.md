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
| membership.accept | Self | Self | Self | Self | Self | Self | Self | Self | No default | Authenticated invitee accepts only the invitation addressed to them |
| membership.restrict | Yes | If delegated | No | No | No | No | No | No | No default | Scoped restriction/suspension with reason and review path |
| membership.revoke | Yes | If delegated | No | No | No | No | No | No | No default | Revocation invalidates access within defined objective |
| role.assign / role.revoke | Yes | If delegated | No | No | No | No | No | No | No default | Cannot grant permissions beyond assigner's authority |
| device.manage | Yes | If delegated | No by default | No | No | No | No | No | No default | Register/revoke devices for tenant/branch |

## 3. Catalogue and inventory

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| catalogue.read / catalogue.search | Yes | Yes | Yes | Yes | Yes | Yes | Read if needed | Read-only grant | No default | Tenant data and permitted shared catalogue |
| catalogue.export | Yes | If delegated | No by default | No | If granted | If granted | If granted | Read-only grant | No default | Tenant-scoped export with audit and field minimization |
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
| inventory.transfer.cancel | Yes | If delegated | If granted | No | If granted | If granted | No | No | No default | Only before dispatch; dispatched transfer needs discrepancy resolution |
| inventory.hold | Yes | Yes | If granted | No | Yes | No | No | No | No default | Any staff may report an issue; authorized role applies hold |
| inventory.release | Yes if authorized | If delegated | If action requires professional review, verified professional only | No | No by default | No | No | No | No default | Release criteria depend on hold reason and law |
| inventory.disposal.request / approve | Yes / separate approver | If delegated | Where required by reviewed policy | No | Request if granted | No | No | No | No default | Evidence and approval rules required |
| inventory.return.disposition | Yes if authorized | If delegated | If required by policy | No | If granted | No | No | No | No default | Separate from customer-return acceptance and refund |
| inventory.export | Yes | If delegated | No by default | No | If granted | If granted | If granted | Read-only grant | No default | Scope-filtered export and audit |

## 4. Supplier and purchasing

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| supplier.read | Yes | Yes | If granted | No | If granted | Yes | Limited | Read-only grant | No default | Tenant scope |
| supplier.create / update | Yes | If delegated | No by default | No | No by default | Yes | No | No | No default | Sensitive supplier data separately protected |
| supplier.block | Yes | If delegated | No | No | No | If delegated | No | No | No default | Reason and audit required |
| purchase_order.read | Yes | Yes | If granted | No | If granted | Yes | If granted | Read-only grant | No default | Order detail and branch scope; supplier-sensitive fields filtered |
| purchase_order.create | Yes | If delegated | No by default | No | If granted | Yes | No | No | No default | Branch and supplier scope |
| purchase_order.submit | Yes | If delegated | No | No | If granted | Yes | No | No | No default | Submission does not equal approval |
| purchase_order.approve | Yes if not requester | If delegated and not requester | No by default | No | No by default | No self-approval | No | No | No default | Threshold and separation rules |
| purchase_order.issue / cancel | Yes | If delegated | No | No | No | Yes if approved | No | No | No default | External supplier communication logged |
| purchase_order.close | Yes | If delegated | No | No | If granted | If granted | If granted | No | No default | Only after operational and financial closure conditions pass |
| goods_receipt.read | Yes | Yes | If granted | No | Yes | If granted | If granted | Read-only grant | No default | Receipt/batch detail; pricing fields separately filtered |
| goods_receipt.create | Yes | Yes | If granted | No | Yes | If granted | No | No | No default | Actual quantity and batch evidence |
| goods_receipt.accept | Yes if authorized | If delegated | If required by policy | No | If granted | If granted | No | No | No default | Discrepancy/inspection policy applies |
| supplier_invoice.read | Yes | If delegated | No | No | No | If granted | Yes | Read-only grant | No default | Invoice and matching evidence; sensitive financial fields restricted |
| supplier_invoice.record | Yes | If delegated | No | No | No | Yes | Yes | No | No default | Recording does not approve payable |
| supplier_invoice.match | Yes if delegated | If delegated | No | No | No | Yes if granted | Yes if granted | No | No default | Matching is distinct from approval; preserve exceptions |
| supplier_invoice.exception.resolve | Yes if authorized | If delegated | No | No | No | If granted | Yes if granted | No | No default | Evidence and reason required; approval separation applies |
| supplier_invoice.dispute | Yes if authorized | If delegated | No | No | No | If granted | Yes if granted | No | No default | Records a supplier dispute; does not approve or settle invoice |
| supplier_invoice.approve | Yes if authorized | If delegated | No | No | No | No self-approval | Yes if delegated | No | No default | Accounting policy and match exceptions |
| supplier_payment.read | Yes | If delegated | No | No | No | If granted | Yes | Read-only grant | No default | Payment references and reconciliation status; bank details separately gated |
| supplier_payment.initiate / confirm | Yes by policy | If delegated | No | No | No | Initiate if granted | Confirm/reconcile if granted | No | No default | Payment confirmation must be authoritative |
| purchasing.export | Yes | If delegated | No | No | If granted | Yes | If granted | Read-only grant | No default | Tenant/branch scope |

## 5. Sales, payments, and finance

| Permission | Owner | Manager | Professional | Cashier | Inventory | Purchasing | Accountant | Auditor | Platform support | Scope / conditions |
|---|---|---|---|---|---|---|---|---|---|---|
| payment.read | Yes | If delegated | If granted | Assigned sales only | No | No | Yes | Read-only grant | No default | Payment attempt/status for permitted sale and branch scope |
| customer_return.read | Yes | If delegated | If assigned | Assigned branch only | No | No | Limited read | Read-only grant | No default | Return status linked to permitted original sale |
| sale.read | Yes | Yes | If granted | Assigned sales only | No by default | No | Limited report | Read-only grant | No default | Original sale and permitted branch scope |
| sale.create / commit | Yes if granted | Yes if granted | Yes if granted | Yes if granted | No by default | No | No | No | No default | Item eligibility and any regulated-action gate evaluated separately |
| sale.discount.request | Yes | Yes | If granted | If granted | No | No | No | No | No default | Threshold and reason |
| sale.discount.approve | Not self-approve where required | If delegated | If delegated | No self-approval | No | No | No | No | No default | Amount/role policy |
| sale.void.request | Yes | Yes | If granted | If granted | No | No | No | No | No default | Linked to original sale |
| sale.void.approve | Yes if not requester | If delegated and not requester | If delegated | No self-approval | No | No | If granted | No | No default | Separate approval policy |
| customer_return.create / submit | Yes | If delegated | If granted | If granted | No | No | No | No | No default | Return is linked to original sale; no automatic refund/restock |
| customer_return.review | Yes if not requester | If delegated and not requester | If designated | No self-approval | No | No | If granted | No | No default | Accept/reject based on policy and evidence |
| payment.initiate | Yes if granted | Yes if granted | If granted | Yes if granted | No | No | If granted | No | No default | Method/provider must be enabled |
| payment.confirm_manual | Yes if delegated | If delegated | No | No self-approval | No | No | Yes if granted | No | No default | Evidence and separation required |
| payment.refund.request / approve | Request | Approve if not requester | If granted | Request only if granted | No | No | Approve/reconcile if granted | No | No default | Provider/cash policy |
| finance.read | Yes | Limited | No by default | No | No | Limited | Yes | Read-only grant | No default | Branch/tenant scope |
| finance.reconcile | Yes if delegated | No by default | No | No | No | No | Yes | No | No default | Source evidence and exception handling |
| finance.export | Yes | If delegated | No | No | No | No | Yes | Read-only grant | No default | Sensitive export audit |
| finance.cash_session.open | Yes | Yes | If granted | Open if assigned | No | No | No | No | No default | Cashier/session and till scope |
| finance.cash_session.count | Yes | Yes | If granted | Count own session if policy permits | No | No | Review if granted | No | No default | Count does not close or rewrite the session |
| finance.cash_session.close | Yes if authorized | If delegated | If granted | Close own session only if policy permits | No | No | Review/close if granted | No | No default | Variance policy and separation apply |

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
| platform.organization.restrict | No | No | No | No | No | No | No | No | Designated platform policy admin | Restrict/suspend a defined organization capability with reason and review path |
| platform.entitlement.manage | No | No | No | No | No | No | No | No | Designated platform entitlement admin | Manage software entitlement only; not legal authorization |
| platform.security.admin | No | No | No | No | No | No | No | No | Designated security admin | Separate privileged platform role |

## 7. Professional and establishment checks

The following are separate checks and must never be represented by a single global verification flag:

- **Professional credential check:** person identity, credential type, issuer, validity, scope, and jurisdiction. Apply only to a specific regulated professional action or a public claim that depends on that credential.
- **Establishment evidence check:** organization/branch, establishment category, issuer, jurisdiction, validity, and capability. Apply only to an operation or claim that requires it.
- **Management entitlement check:** whether the tenant is entitled to use a software capability under the service relationship. This is not legal authorization.
- **Public participation check:** whether the organization/branch explicitly opted into a particular public capability and passed its required checks.

If a professional credential expires, only credential-dependent actions are blocked. If establishment evidence expires, affected operations/public claims are evaluated individually. Neither event automatically removes ordinary staff memberships or unrelated management permissions.


The exact credential and establishment dependencies remain unresolved until the relevant Ethiopian federal/regional rules are reviewed. No role table in this draft overrides applicable law.

### 7.1 Action-specific evidence dependency register (draft)

This register defines the product's default separation rules. It does **not** make a legal determination that a particular establishment, product, or person may lawfully perform an activity. A dependency marked **conditional / unresolved** must be resolved for the launch jurisdiction, establishment category, product category, and exact activity before that capability is enabled.

| Action or capability | Establishment evidence | Individual professional credential | First-release handling |
|---|---|---|---|
| Account registration, sign-in, invitation acceptance, ordinary membership | None | None | Available under identity and abuse controls; never gate on a pharmacy licence or professional credential |
| Organization setup and private management configuration | Evidence of representative authority only where proportionate/required; not automatically a pharmacy operating licence | None | Permit setup and unrelated management while any required establishment review is pending; do not claim legal operating status |
| Private catalogue maintenance, supplier records, purchase orders, purchasing queries | None by default | None | Ordinary tenant-scoped grants; product classification must not be invented from free text |
| Goods receipt, batch capture, stock counts, holds, permitted inventory corrections | None by default for software access; establishment-specific operating obligations remain outside a blanket credential gate | None by default | Allow ordinary stock-management roles; restricted handling/disposal actions require their own reviewed policy |
| In-branch POS sale of a product in a reviewed, supported category | Establishment eligibility for the actual category/location only where the reviewed law requires it | None for ordinary cashier operations unless the exact sale includes a legally reserved professional act | Enable only reviewed product/action combinations; separate ordinary sale recording from professional dispensing/authorization |
| Prescription-controlled, restricted-category, professional dispensing, or other regulated act | Conditional; exact establishment category and licence scope must be confirmed | Conditional; exact professional category, scope, identity match, and current status must be confirmed | Disabled until the action-specific legal matrix and credential verification source are approved; a POS permission is never a substitute |
| Payment initiation, manual confirmation, cash session, reconciliation, ordinary financial reporting | None | None | Gate by finance permissions, evidence, separation, and provider policy—not professional credentials |
| Return review, refund request/approval, and ordinary sale correction | None by default | None by default | Gate by workflow permissions and accounting policy; if the return/disposition involves a separately regulated activity, evaluate that activity independently |
| Inventory disposal, controlled/restricted stock handling, recall execution | Conditional; category, establishment, jurisdiction, method, and any witness requirements need review | Conditional only if the specific action is legally reserved to a qualified professional | Keep disabled until the relevant product-category and regional rules, evidence, and roles are approved |
| Public establishment profile or establishment verification claim | Evidence required only for the specific claim and jurisdiction; publication is separately opt-in | No blanket individual credential requirement | Public discovery is deferred from first release; never imply that an individual credential verifies the whole establishment |
| Public professional credential claim | Not a substitute for establishment evidence | The named person's verified credential and claim scope | Public claim must name the person, scope, issuer/method, and validity; no organization-wide inference |
| Platform support, customer support case creation, and software administration | None | None | Separate service permissions; no legal/professional authority implied |

**Enforcement rules:**

1. The server resolves evidence dependencies from a versioned action-policy registry using the exact action, jurisdiction, establishment category, product classification, and effective date. A client may submit observed facts or evidence references but may not choose the policy, declare itself exempt, or assert that a credential is valid.
2. A policy entry must state \`no_evidence_required\`, \`establishment_evidence_required\`, \`professional_credential_required\`, \`both_required\`, or \`unresolved\`. \`unresolved\` is a release blocker for the affected action/category; it must not be silently treated as \`no_evidence_required\`.
3. A required establishment check binds evidence to the actual legal entity/establishment/branch, issuer, jurisdiction, establishment category, licence scope, validity period, and authoritative/manual verification method.
4. A required professional check binds the credential to the acting person, identity match, issuer, professional category, jurisdiction, permitted scope, validity period, and current status. A role title or uploaded image alone never satisfies it.
5. Evidence is checked at the point of authorization and rechecked for long-running approvals or delayed execution. Expiry, revocation, suspension, policy change, or scope mismatch blocks only the dependent action. It does not retroactively rewrite historical attribution or disable unrelated management access.
6. If authoritative verification is unavailable, the state remains pending/manual-review as defined by policy. The system must not present submitted evidence as verified or silently downgrade a required check.
7. The action-policy registry must retain source references, reviewer/legal approval, policy version, effective dates, decision rationale, and a regression-test set. A change to law, source, category, or policy requires review and controlled rollout.
8. No regulated action may be enabled in production until its policy entry, evidence source/method, permitted actor scope, positive/negative tests, and accountable legal/regulatory approval are recorded.

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
- A permission grant cannot override an action-policy entry marked `unresolved`, or a required establishment/professional evidence check that is missing, expired, revoked, suspended, out of scope, or bound to another person/establishment/branch.
- The client cannot choose the action-policy ID/version, jurisdiction, establishment category, product classification, exemption, or evidence-verification outcome.
- A policy lookup with an unknown jurisdiction, ambiguous product classification, or unsupported establishment category blocks only the affected regulated action and returns a safe review-required outcome; ordinary private management remains usable.
- A delayed approval/execution rechecks the current policy version and required evidence; a credential or establishment status changed after the original request cannot be reused without revalidation.
- Evidence submission, upload, or self-entered licence number never produces a verified status without the configured authoritative/manual review.
- Evidence access is restricted and audited; raw evidence documents do not appear in logs, events, analytics, or push notifications.

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
| Identity and organization | `account.register`, `account.sign_in`, `organization.read`, `organization.manage`, `branch.read`, `branch.manage`, `membership.invite`, `membership.accept`, `membership.restrict`, `membership.revoke`, `role.assign`, `role.revoke`, `device.manage` | This specification; Spec 20 |
| Catalogue | `catalogue.read`, `catalogue.search`, `catalogue.export`, `tenant_catalogue.create`, `tenant_catalogue.update`, `tenant_catalogue.disable`, `catalogue_correction.propose`, `catalogue_correction.review`, `catalogue_correction.approve`, `barcode_mapping.propose`, `barcode_mapping.review`, `regulatory_classification.read`, `regulatory_classification.review` | Spec 16 |
| Inventory | `inventory.read`, `inventory.receive`, `inventory.count`, `inventory.adjust.request`, `inventory.adjust.approve`, `inventory.transfer.create`, `inventory.transfer.dispatch`, `inventory.transfer.receive`, `inventory.transfer.cancel`, `inventory.hold`, `inventory.release`, `inventory.disposal.request`, `inventory.disposal.approve`, `inventory.return.disposition`, `inventory.export` | Spec 13 |
| Supplier and purchasing | `supplier.read`, `requisition.read`, `quotation.read`, `purchase_order.read`, `goods_receipt.read`, `supplier_invoice.read`, `supplier_payment.read`, `supplier.create`, `supplier.update`, `supplier.block`, `supplier.sensitive_data.read`, `requisition.create`, `requisition.submit`, `requisition.approve`, `quotation.record`, `purchase_order.create`, `purchase_order.submit`, `purchase_order.approve`, `purchase_order.issue`, `purchase_order.cancel`, `purchase_order.close`, `goods_receipt.create`, `goods_receipt.accept`, `supplier_invoice.record`, `supplier_invoice.match`, `supplier_invoice.exception.resolve`, `supplier_invoice.dispute`, `supplier_invoice.approve`, `supplier_return.create`, `supplier_return.approve`, `supplier_payment.initiate`, `supplier_payment.confirm`, `supplier_payment.reconcile`, `purchasing.export` | Spec 15 |
| Sales and payments | `sale.create`, `sale.read`, `sale.commit`, `payment.read`, `customer_return.read`, `sale.discount.request`, `sale.discount.approve`, `sale.void.request`, `sale.void.approve`, `customer_return.create`, `customer_return.submit`, `customer_return.review`, `payment.initiate`, `payment.confirm_manual`, `payment.refund.request`, `payment.refund.approve`, `receipt.read`, `receipt.reissue`, `sales.export` | Spec 14 |
| Cash and finance | `finance.read`, `finance.cashbook.read`, `finance.cash_movement.create`, `finance.cash_session.open`, `finance.cash_session.count`, `finance.cash_session.close`, `finance.reconcile`, `finance.reconciliation_exception.resolve`, `finance.expense.create`, `finance.expense.approve`, `finance.journal.read`, `finance.journal.post`, `finance.journal.reverse`, `finance.period.close`, `finance.export`, `finance.sensitive_payment_data.read` | Spec 17; Spec 14 |
| Public participation | `public_profile.configure`, `public_profile.publish`, `public_profile.pause`, `public_profile.update_contact`, `public_profile.update_location`, `public_catalogue.configure`, `public_catalogue.publish`, `public_catalogue.pause`, `public_availability.confirm`, `public_availability.publish`, `public_availability.pause`, `public_enquiry.read`, `public_report.submit` | Spec 19 |
| Moderation and verification | `moderation.case.read`, `moderation.case.triage`, `moderation.content.suppress`, `moderation.decision.record`, `moderation.appeal.review`, `verification.evidence.submit`, `verification.evidence.review`, `verification.claim.publish` | Spec 19; Spec 20 |
| Platform operations and support | `support.case.create`, `support.tenant_data.access`, `platform.organization.restrict`, `platform.entitlement.manage`, `platform.security.admin` | This specification; Specs 20, 28 |

### 11.1 Naming normalization

- Customer returns use `customer_return.*` permissions; stock disposition uses `inventory.return.disposition`; neither implies a payment refund. Supplier invoice permissions use the `supplier_invoice.*` namespace so invoice actions cannot be confused with generic invoices.
- Cash-session actions use the `finance.cash_session.*` namespace because the session creates and reconciles cash-control records. Specification 14 must use these same identifiers when describing POS permissions.
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


## 12. Proposed first-release role-grant baseline (review draft)

This section narrows grant design to the private pharmacy-operations candidate in Specification 25. It is a **proposal for product-owner, pharmacy-operator, accounting, and regulatory review**, not an approved grant set. “Eligible” means a role may be considered for an explicit scoped grant; it does not mean the permission is automatically assigned.

### 12.1 Grant semantics

- Account registration and sign-in are self-service identity actions, not tenant permissions.
- The first organization owner receives only the minimum bootstrap grants required to create the workspace, manage its initial branch, invite staff, and assign explicitly approved roles. Ownership does not imply every operational or regulated permission.
- Every other grant is explicit, tenant-scoped, and branch-scoped where the resource is branch-owned. Role templates are convenience bundles, not authorization evidence.
- A person may hold multiple roles, but each effective permission is still recorded and evaluated independently.
- No professional credential is required for ordinary membership, account access, purchasing, inventory management, or general software administration. A credential gate is added only to the exact regulated action/category that qualified review identifies.
- A role grant or permission is necessary but never sufficient for an action whose action-policy registry requires establishment or professional evidence. The server must evaluate the applicable jurisdiction/category/action policy at execution time; a grant cannot override an unresolved or failed evidence dependency.
- Where an approval action is listed, the requester cannot approve their own request unless a documented, legally/accountingly acceptable single-person exception is approved with compensating review.
- Cashier, inventory, purchasing, and accountant grants are constrained to assigned branches and duties. A job title alone never creates access.
- Public discovery/availability permissions and platform-global support/security permissions are not part of the private first-release grant baseline. They remain disabled or separately controlled until their own release gates pass.

### 12.2 Candidate role eligibility by first-release permission

| Permission identifier | Candidate eligible roles | Mandatory scope and control |
|---|---|---|
| `organization.read` | Owner, Manager; Auditor read-only; Accountant if needed | Own tenant only |
| `organization.manage` | Owner; Manager only by explicit delegation | Tenant-level configuration; no professional authority |
| `branch.read` | Owner, Manager, assigned Professional, Cashier, Inventory, Purchasing, Accountant; Auditor read-only | Assigned branch; accountant/auditor read scopes |
| `branch.manage` | Owner; delegated Manager | Own tenant; preserve branch history |
| `membership.invite` | Owner; delegated Manager | Invitation role/scope cannot exceed inviter delegation |
| `membership.accept` | Intended authenticated invitee | Self only; invitation token and intended identity verified |
| `membership.restrict` | Owner; delegated Manager | Reason, scope, effective time, review path, session/queue revocation |
| `membership.revoke` | Owner; delegated Manager | Preserve attribution and historical records |
| `role.assign` | Owner; delegated Manager within a bounded grant allowlist | Cannot grant beyond own delegation or approve own privilege escalation |
| `role.revoke` | Owner; delegated Manager | Immediate server-side re-evaluation and documented revocation objective |
| `device.manage` | Owner; delegated Manager | Tenant/branch device ownership and revocation audit |
| `catalogue.read`, `catalogue.search` | Owner, Manager, assigned Professional, Cashier, Inventory, Purchasing; Accountant/Auditor read-only where needed | Tenant catalogue and permitted shared catalogue only |
| `catalogue.export` | Owner; Manager, Inventory, Purchasing, Accountant, Auditor only with explicit scope grant | Tenant-scoped export with audit and field minimization |
| `tenant_catalogue.create`, `tenant_catalogue.update` | Owner, Manager; individually assigned Inventory/Purchasing/Professional | Tenant-local catalogue; provenance and version history |
| `tenant_catalogue.disable` | Owner; delegated Manager | Preserve references from historical transactions |
| `catalogue_correction.propose` | Owner, Manager, assigned Professional, Inventory, Purchasing | Proposal only; cannot mutate canonical shared catalogue |
| `inventory.read` | Owner, Manager, assigned Professional, Inventory; Cashier limited to sale-relevant availability; Purchasing limited to purchasing scope; Accountant/Auditor read-only | Branch scope and field-level filtering |
| `inventory.receive`, `goods_receipt.create` | Owner, Manager, Inventory; Purchasing if explicitly assigned | `goods_receipt.create` permits draft creation; `inventory.receive` is required with `goods_receipt.accept` for authorized acceptance/receipt posting. The final `goods_receipt.commit` is an internal continuation, not a separately granted action. Actual branch, item, batch, expiry, unit, quantity and source evidence |
| `goods_receipt.accept` | Owner or delegated Manager; assigned Inventory/Purchasing operator | Inspection/discrepancy policy; approval separation for material exceptions |
| `inventory.count` | Owner, Manager, Inventory; Professional if assigned | Count scope, cutoff, counter identity, device and version |
| `inventory.adjust.request` | Owner, Manager, Inventory | Reason and evidence; requester identity retained |
| `inventory.adjust.approve` | Owner or delegated Manager; never the requester | Threshold, evidence, separation and audit |
| `inventory.transfer.create`, `inventory.transfer.dispatch`, `inventory.transfer.receive`, `inventory.transfer.cancel` | Owner, Manager, Inventory; Purchasing only if explicitly assigned | Grant only if the pilot includes multiple operational branches and transfer workflows pass acceptance; otherwise keep all transfer commands disabled. Source/destination scopes differ; cancellation only before dispatch |
| `inventory.hold` | Owner, Manager, Inventory; assigned Professional where appropriate | Any staff may report concern; only an authorized role applies the hold |
| `inventory.release` | Owner, Manager, Inventory; verified Professional only where the hold reason/legal rule requires it | Evidence-specific release; expiry/recall restrictions remain independently enforced |
| `inventory.disposal.request` | Owner, Manager, Inventory | Grant only if the disposal workflow is explicitly included and its product-category/legal evidence rules are approved; otherwise disabled |
| `inventory.disposal.approve` | Owner or delegated Manager; separate approver where required | Grant only if disposal is included; no self-approval; legal witness/evidence rules must be resolved |
| `inventory.return.disposition` | Owner, Manager, Inventory; Professional only where a reviewed product rule requires it | Separate from customer-return acceptance and refund; returned goods remain non-saleable until disposition |
| `supplier.read` | Owner, Manager, Purchasing; Inventory/Accountant limited | Tenant scope; sensitive banking/contact fields excluded unless separately granted |
| `requisition.read`, `quotation.read` | Owner, Manager, Purchasing; Inventory limited where assigned | Grant only if requisition/quotation workflows are included; otherwise disabled |
| `purchase_order.read` | Owner, Manager, Purchasing; Inventory and Accountant limited; Auditor read-only | Order detail and branch scope; supplier-sensitive fields filtered |
| `goods_receipt.read` | Owner, Manager, Inventory, Purchasing; Accountant limited; Auditor read-only | Receipt/batch detail; pricing fields separately filtered |
| `supplier_invoice.read` | Owner, Manager if delegated, Purchasing, Accountant; Auditor read-only | Grant only if invoice workflow is included; invoice and matching evidence; sensitive financial fields restricted |
| `supplier_payment.read` | Owner, Accountant; Purchasing/Manager only if delegated; Auditor read-only | Grant only if supplier-payment workflow is included; payment references and reconciliation status; bank details separately gated |
| `supplier.create`, `supplier.update` | Owner, Manager, Purchasing | Tenant scope and duplicate review |
| `supplier.block` | Owner; delegated Manager | Reason, impact on open orders and appeal/review path |
| `requisition.create`, `requisition.submit`, `quotation.record` | Owner, Manager, Purchasing; Inventory may create requisitions if assigned | Grant only if pilot research requires requisition/quotation workflows; otherwise disable and use the approved minimal PO path |
| `requisition.approve` | Owner or delegated Manager; not the requester | Threshold and separation rules |
| `purchase_order.create`, `purchase_order.submit` | Owner, Manager, Purchasing; Inventory if assigned | Supplier, branch, approved item/unit and quotation snapshot |
| `purchase_order.approve` | Owner or delegated Manager; not the requester | Threshold and separation rules |
| `purchase_order.issue`, `purchase_order.cancel`, `purchase_order.close` | Owner, Manager, Purchasing; Accountant may close only if explicitly delegated | Issue only approved version; cancellation resolves outstanding quantity; closure checks financial follow-up |
| `supplier_invoice.record` | Purchasing, Accountant, Owner/Manager if assigned | Grant only after invoice matching/payable policy and accountant review are approved; preserve supplier-stated values and document provenance |
| `supplier_invoice.match` | Accountant, Purchasing | Grant only if invoice workflow is included; match to PO/accepted receipt; retain line-level discrepancies |
| `supplier_invoice.exception.resolve` | Accountant; Owner/Manager only by explicit delegation | Grant only if invoice workflow is included; evidence and reason; material exceptions cannot be self-approved |
| `supplier_invoice.dispute` | Purchasing, Accountant, Owner/Manager if assigned | Grant only if invoice workflow is included; dispute blocks payable settlement pending resolution |
| `supplier_invoice.approve` | Accountant or Owner/Manager specifically delegated; not the requester | Grant only if invoice workflow is included; approved matching/accounting policy and separation of duties |
| `supplier_payment.initiate` | Owner; Purchasing only if explicitly assigned | Grant only for an approved supported payment method and accounting workflow; approved obligation, amount/currency, idempotency and evidence |
| `supplier_payment.confirm`, `supplier_payment.reconcile` | Accountant; Owner only under a separately approved control model | Grant only if supplier payments are included; authoritative bank/provider evidence; initiation is not settlement |
| `sale.read` | Owner, Manager, assigned Professional, Cashier for permitted sales history; Accountant/Auditor read-only where needed | Original sale and permitted branch scope |
| `payment.read` | Owner, Manager, Accountant; Cashier for assigned sales; Auditor read-only where approved | Payment attempt/status only; sensitive provider/bank data remains separately gated |
| `customer_return.read` | Owner, Manager, Cashier for assigned branch; Professional if assigned; Accountant/Auditor limited read | Return status linked to an authorized original sale |
| `sale.create`, `sale.commit` | Cashier, Owner, Manager; Professional if assigned | Branch/session; category eligibility and exact regulated-action gate |
| `sale.discount.request` | Cashier, Owner, Manager, assigned Professional | Grant only if the discount policy is approved; threshold and reason |
| `sale.discount.approve` | Owner or delegated Manager; not requester | Grant only if the discount policy is approved; threshold and separation |
| `sale.void.request` | Cashier, Owner, Manager | Original sale and reason |
| `sale.void.approve` | Owner or delegated Manager; not requester | Linked correction; stock disposition remains separate |
| `customer_return.create`, `customer_return.submit` | Cashier, Owner, Manager; Professional if assigned | Original sale, lines, quantities, condition, timing and reason |
| `customer_return.review` | Owner or delegated Manager; designated Professional only where policy requires | Not requester; acceptance does not imply refund or restock |
| `payment.initiate` | Cashier, Owner, Manager; Accountant if specifically assigned | Enabled payment method/provider and sale reference |
| `payment.confirm_manual` | Accountant or Owner/Manager with explicit confirmation grant; never the initiator where separation is required | Evidence review; client assertion is insufficient |
| `payment.refund.request` | Cashier, Owner, Manager | Original payment/sale and eligible amount |
| `payment.refund.approve` | Owner, delegated Manager, or Accountant; not requester | Provider/cash policy, amount threshold and authoritative completion |
| `receipt.read`, `receipt.reissue` | Cashier, Owner, Manager; Accountant read-only if needed | Original transaction; reissue never creates a new sale |
| `finance.read`, `finance.cashbook.read` | Owner, Accountant; Manager limited; Auditor read-only | Branch/tenant scope and sensitive-field minimization |
| `finance.cash_movement.create` | Owner, Manager; Cashier only for assigned session types | Reason, source transaction, actor and session |
| `finance.cash_session.open` | Cashier, Owner, Manager | Assigned branch/till; no conflicting active session under selected policy |
| `finance.cash_session.count` | Assigned Cashier for own session; Manager/Accountant as reviewer | Count evidence; cannot rewrite underlying sales/payments |
| `finance.cash_session.close` | Owner, Manager, Accountant if assigned; Cashier only for own session if policy permits | Variance and late-movement policy; separation where required |
| `finance.reconcile` | Accountant; Owner only if explicitly assigned | Source file/provider evidence, hash, period and currency |
| `finance.reconciliation_exception.resolve` | Accountant; Owner/Manager only by explicit delegation and not requester | Evidence, reason, linked correction and reviewer |
| `finance.export`, `sales.export`, `purchasing.export`, `inventory.export` | Owner; Accountant for finance/sales reports; relevant functional role only for its own export; Auditor read-only where approved | Tenant/branch filters, purpose, audit and data minimization |

### 12.3 Explicitly outside this first-release grant baseline

Unless Specification 25 is amended and its release gates pass, do not grant customer-tenant roles any of the following merely because the permission exists in the canonical registry:
- public profile/catalogue/availability publication, public enquiry handling, or public reporting;
- moderation, verification approval, or publication of verification claims;
- platform organization restriction, entitlement administration, platform security administration, or support tenant-data access;
- full general-ledger posting/reversal, period close, or other deferred accounting capabilities;
- any regulated dispensing, prescription, restricted-category, or clinical workflow that has not received action-specific Ethiopian legal/regulatory review.

#### 12.3.1 Explicit disposition of other canonical permissions

The canonical registry contains 136 identifiers; 92 are mapped to the proposed first-release operation baseline. The remaining identifiers are intentionally not ordinary tenant-role grants. Their disposition is:

- `account.register` and `account.sign_in` are self-service identity actions, not tenant-role grants; their controls are defined by authentication policy.
- `catalogue_correction.review`, `catalogue_correction.approve`, `barcode_mapping.propose`, `barcode_mapping.review`, `regulatory_classification.read`, and `regulatory_classification.review` belong to canonical catalogue governance/review. They are not enabled as first-release tenant roles; tenant-local catalogue maintenance remains separate.
- `supplier.sensitive_data.read` is a separately controlled, purpose-bound sensitive-field permission and is not included in the baseline.
- `supplier_return.create` and `supplier_return.approve` remain excluded with supplier returns unless a complete pilot workflow is explicitly approved.
- `finance.expense.create` and `finance.expense.approve` remain excluded because expense tracking is not in the candidate first release.
- `finance.journal.read`, `finance.journal.post`, `finance.journal.reverse`, and `finance.period.close` remain excluded with full general-ledger and accounting-period capabilities.
- `finance.sensitive_payment_data.read` is separately controlled and is not implied by ordinary payment, finance, or reconciliation access.
- `support.case.create` may be made available to authenticated tenant members as a case-scoped service capability under abuse controls; it is not an operational tenant-role grant. It never grants `support.tenant_data.access`.

For an exact machine-auditable inventory, the **44 canonical identifiers outside the 92-permission first-release baseline** are:

| Domain | Identifiers outside the baseline | Disposition |
|---|---|---|
| Self-service identity | `account.register`, `account.sign_in` | Authentication policy; not tenant-role grants |
| Canonical catalogue governance | `catalogue_correction.review`, `catalogue_correction.approve`, `barcode_mapping.propose`, `barcode_mapping.review`, `regulatory_classification.read`, `regulatory_classification.review` | Deferred governance/review capabilities; no first-release tenant grants |
| Supplier-sensitive data and returns | `supplier.sensitive_data.read`, `supplier_return.create`, `supplier_return.approve` | Sensitive-field access separately gated; supplier returns excluded |
| Deferred finance | `finance.expense.create`, `finance.expense.approve`, `finance.journal.read`, `finance.journal.post`, `finance.journal.reverse`, `finance.period.close`, `finance.sensitive_payment_data.read` | Expense and full-ledger/period-close workflows excluded; sensitive payment fields separately gated |
| Public participation | `public_profile.configure`, `public_profile.publish`, `public_profile.pause`, `public_profile.update_contact`, `public_profile.update_location`, `public_catalogue.configure`, `public_catalogue.publish`, `public_catalogue.pause`, `public_availability.confirm`, `public_availability.publish`, `public_availability.pause`, `public_enquiry.read`, `public_report.submit` | Public surface excluded from first production release |
| Moderation and verification | `moderation.case.read`, `moderation.case.triage`, `moderation.content.suppress`, `moderation.decision.record`, `moderation.appeal.review`, `verification.evidence.submit`, `verification.evidence.review`, `verification.claim.publish` | Platform governance and verification workflows excluded |
| Support and platform administration | `support.case.create`, `support.tenant_data.access`, `platform.organization.restrict`, `platform.entitlement.manage`, `platform.security.admin` | Case creation is a service capability for authenticated members, not an operational role grant; data access and platform administration remain separately controlled |

These dispositions are explicit exclusions or separate control paths, not missing permission mappings. A future release must add the relevant operations, role/scope rules, evidence gates, and tests before enabling any excluded capability.

Support case creation may remain available to all authenticated tenant members, but it does not grant support staff access to tenant data. Platform support access requires its separate case-scoped, time-limited approval.

### 12.4 Required grant resolution before implementation

For each permission selected for the approved release, convert this proposal into an explicit row containing: role, default/delegated/individual grant type, tenant/branch/resource scope, grantor, revocation behavior, requester/approver separation, thresholds, re-authentication, action-specific evidence gate, online/offline eligibility, audit event, and positive/negative tests.

The table above is a review baseline, not a production grant. If a role or permission is not explicitly approved, deny it by default. A permission not required by the selected first-release journeys should remain unassigned and its corresponding UI/API capability disabled.
