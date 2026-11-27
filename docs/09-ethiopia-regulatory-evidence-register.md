# 09 — Ethiopia regulatory evidence register

**Status:** Draft — initial source review, not legal advice  
**Last reviewed:** 2026-11-27  
**Purpose:** Record what primary sources currently support, what they do not establish, and what must be reviewed before regulated functionality is implemented.

## 1. Evidence standard

This register separates:
- what a cited legal source expressly states;
- the product control proposed in response;
- interpretation that requires qualified Ethiopian legal/regulatory review.

A source being official does not make this document a legal opinion. The current consolidated status, amendments, implementing regulations/directives, and regional application must be checked before release.

## 2. Food and Medicine Administration Proclamation No. 1112/2019

**Primary source:** Ethiopian Food and Drug Authority-hosted Federal Negarit Gazette PDF  
https://www.efda.gov.et/wp-content/uploads/2020/06/Food-and-Medcine-Proclamation-1112.pdf

Relevant provisions identified in the published English text:

- The proclamation defines medicine and pharmacy professional, among other regulated terms.
- Article 26 addresses storage, transport, and sale of medicine and medical devices.
- Articles 31–33 address special controls and records for narcotic and psychotropic medicines and precursor chemicals.
- Article 35 addresses medicine prescribing and prescription paper.
- Article 36 addresses dispensing, over-the-counter medicines, professional scope, prescription-related records, and prohibitions concerning expired, damaged, substandard, diverted, or illegal medicine.
- Article 60 includes restrictions concerning medicine and medical-device advertising.

**Product implications proposed for review:**
- Do not represent the platform as a prescribing, diagnosing, or independent dispensing authority.
- Regulated professional actions must be attributable to an appropriately verified professional and constrained by applicable scope.
- The product must not silently enable sale/dispensing of expired, damaged, substandard, diverted, or illegal stock.
- Prescription-related record handling, regulated categories, advertising, and any public-facing ordering flow require a separate reviewed specification.
- Any controlled-category workflow must remain out of scope until the exact current requirements and permissions are reviewed by qualified local experts.

**Not established by this initial review:** the full set of current implementing directives, exact establishment-specific permissions, every record-retention period, and the legal status of all proposed online marketplace, reservation, or delivery models.

## 3. Personal Data Protection Proclamation No. 1321/2024

**Primary source:** Ethiopian Ministry of Justice law record  
https://justice.gov.et/en/law/personal-data-protection-proclamation/

The Ministry of Justice record identifies Proclamation No. 1321/2024, dated 24 July 2024, and marks it **In Force**. The official law portal also provides a download link.

**Product implications proposed for review:**
- Map each processing purpose and identify the responsible data-protection roles before collecting personal data.
- Minimize collection and separate sensitive records from general business operations.
- Define notices, lawful basis, consent where applicable, access controls, retention, data-subject request handling, incident response, processor arrangements, and cross-border transfer review.
- Do not add patient/customer health records to the first release without a dedicated legal, privacy, security, and clinical governance review.
- Treat hosting location and third-party processing as a legal assessment, not as a guessed localization rule.

**Next research required:** review the complete proclamation and implementing instruments clause by clause, including lawful bases, sensitive data, minors, transparency, purpose limitation, rights, security, breach obligations, processor/controller duties, and international transfers.

## 4. Regional licensing and establishment verification

**Primary source surfaced:** Addis Ababa Food and Drug Authority licensing portal  
https://license.aafda.gov.et/

The portal describes establishment licensing services, including new pharmacy licensing, renewal, address change, technical-manager change, and regional registration. It also distinguishes establishment licensing from individual professional competency licensing.

**Product implications proposed for review:**
- Model establishment licences and individual professional credentials as different evidence types.
- Store issuing authority, licence identifier, evidence, effective/expiry dates where supplied, and verification method.
- Do not treat an uploaded document or a submitted application as an approved licence.
- Support region-specific authorities and workflows rather than assuming one federal portal covers every establishment.
- The portal is an operational source for Addis Ababa; it does not establish the complete licensing rules for all Ethiopian regions.

## 5. Ethiopian Standard ES 7085:2024 — Drug Shop Requirements

**Source surfaced:** Ethiopian Standards Institute standards portal PDF  
https://gov.ethiostandards.org/sites/default/files/2023-12/ES%207085%20Drug%20shop%20requirement_2024.pdf

The surfaced standard identifies requirements concerning drug-shop practices, premises, professionals, products, equipment, and materials, and describes applicability to new and existing drug shops.

**Product implications proposed for review:**
- Establishment type must be a first-class configuration and compliance attribute.
- Do not assume a single role model or feature set applies to every pharmacy and drug-shop category.
- Obtain the full standard and assess its current status, applicability, and related directives with qualified local reviewers before turning it into validation rules.

## 6. Electronic Transactions Proclamation No. 1205/2020

**Primary-source Gazette copy:** Federal Negarit Gazette, 26th Year No. 57, Addis Ababa, 30 June 2020; Proclamation No. 1205/2020 begins at Gazette page 12,581. The Ministry of Trade and Regional Integration hosts a copy of the Gazette:
https://www.motri.gov.et/sites/default/files/resource/%E1%8B%A8%E1%8A%A4%E1%88%8C%E1%8A%AD%E1%83%...%20.pdf

**Official Ministry of Justice law-register copy:**  
https://laws.moj.gov.et/registries/view/1414/am

The Gazette text identifies Proclamation No. 1205/2020 as the Electronic Transaction Proclamation and includes definitions for electronic commerce, electronic commerce operators, electronic commerce platform operators, electronic documents, electronic records, and electronic signatures. The Ministry of Justice registry hosts the Gazette record. This replaces the earlier secondary-repository-only citation.

**Product implications proposed for legal review:**
- Preserve electronic records and transaction evidence with integrity, access control, provenance, and reproducible export.
- Do not treat a typed name, checkbox, OTP, or click as legally sufficient for every kind of signature or regulated approval without reviewing the applicable legal requirements and evidence standard.
- Evaluate Pharmalink's legal role against the actual feature set. Private pharmacy-management software, public discovery, public enquiries, and any later marketplace/trading workflow may raise different questions; do not assume that the product's label determines its legal classification.
- Before enabling public orders, matching that facilitates trading, payment collection, or other commerce, obtain a written assessment of the obligations applicable to the actual platform/operator model.
- Review current implementing instruments and any subsequent amendments before relying on this proclamation for a production flow.

**Not established by this initial review:** whether any proposed Pharmalink feature falls within a particular operator category, whether a particular electronic consent/signature mechanism is sufficient for a specific record, or the complete current set of implementing rules and sector-specific obligations. These require clause-level review by qualified Ethiopian counsel.

## 7. Initial regulatory product boundaries

Until review is complete:
- no diagnosis, prescribing, treatment recommendation, or automated clinical decision-making;
- no independent authorization of regulated dispensing;
- no unreviewed medicine advertising or promotion;
- no public promise of guaranteed availability;
- no online ordering, reservation, or delivery flow presented as legally approved;
- no patient-health-record feature without a dedicated approval gate;
- no claim that Pharmalink verifies a licence unless the verification source and method are explicit.

## 8. Required legal review deliverables

Before implementing regulated workflows, obtain:
1. A current federal law and implementing-directive matrix.
2. A region-by-region establishment and professional verification matrix for the launch geography.
3. A product-category and public-display boundary assessment.
4. A written assessment of online ordering, reservation, delivery, and payment model legality.
5. A data protection impact and cross-border processing assessment.
6. A records, retention, inspection, and reporting matrix.
7. Written review of public claims, advertising, and verification language.

## 9. Source quality and caveats

The Food and Medicine Administration Proclamation PDF and Ministry of Justice law record are primary official sources. The Addis Ababa licensing portal is an official regional operational source. The drug-shop standard is surfaced through the Ethiopian Standards Institute portal and must be checked for current edition/status. The electronic-transactions source above is secondary and must be replaced by an official Gazette copy before reliance.

This register is an initial evidence map, not a complete regulatory survey. No legal conclusion should be inferred from an omitted topic.

## 10. Platform access versus regulated activity — product interpretation

The platform owner's requirement is that Pharmalink must not require an individual's professional licence merely to register, sign in, join a pharmacy organization, or use ordinary management functions. A pharmacy may use Pharmalink privately without publishing or selling through the public-facing experience.

This product requirement is distinct from legal requirements governing the pharmacy establishment or a regulated professional act. The application must not represent platform access as proof of legal establishment status, and it must not authorize an action that applicable law reserves to a qualified professional. Where a specific feature requires establishment evidence or a professional credential, gate only that feature or action and explain why the evidence is requested. Do not make the credential a blanket platform-entry requirement.

The exact activities requiring a credential, the relevant establishment categories, and the acceptable evidence sources remain subject to current federal/regional legal review. This register does not infer that all retailers are exempt from professional or establishment obligations; it defines the product's access boundary.


## 11. Privacy implementation specification

The Personal Data Protection Proclamation evidence recorded above is mapped to product and operational controls in [Specification 22](22-privacy-and-personal-data-lifecycle-specification.md). That specification remains Draft until qualified Ethiopian counsel validates legal roles, lawful bases, data-subject rights, retention, breach duties, and cross-border processing for the actual Pharmalink service model.
