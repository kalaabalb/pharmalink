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

## 10. Platform membership is not a professional licence

A person may create an account, join an organization, and use ordinary platform functions according to assigned permissions without presenting an individual professional licence. Do not require professional credentials at general registration, invitation acceptance, sign-in, or routine management use.

Professional credential data is collected and checked only when there is a defined need: for example, a regulated action, a role whose legal scope requires it, or a public verification statement that explicitly depends on that credential. Such checks must be limited to the relevant action and must not automatically block unrelated management functions.

Establishment licence evidence, an individual's professional credential, and Pharmalink's own organization/account verification are separate records and statuses. A pharmacy's choice not to publish or sell through the public-facing app must not prevent its staff from using private management features.

**Permission design consequence:** distinguish platform permissions (manage catalogue, stock, suppliers, reports, staff) from regulated professional permissions. The latter must be gated by applicable role, credential status, and current law; the former must not inherit a blanket professional-licence requirement.

## 11. Independent access, evidence, and publication states

The authorization model must not collapse platform access, establishment licensing, professional credentials, and public participation into one “verified pharmacy” flag.

| Dimension | Subject | Example states | What it controls |
|---|---|---|---|
| Person account | Person | pending activation, active, locked, closed | Authentication only |
| Organization | Tenant | setup, active, suspended, closed | Tenant lifecycle and service relationship |
| Membership | Person–tenant relationship | invited, active, suspended, revoked | Whether the person may act for that tenant |
| Branch membership/scope | Membership–branch relationship | assigned, suspended, removed | Which branch resources are accessible |
| Management entitlement | Tenant and capability | enabled, read-only/grace, suspended, ended | Contractual/service availability, not professional authority |
| Establishment evidence | Organization or branch | not requested, required, submitted, under review, verified, rejected, expired, suspended, superseded | Only the specific operation or public claim for which evidence is required |
| Professional credential | Person and credential scope | not requested, submitted, under review, verified, rejected, expired, suspended, revoked | Only professional actions requiring that credential and scope |
| Public participation | Organization/branch and capability | opted out, draft, submitted, published, paused, rejected, withdrawn | Public projection and discovery |
| Commerce capability | Organization/branch and approved capability | unavailable, review required, enabled, paused, revoked | Only the separately approved commerce workflow |

These are separate aggregates or independently versioned state records. A transition in one dimension must not implicitly mutate another unless a documented business rule explicitly requires it.

### Authorization decision sequence

For each command, the server evaluates:
1. Is the account authenticated and not locked or revoked?
2. Does the person have an active membership in the target tenant?
3. Does the membership have the required permission for the target resource and branch?
4. Is the tenant's relevant management entitlement active for this capability?
5. Is the resource in a state that permits the requested transition?
6. Does this exact action require establishment evidence, and if so is the correct evidence valid for this establishment, branch, region, and action?
7. Does this exact action require a professional credential, and if so is the credential verified, current, and within its scope?
8. Are separation-of-duties, approval, re-authentication, and other risk controls satisfied?

Only applicable checks are required. An irrelevant check must not become an accidental global gate. The server must return a structured, actionable denial reason without exposing private verification evidence.

### Role design constraints

- Organization owners and business administrators may be non-professionals. Their management permissions do not grant professional authority.
- Cashiers, inventory operators, purchasing staff, accountants, and other ordinary staff accounts do not need an individual professional licence merely to join or use permitted functions.
- A professional may also hold management roles, but the professional credential is an additional scoped authorization attribute, not the source of all permissions.
- A role title, uploaded credential image, self-entered licence number, or owner's assertion is not proof of verified professional authority.
- A verified credential must be checked for issuer, identity match, validity, professional category, permitted scope, applicable location, and current status to the extent supported by authoritative sources.
- If a credential expires or is suspended, remove only permissions that depend on it. Preserve unrelated management access and historical attribution.
- If establishment evidence expires or is suspended, evaluate each affected operation and public claim separately. Do not automatically lock every staff account.
- Public listing publication and commerce enablement require explicit organization authorization and capability-specific review. They are not implied by creating a tenant or branch.

## 12. Permission-matrix deliverable

The initial proposed matrix is maintained in [Specification 18](18-proposed-permission-matrix.md). It is a Draft, not yet approved. Before implementation, reconcile it against every module specification and ensure each permission has a stable identifier, purpose, allowed actor/role, tenant/branch/resource scope, management entitlement, establishment evidence dependency, professional credential dependency, approval and re-authentication rules, offline eligibility, audit event, and positive/negative test cases.

The matrix must include allowed and denied examples for owner, manager, pharmacist/professional, cashier, inventory operator, purchasing operator, accountant, auditor, and platform support. “Owner” must never be treated as a universal bypass for professional or regulatory constraints. Organization, branch, invitation, membership, and offboarding behavior is detailed in [Specification 20](20-organization-branch-and-membership-lifecycle-specification.md).
