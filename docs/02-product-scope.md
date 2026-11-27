# 02 — Product scope and capability map

**Status:** Draft — detailed module specifications remain to be written

This document defines the capability map, not every screen, field, validation rule or API operation.

## 1. Pharmacy Operations

### 1.1 Business and branches
- Business profile and one or more branches
- Branch addresses, map locations, contacts and hours
- Branch-specific staff and settings
- Consolidated owner reporting without weakening branch access controls

### 1.2 Identity and access
- Account registration and secure sign-in
- Business onboarding and invitation-based staff membership
- Role templates plus granular permissions
- Branch-scoped assignments
- Session/device management, account recovery and staff offboarding
- Audit records for privileged actions

A role name alone must not authorize an action. The server enforces permission and tenant/branch scope.

### 1.3 Medicine catalogue
Catalogue identity is separate from physical stock. Candidate attributes:
- Generic/active ingredient and brand names
- Strength/concentration, dosage form and route where relevant
- Pack size and unit of measure
- Manufacturer and authorization holder where known
- Registration/authorization reference and verification status
- Storage requirements
- GTIN/barcodes and other identifiers
- Language names, transliterations and search aliases
- Source provenance and last-reviewed date

Authoritative Ethiopian catalogue sources and redistribution rights remain open questions.

### 1.4 Batch inventory
Each physical batch/lot belongs to a catalogue product and branch. Candidate data:
- Batch/lot number and expiry date
- Quantity received and current balance
- Unit/pack conversion
- Supplier and receiving document
- Purchase cost and cost allocation
- Receiving date and storage location
- Inspection/quality status
- Recall, quarantine, damage and disposal status

Stock changes are traceable movements, not unexplained edits to a quantity field.

### 1.5 Inventory operations
- Receive against purchase records
- Controlled opening-balance import
- Branch transfers where permitted
- Authorized adjustments with reasons
- Damaged, missing, returned, quarantined and disposed stock
- FEFO allocation where appropriate
- Block expired, recalled and quarantined stock from sale
- Reorder thresholds and alerts
- Physical-count reconciliation

### 1.6 Suppliers and purchasing
- Supplier profiles and relevant authorization data
- Requisitions, purchase orders and quotations
- Supplier price history
- Partial delivery and backorder handling
- Goods receiving with batch/expiry capture
- Discrepancy, damage and rejection workflows
- Supplier invoices, balances, payments and returns

### 1.7 Point of sale
- Search and barcode scan
- Validate branch stock and batch eligibility
- Configured prices and authorized discounts/overrides
- Payment method and status
- Consistent sale and stock-movement commit
- Receipts and transaction history
- Returns, exchanges and refunds under explicit policy
- Shift opening/closing and cash reconciliation where required

Sale, line items, stock movements and audit records need defined atomicity and idempotency. Concurrent checkout must not oversell.

### 1.8 Prescription and dispensing records
A restricted workflow may be required for prescription-controlled products and professional dispensing records. Scope must be designed with Ethiopian pharmacy professionals and regulatory counsel. The system must not make clinical decisions or replace professional judgment. Prescription and patient-linked data require strict access, retention and privacy controls.

### 1.9 Finance and reporting
- Sales, refunds, discounts and adjustments
- Cost of goods sold and gross margin
- Expenses and operating costs
- Supplier payables
- Payment and cash reconciliation
- Shift/register reconciliation
- Tax/statutory reporting subject to local accounting review
- Branch and consolidated reports
- Permission-controlled exports

Revenue, gross profit, cash received and net profit are distinct measures.

### 1.10 Expiry, recall and quality
- Configurable expiry warnings and batch queues
- Expired-stock sale prevention
- Quarantine/release controls
- Recall notices and affected-batch identification
- Damage, disposal and corrective-action records
- Storage locations and temperature records where applicable
- Cold-chain sensor integration as a later capability

Software warning thresholds do not replace product-specific instructions or regulation.

### 1.11 Public listing controls
- Business-level defaults and product-level overrides
- Eligibility checks
- Availability state and freshness timestamp
- Suppression for expired, recalled or otherwise ineligible stock
- Stale/disconnected data handling

A listing does not mean a medicine is suitable for a particular person or that dispensing is approved.

### 1.12 Notifications
- In-app notifications and event-driven updates
- Optional email, SMS or push after provider and consent review
- User preferences, delivery status, retries and deduplication
- Quiet-hour policy where appropriate

## 2. Public Discovery

### 2.1 Medicine search
Search generic/brand names, strength and dosage form; support approved language variants, transliterations and spelling variation. Show clear product identity and data provenance. Avoid unsafe recommendations or unsupported substitution advice.

### 2.2 Pharmacy discovery
- Optional current-location search
- Manual city/area/address search
- Map and list views
- Distance, directions, opening hours and contact
- Branch profiles and accurately described verification status

### 2.3 Availability
Candidate states:
- Recently confirmed
- Limited — only if reliably supported
- Unavailable
- Unknown
- Stale — last update too old

Do not expose exact stock quantities by default. Every availability state needs a last-confirmed time and freshness policy. Offline pharmacies cannot appear freshly confirmed. Public listing and availability are opt-in, time-bounded projections governed by [Specification 19](19-public-discovery-and-availability-specification.md). A pharmacy can operate in management-only mode. Medicine/product publication remains disabled until the applicable legal category and content rules are reviewed.

### 2.4 Public trust and governance
- Explain exactly what a platform verification badge means
- Show information source and freshness
- Provide correction/report mechanisms
- Moderate user-submitted content if reviews are later approved
- Avoid paid medicine ranking or promotions that compromise search integrity or violate restrictions

## 3. Platform Operations

### 3.1 Onboarding and verification
- Application intake and secure document upload
- Review queues and additional-information requests
- Reviewer identity and decision records
- Expiry/revalidation reminders
- Suspension and appeal process
- Public status with precise meaning

Establishment licensing and individual professional credentials are separate concepts.

### 3.2 Subscription and billing
- Configurable plans and entitlements
- Trial and subscription lifecycle
- Invoices and payment records
- Local payment-provider integrations
- Failed-payment handling and grace periods
- Cancellation, refunds and plan changes
- Read-only/restricted states and data export/retention rules

Subscription expiry must not silently delete historical pharmacy records.

### 3.3 Support and moderation
Support cases, complaints, disputes, listing corrections, content moderation, abuse reports, restricted internal notes and case audit history.

### 3.4 Platform security and operations
Administrative identity, strong authentication, least privilege, audit trail, service/event/queue monitoring, backup/restore monitoring, incident response and data-access governance.

## 4. Cross-cutting capabilities

- Multi-tenancy and branch isolation
- Localization and accessibility
- Currency, calendar, date and timezone handling
- Offline operation and synchronization
- Event-driven communication
- Provenance and auditability
- Privacy, retention and security
- Observability and resilience
- Controlled import/export and migration

## 5. Deferred or excluded until separately approved

- Diagnosis, prescribing or treatment recommendations
- AI medicine selection or substitution
- Automated dispensing approval
- Unreviewed medicine advertising
- Promotion of restricted medicines
- Guaranteed stock reservation or online checkout
- Delivery logistics
- Custody of customer funds or stored-value wallet
- Government integration without authorization
- Predictive purchasing before reliable history exists
- Customer reviews before moderation/liability rules exist

## 6. Phasing principle

The first release should solve a coherent, safe problem for a real pharmacy cohort. Do not attempt to ship everything at once. Foundational data ownership, tenant isolation, batch traceability, auditability and event contracts must nevertheless be designed before dependent modules.

## 7. Platform access versus optional public participation

The platform must support pharmacies that want management software only.

- A pharmacy can register an organization and use permitted management capabilities without opting into public discovery or selling through the platform.
- Public profile publication, product publication, availability sharing, public enquiries, and any future order/commerce feature are separate opt-in capabilities.
- A pharmacy can enable some public capabilities while keeping others disabled, subject to approved product rules and law.
- Disabling public participation must not disable private management, inventory, purchasing, reporting, or access to the pharmacy's own records.
- Professional credentials are not a blanket prerequisite for account registration, staff access, or ordinary management functions. Credential checks are attached only to specific regulated actions, approval responsibilities, or verification claims where applicable.
- Establishment licence information, where needed for a regulated workflow or a clearly described public verification claim, is distinct from an individual's professional credential. Do not turn either into a universal sign-up gate without a demonstrated legal basis.

The initial public-facing feature set remains discovery and information. Online ordering, reservation, payment, delivery, and other commerce functions remain separate decisions requiring legal and operational review.

## 8. Management-only is a complete product mode

Management-only is a first-class supported operating mode, not a trial, degraded tier, temporary onboarding state, or incomplete version of public commerce.

A pharmacy organization may use approved private operational capabilities while choosing not to publish a public profile, catalogue, product, price, or availability. Public-facing capabilities are independent entitlements and publication controls. Enabling one does not silently enable another.

The product must distinguish platform account state, organization membership state, management-service entitlement, establishment-evidence state, individual professional-credential state, public-participation state, and commerce-capability state. No one state substitutes for another. A pending public-listing review must not disable private management. Turning off public participation must not delete business records, inventory history, staff memberships, or audit evidence.

The exact management features and any legal prerequisites for particular establishment types remain subject to market and regulatory validation. Platform access itself does not establish legal authority to operate a pharmacy.
