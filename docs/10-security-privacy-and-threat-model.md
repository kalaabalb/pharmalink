# 10 — Security, privacy, and threat model

**Status:** Draft — threat-model baseline; requires security and legal review.

## 1. Security objectives

Protect:
- tenant and branch confidentiality;
- integrity of stock, sales, payment, financial, catalogue, and verification records;
- availability of essential pharmacy business records;
- personal data and any future sensitive health-related data;
- authenticity of establishment and professional verification;
- integrity of event delivery and offline synchronization;
- trustworthiness of public information.

Security is a system property. UI hiding, client-side validation, obscured identifiers, and unverified user claims are not security controls.

## 2. Trust boundaries

Identify and document boundaries between:
1. public internet and platform edge;
2. unauthenticated and authenticated sessions;
3. one tenant and another tenant;
4. one branch and another branch;
5. ordinary tenant users and privileged tenant roles;
6. tenant users and platform support/administrators;
7. application services and external providers;
8. server-authoritative state and device-local offline state;
9. canonical reference data and tenant-supplied data;
10. public projections and private operational records.

Every boundary requires authentication where appropriate, authorization, input validation, logging, and failure-safe behavior.

## 3. Threat scenarios to test

- A user changes tenant or branch identifiers in a request to access another pharmacy's records.
- A revoked staff member continues using an old session or refresh token.
- A user manipulates role claims or submits a privileged request directly to an API.
- A support agent uses a legitimate support tool to browse unrelated tenant data.
- A malicious or compromised device replays queued commands after access has been revoked.
- Duplicate or forged payment callbacks alter payment state.
- An attacker submits malformed catalogue data or misleading verification evidence.
- A public endpoint leaks internal prices, quantities, staff data, customer data, or operational metadata.
- An event subscription exposes data across tenant or branch boundaries.
- Logs, analytics, crash reports, or push notifications expose sensitive values.
- A dependency, integration token, or deployment secret is compromised.
- A backup, export, or object-storage bucket is exposed.
- A denial-of-service event prevents legitimate users from accessing records.
- A user attempts to erase or alter audit evidence through ordinary correction flows.

## 4. Authentication and session requirements

- Use established authentication mechanisms; do not invent custom password cryptography.
- Require secure transport for all authenticated traffic.
- Store passwords only using an established, appropriately configured password-hashing scheme if password authentication is used.
- Protect session and refresh tokens against theft and replay.
- Support session revocation, account recovery, and re-authentication for high-impact actions.
- Define rate limits and abuse detection for login, recovery, invitations, and verification endpoints.
- Do not expose whether a private account exists through unnecessarily distinguishable error messages.
- Define a policy for shared devices and unattended sessions.

Exact session durations and authentication factors require threat and usability review.

## 5. Authorization and tenant isolation

- Enforce tenant, branch, membership, role, permission, and resource-state checks on the server.
- Derive tenant context from authenticated membership, not from an untrusted request field alone.
- Scope database queries and background jobs to an explicit tenant context.
- Ensure exports, files, search indexes, caches, event channels, and analytics follow the same tenant boundary.
- Test horizontal privilege escalation, vertical privilege escalation, guessed IDs, bulk endpoints, and asynchronous job access.
- Privileged support access must be purpose-bound, time-limited, approved where required, and fully audited.

## 6. Data classification and minimization

Candidate classifications:
- **Public:** intentionally published pharmacy profile fields.
- **Internal:** non-sensitive operational configuration.
- **Confidential business:** stock, pricing, supplier, sales, finance, staff, and subscription records.
- **Personal:** account, staff, contact, and identity information.
- **Sensitive personal:** any health-related or other legally defined sensitive personal data.
- **Security restricted:** credentials, secrets, access logs, incident evidence, and privileged-access records.

Collect only fields necessary for an approved purpose. Patient/customer health records remain excluded until a dedicated review approves purpose, lawful basis, retention, access, and safeguards.

## 7. Encryption, secrets, and infrastructure

- Encrypt traffic in transit.
- Encrypt sensitive data at rest and protect encryption keys separately from data.
- Store secrets in a managed secret facility; never commit secrets to Git or place them in client bundles.
- Rotate credentials and revoke compromised keys.
- Separate production, staging, and development credentials and datasets.
- Do not use real customer or patient data in development or test environments unless a specific lawful and approved process exists.
- Apply least privilege to service identities and integration credentials.
- Maintain dependency and vulnerability review.

## 8. Audit and monitoring

Audit records should capture actor or system identity, action, target, tenant/branch scope, time, outcome, reason, and correlation reference. Avoid copying sensitive payloads into logs. Audit access itself must be controlled and logged.

Monitor authentication anomalies, authorization denials, privileged access, export activity, event delivery failures, unusual data access, payment inconsistencies, and backup/restore failures. Define incident severity, escalation, evidence preservation, communication, and recovery procedures.

## 9. Privacy lifecycle

Before collecting personal data, document:
- purpose and necessity;
- data categories and data subjects;
- lawful basis and required notice/consent process;
- controller/processor and third-party roles;
- access and disclosure recipients;
- retention and deletion schedule;
- cross-border transfer and hosting assessment;
- rights-request handling;
- security and breach response.

Do not assume consent is the only lawful basis or that consent can cure an otherwise impermissible use. Legal review must determine the correct basis for each purpose.

## 10. Acceptance criteria

- Automated tests demonstrate tenant and branch isolation across APIs, queries, events, exports, caches, and jobs.
- Revoked memberships and sessions cannot continue privileged access beyond the defined revocation objective.
- No secrets are present in source control or client-delivered assets.
- Sensitive data is excluded from logs and push payloads by default.
- Privileged support access is attributable and reviewable.
- Security incidents have an owner, severity, response procedure, and evidence trail.
- Data collection and retention have documented purposes and approval.

## 11. Open decisions

- Which authentication factors are appropriate for owners and high-impact roles?
- What are the session and token revocation objectives?
- Which cloud regions and subprocessors may be used after legal review?
- What security assurance or independent testing is required before launch?
- Which data classes, if any, will be present in the first release beyond staff and business records?
