# 19 — Public discovery, pharmacy profiles, and availability specification

**Status:** Draft — Ethiopian legal/regulatory review, pharmacy field validation, and usability testing required  
**Priority:** High — public information must be accurate, privacy-preserving, and honest about freshness  
**Scope:** Public pharmacy discovery, pharmacy profiles, medicine/product search, public availability signals, publication controls, freshness, moderation, and public-user interactions. This specification does not authorize online sale, reservation, dispensing, delivery, diagnosis, or prescribing.

## 1. Product boundary

Public Discovery is an optional platform surface. A pharmacy can use private management features without publishing a profile, product, stock status, or contact information through Pharmalink.

Public publication is independently controlled at organization, branch, and capability/field level. Disabling public participation must not:
- disable private management or POS;
- delete private catalogue, stock, purchasing, sales, or financial records;
- erase audit history;
- interrupt lawful record export;
- require a pharmacy to continue selling through the platform.

A public listing is not proof that a pharmacy is legally authorized to operate, that a product is registered, that a medicine is appropriate for a person, or that stock will still be available when the person arrives.

The initial discovery service is informational. It does not create a reservation, sale, prescription, dispensing relationship, payment obligation, delivery promise, or guaranteed stock hold.

## 2. Public user goals and prohibited assumptions

Candidate public-user goals:
- find a pharmacy by name or approximate location;
- view pharmacy-published contact, location, opening hours, and service information;
- search for a medicine/product identity using safe, accurate identity fields;
- see a pharmacy's recent availability signal where the pharmacy has explicitly enabled it;
- contact the pharmacy independently through an enabled contact channel;
- report inaccurate, unsafe, misleading, or outdated public information.

The platform must not assume that:
- a search indicates medical need or a diagnosis;
- a searched medicine is prescribed to the user;
- a pharmacy's availability signal guarantees stock at arrival;
- a public user is entitled to purchase a restricted product;
- the nearest result is the best or legally suitable provider;
- a profile badge means every staff member is professionally credentialed;
- a product listing is an advertisement approved by a regulator.

Do not use search history to infer health status, build advertising profiles, or expose a person's search terms to pharmacies.

## 3. Publication opt-in and control hierarchy

Public publication requires an explicit authorized action and a recorded consent/configuration state. Use separate controls for:
- organization public presence;
- branch public presence;
- branch address/location;
- contact methods;
- opening hours and temporary closures;
- service descriptions;
- product/medicine catalogue visibility;
- availability signal publication;
- photos and other media;
- any future ordering or commerce capability.

Default state for a new tenant/branch is private. A branch must not become publicly discoverable merely because it exists in the management database.

The system records who enabled each capability, when, the scope, the terms/version shown, and any later changes. An owner may delegate publication management only through an explicit permission. The platform must not treat a staff member's ordinary management login as consent to publish.

Pausing or disabling one public capability must not implicitly disable unrelated private capabilities. A tenant may pause one branch while leaving another branch public, subject to legal and operational constraints.

## 4. Pharmacy and branch public profile

Candidate profile fields:
- public display name and any approved trading name;
- branch-specific address and map point;
- contact phone and selected contact methods;
- opening hours by day, exceptions, and temporary closure;
- public service description;
- accessibility and language information where confirmed;
- profile image/logo uploaded by the tenant with rights confirmation;
- public verification/status wording;
- last profile confirmation/update date where useful.

Do not expose:
- private owner/staff contact details;
- internal branch notes, staff schedules, supplier details, security settings, or financial data;
- precise internal stock quantities by default;
- patient/customer identities or transaction history;
- establishment documents or professional credential documents;
- device identifiers, private location data, or security metadata.

Address and map coordinates require explicit confirmation by an authorized tenant user. Location precision should be appropriate to the public purpose. Do not expose a user's current precise location to a pharmacy or third party.

## 5. Verification claims and trust indicators

Use distinct, precise statuses. Candidate statuses:
- **Profile submitted:** tenant has supplied public details; not independently verified.
- **Identity checked:** the platform has completed a defined identity check for the entity/person stated.
- **Establishment evidence reviewed:** specified establishment evidence has been reviewed for a stated jurisdiction, category, and validity period.
- **Credential evidence reviewed:** a specific person's credential has been reviewed for the stated scope and validity period.
- **Publication paused:** one or more public capabilities are unavailable.
- **Evidence expired / review required:** a time-bounded check is no longer current or requires reassessment.

Do not collapse these into a single “verified pharmacy” flag. Public wording must state what was checked, by whom (where appropriate), scope, date, and limitations. The platform must not claim regulator endorsement or live registry validation unless a current authoritative source and legal basis support that exact claim.

A person's professional credential is not a blanket requirement for a pharmacy's public profile. A professional credential badge, where legally appropriate, applies to that named individual and specific claim—not to the whole organization or every staff member.

An establishment's legal authority to operate and a person's professional authority are separate from Pharmalink account eligibility and software entitlement. Exact verification requirements require current Ethiopian federal and regional legal review.

## 6. Medicine/product public catalogue

A tenant may independently opt into publishing selected catalogue entries. Publication is not automatic from private catalogue creation, purchase, receipt, or POS use.

Each public product record must reference a reviewed canonical product identity and contain only fields approved for publication, potentially:
- public product name and brand;
- strength, dosage form, and presentation;
- manufacturer/authorization holder where sourced and permitted;
- source-backed product identifier/status where legally appropriate;
- public description that does not make unsupported clinical claims;
- the publishing pharmacy/branch and its status.

Do not publish a provisional or ambiguous tenant-local item as a confirmed canonical medicine identity. Do not infer a product's regulatory status, prescription category, substitution equivalence, advertising eligibility, or clinical suitability from text or a model-generated description.

Medicine advertising, promotion, price publication, and product availability displays may be restricted by law or category. These controls must remain disabled until the applicable federal/regional rules and permitted categories are reviewed by qualified Ethiopian counsel/regulatory experts. A technical ability to publish is not legal permission to publish.

## 7. Availability signal model

Public availability is a time-bounded, pharmacy-confirmed signal, not a reservation or guarantee.

Candidate states:
- **Recently confirmed:** an authorized source confirmed eligible stock within the configured freshness window.
- **Limited:** a pharmacy-approved low-availability band may be shown only if reliable and legally permitted.
- **Unavailable:** the pharmacy has explicitly confirmed no available stock for the relevant product/branch.
- **Unknown:** no reliable current signal exists.
- **Stale:** a previously confirmed signal exceeded its freshness window.
- **Not published:** the pharmacy has not opted to publish availability.

Do not treat missing stock data as “unavailable.” Do not treat a stale positive value as current. Public exact quantities are off by default. The product owner must approve any availability bands only after pharmacy research, privacy review, and legal review.

Each signal should record:
- tenant and branch;
- exact canonical product/presentation identifier and version;
- source method (manual confirmation, validated stock-derived projection, or approved integration);
- confirmation actor/system;
- confirmed-at time and server-recorded time;
- freshness deadline/policy version;
- status and permitted display band;
- source stock snapshot/version or audit reference;
- publication consent/configuration state.

A signal derived from stock must respect batch eligibility, expiry, hold/recall status, branch scope, unit conversions, pending transactions, and concurrent reservations if reservations are ever introduced. Only eligible, sellable stock can contribute. The derivation must not expose exact internal quantities.

## 8. Freshness, stale transitions, and recovery

Freshness thresholds are product decisions to validate by pharmacy type, connectivity, update method, and user expectation. Do not hard-code a threshold without evidence.

Rules:
1. Every published positive signal has a server-side expiry/deadline.
2. If no qualifying confirmation arrives by the deadline, the public projection changes to Stale or Unknown automatically.
3. Offline local changes do not refresh public freshness until accepted by the server.
4. Reconnecting devices synchronize authoritative changes and recover missed events before updating the projection.
5. A delayed event must not overwrite a newer aggregate version or later confirmation.
6. If the event pipeline or projection is unhealthy, the platform must fail closed to Stale/Unknown rather than keep presenting an old positive signal as current.
7. The public UI displays a human-readable last-confirmed time or freshness label appropriate to the user's locale.
8. Manual confirmation must be attributable to an authorized branch user and auditable.

A scheduled expiry process may be used to transition freshness state. This is not a reason to poll continuously for unchanged data. Use durable scheduling/expiry jobs and event subscriptions for updates.

## 9. Search, ranking, and result explanation

Medicine search must use the canonical identity rules in [Specification 16](16-medicine-catalogue-and-product-identity-specification.md). Pharmacy search may use confirmed public names, address, locality, and explicitly published service fields.

Search behavior:
- distinguish exact product identity from candidate/ambiguous matches;
- show strength, form, and presentation where needed to prevent misleading matches;
- do not silently substitute a different strength, formulation, or product;
- provide clear “no reliable match” behavior;
- do not treat spelling similarity as clinical equivalence;
- localize names and aliases with provenance;
- provide a way to report a wrong or misleading result.

Ranking must be explainable and must not imply clinical suitability. Candidate ranking factors include text relevance, approximate distance where the user voluntarily supplies location, profile completeness, and freshness of the published signal. Any weighting must be tested for fairness and manipulation.

Do not sell ranking placement or allow paid promotion to override identity accuracy, safety controls, or legal restrictions. Sponsored placement is excluded until a separate legal, product, and transparency review.

A public user may search without creating an account. Account creation, if later needed for a separate capability, must not be forced for ordinary discovery.

## 10. Location privacy

- Ask for device location only when the user invokes a location-dependent feature.
- Explain why location is requested and provide a manual locality/address search alternative.
- If location permission is denied, discovery remains usable without precise location.
- Prefer coarse or on-device distance calculation where practical; send only the minimum location precision needed.
- Do not retain precise user location or location-search history by default.
- Do not expose a user's search, device location, IP-derived location, or identity to pharmacies.
- Define retention and deletion behavior for server logs, abuse prevention, and security separately and transparently.
- Never place precise location or medicine search terms in push-notification payloads, analytics identifiers, or public URLs.

## 11. Contact and user enquiry workflow

If public contact is enabled, define exactly which channel is used and what information is disclosed. A simple click-to-call or external contact action may not require Pharmalink to store the user's identity or message.

If an in-platform enquiry is introduced:
- state what personal information is collected and why;
- collect the minimum required fields;
- warn users not to submit unnecessary sensitive medical information;
- separate contact consent from marketing consent;
- define delivery, response, abuse-reporting, retention, deletion, and access rules;
- do not turn an enquiry into a reservation or sale unless separately approved;
- do not expose enquiries to staff who lack the relevant branch permission.

Do not promise response times unless the pharmacy explicitly adopts and monitors them.

## 12. Public reports, moderation, and appeals

Provide public reporting categories such as:
- incorrect pharmacy details;
- closed or relocated branch;
- misleading profile or verification claim;
- incorrect product identity;
- stale or misleading availability;
- prohibited or unsafe content;
- privacy or impersonation concern.

A report is an allegation, not proof. The moderation workflow should:
1. accept the report with minimal required information;
2. issue a case ID and acknowledgement;
3. triage severity and immediate public risk;
4. preserve relevant evidence and access logs;
5. apply proportionate temporary restrictions only where justified;
6. request information from the tenant where appropriate;
7. record a reasoned decision and reviewer;
8. notify affected parties where lawful and safe;
9. provide an appeal/correction route;
10. retain the decision history and review deadline.

Urgent safety or impersonation risks may justify temporary suppression while investigated. Suppression of public content must not delete private management records or be represented as a legal finding. Platform moderation is distinct from a regulator's enforcement action.

Define appeal eligibility, time windows, independent reviewer, evidence standard, decision SLA, and escalation process before launch.

## 13. Public data architecture and projection

The public discovery service should read from a deliberately minimized public projection, not query private operational tables directly.

Projection requirements:
- explicit allowlist of public fields;
- source tenant/branch and publication configuration;
- source aggregate/version and last update;
- verification claim and evidence freshness metadata;
- public availability status/freshness;
- moderation/suppression state;
- deletion/unpublish tombstone;
- locale/search normalization metadata.

Every projection update is driven by committed domain events and idempotent consumers. Projection rebuilds must be deterministic and must not resurrect paused, deleted, or suppressed content.

Public API authorization must be separate from tenant management authorization. Public endpoints expose only public projection fields. Administrative/support access to private source records is separately authorized and audited.

## 14. Abuse, reliability, and service protection

Threats include pharmacy impersonation, false stock claims, catalogue poisoning, scraping, enumeration, denial of service, fake reports, malicious links/uploads, location harvesting, and attempts to infer customer health interests.

Required controls:
- rate limits and abuse detection on public search, profile, report, and media endpoints;
- input validation and output encoding;
- safe media type/size processing and malware scanning where supported;
- no public enumeration of private tenant IDs or internal stock records;
- monitoring for automated scraping and unusual query patterns;
- bot/abuse controls that preserve accessibility;
- audit of publication, verification, and moderation changes;
- incident playbooks for impersonation, misleading availability, data exposure, and compromised tenant accounts.

Availability of the public discovery service must not be coupled to the ability of a pharmacy to perform private management operations. A public search outage must not block POS, stock, purchasing, or lawful exports.

## 15. Permissions and audit events

Candidate permissions:
- public_profile.configure;
- public_profile.publish;
- public_profile.pause;
- public_profile.update_contact;
- public_profile.update_location;
- public_catalogue.configure;
- public_catalogue.publish;
- public_catalogue.pause;
- public_availability.confirm;
- public_availability.publish;
- public_availability.pause;
- public_enquiry.read;
- public_report.submit;
- moderation.case.read;
- moderation.case.triage;
- moderation.content.suppress;
- moderation.decision.record;
- moderation.appeal.review;
- verification.evidence.submit;
- verification.evidence.review;
- verification.claim.publish.

Candidate events:
- PublicProfilePublicationConfigured;
- PublicProfilePublished;
- PublicProfilePaused;
- PublicProfileChanged;
- PublicCatalogueEntryPublished;
- PublicCatalogueEntryPaused;
- PublicAvailabilityConfirmed;
- PublicAvailabilityUpdated;
- PublicAvailabilityBecameStale;
- PublicEnquiryReceived;
- PublicReportSubmitted;
- ModerationCaseOpened;
- PublicContentSuppressed;
- ModerationDecisionRecorded;
- ModerationAppealFiled;
- VerificationClaimChanged.

Every event is attributable, versioned, and minimal. Sensitive documents, customer search terms, private stock quantities, and personal location data must not appear in public event payloads.

## 16. Required test scenarios

Automated and operational tests must prove:
- a new tenant and branch are private by default;
- management-only tenants can fully use private features without public profile publication;
- a staff member without publication permission cannot publish by changing a client request;
- pausing a profile does not delete or disable private management records;
- pausing one branch does not unintentionally publish or pause another;
- product publication cannot bypass canonical identity, legal category, or tenant opt-in checks;
- ambiguous/provisional product identities cannot be presented as confirmed medicine matches;
- expired, held, recalled, or otherwise ineligible stock is excluded from availability;
- offline stock changes do not refresh server-confirmed availability;
- availability automatically becomes stale/unknown at its deadline;
- delayed/replayed events cannot replace a newer availability version;
- projection rebuild cannot resurrect unpublished or suppressed content;
- exact stock quantity and private supplier/customer information never appear in public APIs;
- location denial does not prevent non-location-based discovery;
- public search terms and precise user location are not exposed to pharmacies;
- moderation reports cannot directly alter publication state without authorized review;
- support agents cannot access private tenant data without an approved case-scoped grant;
- public service failure does not block private POS or inventory operations;
- public APIs are rate-limited and do not reveal internal tenant identifiers.

## 17. Open decisions before implementation

1. Which public discovery capabilities are lawful for each pharmacy category and region at launch?
2. Are medicine/product names, brands, images, prices, and availability displays permitted, and under what restrictions?
3. What exact public verification wording is legally supportable and what evidence/source can substantiate it?
4. Which public fields are required for a useful pharmacy profile?
5. What freshness windows are acceptable for manual versus stock-derived confirmation?
6. Which availability bands, if any, are useful and safe? Should availability be status-only?
7. Should discovery be available without an account at launch? (Proposed default: yes.)
8. Which local languages, transliterations, and search aliases are required for launch?
9. Which location precision and map provider/data licensing are acceptable?
10. Is in-platform enquiry needed, or are tenant-published external contact channels sufficient?
11. What moderation staffing, escalation coverage, appeal SLA, and evidence retention can the platform actually operate?
12. What are the privacy lawful basis, retention, deletion, and user-rights requirements for public discovery logs and enquiries?
13. Are any paid placements or public ratings/reviews in scope? (Proposed default: no.)
14. What availability projection outage behavior and recovery objectives are acceptable?

Resolve legal and operational decisions before enabling public product/medicine promotion or availability publication. Until then, keep affected capabilities disabled by default and clearly mark this specification Draft.
