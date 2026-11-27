# Pharmalink

**Working name:** Pharmalink  
**Initial market:** Ethiopia  
**Phase:** Product discovery and specification  
**Implementation:** Not started

Pharmalink is a proposed integrated pharmacy-operations and medicine-discovery platform for Ethiopia. Its intended product has three connected surfaces:

1. **Pharmacy Operations:** business and branch management, staff permissions, medicine catalogue, batch inventory, purchasing, suppliers, POS, returns, finance, reporting, expiry, recalls and compliance workflows.
2. **Public Discovery:** medicine and pharmacy search, location discovery, pharmacy profiles, availability freshness, contact details and directions.
3. **Platform Operations:** pharmacy onboarding and verification, subscriptions, billing, support, moderation, policy administration and service monitoring.

These surfaces share authoritative domain data and security controls but have distinct interfaces and permissions.

## Current objective

Complete and challenge the product, operational, regulatory, UX and technical specifications before implementation begins. Documents are drafts until explicitly reviewed and approved.

## Non-negotiable principles

- Correctness, safety and auditability before convenience.
- Medicine catalogue identity is separate from physical stock batches.
- Tenant and branch isolation is enforced server-side.
- Stock changes and financial corrections are traceable.
- Use event subscriptions for relevant live updates instead of routine polling for unchanged data.
- Design offline operation and synchronization explicitly.
- Replace obsolete implementations instead of accumulating unnecessary V1/V2 layers, compatibility shims or permanent fallbacks.
- Migrate and verify required business data before destructive replacement.
- No fake functionality, availability, verification or integrations.
- AI is optional, permission-aware and not a clinical decision-maker.
- Regulatory claims require current evidence and qualified local review.

## Documentation

- [Specification workflow](docs/README.md)
- [Project charter](docs/01-project-charter.md)
- [Product scope](docs/02-product-scope.md)
- [Principles and decisions](docs/03-principles-and-decisions.md)
- [Risks and open questions](docs/04-risks-and-open-questions.md)

This repository is the authoritative planning record. Chat discussions and agent proposals are inputs; approved repository decisions govern the project.

“Pharmalink” is a working name, not trademark or domain clearance.

## Platform access and optional public commerce

Pharmalink is a platform with pharmacy-management capabilities and a separate public-facing discovery/access experience. A pharmacy may use the management system privately without publishing its catalogue, advertising availability, accepting orders, or selling through the public-facing experience. Public listing and any commerce-related capabilities are opt-in, separately configurable, and subject to applicable legal and operational review. A professional credential is not a general requirement to create an account or use ordinary platform-management features; credential checks apply only to specific regulated activities or claims where required.

## Production release standard

The intended outcome is a deployable, functional product for a validated first-release scope—not a demo. Every enabled capability must use real persistent data, server-enforced permissions, actual business transactions, defined failure/recovery behavior, automated tests, monitoring, and documented operations. Mock data and simulated success are prohibited in production. See the [production deployability contract](docs/12-functional-product-and-production-deployability-contract.md).
