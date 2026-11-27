# 23 — User experience, accessibility, and localization specification

**Status:** Draft — pharmacy observation, public-user research, accessibility testing, and language review required  
**Priority:** High — workflows must be usable in real pharmacy conditions, including shared devices and unreliable connectivity  
**Scope:** Navigation, interaction patterns, form behavior, transaction feedback, responsive layouts, accessibility, localization, low-bandwidth behavior, and user-facing error states across the pharmacy, public, and platform surfaces.

## 1. UX product principles

1. Optimize for accurate completion of real pharmacy work, not visual novelty or demo interactions.
2. Make high-frequency tasks direct and predictable; avoid unnecessary screens and repeated data entry.
3. Never hide a business-critical state behind color, animation, or icon alone.
4. Keep private pharmacy operations distinct from public discovery and platform administration.
5. Show only actions the user can perform, but always enforce permission on the server.
6. Preserve user-entered work through safe drafts and explicit recovery behavior.
7. Make loading, empty, denied, stale, offline, queued, conflict, success, and failure states explicit.
8. Use confirmation proportional to risk; avoid confirmation dialogs for every low-risk action.
9. Do not report success until the authoritative operation has committed.
10. Design for real devices, real network conditions, and the actual language/number/date conventions of launch customers.

## 2. Product surfaces and navigation

The product has three distinct surfaces:
- **Pharmacy Operations:** organization/branch workspaces for staff and management.
- **Public Discovery:** public pharmacy and approved product information.
- **Platform Operations:** restricted support, verification, moderation, billing, and service operations.

The interface must make the current surface, organization, branch, and account role visible. Switching tenant or branch changes the data scope and must be obvious. Avoid mixing platform support controls into ordinary pharmacy navigation.

Candidate Pharmacy Operations navigation groups:
- Home/overview;
- Point of sale (if enabled);
- Inventory and batches;
- Products/catalogue;
- Purchasing and suppliers;
- Finance and reports;
- Staff and access;
- Branch settings;
- Notifications/tasks;
- Help/support.

The final navigation order and labels must be validated with target pharmacy staff. Do not show unavailable capabilities as active menu items. A capability may be labelled planned or unavailable only where that information is useful and not confusing.

Public Discovery must have a separate navigation and privacy model. Public users should not need to understand tenant administration, branch permissions, or internal stock operations.

## 3. Role-aware experience

The UI should adapt to the user's active tenant, branch, and granted permissions:
- hide irrelevant actions where appropriate;
- explain why a needed action is unavailable;
- show the responsible role or next step when safe;
- never expose confidential data merely because a screen is hidden;
- avoid role names as a substitute for explaining actual permission.

When a user belongs to multiple organizations, the active organization must be prominent. When a branch-specific action is performed, the branch must be clear before commit.

Permission changes should update active sessions promptly. If a permission is revoked while a screen is open, the next protected action is denied by the server and the interface explains that access changed; it must not display a false success.

## 4. Workflow interaction standards

### Forms

- Label every field persistently; placeholders are not labels.
- Group fields by task and explain units, accepted formats, and consequences.
- Preserve valid entered values after validation errors.
- Validate locally for immediate feedback and on the server for authoritative rules.
- Identify the specific field and correction needed.
- Do not silently coerce medicine strength, units, currency, dates, or identifiers.
- Show required/optional status clearly.
- Avoid asking for personal or sensitive information unless the workflow requires it.
- Support keyboard navigation, autofill where safe, and accessible focus order.

### Tables and lists

- Provide clear column labels, sorting/filtering state, and result counts.
- Keep critical identifiers and status visible at responsive widths.
- Do not rely on hover-only actions.
- Provide accessible alternatives to dense tables on small screens.
- Distinguish item identity from batch/lot identity.
- Show units and currency consistently.
- Make pagination or incremental loading predictable; preserve filters when navigating back.

### High-impact actions

For sale commit, stock adjustment, batch release, disposal, supplier payment, refund, permission change, public publication, and similar actions:
- summarize the object and material effect before confirmation;
- show the active tenant/branch;
- show amount, quantity, product/batch, and destination where relevant;
- require a reason or second approval where policy requires;
- prevent double submission;
- show pending state while the server processes;
- show a durable reference after commit;
- provide a traceable correction path rather than silent editing.

Do not use dark patterns, preselected optional consent, or urgency language to force actions.

## 5. State and feedback vocabulary

The interface must use consistent meanings:

| State | Meaning | Required user feedback |
|---|---|---|
| Draft/local | Saved only on this device or in an explicitly local draft | State that it is not yet on the server |
| Queued | Waiting for synchronization | Explain pending status and what cannot happen yet |
| Submitting | Command sent; outcome not yet authoritative | Prevent duplicate action and show progress |
| Confirmed/committed | Server accepted and committed the transaction | Show reference and next step |
| Rejected | Server did not accept the command | Explain reason and safe correction |
| Needs review | Human/authorized decision is required | Identify the review owner or path |
| Offline | No current server connection | Show what remains usable and what is blocked |
| Stale | Data may no longer be current | Show last confirmed time and avoid implying freshness |
| Conflict | Local and server versions differ | Explain the conflict without silently overwriting |
| Restricted | Capability is unavailable under current rules | State the scope and review/appeal route where appropriate |

Never use “Saved” ambiguously when the user needs to know whether the server committed the action.

## 6. Notifications and task handling

Notifications should be actionable, scoped, and low-noise.

Candidate notification classes:
- security and account access;
- membership/invitation;
- approval/review tasks;
- stock/expiry/recall;
- purchase order and supplier delivery;
- payment/reconciliation;
- public listing/availability;
- service/billing;
- system outage and recovery.

Each notification defines:
- recipient roles and tenant/branch scope;
- triggering event;
- urgency and delivery channel;
- minimal safe content;
- deduplication key;
- expiry and read state;
- deep-link authorization recheck;
- retry and failure behavior.

Do not include prescription details, medicine search terms, sensitive customer data, credentials, full bank/payment details, or private stock quantities in ordinary push/SMS previews. A notification is not the source of truth; opening it must load the current authorized record.

## 7. Accessibility requirements

Target WCAG 2.2 AA as the design and test baseline, subject to formal accessibility review.

Required behavior:
- full keyboard operation for all critical flows;
- visible focus indicators;
- semantic headings, landmarks, labels, and controls;
- screen-reader names and status announcements;
- error identification and correction suggestions;
- sufficient text and control contrast;
- no status conveyed by color alone;
- scalable text and responsive reflow;
- touch targets sized for practical use;
- reduced-motion support;
- no flashing or disorienting animation;
- accessible tables, dialogs, menus, charts, and date controls;
- accessible authentication and recovery;
- captions/transcripts for instructional media where used.

Critical workflows (sign-in, receiving stock, sale commit, payment status, stock adjustment, and permission management) require keyboard and screen-reader testing, not only automated scans.

## 8. Localization and language support

Language support must be based on target-customer research and professional translation review. Candidate launch languages may include Amharic and English, with additional Ethiopian languages evaluated from actual customer needs.

Requirements:
- store canonical data independently of localized display labels;
- support Unicode and the relevant Ethiopic scripts without data loss;
- support search aliases and transliteration where validated;
- use reviewed translations for medicine identity, units, safety warnings, finance, and legal notices;
- do not machine-translate safety-critical or legal text into production without qualified review;
- allow a user preference where appropriate, without changing organization-wide canonical records;
- define fallback language and missing-translation behavior;
- test sorting, search, truncation, line wrapping, and font rendering across supported scripts.

Language selection does not imply a particular user's ethnicity or identity. Do not infer language preference from name, location, or other sensitive attributes.

## 9. Dates, time, numbers, units, and currency

Store canonical timestamps in a consistent machine-readable format and render them in the relevant branch/user timezone with an explicit locale.

Decisions required:
- whether the Ethiopian calendar is supported in addition to Gregorian display;
- whether users can select calendar and numeral preferences;
- how ambiguous date entry is prevented;
- whether Arabic/Western or Ethiopic numerals are supported in input and display;
- currency formatting and decimal precision;
- medicine strength/concentration units and normalization;
- stock purchase, pack, and sale units;
- timezone behavior for branches in different regions.

Never store a localized date string as the only authoritative timestamp. Preserve canonical instant, timezone context, and source date where legally/business relevant. Do not silently convert an ambiguous date or unit.

Money calculations use exact decimal/minor-unit rules from the finance specification. Display formatting must not change stored value or rounding.

## 10. Low-bandwidth, offline, and device constraints

The interface must be usable under constrained connectivity and on the actual device mix found in pharmacy research.

Requirements:
- prioritize critical task content over decorative media;
- avoid large automatic downloads and unnecessary animation;
- cache only approved data with visible freshness/version;
- show connection and synchronization state without constant disruptive alerts;
- do not claim a command is committed while offline;
- provide clear blocked/allowed action boundaries;
- prevent local drafts from leaking across shared-device accounts;
- clear or lock sensitive cached data on logout/device revocation according to policy;
- handle app suspension, battery constraints, reconnect, and interrupted uploads;
- avoid frequent polling for unchanged data; use event subscriptions and bounded recovery synchronization.

Offline support is capability-specific. The UI must not imply that every screen or transaction works offline.

## 11. Responsive and device support

The release must declare supported:
- desktop browsers and minimum versions;
- tablets and phone screen sizes;
- operating systems and mobile browser/app behavior;
- printers, barcode scanners, cash drawers, and other peripherals if included.

Critical workflows must be tested at the supported widths and with real target devices. Do not claim peripheral support based only on a browser simulator. Device permissions, disconnection, and reconnection require explicit states.

## 12. Content, trust, and error writing

Use concise, factual language. Error messages should state:
- what failed;
- whether the operation committed;
- whether retry is safe;
- what the user can do next;
- a support/reference ID where useful.

Never tell the user to retry a payment or stock commit if the outcome is unknown until reconciliation. Avoid blaming the user or exposing stack traces, provider internals, security rules, or personal data.

Verification and availability wording must match the actual source, scope, and freshness. Avoid labels such as “guaranteed,” “official,” “certified,” or “in stock” unless the evidence and legal review support the exact claim.

## 13. UX research and validation

Before implementation, observe target pharmacy staff completing representative tasks using their current process. Include owners, managers, cashiers, inventory/purchasing staff, accountants, and public users where relevant.

Research should measure:
- task completion and error rates;
- time and repeated data entry;
- common interruptions;
- device and peripheral use;
- connectivity conditions;
- language and literacy needs;
- shared-device and account-switching behavior;
- accessibility needs;
- trust in availability and verification wording;
- comprehension of offline, pending, and stale states.

Do not treat internal team preference or a polished prototype as proof of workflow usability.

## 14. Required test scenarios

Automated and usability tests must prove:
- every critical action has a clear pending, committed, rejected, and unknown state;
- forms preserve valid values and identify field-level errors;
- no high-impact command can be double-submitted to create duplicate effects;
- current tenant and branch are clear before a scoped action;
- role revocation is reflected on the next protected action;
- keyboard-only users can complete critical workflows;
- screen readers announce labels, validation, and transaction state;
- color is never the only signal for status;
- text scaling and narrow screens do not hide critical actions or amounts;
- supported scripts render, search, and sort correctly;
- unreviewed translations cannot replace safety/legal text;
- dates, timezone, currency, and units are unambiguous;
- offline drafts are visibly distinct from server-committed records;
- sensitive local caches are not available to a different user after logout;
- notification previews do not disclose sensitive data;
- reduced-motion settings are respected;
- actual supported printers/scanners are tested end-to-end where included.

## 15. Open decisions before implementation

1. Which pharmacy roles and daily tasks should define the first-release navigation?
2. Which devices, browsers, tablets, scanners, and printers are used by target pharmacies?
3. Which languages are required at launch, and who will review translations?
4. Is Ethiopian calendar display required at launch, and for which workflows?
5. Which numeral, currency, unit, and date input conventions should be supported?
6. What accessibility accommodations are required by actual users?
7. Which offline capabilities are essential and what local data may be cached?
8. What is the correct shared-device logout/lock behavior?
9. Which notifications are essential, and which channels can be operated reliably?
10. What usability targets should be used for high-frequency pharmacy tasks?
11. Which public-user discovery tasks need to work without account registration?
12. Which peripheral integrations are required for the initial production cohort?

Finalize UX decisions through observed workflow research and accessibility testing. A visually complete screen is not a completed feature unless its actions, permissions, data states, and recovery behavior work end to end.
