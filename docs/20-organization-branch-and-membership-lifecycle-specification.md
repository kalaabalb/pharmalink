# 20 — Organization, branch, and membership lifecycle specification

**Status:** Draft — customer discovery, Ethiopian legal review, privacy review, and operational validation required  
**Priority:** Critical — tenant identity and membership boundaries underpin every private and public capability  
**Scope:** Customer organization records, tenant boundaries, branches, invitations, memberships, role assignment, access recovery, ownership changes, organization suspension, subscription/entitlement boundaries, export, and offboarding.

## 1. Purpose and terminology

This specification defines how a pharmacy business becomes a Pharmalink customer, how its branches and staff are represented, and how access changes over time.

Keep these concepts separate:

- **Person/account:** an individual identity used to authenticate.
- **Customer organization:** the customer workspace and tenant security boundary in Pharmalink.
- **Legal entity:** the real-world legal person or business that may own or operate one or more establishments. The relationship to a customer organization must be explicit and verified to the degree required.
- **Establishment/branch:** a real-world operating location with its own address, evidence, operational configuration, and branch-scoped data.
- **Membership:** a person's relationship to a customer organization.
- **Role assignment:** a set of permissions granted to a member at a defined scope.
- **Establishment evidence:** evidence about a particular establishment, category, issuer, jurisdiction, and validity period.
- **Professional credential:** evidence about a particular person and their professional scope.
- **Service entitlement:** a software capability available under the customer's service plan or agreement.
- **Public participation:** a separate, explicit opt-in to one or more public capabilities.

These concepts must not be collapsed into one “verified pharmacy” or “licensed user” flag.

A customer organization may be used for private management without public selling or public listing. Ordinary account registration, sign-in, invitation acceptance, and permitted management work do not require an individual's professional licence.

## 2. Tenant and legal-entity relationship

The tenant is the server-enforced customer data boundary. It is not automatically proof of a legal entity, pharmacy licence, or right to operate.

Before implementation, field research must determine whether the usual customer relationship is:
- one tenant per legal entity;
- one tenant per pharmacy business/brand, potentially representing multiple legal entities;
- one tenant per owner/operator with multiple independently licensed establishments;
- another locally appropriate structure.

Do not assume a one-to-one mapping between tenant, legal entity, owner, and establishment. The data model must support an explicit, reviewable relationship without allowing cross-tenant data access.

Candidate organization fields:
- stable organization ID;
- tenant ID (may be the same identifier only if the architecture explicitly chooses that);
- registered/legal name and trading name, with provenance;
- organization type and customer category;
- legal-entity identifiers where needed and lawful;
- authorized representatives and evidence of their authority where required;
- billing/service contact distinct from operational staff;
- lifecycle status and status reason;
- locale, timezone, and supported currency configuration;
- data export/offboarding configuration;
- created/updated timestamps and actor.

Avoid collecting owner identity documents or sensitive personal data merely because a person wants to use the software. Collect only what is needed for a defined legal, security, billing, or verification purpose.

## 3. Organization lifecycle

Candidate states:
- **Draft:** setup started; not yet operationally activated.
- **Pending review:** only a defined review is outstanding; identify its scope.
- **Active:** permitted software capabilities are available subject to role, evidence, and entitlement rules.
- **Restricted:** one or more specified capabilities are disabled with reason and review date; unrelated capabilities remain available.
- **Suspended:** access to specified service capabilities is suspended under an approved policy; data access, export, legal record retention, and support pathways remain defined.
- **Closing:** customer requested offboarding; new operational activity may be limited under policy while export and retention are handled.
- **Closed:** service relationship ended and data has reached the documented retention/deletion stage.

Avoid using “pending verification” as a global state that blocks every capability. Store separate state machines for identity review, establishment evidence, professional credentials, subscription/entitlement, security restrictions, and public participation.

A state transition must record actor/system, time, reason, scope, prior state, new state, policy/version, notification, and appeal or review path where applicable.

## 4. Organization onboarding workflow

1. A person creates an account or signs in using an approved authentication method.
2. The person creates a customer organization or accepts an invitation to an existing one.
3. The system checks for possible duplicate customer organizations without automatically merging them.
4. The creator states their relationship and authority to represent the organization; the exact evidence requirement is risk- and law-dependent.
5. The organization selects management-only as the default operating mode, unless it explicitly configures a public capability.
6. The system creates the initial organization workspace and records the initial owner/admin membership.
7. Branches may be created in draft/private setup state. This is not a representation that they are legally authorized to operate.
8. Establishment evidence is requested only when a specific capability or legal review requires it.
9. Professional credential evidence is requested only for a specific person/action/claim that requires it.
10. The organization configures its branches, catalogue, roles, suppliers, and operational settings.
11. The system records any service plan/entitlement separately from identity and legal evidence.
12. The organization can begin using each permitted management capability once its own prerequisites pass.

If an identity, authority, establishment, or credential check is pending, show the exact affected capability and next step. Do not silently disable unrelated private management functions.

## 5. Branch lifecycle and ownership

Candidate branch fields:
- stable branch ID and tenant ID;
- public/internal name;
- linked legal entity and operator, where known;
- address, locality, coordinates, and address confirmation provenance;
- branch category and service configuration;
- establishment evidence references;
- timezone and operating hours;
- branch-specific catalogue, stock, suppliers where applicable, POS, staff assignments, and financial scope;
- public publication configuration;
- operational status and closure reason.

Candidate states:
- draft;
- active for permitted internal operations;
- restricted;
- temporarily closed;
- permanently closed;
- archived.

Branch status must not be confused with public profile status. A branch may be active for private management while not publicly listed, subject to applicable law. Conversely, pausing a public profile does not automatically close the physical establishment or disable private records.

A branch transfer, rename, address correction, legal-entity change, or permanent closure must preserve history and transaction ownership. Do not rewrite old sales, stock movements, or evidence to make them appear to have occurred under a new branch.

## 6. Invitations and membership lifecycle

Candidate membership states:
- invited;
- accepted/pending activation;
- active;
- restricted;
- suspended;
- revoked;
- expired.

Invitation rules:
- invitation targets a verified account identifier or a controlled invitation flow;
- invitation includes tenant, intended branch scope, proposed role/permissions, inviter, expiry, and terms;
- invitation token is single-use, time-limited, and stored safely;
- accepting an invitation does not automatically prove a professional credential or legal authority;
- the invitee must authenticate as the intended person;
- role/branch scope is shown before acceptance;
- expired, revoked, or superseded invitations cannot be reused;
- an inviter cannot grant permissions they do not possess or delegate;
- invitation delivery failures and wrong-recipient reports have safe recovery paths.

A person may belong to multiple customer organizations. Each membership is independent, and switching organizations must change the server-side authorization context. Never trust a tenant ID supplied by the client as proof of membership.

## 7. Role and permission assignment

Use the explicit permission identifiers in [Specification 18](18-proposed-permission-matrix.md), reconciled with the module specifications.

Role assignments:
- are tenant-scoped and optionally branch-scoped;
- have actor, time, reason, and effective period;
- cannot grant a permission beyond the assigner's delegation authority;
- support temporary and revocable grants;
- must not infer a professional credential from role title;
- must not infer organization ownership from being a professional;
- require separation of duties for designated high-risk actions.

When a person's role changes, all existing sessions and active subscriptions must observe the change within a defined revocation objective. Queued offline commands are re-authorized on the server at synchronization; commands that no longer pass are rejected with a safe explanation and audit record.

## 8. Owner succession, recovery, and emergency access

The system must prevent an organization from becoming permanently inaccessible because one owner loses access, leaves the business, or becomes unavailable.

Define:
- more than one trusted organization administrator where appropriate;
- verified ownership/authority transfer process;
- identity recovery and account compromise response;
- disputed ownership escalation and evidence review;
- deceased/incapacitated owner process subject to applicable law;
- temporary emergency access with approval, time limit, narrow scope, alerting, and post-event review.

Support staff must not simply change ownership based on an email request. Ownership transfer must preserve historical actor attribution and avoid granting access to a disputed claimant before review.

Emergency access is not a broad permanent administrator role and must not bypass legal/professional restrictions.

## 9. Evidence and verification lifecycle

Establishment evidence and professional credentials are separate records with separate subjects and scopes.

Each evidence record should preserve:
- subject person or organization/branch;
- evidence type and claimed issuer;
- jurisdiction and establishment/professional category;
- reference/identifier and validity period;
- secure document reference where necessary;
- source and verification method;
- submitted, reviewed, approved, rejected, expired, revoked, or superseded state;
- reviewer, decision rationale, timestamps, and appeal/re-review history.

Uploaded evidence is not verified merely because it is present. Reviewers must be assigned, conflicts of interest prevented, evidence access logged, and retention/deletion rules applied.

An expired professional credential affects only the actions that depend on it. An expired establishment record affects only operations or public claims that depend on it. Do not automatically revoke every ordinary membership.

## 10. Subscription, entitlement, and service restrictions

Subscription and payment are service-contract concerns, not legal credentials or ownership.

The service model must define:
- plans and included capabilities;
- trial, billing cycle, tax, currency, grace period, failed-payment behavior, cancellation, refunds, and disputes;
- seat/branch/usage limits, if any;
- entitlement activation and expiration;
- notice periods and administrator contact;
- data access/export during suspension and after termination.

A failed subscription payment must not erase or corrupt operational data. Suspension must be capability-specific where practical, with a safe read/export mode for lawful records and customer continuity. Do not block access to required records or prevent lawful export as a debt-collection shortcut.

The first release's pricing, free tier, trial duration, and enforcement policy remain commercial decisions requiring customer research and legal/accounting review.

## 11. Restriction, suspension, and offboarding

Restriction/suspension reasons may include security incident, suspected account compromise, legal direction, serious abuse, service non-payment under agreed terms, or evidence expiry. Each reason has its own authority, evidence, scope, duration, and appeal path.

Do not combine:
- a public-content moderation action;
- an establishment evidence expiry;
- a professional credential expiry;
- a billing dispute;
- an account security lock;
- a tenant-wide legal suspension.

They have different consequences and review owners.

Offboarding workflow:
1. authenticate an authorized customer representative;
2. record the request and effective date;
3. explain the effect on users, branches, public publication, integrations, and billing;
4. stop or transfer public publication according to the customer's choice and legal requirements;
5. provide a complete, documented export in usable formats;
6. verify export integrity and delivery;
7. preserve records subject to statutory, contractual, dispute, and security retention duties;
8. delete or de-identify remaining data according to an approved schedule;
9. revoke sessions, API credentials, devices, and integrations;
10. issue closure confirmation and retain a minimal closure audit record.

A public profile removal is not tenant offboarding. Membership revocation is not account deletion. Tenant closure is not permission to erase records that must lawfully be retained.

## 12. Tenant isolation and data ownership

Every tenant-owned record must have explicit tenant ownership, and branch-owned records must have explicit branch scope or a documented organization-wide scope.

Server-side isolation must apply to:
- API reads and writes;
- background jobs and scheduled tasks;
- event topics and consumer subscriptions;
- search indexes and caches;
- file/object storage;
- exports and reports;
- support tooling;
- analytics and logs where tenant data appears;
- backups and restore procedures.

Never rely only on client-supplied tenant/branch IDs. Derive authorized scope from authenticated membership and server-side resource relationships.

Cross-tenant platform data (billing, verification, moderation, security) must have explicit ownership and access policy. Support access is purpose-bound, case-scoped, time-limited, and audited.

## 13. Offline and event-driven behavior

- Organization/branch setup and invitations may be drafted offline only where local data protection and replay rules are approved.
- Membership activation, role changes, ownership transfer, evidence decisions, suspension, entitlement changes, and offboarding are online-only authoritative transitions.
- A local pending invitation or role change must not be represented as active server authorization.
- Membership, permission, branch, and entitlement changes emit committed versioned events.
- Existing sessions and clients subscribe to relevant changes and refresh authorization; do not continuously poll for unchanged membership state.
- Revocation events invalidate or re-evaluate sessions, caches, queued commands, and active device grants within the approved objective.
- Missed events are recovered through durable replay/cursor or authoritative snapshot reconciliation.

## 14. Required test scenarios

Automated and operational tests must prove:
- a person can create an account and join as an ordinary member without a professional credential;
- a non-professional owner can use permitted management functions;
- one person can hold independent memberships in multiple tenants without cross-tenant leakage;
- client-supplied tenant/branch IDs cannot bypass membership scope;
- invitation tokens expire, are single-use, and cannot be accepted by the wrong identity;
- an inviter cannot grant permissions they do not hold or delegate;
- a professional role title does not grant regulated authority without the required verified credential;
- a professional credential does not grant organization ownership or unrelated finance access;
- branch closure does not rewrite historical transactions;
- public profile pause does not disable private operations;
- credential expiry blocks only dependent actions;
- entitlement suspension does not erase data or block lawful export;
- account recovery and owner succession require approved evidence and preserve audit history;
- revoked membership/permissions invalidate existing sessions and queued offline commands within the defined objective;
- tenant isolation holds in API, jobs, events, caches, files, reports, exports, and backups;
- offboarding export can be restored/read and matches source counts/hashes;
- deletion jobs preserve records under legal hold and produce a verifiable deletion report for eligible data.

## 15. Open decisions before implementation

1. What is the real-world relationship between one Pharmalink tenant, legal entity, owner/operator, and licensed establishment?
2. Can one tenant represent multiple legal entities, or must each legal entity have its own tenant?
3. What evidence is proportionate to verify the creator's authority to represent a business?
4. What customer categories and establishment types are in the initial target cohort?
5. What is the minimum branch model and which operational records are branch-scoped versus organization-wide?
6. Which role templates are useful defaults, and which must be customized by each customer?
7. What is the invitation identity mechanism and recovery process in low-connectivity settings?
8. What is the revocation objective for active sessions, mobile devices, and queued offline actions?
9. What owner succession and disputed ownership process is legally and operationally supportable?
10. Which subscription plans, grace periods, restrictions, and safe export modes are commercially viable?
11. What exact retention/deletion schedules apply to tenant, billing, verification, security, and support records?
12. Which exports are required, in what formats, and how are their integrity and completeness proven?
13. What support access can be provided without exposing unrelated tenant data?
14. Which regional and federal establishment rules alter onboarding or branch capability eligibility?

Resolve tenant/legal-entity modeling and branch ownership before database schema lock. Do not build authorization around an assumed one-tenant/one-licence/one-professional relationship.
