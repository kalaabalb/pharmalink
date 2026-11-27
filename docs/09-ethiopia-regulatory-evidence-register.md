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

## 4. Regional establishment and professional licensing authorities

**Addis Ababa establishment licensing source:** Addis Ababa Food and Drug Authority (AAFDA) licensing portal  
https://license.aafda.gov.et/

The current portal describes pharmacy and drug-establishment services including new establishment licensing, renewal, address change, technical-manager change, and regional registration. It states that AAFDA licenses establishments and that individual health-professional competency licences are issued by the Addis Ababa Health Bureau.

**Addis Ababa professional licensing source:** Addis Ababa Health Bureau (AAHB) licensing portal  
https://license.aahb.gov.et/

The AAHB portal describes professional competency licensing, renewal, upgrade, regional licence transfer/registration, and related professional-record services. These portals are jurisdiction-specific operational sources; they do not establish the complete process for every Ethiopian region.

**Product implications proposed for review:**
- Model establishment licences/certificates and individual professional competency credentials as different evidence types, with different issuing authorities and verification paths.
- Do not infer that an AAFDA establishment licence verifies every employee, or that an individual's professional licence proves that an establishment is licensed.
- Record the authority, jurisdiction, establishment/person subject, category, scope, validity, source, and verification method. Submission confirmation is not licence approval.
- Support region-specific authorities and processes. Do not hard-code Addis Ababa authorities as if they issue or verify every regional credential.
- Preserve private management access independently from a claim that the establishment is legally authorized to operate. Do not display a legal operating claim until the relevant establishment evidence is verified.

## 5. Ethiopian Standards Institute standards: pharmacy and drug-shop requirements

The Institute of Ethiopian Standards (IES) publishes separate 2024 standards for community pharmacies and drug shops. They must not be treated as interchangeable establishment categories.

### 5.1 ES 7084:2024 — Pharmacy requirements

**IES source:**  
https://www.ethiostandards.org/sites/default/files/2023-11/ES%207084_2024.pdf

The standard's stated scope covers pharmacy practices, premises, professionals, and products for new and existing pharmacies throughout Ethiopia, excluding pharmacy services provided as part of health-facility services. Its text defines the pharmacy/technical-manager/professional concepts and includes establishment competence and professional-practice requirements.

Relevant clauses surfaced for legal review include:
- Section 3 defines a pharmacy professional by reference to a pharmacist, druggist, or pharmacy technician licensed by the appropriate health-professional regulatory organ; it defines the technical manager as a licensed pharmacist designated for responsibility over the pharmacy and named in the certificate of competence.
- Sections 4.1 and 4.4 address a certificate of competence and the requirement that a pharmacy not operate without a registered and licensed pharmacist.
- Section 5.1.2 assigns dispensing responsibilities to pharmacy professionals and includes the dispensing workflow; the OTC exception in the prescription rule must not be assumed to remove professional-scope requirements.
- Sections 5.1.7–5.1.8 address procurement, stock management, receiving, traceability, and records; the standard assigns specific responsibilities to the technical manager or pharmacy professional.
- Section 5.1.13 addresses medicines-waste management and assigns responsibility to pharmacy professionals.

The standard uses the term **certificate of competence** for a permit connected to a person carrying out pharmacy services, and ties the technical-manager designation to that certificate. This is not safely interchangeable with either the establishment licence or the individual's professional competency licence. The exact legal subject, issuer, scope, and relationship of this certificate must be confirmed with the relevant authority and qualified local counsel before the data model treats it as equivalent to either.



**Relevant provisions surfaced for qualified review:** Sections 3–4 address pharmacy, technical-manager, pharmacy-professional and certificate-of-competence concepts and general operating requirements; Section 5 addresses practice, professional dispensing, procurement/stock management, records, and other pharmacy operations. The full standard and applicable regional laws/directives must be reviewed before translating any clause into a product rule.

### 5.2 ES 7085:2024 — Drug-shop requirements

**IES source:**  
https://gov.ethiostandards.org/sites/default/files/2023-12/ES%207085%20Drug%20shop%20requirement_2024.pdf

The standard has a separate scope and definitions for drug shops, including its own technical-manager and professional categories. It addresses establishment competence, professional practice, procurement/stock management, records, and other drug-shop requirements. It is not a substitute for the pharmacy standard.

Relevant clauses surfaced for legal review include:
- Section 3 defines the drug-shop technical manager and pharmacy-professional categories, and defines a certificate of competence for carrying out pharmacy services at community-pharmacy level.
- Sections 4.1 and 4.4 address the certificate of competence and the requirement that a drug shop not operate without a registered and licensed pharmacist, druggist, or level IV pharmacy technician.
- Section 5.1.2 addresses dispensing responsibilities; Sections 5.1.5–5.1.6 address procurement, stock management, records, and supporting documentation.
- Section 5.1.10 addresses medicines-waste management and assigns responsibility to pharmacy professionals, while also requiring staff training and appropriate handling controls.

As with ES 7084, the certificate of competence must be modeled as a distinct, establishment-linked person-authority record until its exact legal nature and issuer are confirmed. Do not collapse it into the establishment licence or professional competency licence.


**Product implications proposed for review:**
- Establishment category must be a first-class, jurisdiction-specific compliance attribute. Do not apply a pharmacy rule set to a drug shop or vice versa.
- Preserve the distinction between a legal establishment/operating requirement, a person's professional competency credential, a software account, and a tenant's management entitlement.
- Software account registration, invitation acceptance, sign-in, and ordinary private management are not themselves proof of authority to operate an establishment or perform a regulated professional act.
- Separate administrative data entry from any professional approval, accountable sign-off, or regulated service that applicable law/standard reserves to a qualified person. Do not let an ordinary POS or inventory permission imply professional authority.
- Keep regulated workflows disabled where the applicable establishment category, professional scope, product classification, or current legal rule is unresolved.
- Verify the current edition/status, applicability, implementing directives, and regional interpretation with qualified Ethiopian reviewers before turning either standard into automated eligibility rules.

## 6. Electronic Transactions Proclamation No. 1205/2020

**Primary-source Gazette copy:** Federal Negarit Gazette, 26th Year No. 57, Addis Ababa, 30 June 2020; Proclamation No. 1205/2020 begins at Gazette page 12,581. The Ministry of Trade and Regional Integration hosts a copy of the Gazette:

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
2. A region-by-region, establishment-category-by-establishment-category, and action-by-action establishment/professional evidence matrix for the launch geography. It must distinguish ordinary sale recording from legally reserved dispensing or other professional acts, identify exact product categories and evidence sources, and state which checks are not required.
3. A product-category and public-display boundary assessment.
4. A written assessment of online ordering, reservation, delivery, and payment model legality.
5. A data protection impact and cross-border processing assessment.
6. A records, retention, inspection, and reporting matrix.
7. Written review of public claims, advertising, and verification language.

## 9. Source quality and caveats

The Food and Medicine Administration Proclamation PDF and Ministry of Justice law record are primary official sources. The Addis Ababa licensing portal is an official regional operational source. The drug-shop standard is surfaced through the Ethiopian Standards Institute portal and must be checked for current edition/status. The Electronic Transactions Proclamation is linked to the Ministry of Justice official law registry in Section 6; the Gazette-hosted copy and current amendments/implementing instruments must still be checked during clause-level legal review.

This register is an initial evidence map, not a complete regulatory survey. No legal conclusion should be inferred from an omitted topic.

## 10. Platform access versus regulated activity — product interpretation

The platform owner's requirement is that Pharmalink must not require an individual's professional licence merely to register, sign in, join a pharmacy organization, or use ordinary management functions. A pharmacy may use Pharmalink privately without publishing or selling through the public-facing experience.

This product requirement is distinct from legal requirements governing the pharmacy establishment or a regulated professional act. The application must not represent platform access as proof of legal establishment status, and it must not authorize an action that applicable law reserves to a qualified professional. Where a specific feature requires establishment evidence or a professional credential, gate only that feature or action and explain why the evidence is requested. Do not make the credential a blanket platform-entry requirement.

The exact activities requiring a credential, the relevant establishment categories, and the acceptable evidence sources remain subject to current federal/regional legal review. This register does not infer that all retailers are exempt from professional or establishment obligations; it defines the product's access boundary.


## 11. Privacy implementation specification

The Personal Data Protection Proclamation evidence recorded above is mapped to product and operational controls in [Specification 22](22-privacy-and-personal-data-lifecycle-specification.md). That specification remains Draft until qualified Ethiopian counsel validates legal roles, lawful bases, data-subject rights, retention, breach duties, and cross-border processing for the actual Pharmalink service model.
