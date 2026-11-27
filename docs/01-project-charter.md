# 01 — Project charter

**Status:** Draft — owner review required  
**Working name:** Pharmalink  
**Initial market:** Ethiopia  
**Category:** Pharmacy operations software and medicine/pharmacy discovery

## 1. Purpose and vision

Build a dependable digital platform that helps pharmacy businesses manage authorized daily operations and helps the public discover pharmacies and understand whether medicine availability information is current.

The problems this project intends to address—fragmented operations, weak batch/expiry visibility, poor stock visibility and limited public pharmacy information—are hypotheses. Validate them with Ethiopian pharmacy owners, pharmacists, employees and customers before treating them as proven market facts.

## 2. Intended outcomes

### Pharmacy businesses
- Reliable stock visibility by medicine, batch and branch.
- Better expiry, recall, damage and quarantine control.
- Traceable purchasing, receiving, sales, returns and adjustments.
- Useful financial and operational reporting.
- Controlled staff access.
- Multi-branch management without mixing branch records.
- Continued operation during defined connectivity outages, with explicit offline safety limits.

### Public users
- Search medicine names, strengths and dosage forms.
- Discover pharmacies by location and service information.
- Understand whether availability data is current, stale, unavailable or unknown.
- Contact a pharmacy before travelling.
- Never mistake stale information for guaranteed stock.
- Receive factual information, not diagnosis or treatment recommendations.

### Platform operator
- Operate onboarding and verification workflows.
- Manage subscriptions and billing transparently.
- Handle support, complaints and disputes.
- Moderate public content and enforce platform policies.
- Monitor security, event delivery and service health.

## 3. Product surfaces

### Pharmacy Operations
Business-facing application for owners and authorized staff: business/branch setup, access control, catalogue, batch inventory, suppliers, purchasing, POS, returns, finance, reporting, expiry/recall, compliance, notifications and settings.

### Public Discovery
Public experience for medicine and pharmacy search, location-aware discovery, pharmacy profiles, availability freshness, contact and directions. Ordering, reservations, payments, delivery and medicine promotion are not assumed to be in scope until separately assessed.

### Platform Operations
Restricted internal console for onboarding, verification, subscription administration, support, moderation, policy and monitoring. It must not provide unrestricted access to sensitive pharmacy or customer data.

## 4. Stakeholders

| Stakeholder | Main interests |
|---|---|
| Pharmacy owner | Business, branches, staff, stock and finance |
| Pharmacy manager | Daily operations and oversight |
| Pharmacist | Safe and compliant professional workflows |
| Sales/cashier staff | Fast, authorized checkout |
| Inventory/purchasing staff | Receiving, batches and suppliers |
| Accountant | Reconciliation and financial records |
| Public user | Trustworthy pharmacy discovery |
| Platform operator | Sustainable operations and security |
| Regulators/authorities | Compliance with applicable rules |
| Suppliers/distributors | Orders, receiving and reconciliation |
| Payment providers | Secure settlement and reconciliation |

## 5. Business model hypothesis

Initial hypothesis: B2B subscriptions from pharmacies, potentially tiered by branch count or capability. Public discovery is expected to be free initially.

A 90-day free trial has been discussed as a possibility, not approved policy. Pricing, trial length, plan entitlements, payment methods, tax treatment, grace periods, suspension, refunds and cancellation require market and legal review.

Paid placement must not distort medicine search relevance or undermine public trust.

## 6. Scope boundaries

Intended domains:
- Pharmacy business and branch management
- Staff identity, roles and permissions
- Medicine catalogue and search aliases
- Batch inventory and traceability
- Suppliers and purchasing
- POS, sales, returns and refunds
- Finance and reporting
- Expiry, recall, quarantine and disposal
- Offline-aware operations and event-driven updates
- Public pharmacy and medicine discovery
- Pharmacy onboarding and verification
- Subscription and platform administration
- Audit, privacy, security and monitoring

Not assumed:
- Diagnosis, prescribing or treatment recommendations
- Automated dispensing decisions
- Unlicensed online sale or delivery
- Public advertising of restricted medicines
- Custody of customer funds or a wallet
- Government integrations without authorization
- AI-controlled stock, price, prescription or financial changes
- Guaranteed live stock when a pharmacy is offline
- A national verified medicine registry without an authorized authoritative source

## 7. Success measures

Set numerical targets only after pilot baselines. Candidate measures:
- Inventory accuracy by branch and batch
- Completeness of batch/expiry records
- Overselling and stock-conflict rate
- Routine sale completion time
- Offline synchronization success and conflict rate
- Event delivery latency/failure rate
- Public availability freshness
- Pharmacy onboarding and verification turnaround
- Staff adoption and pharmacy retention
- Support resolution time
- Security incidents
- Backup restore-test success

## 8. Constraints to validate

- Internet reliability and cost
- Android-first usage and device capability
- Barcode scanners and receipt printers
- Local languages, scripts and transliteration
- Local payment-provider capabilities
- Ethiopian calendar display and timezone handling
- Privacy and health-data obligations
- Variation in pharmacy size and workflow
- Availability and licensing of authoritative medicine data

## 9. Guiding principles

1. Public and patient safety outrank convenience.
2. Business records must be accurate, traceable and recoverable.
3. Each fact has one authoritative owner.
4. Catalogue identity differs from batch stock.
5. Access is scoped by tenant, branch and permission.
6. Event-driven updates are preferred to unnecessary polling.
7. Offline behavior is designed and tested.
8. Obsolete implementations are replaced after safe data migration.
9. No fake data, availability, verification or integrations.
10. AI is optional and not authoritative for clinical decisions.
11. Regulatory claims require current evidence and qualified review.
12. Prefer reliable simplicity over premature microservices or decorative complexity.

## 10. Specification readiness gate

Technical design may begin only after:
- Major actors and end-to-end workflows are documented.
- Functional requirements and business rules are testable.
- Domain entities, ownership and lifecycle are specified.
- Permissions and tenant boundaries are explicit.
- Offline limits and conflict rules are defined.
- Event contracts and delivery guarantees are documented.
- Security, privacy and regulatory risks have treatment plans.
- UX structure and critical screens are specified.
- External integrations are evaluated.
- Non-functional requirements have measurable acceptance criteria.
- Tests cover normal, exceptional, concurrent and failure cases.
- Operations, backup, recovery and support are defined.
- High-risk unknowns are resolved or explicitly accepted with mitigation.

This charter does not claim these conditions are already satisfied.

## 11. Platform access is separate from public selling

Pharmalink serves two distinct usage modes within one platform: (1) private pharmacy/business management and (2) optional public-facing discovery and, only if separately approved, commerce-related capabilities. A pharmacy retailer may use management functions without publishing products, exposing stock availability, receiving public enquiries, accepting orders, or selling through Pharmalink. These choices are controlled by the pharmacy at business and branch level.

Account creation, platform access, and ordinary management use must not be conditioned on an individual's professional licence. A professional credential is relevant only when a specific workflow or legal requirement calls for a qualified professional to perform or approve that action. Establishment licensing and professional credentials are different evidence types; any verification needed for a public trust claim or a regulated capability must be proportionate, transparent, and legally reviewed. This product rule does not waive any legal obligation a pharmacy has to operate its physical business lawfully.
