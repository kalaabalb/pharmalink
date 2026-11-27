# 21 — External integrations and provider lifecycle specification

**Status:** Draft — provider selection, contract review, privacy/security review, and operational validation required  
**Priority:** High — production behavior must not depend on undocumented third-party assumptions  
**Scope:** External identity/authentication, SMS/email delivery, payments, maps/geocoding, medicine catalogue sources, accounting exports, file/object storage, monitoring, and other external service boundaries.

## 1. Purpose and integration policy

This document defines how Pharmalink evaluates, connects, operates, secures, monitors, and exits external providers.

An integration is a production dependency only after its provider, purpose, data flow, legal basis, contract, service behavior, failure handling, and operational owner are documented and tested.

Do not add an integration because a demo needs a button to work. Do not claim a provider is connected when the system is using a mock, local fixture, manual database update, or unverified endpoint.

Avoid building a generic integration marketplace or speculative abstraction framework. Use a small, explicit adapter boundary only where it protects a real domain contract or allows a documented provider to be changed. Do not preserve obsolete provider behavior indefinitely for hypothetical future compatibility.

If a provider is not selected or cannot be operated safely, the affected capability remains disabled or uses a clearly documented manual process approved for the release. Never silently substitute a fake provider.

## 2. Integration inventory and ownership

Maintain a versioned integration register with:
- integration ID and business capability;
- provider and product/service;
- production owner and technical owner;
- environments and endpoints;
- data categories sent/received;
- purpose and lawful basis/consent where applicable;
- countries/regions of processing and storage;
- provider contract, terms, DPA, and security evidence;
- authentication method and secret owner;
- API/webhook contract version;
- rate limits, quotas, timeout, and retry behavior;
- availability and support commitments;
- cost model and billing owner;
- incident contact and escalation route;
- data retention/deletion and export behavior;
- provider exit and migration plan;
- last security, privacy, and operational review.

A provider integration must not go live without a named accountable owner. Provider credentials must be environment-specific and must not be committed to Git, client bundles, screenshots, logs, or support tickets.

## 3. Candidate integration categories

These are candidate categories, not provider selections or commitments.

| Category | Purpose | Required decision before production |
|---|---|---|
| Authentication/identity | Account authentication, verification, recovery | Supported methods, account recovery, MFA policy, abuse controls, local accessibility |
| SMS/email delivery | Invitations, security alerts, service notices, optional OTP | Delivery provider, sender registration, consent, retry, cost, local reachability |
| Payment collection | Customer subscription billing and any separately approved customer payments | Licensed provider, settlement semantics, fees, refunds, webhook verification, reconciliation |
| Maps/geocoding | Pharmacy address confirmation and public discovery maps | Data licensing, coverage, location precision, caching, attribution, provider outage behavior |
| Medicine catalogue source | Canonical medicine/product data | Authority, licence/redistribution terms, source provenance, update process |
| File/object storage | Private evidence, receipts, exports, backups, media | Encryption, region, access control, retention, versioning, lifecycle, recovery |
| Accounting export/integration | Customer accounting workflow | Supported format/API, mapping, idempotency, correction, reconciliation |
| Monitoring/alerting | Service health, error and security alerts | Data minimization, region, access, alert ownership, retention |
| Analytics | Product/operational metrics | Purpose, consent/legal basis, aggregation, minimization, no health inference |
| Support tooling | Case management and customer support | Case-scoped access, data transfer, retention, audit, vendor controls |

The first release should select only integrations needed for its validated vertical slice. Do not integrate every category by default.

## 4. Provider selection and due diligence

For each candidate provider, evaluate:
- legal right to use the service for Pharmalink's business and launch jurisdiction;
- current registration/licensing status where regulated;
- geographic coverage and reliability in target pharmacy locations;
- API maturity, documentation, versioning, and deprecation policy;
- authentication and cryptographic controls;
- data processing, storage region, subprocessors, and cross-border transfer;
- privacy terms, retention, deletion, and breach notification;
- service availability, support hours, incident communication, and recovery;
- rate limits, quotas, pricing, hidden fees, and volume scaling;
- sandbox/production differences;
- webhook/event support and reconciliation capability;
- data portability and exit cost;
- operational ability to monitor and support the integration.

A marketing page or sandbox success is not sufficient due diligence. Store evidence, decision, reviewer, date, and unresolved risks.

## 5. Integration boundary and contract

Each integration defines:
- the Pharmalink domain capability it supports;
- inbound/outbound data schema and version;
- authentication and authorization;
- request correlation and idempotency;
- timeout and retry policy;
- rate-limit handling;
- provider error mapping;
- webhook signature/authentication and replay protection;
- source-of-truth ownership for each field/state;
- audit and observability requirements;
- data minimization and retention;
- outage and recovery behavior.

Domain logic must not be spread across UI components or provider-specific callbacks. Provider adapters translate an external contract into an explicit internal command/event; they do not own the pharmacy's business truth.

Do not let an external provider directly mutate arbitrary tenant records. All changes pass through authenticated, validated domain commands and server-side authorization.

## 6. Webhooks, events, and asynchronous delivery

Where a provider supports webhooks/events:
- verify signatures using the provider's documented method;
- validate timestamp/nonce or equivalent replay controls;
- store provider event ID and payload hash;
- process idempotently;
- acknowledge only after durable receipt according to the provider contract;
- handle duplicate, delayed, out-of-order, and missing events;
- preserve a minimal redacted audit record;
- never trust a client-supplied “payment succeeded” or similar status.

Provider events are external inputs, not Pharmalink domain events. Verify and process them; only after the corresponding internal transaction commits should Pharmalink publish its own domain event through the transactional outbox.

Use bounded retries with backoff, jitter, maximum attempts, dead-letter handling, and an operator-visible recovery path. Do not retry permanent validation/authentication failures indefinitely.

If a provider has no webhook support, document the exact bounded reconciliation schedule, provider limits, and reason polling is unavoidable. Do not use frequent polling as a default substitute for event delivery.

## 7. Payment provider contract

Subscription billing and any future customer checkout are distinct payment use cases and may require different providers and legal reviews.

For each payment flow, specify:
- payer/payee and merchant-of-record roles;
- supported payment methods and currencies;
- authorization, capture, settlement, reversal, refund, dispute, and chargeback semantics;
- fee and tax treatment;
- transaction and idempotency identifiers;
- signed callback/webhook contract;
- reconciliation file/API and settlement timing;
- partial payments and failure behavior;
- customer notification responsibilities;
- provider outage behavior;
- support/dispute ownership;
- data retained by Pharmalink versus provider.

Do not store raw card credentials or payment secrets. Do not mark a transaction paid from a redirect query parameter or client callback. A payment status becomes confirmed only after a verified authoritative provider response or an approved, audited manual confirmation policy.

Payment provider integration must pass duplicate, replay, delayed callback, reversal, refund, settlement mismatch, and reconciliation tests before production.

## 8. Authentication and message-delivery providers

Authentication is a core security boundary. Provider selection must consider recovery, account enumeration, phishing resistance, session revocation, MFA options, device security, and access from low-connectivity areas.

SMS/email are delivery channels, not proof of a person's professional credential or business authority. A successful OTP proves only control of a delivery endpoint under the defined method and time; it does not establish legal identity or professional authorization.

Message delivery must:
- use approved templates and locale;
- minimize sensitive content;
- avoid medicine names, health searches, prescription data, or financial details in ordinary SMS/push previews;
- distinguish transactional/security messages from marketing;
- respect consent and unsubscribe requirements where applicable;
- record provider message IDs and delivery states;
- retry within bounded policy;
- prevent duplicate sends through idempotency;
- provide an accessible alternative where SMS/email is unavailable.

## 9. Maps and geocoding

Maps/geocoding must support confirmed pharmacy branch locations without exposing a public user's precise location.

Before use, confirm:
- coverage and address quality in Ethiopia and target regions;
- geocoding accuracy and confidence indicators;
- map tile/geocoding licensing and attribution;
- caching and offline-use rights;
- provider retention of queries/device location;
- permitted use in commercial public discovery;
- cost and quota behavior;
- fallback behavior when provider is unavailable.

Do not silently convert an uncertain geocode into a confirmed address. Tenant representatives must review and confirm branch locations. If maps fail, manual address/locality discovery should remain usable where supported.

## 10. Medicine catalogue data providers

Catalogue ingestion is governed by [Specification 16](16-medicine-catalogue-and-product-identity-specification.md). A data provider must be evaluated for authority, coverage, update cadence, licensing, redistribution, correction process, and source continuity.

Do not copy, scrape, redistribute, or imply endorsement of catalogue data without confirmed rights. Do not label a third-party dataset as an official national registry unless the competent authority and source support that claim.

If the source becomes unavailable or licence expires, preserve lawful historical transaction references, stop unauthorized new redistribution, and follow a documented remediation/exit plan.

## 11. File storage, exports, and backups

Private evidence, financial documents, receipts, and customer exports require separate access and retention policies from public media.

Storage requirements:
- encryption in transit and at rest;
- server-side authorization on every retrieval;
- short-lived access links where appropriate;
- object-level tenant/branch ownership;
- malware scanning and safe content-type validation for uploads;
- immutable/versioned evidence where required;
- audit of view/download/delete;
- retention and legal-hold enforcement;
- integrity checks for export and backup objects;
- region and cross-border processing review.

A storage provider is not a backup strategy by itself. Backup and restore requirements remain governed by [Specification 11](11-nonfunctional-requirements-and-release-gates.md) and the production deployability contract.

## 12. Provider outage and degraded operation

Each integration must define:
- timeout and circuit-breaker behavior where appropriate;
- which workflows can continue without the provider;
- what is queued and for how long;
- what is rejected immediately;
- how users see pending/failed state;
- how retries and reconciliation recover state;
- whether stale external data is hidden or labelled;
- escalation and incident communication.

Provider outage must not silently convert an authoritative action into success. For example, a payment timeout is pending/unknown until reconciled, not automatically failed or paid. A map provider outage must not prevent private stock receiving. An SMS outage must not activate an invitation that was never accepted.

Degraded behavior must be explicit and tested, not an unbounded queue of irreversible commands.

## 13. Secrets, access, and security operations

- Store secrets in an approved secret manager or protected deployment environment.
- Use separate least-privilege credentials per environment and service.
- Rotate credentials and revoke compromised keys through a documented process.
- Never log authorization headers, secrets, raw payment tokens, or full sensitive payloads.
- Restrict provider dashboard access and review it periodically.
- Use signed webhooks and allowlisted egress only where the provider contract supports it.
- Monitor unusual provider API usage, unexpected spend, webhook failures, and credential changes.
- Include third-party compromise and data breach in incident response.
- Review subprocessors and provider terms when they change.

## 14. Provider change and exit

Changing a provider requires:
1. documented reason and approval;
2. data and contract compatibility assessment;
3. migration/dual-running plan only where necessary and time-limited;
4. reconciliation of in-flight transactions;
5. credential revocation and access removal;
6. data export/deletion request as contractually and legally appropriate;
7. monitoring and rollback criteria;
8. closure evidence and updated integration register.

Do not retain obsolete provider adapters indefinitely. Remove them after verified migration and required retention. Preserve provider transaction references needed for audit and dispute resolution.

## 15. Required test scenarios

Automated and operational tests must prove:
- a mock/sandbox provider cannot be enabled in production by configuration mistake;
- provider secrets never appear in source control, client assets, logs, or error messages;
- invalid signatures and replayed webhooks are rejected;
- duplicate provider events cannot duplicate internal financial or business effects;
- delayed/out-of-order events cannot overwrite newer authoritative state;
- payment redirects/client payloads cannot mark a payment as settled;
- provider timeout produces an explicit pending/unknown state and reconciliation path;
- provider outage does not block unrelated private pharmacy operations;
- message retries do not send duplicate invitations/security notices;
- OTP success does not imply professional credential or establishment verification;
- geocoding uncertainty remains visible until branch confirmation;
- public search remains usable without precise location permission;
- data provider licence expiry triggers an explicit publication/ingestion decision;
- storage access is tenant-scoped and every sensitive document access is audited;
- provider change preserves transaction references and reconciles in-flight activity;
- production monitoring detects webhook backlog, dead-letter growth, quota exhaustion, and unexpected cost.


## 15A. Service-wide incident coordination

Provider-specific outage behavior is governed by this specification; service-wide incident command, customer communications, alerting, escalation, and post-incident review are defined in [Specification 28](28-production-operations-incident-response-and-service-continuity-specification.md). A provider incident must preserve the distinction between external delivery state and Pharmalink's authoritative business state.

## 16. Open decisions before implementation

1. Which external integrations are essential to the first validated vertical slice?
2. Which Ethiopian/local providers are legally available and operationally reliable for target regions?
3. Which authentication and recovery methods work under low connectivity and shared-device conditions?
4. Which SMS/email provider, sender registration, message pricing, and delivery commitments are acceptable?
5. Is subscription payment automated at launch or handled through an approved manual process?
6. Which payment provider and settlement/reconciliation model is permitted for any future customer transaction flow?
7. Which map/geocoding provider offers lawful coverage and suitable privacy terms?
8. Which medicine catalogue source can be licensed and maintained?
9. Which storage/backup region and cross-border processing arrangements are acceptable?
10. Which accounting export formats and integrations do target pharmacies actually use?
11. What provider outage objectives and manual recovery procedures can the operations team support?
12. Who owns provider contracts, spend, secrets, incidents, and offboarding?
13. Which integrations may be deferred without making the chosen first release incoherent?

No provider is approved by this document. Provider selection and production activation require documented due diligence, contract review, security/privacy assessment, and end-to-end acceptance evidence.
