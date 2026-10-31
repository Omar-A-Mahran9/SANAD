# SANAD — User Roles, Actors & Permissions

> Phase 1 deliverable · Status: **DRAFT — awaiting stakeholder approval** · Date: 2026-11-01
> Sources: UX = SANAD UX Case Study; SAD = System Analysis & Design Report. See `SYSTEM_SCOPE.md §1`.

## 1. Role Reconciliation (UX audiences ↔ SAD roles)

| UX target audience [UX §2] | SAD role(s) [SAD §3] | Reconciliation |
|----------------------------|----------------------|----------------|
| People with Disabilities | Patient | Direct match. The person may also be a **dependent** who has no login of their own. |
| Families & Caregivers | Patient (account holder with dependents) **and** Family / Delegate | UX treats the family member as the main actor [UX US-02]. The SAD covers two different relationships: a patient who *creates dependents*, and a delegate who *receives consent*. Both are kept (see §3). |
| Doctors & Specialists | Doctor | Direct match. |
| Specialized Service Providers (الإخصائيين: therapists etc.) | **No SAD equivalent** | ⚠ Gap. OQ-001 asks whether they're modelled as Doctors with a non-physician provider type, or as a separate role. |
| Clinics & Rehabilitation Centers | Clinic (organization) | Rehab centers are assumed to be clinics with a different type. OQ-001. |
| Clinic Staff & Administrators | Clinic Staff, Clinic Administrator | The SAD splits the UX audience in two. Kept. |
| Platform Administrators | Platform Support, Provider Verifier, Content/Review Moderator, Finance Admin, Security/Compliance Admin, Super Admin | UX describes one "Admin" persona who does everything [UX §12.3]. The SAD **requires separation of duties** [SAD §3.1]. **SAD granularity is adopted**. The UX admin journey stays valid as a composite of these roles. |
| Partners & Advertisers | Advertiser / Partner | Direct match. Whether they get a self-service portal is unclear (OQ-028). |
| — | Visitor | SAD only. |
| — | *(external)* Lab / Imaging Provider | Appears only in the SAD Fig.1 use-case diagram. It isn't in the role table and has no portal (OQ-013). |
| — | *(external)* Payment Gateway | System actor [SAD Fig.1]. |
| *(implied by UX community features)* | — | Community moderator, group admin and event organizer **aren't defined anywhere** (OQ-023, OQ-026). |

## 2. Actor Catalog

Legend for **Auth**: `None`, `Verified contact` (email and/or mobile OTP), `MFA required`, `MFA + JIT`.

| ID | Actor | Type | Description | Auth | Source |
|----|-------|------|-------------|------|--------|
| R01 | **Visitor** | Human, anonymous | Browses public pages, searches providers, views public profiles, reads approved reviews and marketing content. | None | SAD §3 |
| R02 | **Patient** | Human | Manages own profile, searches and matches, books, pays, sees their own medical history, uploads documents, grants consent, reviews completed visits. Can create and manage dependents. | Verified contact | SAD §3; UX US-01 |
| R03 | **Family Member / Caregiver (account holder)** | Human | Manages **dependent profiles** they created and follows their care journey, including booking, records and timeline. Persona: "The Coordinating Parent" [UX §9.1]. | Verified contact | UX §2, US-02, §9.1; SAD FR-PAT-02 |
| R04 | **Family / Delegate (consent grantee)** | Human | Has access **only** to the records and functions another patient explicitly granted through consent or delegation. | Verified contact | SAD §3, FR-PAT-03, UC-12 |
| R05 | **Dependent** | Data subject (may have no login) | The person cared for, such as a child or an adult with a disability. Owns the clinical record. | n/a or Verified contact | SAD UC-02, ERD `DEPENDENTS` |
| R06 | **Doctor / Specialist** | Human, provider | Manages profile and availability. Sees authorized booked patients, documents encounters, issues prescriptions and orders, reviews results, views own metrics. | **MFA required** [SAD §13.1] | SAD §3; UX US-11/12, §9.2 |
| R07 | **Specialized Service Provider** | Human, provider | A therapist or other non-physician specialist [UX §2]. Permissions are undefined (OQ-001). | TBD | UX §2 only |
| R08 | **Clinic Staff** | Human, org member | Manages clinic profile, doctors, schedules, appointments and operational reports **within the assigned staff role**. | Verified contact (MFA TBD, OQ-009) | SAD §3 |
| R09 | **Clinic Administrator** | Human, org admin | Manages clinic staff permissions, provider affiliations, schedule templates, billing summaries and clinic-level reports. Persona: "The Clinic Administrator" [UX §9.3]. | **MFA required** | SAD §3, §13.1; UX US-13 |
| R10 | **Platform Support** | Human, internal | Resolves account and booking issues. Has *limited* access to sensitive clinical content, and **all** support access is audited. | **MFA required** | SAD §3 |
| R11 | **Clinical / Provider Verifier** | Human, internal | Reviews doctor and clinic credentials, and activates or suspends verification status. | **MFA required** | SAD §3; UX US-14 |
| R12 | **Content / Review Moderator** | Human, internal | Moderates reviews, comments and ad content. **No default access to clinical history.** | **MFA required** | SAD §3 |
| R13 | **Finance Administrator** | Human, internal | Gateway transactions, refunds, settlements, reconciliation. **No clinical access.** | **MFA required** | SAD §3, §13.2 |
| R14 | **Security / Compliance Administrator** | Human, internal | Access logs, security events, consent and audit evidence, retention, incident workflows. | **MFA required** | SAD §3; UC-14 |
| R15 | **Super Admin** | Human, internal | System configuration and emergency operations. Uses a **separate privileged account**, MFA, **just-in-time elevation** and enhanced audit. | **MFA + JIT** | SAD §3 |
| R16 | **Advertiser / Partner** | Human, external org | Submits campaigns and assets and sees campaign analytics only. **No access to patient records or condition-level private profiles.** | Verified contact | SAD §3; UX §2 |
| R17 | **Payment Gateway** | System | Hosted payment and tokenization; sends signed webhooks. | Signature | SAD Fig.1, FR-PAY-02 |
| R18 | **Lab / Imaging Provider** | External (undefined) | Shown in Fig.1 attaching lab and imaging records. | TBD | SAD Fig.1 only → OQ-013 |
| R19 | **Scheduler / Worker** | System | Reminders, expiry checks, campaign activation, cleanup, reports, scans. | Service identity | SAD §15.1 |
| R20 | **AI Assistant (Ask SANAD / Chatbot)** | System | Reads platform data on the user's behalf. Must not exceed **the requesting user's** permissions (OQ-021). | Inherits user | UX §6 |

## 3. Family Relationship Models (critical for SANAD)

The SAD and the ERD describe **three** different ways one person can act for another. They must be kept apart.

| Model | Who creates it | Who owns the clinical record | Access basis | Source |
|-------|----------------|------------------------------|--------------|--------|
| **A. Dependent** | The account holder (patient or family member) creates a profile for someone with no account, such as a child | The dependent, as data subject | Relationship (parent/guardian) | FR-PAT-02, UC-02, ERD `DEPENDENTS(patient_id, name/relationship, consent_status)` |
| **B. Delegate** | A patient with their own account grants access to another user | The granting patient | Explicit scoped consent: person, scope, date range, revocable | FR-PAT-03, UC-12, ERD `CONSENT_GRANTS` |
| **C. Legal guardian of an adult** | Not defined | The adult dependent | "Legal guardian rule" is named but not specified | UC-02 / UC-12 alternatives |

⚠ **Important gaps** (OQ-006, OQ-007, OQ-008):
- The ERD gives `DEPENDENTS` only a name and relationship, while clinical entities link to `PATIENT_PROFILES`. A dependent therefore can't be booked or hold records unless it's also a patient profile. The model needs a decision.
- SANAD's primary audience includes **adults with disabilities who are cared for by family**. The SAD says "adult family members do not automatically see another adult's records" [SAD §13.3]. The guardianship verification process decides whether the core UX journey works for adult dependents.
- Rules for when a minor dependent turns 18 (handover of record ownership) aren't defined.

## 4. Permission Matrix (target state)

Legend: ✅ full · 🔶 scoped (own / relationship / consent / clinic membership / assigned role) · 👁 read-only · 🧾 audited & reason-required · ❌ none · ❓ undefined (see OQ).
Columns: VIS Visitor · PAT Patient · FAM Family account-holder · DEL Delegate · DOC Doctor · CST Clinic Staff · CAD Clinic Admin · SUP Support · VER Verifier · MOD Moderator · FIN Finance · SEC Security/Compliance · SA Super Admin · ADV Advertiser.

| Capability | VIS | PAT | FAM | DEL | DOC | CST | CAD | SUP | VER | MOD | FIN | SEC | SA | ADV |
|------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:--:|:---:|
| Browse public site, search, view public profiles & approved reviews | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Register / sign in | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Manage own profile | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Create / manage dependents | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | 🧾 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Grant / revoke consent | ❌ | 🔶 own | 🔶 for dependents ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | 👁 evidence | ❌ | ❌ |
| Book / reschedule / cancel appointment | ❌ | 🔶 own | 🔶 dependents | 🔶 if scope grants | 🔶 own schedule | 🔶 clinic | 🔶 clinic | 🧾 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Pay provider | ❌ | 🔶 | 🔶 | 🔶 if scope ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | 👁 | ❌ | ❌ | ❌ |
| View medical timeline / records | ❌ | 🔶 own | 🔶 dependents | 🔶 consent scope | 🔶 care relationship | ❓ OQ-010 | ❓ OQ-010 | 🧾 limited | ❌ | ❌ | ❌ | 👁 audit metadata only | 🧾 break-glass | ❌ |
| Upload medical documents | ❌ | 🔶 own | 🔶 dependents | 🔶 if scope | 🔶 relationship | ❓ | ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Record encounter / prescription / orders | ❌ | ❌ | ❌ | ❌ | 🔶 authorized appointment | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Record lab / imaging result | ❌ | ❌ | ❌ | ❌ | 🔶 relationship/consent | ❓ | ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Export medical record | ❌ | 🔶 own | 🔶 dependents | ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Submit review (completed appointment) | ❌ | 🔶 | 🔶 dependents' appts ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Manage provider profile & availability | ❌ | ❌ | ❌ | ❌ | 🔶 own | 🔶 clinic doctors | 🔶 clinic | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Submit credentials | ❌ | ❌ | ❌ | ❌ | 🔶 own | ❌ | 🔶 clinic | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Approve / suspend verification | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Manage clinic staff & permissions | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ own clinic | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Clinic reports & billing summaries | ❌ | ❌ | ❌ | ❌ | 👁 own | 🔶 by role | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Moderate reviews / comments / ad content | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Transactions, refunds, reconciliation | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | 👁 summaries | 👁 | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |
| Audit log search / export | ❌ | 👁 own access history | 👁 dependents | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Users & roles administration | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | 🔶 clinic | 🧾 support actions | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| System configuration (taxonomy, weights, policies, flags) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ JIT | ❌ |
| Manage campaigns & creatives | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❓ provider promotion | ❌ | ❌ | 👁 review | ❌ | ❌ | ✅ | 🔶 own ❓ OQ-028 |
| Campaign analytics (aggregate only) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | 🔶 own |
| Join community groups / group chat | ❌ | ❓ | ❓ | ❓ | ❓ | ❌ | ❌ | ❌ | ❌ | ❓ | ❌ | ❌ | ❌ | ❌ |
| Create / manage events | ❌ | ❌ | ❌ | ❌ | ❓ "specialist-recommended" | ❓ | ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❓ | ❌ |
| Register for events | ❓ | ✅ | ✅ | ❓ | ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Use Ask SANAD / AI chatbot | ❓ | ✅ | ✅ | ❓ | ❓ | ❓ | ❓ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

## 5. Access-Control Principles (normative, SAD §3.1 & §13)

1. **Deny by default.** Grant only the minimum necessary permissions.
2. **RBAC plus relationship/attribute checks**: patient ownership, doctor↔appointment relationship, clinic membership, consent scope.
3. **Database-level boundary.** The SAD specifies PostgreSQL RLS. Under the approved MySQL stack an equivalent control has to be approved (OQ-040).
4. **Admin authority ≠ clinical visibility.** No generic admin role grants clinical-record access.
5. **Break-glass** is used only where legally required, with an explicit reason, a time limit, and an immediate audit record and alert.
6. **MFA** is required for doctors, clinic admins and every platform-admin role. Super Admin also needs a separate account and JIT elevation.
7. **Finance permissions are separate from clinical permissions.**
8. **Advertisers never reach patient data.** Not records, and not condition-level private profiles.
9. Every sensitive access and privileged change produces an **append-only audit event**.

## 6. Open Role Questions (summary; detail in OPEN_QUESTIONS.md)

OQ-001 Specialized service providers / rehab centers · OQ-006 dependent data model · OQ-007 adult-guardian rule · OQ-008 minor coming of age · OQ-009 MFA for clinic staff · OQ-010 clinic staff clinical access · OQ-011 doctor access to prior history · OQ-013 lab/imaging actor · OQ-023 community roles · OQ-026 event organizers · OQ-028 advertiser portal · OQ-021 AI data access.
