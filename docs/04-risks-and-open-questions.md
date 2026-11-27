# 04 — Risk and open-question register

**Status:** Initial register — maintain throughout discovery

Ratings are preliminary. Risks and questions must not be silently treated as resolved.

## 1. Initial risks

| ID | Risk | Severity | Required response |
|---|---|---:|---|
| R-01 | Stale availability misleads customers | High | Freshness policy and stale/unknown states |
| R-02 | Missing or incorrect batch/expiry data | High | Receiving controls, validation and reconciliation |
| R-03 | Concurrent checkout oversells stock | High | Atomic allocation and concurrency tests |
| R-04 | Offline devices create conflicting stock histories | High | Idempotency, conflict rules, limits and reconciliation |
| R-05 | Cross-tenant access exposes private data | Critical | Server-side isolation and security testing |
| R-06 | Public medicine content or promotion breaches rules | High | Content governance and local regulatory review |
| R-07 | Personal/health data is hosted or transferred unlawfully | Critical | Data mapping and legal residency/transfer assessment |
| R-08 | Verification badge implies government endorsement | High | Precise wording, evidence trail and revalidation |
| R-09 | Poor catalogue matching selects the wrong product | High | Authoritative sources, provenance and pharmacist review |
| R-10 | Payment reconciliation errors cause disputes | High | Provider assessment and immutable payment records |
| R-11 | Lost device exposes offline data | High | Encryption, local access controls and session/device revocation |
| R-12 | Backups cannot be restored | High | Restore drills and recovery objectives |
| R-13 | AI invents figures or gives unsafe advice | High | Permission-aware retrieval and clinical restrictions |
| R-14 | Subscription suspension blocks required records | High | Read-only/export and retention policy |
| R-15 | External provider outage disrupts core service | Medium/High | Provider assessment, graceful degradation and exit plan |

## 2. Regulatory and evidence questions

These are not legal conclusions. Research current primary sources and obtain qualified Ethiopian advice where interpretation is required.

| ID | Question | Impact |
|---|---|---|
| Q-01 | What licences/authorizations apply to the operator and each proposed activity? | Launch eligibility |
| Q-02 | What EFDA rules govern public medicine catalogue content, online information and advertising? | Public search and content |
| Q-03 | Which medicine classes may be displayed publicly and under what conditions? | Catalogue eligibility |
| Q-04 | What rules apply if reservation, ordering, payment, delivery or fulfilment is added? | Scope/licensing |
| Q-05 | Which establishment and professional credentials must be checked and through which authoritative source? | Verification |
| Q-06 | What localization, storage and cross-border transfer obligations apply to each data category? | Hosting |
| Q-07 | What lawful basis, consent, data rights, retention and breach-notification duties apply? | Privacy operations |
| Q-08 | What pharmacy and prescription record-retention rules apply? | Data lifecycle |
| Q-09 | What tax, invoicing, fiscal-device and accounting requirements apply? | POS/finance |
| Q-10 | What payment-provider, merchant onboarding and settlement rules apply? | Billing |

## 3. Market and workflow questions

| ID | Question | Research method |
|---|---|---|
| Q-11 | Which pharmacy types are the initial target: independent retail, chains, hospital or others? | Interviews and pilot recruitment |
| Q-12 | How are stock receiving, batch and expiry recorded today? | Observe receiving |
| Q-13 | How common are barcodes and usable product identifiers? | Field observation |
| Q-14 | Which phones, computers, printers and scanners are common? | Field survey |
| Q-15 | How long are connectivity outages, and what must work offline? | Interviews and site tests |
| Q-16 | Which counter and supplier payment methods are used? | Owner/accountant interviews |
| Q-17 | Which languages/scripts are essential in the first pilot? | User research |
| Q-18 | What subscription price is acceptable to target pharmacies? | Pricing research |
| Q-19 | What is the smallest useful first release? | Workflow prioritization |
| Q-20 | Are freshness-qualified availability states sufficient, or is quantity visibility needed? | Customer research |

## 4. Medicine data questions

- Is there a lawful, reliable, machine-readable source for Ethiopian medicine registration and product data?
- Can data be obtained and redistributed?
- How are generic names, brands, strengths, dosage forms and pack sizes normalized?
- How are duplicates and misleading barcode matches handled?
- How are unregistered, discontinued, recalled or uncertain products represented?
- Who reviews catalogue corrections?
- What source provenance and review dates must be recorded or displayed?

## 5. Offline and event-system questions

- Which operations are permitted offline?
- Can offline sales be limited to device/branch-allocated stock?
- How are disconnected devices prevented from selling the same last units?
- Which actions require online authorization?
- How are prescription restrictions enforced offline?
- How are event ordering, replay and deduplication handled?
- How does a client reconcile after missing events?
- What is the authoritative conflict-resolution process?
- What synchronization delay and recovery objectives are acceptable?
- Which events contain sensitive data, and how are subscriptions scoped?

## 6. Operational questions

- Who reviews onboarding and verification?
- What evidence is required before public listing?
- What are appeal and suspension procedures?
- What support hours and response commitments are realistic?
- Who handles incorrect medicine data or availability incidents?
- What backup frequency and recovery objectives are needed?
- What happens when payment, maps, SMS, push or AI providers fail?
- What data can a pharmacy export when leaving?

## 7. Research gates

Before architecture is locked:
- Interview owners, pharmacists, cashiers and inventory staff.
- Observe a complete receiving-to-sale workflow.
- Validate offline requirements under real connectivity.
- Review current Ethiopian regulatory and data-protection sources.
- Evaluate medicine catalogue data availability and licensing.
- Compare local payment and messaging providers.
- Test discovery concepts with customers.
- Record evidence, contradictions and decisions here.

Every high-severity risk needs an owner, mitigation, evidence and residual-risk decision before production readiness.
