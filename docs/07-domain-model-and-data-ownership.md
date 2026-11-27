# 07 — Domain model and data ownership

**Status:** Draft — conceptual model, not a database schema  
**Purpose:** Establish domain boundaries and authoritative ownership before selecting technology.

## 1. Modeling rules

- Give every durable entity a stable identifier independent of its display name.
- Store timestamps with timezone-aware semantics; preserve the original business-local date/time where relevant.
- Represent money with exact decimal/minor-unit semantics and an explicit currency. Do not use binary floating-point for financial amounts.
- Store source, provenance, review state, and effective dates for externally sourced or regulated data.
- Distinguish master/reference data from tenant-owned configuration and branch-owned operational facts.
- Use explicit state machines for records with lifecycle transitions.
- Avoid denormalized copies as independent sources of truth. Derived views must be rebuildable from authoritative records.
- Sensitive personal data must be minimized and separated from general operational metadata.

## 2. Identity and organization domain

Candidate entities:
- **Person identity:** account-level identity and authentication linkage.
- **Organization:** customer/legal business grouping; tenant security boundary.
- **Branch:** operating location belonging to one organization.
- **Membership:** relationship between a person and organization, including state and dates.
- **Role assignment:** role and scope granted to a membership.
- **Permission grant:** explicit capability and resource scope where role templates are insufficient.
- **Credential claim:** a person's claimed professional credential and supporting evidence.
- **Credential verification:** a separate decision record with reviewer, source, date, outcome, and validity.
- **Establishment evidence:** licence or other required evidence tied to an organization/branch and issuing authority.
- **Device registration:** approved device identity, tenant/branch assignment, trust state, and revocation state.
- **Customer tenant:** server-enforced data-isolation boundary and service customer workspace.
- **Legal entity:** real-world legal person/business, distinct from a Pharmalink tenant and from an establishment.
- **Organization authority record:** evidence of a person's authority to create/manage a customer workspace where required.
- **Invitation:** single-use, time-limited offer to join a tenant with proposed scope and permissions.
- **Service entitlement:** separately versioned software capability entitlement, not a legal licence.


Identity is not the same thing as organization membership. A person may belong to multiple organizations. Revoking one membership must not delete the person's account or historical attribution.

## 3. Medicine and sellable-item domain

Keep these concepts separate:

- **Canonical catalogue item:** shared reference identity for a regulated product, with source and review provenance.
- **Product presentation/pack:** the exact presentation or pack identity where the source data distinguishes it.
- **Brand/generic naming attributes:** names and aliases with provenance; not unique identifiers by themselves.
- **Barcode/identifier mapping:** code, issuing/source context, scope, and confidence; one item may have multiple codes and codes may be reused or incorrectly captured.
- **Tenant catalogue entry:** a pharmacy's local enablement and display configuration linked to a canonical item.
- **Price record:** branch/tenant-owned value with currency, effective period, author, and approval where required.
- **Regulatory classification metadata:** source-backed classification with jurisdiction and effective date; not free-form user authority.
- **Catalogue correction proposal:** a proposed change with source evidence and review outcome.

The catalogue must not treat two items as equivalent merely because their names look similar. No automated clinical substitution or recommendation is in scope.

## 4. Inventory and traceability domain

Candidate entities:
- **Stock batch/lot:** a traceability grouping with the identifiers and dates available from source documentation.
- **Stock position:** a derived quantity view for a branch and eligible stock grouping.
- **Stock movement:** immutable event/ledger record describing an accepted change.
- **Stock reservation:** excluded from initial public discovery; only introduce if a separately approved reservation workflow exists.
- **Stock count session:** observed quantities and counted-by attribution.
- **Adjustment proposal:** a reviewed explanation for a discrepancy.
- **Transfer:** source branch, destination branch, line items, dispatch/receipt states, and custody trail.
- **Quality/hold case:** reason, scope, status, evidence, decision authority, and disposition.
- **Recall notice:** source and affected scope, with matching confidence and review outcome.

The stock position is a projection of accepted movements, not the primary record. A correction appends a compensating movement; it does not rewrite historical movement rows. The system must prevent negative or duplicated stock through concurrency control and explicit policy.

## 5. Supplier and purchasing domain

Candidate entities:
- Supplier; supplier contact; supplier document.
- Purchase order and revision.
- Purchase order line.
- Goods receipt and receipt line.
- Supplier invoice.
- Supplier payment/reconciliation record.
- Discrepancy case.

Order, receipt, invoice, and payment are different business facts. Their relationships must be explicit; none is proof of another.

## 6. Sales and finance domain

Candidate entities:
- Sale; sale line; sale state transition.
- Payment attempt; provider transaction; payment confirmation; reconciliation.
- Receipt/invoice document and document numbering sequence.
- Return; return line; void request; approval; reversal.
- Cash session/shift and cash movement.
- Journal/ledger entry; account mapping; reconciliation item.
- Tax configuration/version and applied tax snapshot.

Finalized financial entries should be immutable. A correction creates linked entries with a reason and actor. Store the tax/pricing configuration version applied to the transaction so later configuration changes do not rewrite history. Accounting and tax treatment require local professional review.

## 7. Public discovery domain

Candidate entities:
- Public pharmacy profile and publication settings.
- Public branch listing and location.
- Public catalogue listing.
- Availability snapshot with branch, item, status, confirmation time, source revision, and freshness expiry.
- Publication event and pause/suppression reason.

Public availability is a time-bounded assertion, not a promise. It must not expose exact stock quantities by default. The public projection must be derived only from explicitly publishable data and must not leak patient, staff, supplier, or internal financial information.

## 8. Platform and commercial domain

Candidate entities:
- Onboarding application and evidence.
- Verification case, review, decision, appeal, and re-review.
- Subscription plan/version; subscription; invoice; payment; entitlement; grace/suspension state.
- Support case; access grant; privileged access session.
- Platform policy and policy version.
- Security event and operational incident.

A commercial entitlement is not a data-ownership boundary. Subscription changes must not transfer or erase ownership of tenant data.

## 9. Event and audit domain

- **Domain event:** a fact that has occurred and is committed, with a stable event ID, aggregate ID/version, tenant/branch scope, event type, schema version, occurrence time, and correlation/causation metadata.
- **Outbox record:** event publication intent committed atomically with the domain transaction.
- **Delivery attempt:** destination, attempt number, status, timing, and error class.
- **Consumer inbox/deduplication record:** processed event identity and outcome.
- **Audit record:** actor/system identity, action, scope, target, time, outcome, reason, and relevant before/after references.

Domain events and audit records serve different purposes. Events communicate state changes; audit records provide a controlled accountability trail. Do not assume an event bus is a permanent audit archive.

## 10. Data ownership and access

| Data class | Authoritative owner | Default access |
|---|---|---|
| Account identity | Platform identity service | Account holder and narrowly authorized security operations |
| Organization and branch | Tenant organization | Authorized tenant members within scope |
| Staff membership and roles | Tenant organization | Authorized owner/admin roles |
| Canonical catalogue | Platform-controlled, source-governed reference data | Read access; controlled correction proposals |
| Local catalogue and prices | Tenant/branch | Authorized tenant members |
| Stock movements and sales | Tenant/branch | Authorized operational roles; audited |
| Patient/customer records, if ever approved | Not defined; excluded pending legal and clinical review | No implementation until purpose, lawful basis, retention, and access are approved |
| Public profile | Publishing pharmacy, subject to platform moderation | Public fields only |
| Payment credentials | Payment provider where possible | Never store raw provider secrets or payment credentials unnecessarily |
| Platform security logs | Platform operator | Restricted security/compliance access |

## 11. Retention, deletion, and export

Define retention by record class and applicable law before implementation. Account deletion must not silently erase records that must lawfully be retained; retained records should be minimized, access-restricted, and de-identified where legally appropriate. Tenant exit must support a documented export process, integrity verification, and deletion schedule after retention obligations are satisfied.

## 12. Acceptance criteria

- A medicine catalogue item cannot be confused with a batch, stock position, or local price.
- Every stock position can be reconstructed from accepted movements.
- Every finalized sale and financial correction has traceable linked records.
- Public projections cannot expose private tenant or personal data.
- Every record has an explicit tenant/branch ownership rule or is explicitly platform-global.
- No sensitive patient-data domain is introduced without a separate approved specification.
- Domain events are versioned and distinct from audit records.

## 13. Independent access and verification aggregates

The model must represent the following independently. Avoid a single organization `verified` boolean.

- **PersonAccount:** authentication identity and account lifecycle.
- **Organization/Tenant:** customer workspace and its platform relationship.
- **Membership:** person-to-organization relationship, invitation state, activation/revocation, and attribution.
- **Branch:** operational location under an organization, with its own configuration and lifecycle.
- **BranchAssignment:** membership-to-branch scope, including effective dates and revocation.
- **PermissionGrant:** explicit permission, scope, grantor, reason, effective period, and revocation.
- **ManagementEntitlement:** organization capability/plan state and contractual limits. It does not represent legal or professional authorization.
- **EstablishmentEvidence:** evidence concerning an organization or branch, including type, issuer, jurisdiction, identifier, evidence reference, validity dates, verification method, status, reviewer, and decision history.
- **ProfessionalCredential:** evidence concerning an individual, including credential category, issuer, jurisdiction, identifier, evidence reference, validity dates, and asserted scope.
- **CredentialVerificationDecision:** immutable decision record tied to a credential version, with source, method, reviewer, timestamp, outcome, reason, and limitations.
- **PublicParticipationSetting:** explicit opt-in and field/capability-level publication settings for an organization or branch.
- **PublicProjection:** derived public-safe representation with publication version, source revision, visibility state, and suppression/pause reason.
- **CapabilityReview:** review of a particular operation or public capability, with required evidence and outcome. It must not become a universal tenant gate.
- **VerificationAppeal:** appeal and re-review history linked to the original decision without overwriting it.

### State and data invariants

1. An account can exist without any organization membership.
2. A membership can be active without a professional credential.
3. An organization can maintain private management configuration without public publication.
4. A branch record can exist in a draft/private setup state without the product claiming that the branch is legally authorized to operate.
5. Establishment evidence belongs to the organization or branch and jurisdiction it actually describes; it is not copied to staff members as their credential.
6. Professional credentials belong to a person and have a defined category and scope; they are not inherited by the organization, owner, or coworkers.
7. Verification decisions are versioned and attributable. New evidence or a re-review creates a new decision record rather than silently rewriting the old one.
8. Public projections are derived data. Pausing or withdrawing publication removes or suppresses the public projection but does not delete private source records or historical publication events.
9. A failed public review cannot mutate account, membership, stock, sales, supplier, or finance lifecycle.
10. Expiry/revocation of one credential affects only actions whose authorization rules depend on that credential. Historical actions retain their original actor and authorization context for audit.
11. Subscription/entitlement state is separate from ownership and retention of tenant data. Suspension must not erase records or prevent lawful export and required record access.
12. Every verification decision and every change to permission, membership, publication, or entitlement has actor, time, reason, scope, and correlation metadata.

### Ownership and deletion additions

The tenant owns its private operational records subject to applicable law and service terms. The platform is a processor/service provider where the applicable legal relationship so determines; that role must be confirmed contractually and legally, not hard-coded as an assumption. The platform controls its own security, billing, moderation, and verification decision records, while evidence supplied by a tenant must retain provenance and access restrictions.

Deleting a public profile is not deleting the tenant. Closing a membership is not deleting the person's identity. Expiring a credential is not deleting its verification history. Tenant exit must preserve records that must lawfully be retained, provide a verified export, and apply a documented deletion schedule to remaining data.


## 14. Organization and tenant relationship invariants

The organization, tenant, legal-entity, establishment, membership, role assignment, professional credential, establishment evidence, public participation, and service entitlement relationships are specified in [Specification 20](20-organization-branch-and-membership-lifecycle-specification.md).

The data model must not assume one tenant equals one legal entity, one licence, one branch, one owner, or one professional. The customer/tenant relationship must be explicit and reviewed before schema lock. A branch is a real-world establishment record and may have its own evidence, status, and public projection.

Every tenant-owned record has an authoritative tenant owner. A branch-owned record has a branch reference or a documented organization-wide scope. Legal-entity links and evidence must retain effective dates and provenance so ownership changes do not rewrite historical transaction attribution.
\n\n## Architecture implementation boundary\n\nThis document remains the conceptual domain and data-ownership model, not a physical schema. Module ownership, dependency direction, database integrity, tenant-isolation implementation, API contracts, and deployment boundaries are governed by [Specification 29](29-application-architecture-api-database-and-deployment-contract.md). A physical schema must preserve the ownership and projection rules defined here; it must not turn conceptual entities into shared writable tables without an explicit owner.\n