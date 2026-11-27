# 22 — Privacy and personal-data lifecycle specification

**Status:** Draft — Ethiopian privacy counsel review, controller/processor role assessment, and customer research required  
**Priority:** Critical — pharmacy operations can process information that reveals highly sensitive personal circumstances  
**Scope:** Personal-data inventory, purpose limitation, lawful basis, transparency, minimization, access, correction, retention, deletion, data-subject requests, processor controls, cross-border transfers, privacy incidents, and privacy-by-design acceptance.

## 1. Purpose and legal interpretation boundary

This specification turns privacy principles into product and operational requirements. It is not a legal opinion and does not itself determine the lawful basis, statutory role, retention period, or cross-border transfer permission for any processing activity.

The Ethiopia regulatory evidence register identifies the Personal Data Protection Proclamation No. 1321/2024 as a key legal source. Qualified Ethiopian counsel must map the current law, implementing rules, regional requirements where relevant, and the actual Pharmalink service model to each processing activity before production.

Do not assume that every Pharmalink customer is a controller, that Pharmalink is always a processor, or that a label in a contract determines the legal role. Roles must be assessed by the actual purposes and means of each processing activity.

## 2. Privacy design principles

1. Collect the minimum personal data needed for a defined, documented purpose.
2. Do not collect data merely because a future feature might use it.
3. Do not infer health status, diagnosis, treatment, religion, income, or other sensitive attributes from searches or pharmacy transactions for advertising or unrelated profiling.
4. Keep public discovery usable without a public-user account wherever possible.
5. Keep private pharmacy operations separate from public discovery and platform analytics.
6. Do not expose patient/customer identities, prescription details, or transaction histories in public profiles, search results, URLs, analytics, or push notifications.
7. Provide clear notices before data collection and before material changes of purpose.
8. Define retention at the record-class and purpose level; do not retain everything forever by default.
9. Restrict internal access by purpose, role, scope, and time; log sensitive access.
10. Make privacy rights and deletion workflows operational, not merely policy-page promises.
11. Preserve records that must lawfully be retained, while restricting their use and access.
12. Treat backups, logs, exports, support tickets, analytics, and third-party processors as part of the data lifecycle.

## 3. Data inventory and classification

Maintain a data inventory with:
- data element/category;
- data subject;
- source and collection method;
- purpose and system/process;
- legal basis/consent assessment;
- controller/processor/other role assessment;
- recipients and subprocessors;
- storage/processing location;
- sensitivity and risk classification;
- access roles;
- retention trigger and period;
- deletion/anonymization method;
- rights/notice requirements;
- security controls;
- owner and review date.

Candidate classifications:

| Class | Examples | Minimum handling |
|---|---|---|
| Public | Opted-in pharmacy profile fields | Publication allowlist, correction and takedown controls |
| Internal business | Tenant settings, supplier catalogue, staff role names | Tenant/branch access controls, audit |
| Confidential | Financial records, supplier terms, customer contact records | Least privilege, encryption, export controls |
| Sensitive personal | Identity evidence, credentials, account recovery data | Purpose-bound access, strong audit, minimized retention |
| Potentially health-revealing | Prescription/dispensing records, medicine purchase history, health-related enquiries/searches | Highest practical minimization, strict purpose/access, no advertising use, legal review |
| Security secret | Password hashes, API keys, session tokens, signing keys | Secret management, no ordinary logging, rotation/revocation |

The classification of a field depends on context and applicable law. A medicine name may be public catalogue information in one context and health-revealing personal data when linked to an identifiable customer.

## 4. Processing purposes and data minimization

Every processing purpose must be documented before implementation. Candidate purposes include:
- account authentication and recovery;
- organization membership and role enforcement;
- pharmacy operations and required transaction records;
- security, fraud prevention, and abuse response;
- customer support;
- billing and service administration;
- public profile and availability publication;
- optional user enquiry delivery;
- service reliability and aggregated product analytics;
- legally required recordkeeping and regulatory response.

For each purpose define required fields, optional fields, prohibited fields, retention, recipients, and access.

Do not use operational pharmacy data for:
- targeted advertising based on medicines or inferred health interests;
- unrelated user profiling;
- sale or sharing of identifiable transaction histories;
- training general-purpose models without a separately reviewed legal basis, contractual controls, and appropriate notice;
- ranking or employment decisions unrelated to the original purpose.

AI features, if later approved, require their own purpose, data-flow, retention, access, and human-review assessment.

## 5. Public discovery privacy

Public discovery should not require an account for ordinary search unless a separately justified capability needs one.

For public search:
- do not collect a person's name, phone, or account identity by default;
- do not expose medicine search terms to pharmacies;
- do not attach precise location to search logs unless necessary and lawfully justified;
- provide manual locality search if location permission is denied;
- use coarse/ephemeral location where possible;
- avoid persistent search histories by default;
- do not use medicine searches for ad targeting or health-interest inference;
- keep security logs separate from product analytics and apply different retention;
- do not include sensitive terms or coordinates in URLs, referrers, notification payloads, or analytics event names.

If public enquiry is enabled, show a just-in-time notice describing the recipient, data fields, purpose, and expected handling. Avoid prompting users to provide diagnoses or unnecessary medical details.

## 6. Pharmacy customer, prescription, and transaction data

The first release must determine whether it needs to store identifiable end-customer records, prescription details, or dispensing records at all. Do not collect them simply because a pharmacy POS exists.

If a feature requires personal or health-related data:
- document the exact business/legal purpose;
- identify the responsible legal entity and roles;
- identify the data subject and expected recipients;
- determine whether the data is necessary or whether a non-identifying transaction record is sufficient;
- define notice, lawful basis, access, correction, retention, and deletion rules;
- restrict staff access to the minimum required for their task;
- avoid copying data into logs, analytics, notifications, or support tools;
- encrypt and audit access;
- test tenant/branch isolation and export behavior.

A sale record may be operationally necessary without attaching a named customer. The product must not make customer identity mandatory unless a validated legal/business requirement exists.

## 7. Notices, consent, and preference management

Notices must be clear, accessible, localized, and shown at the relevant point of collection or use.

Separate:
- account/service terms;
- privacy notice;
- optional public publication consent/configuration;
- optional enquiry/contact disclosure;
- optional marketing consent;
- optional analytics or research consent where required;
- provider-specific notices where data is transferred to an external service.

Do not bundle optional marketing or public publication consent into general account registration. Consent must be recorded with subject, purpose, scope, notice version, timestamp, method, and withdrawal event where applicable.

A withdrawal must stop future optional processing/publication where feasible. It does not automatically erase records that must lawfully be retained; those records must be restricted to the remaining lawful purpose.

## 8. Data-subject rights workflow

The exact rights, exemptions, identity verification, deadlines, and appeal procedures must be mapped from applicable Ethiopian law by qualified counsel.

The product must be capable of supporting the approved rights workflow, potentially including:
- request access or confirmation of processing;
- obtain a usable copy;
- correct inaccurate personal data;
- request deletion or restriction where applicable;
- object or withdraw consent where applicable;
- request information about recipients and processing;
- complain or appeal through the appropriate route.

Workflow:
1. Receive the request through a documented channel.
2. Verify the requester proportionately without collecting excessive new identity data.
3. Determine whether Pharmalink, the customer organization, or another party is responsible for the record.
4. Identify all relevant systems, processors, backups, exports, and legal holds.
5. Assess applicable rights, exceptions, retention duties, and third-party impacts.
6. Record approval, partial fulfillment, refusal, or need for clarification with reasons.
7. Execute access/correction/deletion/restriction through controlled jobs.
8. Verify completion across active systems and applicable processors.
9. communicate the outcome within the legally approved timeline.
10. Preserve a minimal request audit record without retaining unnecessary copies of the requested data.

A deletion request must not silently remove required financial, pharmacy, safety, security, or legal records. Where retention is required, restrict access and use to that purpose and explain the limitation as permitted by law.

## 9. Retention, deletion, and anonymization

Create a retention schedule by data class and legal/business purpose. Do not use one global retention period.

For every class define:
- retention trigger (creation, closure, last activity, contract end, legal event);
- minimum and maximum retention period after legal review;
- legal-hold conditions;
- deletion/anonymization method;
- backup expiry behavior;
- processor deletion confirmation;
- audit evidence of completion.

Candidate classes requiring separate schedules:
- account and authentication records;
- membership and access history;
- establishment/professional verification evidence;
- pharmacy operational and financial records;
- prescription/dispensing/customer records if enabled;
- public profile and availability projections;
- public search/security logs;
- enquiries and support tickets;
- billing and payment records;
- analytics/telemetry;
- uploaded files and exports;
- backups and disaster-recovery snapshots.

Anonymization must be tested for re-identification risk. Removing a name while retaining unique combinations of branch, date, product, and transaction may not produce anonymous data.

## 10. Processor and third-party data sharing

Every external provider must be governed by [Specification 21](21-external-integrations-and-provider-lifecycle-specification.md).

Before sending personal data to a provider, document:
- purpose and minimum fields;
- recipient and subprocessor;
- contractual role and instructions;
- processing/storage locations;
- security measures;
- retention/deletion;
- breach notification and cooperation;
- data-subject request support;
- transfer mechanism/legal basis where cross-border;
- audit/assurance rights;
- provider exit and data return/deletion.

Do not send sensitive pharmacy/customer data to analytics, crash reporting, AI, messaging, or support vendors unless specifically approved and minimized.

## 11. Cross-border transfers and hosting location

Hosting region, backups, support access, analytics, messaging, payment, and monitoring can all create cross-border data flows.

Before production:
- map each data flow and physical/administrative access location;
- identify legal restrictions and permitted transfer mechanisms;
- assess provider/subprocessor locations and remote support access;
- document safeguards, approvals, and notices;
- determine whether local hosting or localization is required for any data category;
- ensure contracts and technical configuration match the approved architecture.

Do not assume that selecting an Ethiopian cloud region means all data remains in Ethiopia. Do not assume cross-border processing is prohibited or permitted without legal analysis.

## 12. Privacy incident and breach response

Privacy incidents include unauthorized access, disclosure, loss, alteration, deletion, misdelivery, credential compromise, excessive internal access, provider breach, and unintended public exposure.

Response workflow:
1. detect and report through a documented channel;
2. preserve evidence and relevant logs;
3. contain access/exposure without destroying forensic evidence;
4. identify affected data, subjects, systems, providers, regions, and time window;
5. assess likely impact and ongoing risk;
6. involve privacy/security/legal owners;
7. determine notification duties and deadlines under applicable law;
8. notify affected parties/authorities where legally required and approved;
9. remediate root cause and rotate/revoke credentials as needed;
10. record decisions, communications, and lessons learned;
11. verify controls and monitor for recurrence.

Do not make public assurances or delay legally required notifications while waiting for a complete root-cause analysis. Maintain a decision log and escalation contacts.

## 13. Privacy engineering and access controls

Required controls:
- field-level minimization and allowlisted API response schemas;
- server-side tenant/branch authorization;
- purpose-bound support and verification access;
- encryption in transit and at rest;
- secure secret and key management;
- sensitive document access logging;
- redaction of logs and error responses;
- no sensitive data in URLs or push payloads;
- safe export and temporary file lifecycle;
- deletion/restriction propagation to indexes, caches, processors, and public projections;
- privacy tests in CI and release acceptance;
- privacy review for new fields, integrations, AI features, analytics, and public capabilities.

Sensitive data should not be copied into multiple services merely for convenience. Keep a clear source of truth and a documented lifecycle.

## 14. Required test scenarios

Automated and operational tests must prove:
- ordinary public search works without account registration;
- search terms and precise location are not exposed to pharmacies;
- public discovery does not create persistent health-interest profiles;
- customer identity is not mandatory for a sale unless the approved workflow requires it;
- sensitive records are not included in logs, analytics, URLs, or push previews;
- a user cannot access another tenant's personal or pharmacy customer data;
- support access is purpose-bound, time-limited, and audited;
- optional publication and marketing consent are separate from account registration;
- withdrawal of public publication removes public projections but preserves required private records;
- data-subject request workflow identifies records across active systems and processors;
- deletion jobs honor legal holds and approved retention schedules;
- backups expire according to the approved schedule and cannot be accessed by ordinary users;
- cross-border providers cannot be enabled without an approved data-flow record;
- provider deletion requests and confirmations are traceable;
- incident workflow captures scope, decisions, and legally required notifications;
- anonymized analytics cannot reasonably be joined back to an identifiable person under the approved threat model.

## 15. Open decisions before implementation

1. Which customer, patient, staff, owner, supplier-contact, and public-user data is actually necessary for the first release?
2. Does the first release store identifiable customer, prescription, or dispensing records at all?
3. For each processing purpose, who is controller, processor, joint controller, or another legally defined party?
4. What lawful basis and notice are required for each data category and purpose?
5. Which records are health data or otherwise specially protected under applicable law?
6. What are the statutory retention periods for operational, financial, prescription, dispensing, security, and verification records?
7. What data-subject rights, exemptions, response deadlines, and appeal routes apply?
8. Which hosting, backup, support, analytics, messaging, and payment regions are legally acceptable?
9. Which cross-border transfers require approval or safeguards?
10. What privacy incident notification duties and timelines apply?
11. Which analytics are necessary, and can they be aggregated or processed without personal data?
12. What contract terms and processor controls are required for each external provider?
13. What evidence is needed to prove deletion/restriction has propagated to backups, caches, indexes, and processors?
14. Which privacy officer/legal owner will approve new processing purposes and material product changes?

Do not enable sensitive customer/prescription records, health-related profiling, or cross-border provider processing until these questions are resolved and documented by qualified reviewers.
