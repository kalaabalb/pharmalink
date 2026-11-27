# 16 — Medicine catalogue and product-identity specification

**Status:** Draft — authoritative data-source, licensing, regulatory, and pharmacy-practice validation required  
**Priority:** Critical — incorrect product identity can propagate into purchasing, stock, sales, reporting, recall, and public discovery  
**Scope:** Canonical medicine identity, product presentations, tenant catalogue entries, identifiers/barcodes, provenance, search, correction, publication, and data governance.

## 1. Purpose and safety boundary

The catalogue must help users identify the exact product they intend to manage. It is not a clinical decision system and must not diagnose, prescribe, recommend treatment, or infer that two products are clinically interchangeable.

A matching name is not enough to establish identity. Strength, dosage form, route, concentration, release characteristics, pack size, manufacturer/authorization-holder, and jurisdictional registration may distinguish products. The exact identity dimensions required for each product category must be validated with pharmacy professionals and authoritative data sources.

## 2. Identity layers

Keep these records separate:

1. **Canonical medicine/product identity:** a source-governed reference record for an identifiable product.
2. **Product presentation:** the exact strength, dosage form, concentration, route, release type, and other defining characteristics.
3. **Pack configuration:** units per pack, package level, and sellable/receivable unit relationships.
4. **Brand and naming records:** brand, generic/ingredient names, language variants, transliterations, aliases, and spelling variants with provenance.
5. **Manufacturer/authorization-holder relationship:** a sourced relationship with effective dates and jurisdiction where known.
6. **Regulatory registration/authorization record:** jurisdiction-specific identifier, status, source, and validity/effective dates where authoritative data is available.
7. **Barcode/identifier mapping:** code type, value, issuer/source, packaging level, effective period, and confidence.
8. **Tenant catalogue entry:** a pharmacy's local enablement, display name, local code, preferred supplier, purchase unit, sale unit, reorder settings, and local status.
9. **Branch-specific commercial settings:** price, tax configuration reference, availability publication setting, and other branch-owned values.

A tenant-local label or barcode must not mutate canonical identity. A branch price must not become a catalogue-wide price.

## 3. Candidate canonical attributes

The canonical model may need:
- stable internal identifier;
- generic/active ingredient names and structured ingredient composition where sourced;
- brand/trade name;
- strength and concentration with normalized unit;
- dosage form;
- route of administration where applicable;
- release characteristics where applicable;
- product presentation and pack size;
- manufacturer and authorization holder;
- country/jurisdiction of authorization;
- registration/authorization identifier and current source-backed status;
- storage and handling attributes where authoritative;
- prescription/controlled-category classification only when source-backed and legally reviewed;
- language display names and search aliases;
- source, source record ID, retrieval/effective date, review status, and change history.

Do not infer missing attributes from a similar product, image, barcode prefix, or model-generated text. Unknown values remain unknown.

## 4. Provenance and source governance

Every externally sourced attribute must carry provenance sufficient to answer:
- who or what supplied the value;
- which source record or document supports it;
- when it was retrieved and when it is effective;
- whether it is authoritative, licensed, user-submitted, or inferred;
- who reviewed it and what decision was made;
- what downstream records depend on it.

A source hierarchy must be defined after research. Do not assume an unofficial website, distributor spreadsheet, barcode lookup, or user upload is authoritative for regulatory status.

Before importing a catalogue source, confirm:
- legal right to access, store, normalize, display, and redistribute the data;
- permitted use in commercial software and public search;
- attribution requirements;
- update frequency and correction process;
- API/file availability, rate limits, and service terms;
- source continuity and exit strategy if access ends.

No catalogue source may be represented as an official national registry unless the competent authority and exact source support that claim.

## 5. Data model and versioning

Canonical records should be versioned. A correction creates a new version or an attributable change record; it must not silently rewrite historical transactions.

Tenant catalogue entries link to a canonical item/version while preserving tenant-specific fields. If a canonical identity is merged, split, deprecated, or corrected:
- preserve old identifiers as aliases/redirects with provenance;
- retain historical links from stock, purchase, sale, and recall records;
- identify affected tenant entries and batches;
- create a review/migration plan for ambiguous mappings;
- never automatically merge physical stock from two formerly separate identities;
- provide a rollback/reconciliation path for incorrect catalogue corrections.

Product identifiers must be stable and not derived solely from display names or mutable external codes.

## 6. Barcode and packaging identity

A barcode is an identifier candidate, not a complete proof of medicine identity.

Store:
- code system/type;
- exact value;
- packaging level (unit, inner pack, outer pack, case) where known;
- linked product/presentation/pack;
- source and verification status;
- effective dates and confidence;
- tenant-specific scan observations separately from canonical mapping.

Scanning must show the resolved product identity for confirmation where ambiguity or high risk exists. If the code is unknown, duplicated, malformed, or maps to conflicting records, do not silently choose the first result. Allow a controlled review/correction path.

Unit conversion must be explicit and versioned. A case-to-pack or pack-to-unit conversion must not be guessed. A change to conversion must not rewrite quantities in historical receipts or sales.

## 7. Matching and duplicate-resolution policy

Candidate matching signals:
- exact authoritative registration identifier;
- exact verified barcode at the correct packaging level;
- normalized ingredient/composition;
- strength/concentration and unit;
- dosage form and route;
- brand/manufacturer/authorization holder;
- pack size and presentation.

Matching outcomes:
- **Exact verified match:** all required identity dimensions agree and source supports the match.
- **Candidate match:** likely match but one or more dimensions need review.
- **Ambiguous:** multiple plausible records or conflicting attributes.
- **No match:** no supported canonical identity.
- **Conflicted/deprecated:** source correction or governance action required.

Automated matching may rank candidates but must not merge records or create a clinical equivalence decision. Human review is required for ambiguous/conflicted cases. The reviewer, evidence, decision, and affected links are audited.

## 8. Search and user experience

Search should support:
- generic, brand, and tenant-local names;
- spelling variants and approved transliteration;
- strength, dosage form, concentration, and pack attributes;
- barcode scanning;
- partial/offline catalogue search where a permitted local snapshot exists.

Search results must visually distinguish exact identity from candidates. The user must be able to inspect the identity-defining attributes before selecting an item for stock, purchase, sale, or public publication.

Search ranking must not:
- silently substitute a different strength or form;
- rank a sponsored result above a more accurate medicine identity;
- treat popularity as clinical equivalence;
- hide uncertainty or source status.

If the catalogue lacks a reliable match, the user may create a tenant-local provisional item only under a controlled policy. It must remain visibly provisional and must not be exposed publicly or used in regulated workflows until required review is complete.

## 9. Tenant and branch controls

- Tenants may add local display names and internal codes without changing canonical data.
- Tenant catalogue activation is distinct from public publication.
- Branch price, stock, and availability are separate from canonical medicine identity.
- A tenant can disable a local item while retaining historical transaction references.
- A canonical correction must not silently change the meaning of finalized sales or purchases.
- Public discovery uses an approved public projection and only source-backed fields intended for publication.

## 10. Regulatory classification and safety controls

Regulatory classifications must be jurisdiction-specific, time-bounded, and source-backed. The product must not infer prescription status, controlled category, dispensing authority, advertising eligibility, or legal sale conditions from a free-text label.

Where a classification is uncertain or a source is unavailable:
- mark the value unknown/pending review;
- block only the specific action that requires the classification;
- do not invent a permissive default;
- preserve ordinary unrelated management access;
- record the source and decision needed to resolve the uncertainty.

The current applicable Ethiopian federal and regional rules must be reviewed before enabling regulated classifications or actions. This specification does not itself determine those classifications.

## 11. Catalogue correction and quality workflow

1. A user reports an incorrect or missing product attribute with supporting source evidence.
2. The system records a correction proposal without immediately changing canonical data.
3. A reviewer compares authoritative sources, source terms, and affected records.
4. The reviewer accepts, rejects, requests more evidence, or marks the record conflicted.
5. Accepted corrections create a new version and a change event.
6. The system identifies affected tenant entries, batches, purchase orders, sales, and public projections.
7. High-impact changes require a migration/reconciliation plan and controlled communication.
8. Public projections are refreshed or suppressed as appropriate; private historical records remain traceable.

A user must not be able to approve their own high-impact canonical correction where separation of duties is required.

## 12. Offline and event behavior

- Catalogue snapshots may be cached for authorized local search with version and last-synchronized metadata.
- Offline search must label potentially stale data and must not claim current regulatory status.
- Canonical updates are published as versioned events; clients subscribe and update only affected records.
- Do not poll continuously for catalogue changes.
- A reconnecting client uses an event cursor/replay or authoritative snapshot to recover missed updates.
- Tenant-local draft items may be queued for review, but cannot become canonical or publicly verified solely because they were created offline.
- Historical transaction links retain the canonical version/tenant entry used at the time of the transaction.

## 13. Permissions and audit

Candidate permissions:
- catalogue.read;
- catalogue.search;
- tenant_catalogue.create;
- tenant_catalogue.update;
- tenant_catalogue.disable;
- catalogue_correction.propose;
- catalogue_correction.review;
- catalogue_correction.approve;
- barcode_mapping.propose;
- barcode_mapping.review;
- regulatory_classification.read;
- regulatory_classification.review;
- catalogue.export;
- public_catalogue.publish.

The final permission matrix must define platform-global versus tenant scope, branch scope, approval separation, source-evidence access, re-authentication, audit events, and offline eligibility.

## 14. Required test scenarios

Automated tests must prove:
- different strengths, forms, routes, release types, or pack configurations are not silently merged;
- exact barcode matches respect packaging level and source provenance;
- unknown/conflicting barcodes do not silently select a product;
- unit conversions are explicit and versioned;
- tenant-local labels/prices do not mutate canonical identity;
- canonical correction preserves historical transaction identity and provenance;
- merge/split/deprecation identifies affected stock and requires a safe migration;
- ambiguous matches cannot be used in restricted workflows or public verification claims;
- offline snapshots display last-sync/freshness and recover missed events;
- public catalogue projection excludes private tenant fields;
- revoked reviewer permissions prevent pending correction approval;
- source removal or licence expiry triggers a documented data-use review rather than silent deletion or unauthorized redistribution.

## 15. Open decisions before implementation

1. Which official or licensed medicine catalogue sources can be used in Ethiopia, and under what redistribution terms?
2. Which product categories are in the first release?
3. What are the minimum identity-defining attributes for each supported category?
4. How should products without a reliable canonical match be handled operationally?
5. Which barcode standards and packaging levels are present in target pharmacies?
6. Who owns canonical correction review and what is the SLA?
7. Which data may be shown publicly, and what source/verification wording is permitted?
8. How are Ethiopian language names, transliteration, spelling variants, and scripts normalized?
9. What source update frequency and stale-data behavior are required?
10. What retention and historical-link rules apply when a canonical record is withdrawn or corrected?
11. What product classification evidence is required before enabling a sale/dispensing-related control?
12. Which import formats and existing pharmacy catalogues must be supported?

Resolve source access and licensing before building a catalogue ingestion pipeline. Do not seed production with invented medicine records or present unverified catalogue content as authoritative.
