# 05 — Actors, personas, and role model

**Status:** Draft — proposed requirements, not regulatory approval  
**Scope:** Identity, organization membership, roles, branch access, and sensitive actions.

## 1. Design objective

A person may hold more than one operational responsibility, but permissions must be explicit, scoped, reviewable, and enforced by the server. A user interface hiding a control is not authorization. Every protected operation must be authorized against the authenticated identity, active organization, branch scope, resource ownership, and current resource state.

## 2. Actor groups

### Pharmacy organization
- **Owner:** accountable for the business relationship, organization configuration, subscription, staff access, and high-impact approvals.
- **Branch manager:** runs an assigned branch; manages permitted staff, operational settings, and branch-level activity.
- **Responsible pharmacy professional / technical manager:** professional role whose identity, credential status, scope, and appointment must be verified according to the applicable regulator. The product must not infer professional authority from a job title.
- **Pharmacy professional:** performs only actions permitted by verified professional status, applicable law, organization policy, and branch assignment.
- **Cashier / sales operator:** handles permitted point-of-sale tasks without automatic access to clinical, personnel, or financial-administration data.
- **Inventory operator:** handles permitted receiving, counting, transfers, and stock records without unrestricted access to financial or staff data.
- **Purchasing operator:** prepares supplier and purchase workflows; approval and receipt duties may be separated.
- **Accountant / finance operator:** reviews financial records, reconciliations, and reports; does not automatically receive operational or professional permissions.
- **Auditor / read-only reviewer:** receives narrowly scoped read access and export permissions only where explicitly granted.

One person may have multiple roles. Role combination must not silently broaden access beyond the union of explicitly granted permissions and applicable constraints.

### Public and platform actors
- **Public visitor:** searches public listings and sees only information intentionally published by a pharmacy.
- **Registered public user:** may manage their own profile and preferences. A public account does not grant access to pharmacy operational records.
- **Platform support agent:** handles support cases with time-limited, case-scoped access; no default ability to browse private pharmacy records.
- **Verification reviewer:** reviews establishment and credential evidence; cannot approve their own submission or alter source evidence.
- **Platform administrator:** manages platform configuration and operational controls. This role is not a superuser for pharmacy business records by default.
- **Security / compliance auditor:** accesses security and audit evidence under documented purpose and least privilege.
- **External service identity:** narrowly scoped integration identity for payment, messaging, maps, or other approved services. It is not a human user.

## 3. Authorization hierarchy

The authorization context has these dimensions:

1. **Platform tenant:** the pharmacy business or other customer organization.
2. **Branch:** a specific operating location belonging to that tenant.
3. **Membership:** the person’s active relationship with the tenant.
4. **Role and permission grants:** explicitly assigned capabilities.
5. **Resource scope:** the particular branch, record, or permitted set of records.
6. **Professional/regulatory constraints:** verified credential and scope checks for regulated actions.
7. **Workflow state:** whether the requested action is valid at the current state.

A request must fail closed if any required dimension is missing, expired, revoked, or ambiguous. Client-supplied tenant IDs, branch IDs, role names, or ownership claims are never trusted without server-side validation.

## 4. Permission model

Use permissions as stable action identifiers, grouped into domains. Candidate domains:

- organization.read / organization.manage
- branch.read / branch.manage
- membership.read / membership.invite / membership.revoke
- role.assign / role.revoke
- catalogue.read / catalogue.propose / catalogue.approve
- stock.read / stock.receive / stock.count / stock.adjust / stock.transfer
- supplier.read / supplier.manage
- purchase.create / purchase.submit / purchase.approve / purchase.receive
- sale.create / sale.void.request / sale.void.approve
- return.create / return.approve
- finance.read / finance.reconcile / finance.export
- report.read / report.export
- listing.read / listing.publish / listing.pause
- audit.read
- subscription.read / subscription.manage
- platform.case.read / platform.case.manage
- verification.review / verification.approve

This is a candidate vocabulary, not a final permission matrix. Separate permissions for initiating, approving, reversing, exporting, and administering high-impact actions. Do not create a single broad “manager” permission that bypasses controls.

## 5. Separation of duties and high-risk actions

The system should support configurable approval thresholds, while retaining safe defaults:

- A person must not approve their own high-impact adjustment when a second authorized reviewer is available and required by policy.
- Role grants, owner changes, credential changes, branch ownership changes, and payout/payment configuration changes require re-authentication and an auditable approval path.
- Financial corrections are represented as linked reversals or adjustments, not silent edits to finalized records.
- Stock corrections record reason, actor, timestamp, affected branch, affected batch/stock unit where applicable, and supporting evidence where required.
- Sale voids, returns, and post-close corrections require explicit policy and audit events.
- Platform staff access to tenant data is purpose-bound, time-limited, logged, and reviewable.

If a small pharmacy has only one eligible person, the product must define a documented exception workflow rather than silently dropping the control.

## 6. Account and membership lifecycle

Required states include invited, active, suspended, revoked, and expired where relevant. Invitations must be bound to an organization and intended role scope. Revocation must invalidate active sessions and outstanding access tokens within a defined security objective. Removing a user from one organization must not delete their identity or membership in another organization.

A staff member leaving a branch must lose branch access promptly. Historical transactions retain the actor identity and attribution needed for audit; they must not be reassigned to a generic account.

## 7. Credential and establishment verification

Store claims and evidence separately from verification decisions. Record issuer, identifier, evidence source, effective date, expiry date if present, reviewer, decision, decision time, and reason. A pending, rejected, expired, or unverified credential must not be displayed as verified or used to authorize restricted workflow capabilities.

Verification status is not a substitute for the regulator’s authoritative register. Where direct authoritative verification is unavailable, label the method and confidence honestly and require manual review. Support re-verification and suspension when evidence expires or is challenged.

## 8. Acceptance criteria

- Cross-tenant and cross-branch access attempts are denied server-side, including direct API calls and guessed identifiers.
- Permission changes take effect within the documented revocation objective.
- Each sensitive action records actor, scope, time, action, outcome, and reason/context.
- A user can hold multiple roles without gaining implicit platform-wide access.
- Support and platform administration do not have unrestricted default access to private tenant data.
- Professional or establishment status is never inferred from self-entered text alone.
- Authorization tests cover allowed, denied, expired, revoked, cross-tenant, cross-branch, and invalid-state cases.

## 9. Open decisions

- Which organization roles are mandatory for the first supported pharmacy type?
- Which actions require two-person approval, and what documented exception applies to single-professional sites?
- Which credentials can be verified electronically, from which authoritative source, and how often?
- Should a single person be allowed to own multiple branches under one tenant, subject to local licensing rules?
- What is the support-access approval and emergency-access process?
