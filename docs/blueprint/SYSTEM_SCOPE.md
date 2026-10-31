# SANAD — System Scope

> Phase 1 deliverable · Status: **DRAFT — awaiting stakeholder approval** · Date: 2026-11-01

## 1. Sources

| Code | Document | Location | Notes |
|------|----------|----------|-------|
| **UX** | SANAD UX Case Study (Maha Omran, Senior UI/UX Designer) | `docs/source/SANAD_UX_Case_Study.pdf` (15 pp.) | Product vision, audience, 31 features, 15 user stories, personas, 3 journeys, design system. |
| **SAD** | Doctor–Patient Medical Platform — System Analysis & Design Report (Aug 2026) | `docs/source/System_Analysis_Design.pdf` (29 pp.) | 40 functional reqs, 15 NFRs, 14 use cases, roles, business rules, 48-page inventory, security, deployment. Uses the working title "Medical Platform". |

Citation format used across all blueprint documents: `[UX §6]`, `[SAD §4 FR-BOOK-02]`, `[SAD Fig.4]`.

Companion documents: `PRODUCT_REQUIREMENTS.md`, `FEATURE_MATRIX.md`, `USER_ROLES.md`, `USER_JOURNEYS.md`, `BUSINESS_RULES.md`, `OPEN_QUESTIONS.md`, `REQUIREMENTS_TRACEABILITY.md`.

## 2. Product Definition (reconciled)

**SANAD (سند)** is a connected care platform. It brings specialized care services and trusted, verified providers together in one place. People with disabilities and their families use it to discover, book, manage and follow their care journey [UX §5].

The SAD describes the technical core of that platform. It's a healthcare mediation platform that connects patients and families with verified doctors and clinics. It covers matching, booking, reviews, payments and a longitudinal medical record [SAD §1].

The two documents frame the product differently, and the reconciled scope has to hold both:

| Dimension | UX Case Study | SAD |
|-----------|---------------|-----|
| Primary audience | People with disabilities & their families [UX §2] | Patients and patient families (general) [SAD §1] |
| Provider types | Doctors & specialists, **specialized service providers (therapists)**, clinics **& rehabilitation centers** [UX §2] | Doctors and clinics only [SAD §3] |
| Value focus | The whole care journey, family empowerment, community, AI guidance | Discovery, booking, medical record, payments, reviews, ads |
| Language | Bilingual Arabic/English, Tajawal font [UX §1–14] | English; Arabic/RTL "if future Arabic UI is required" [SAD NFR-I18N-01] |

→ See OQ-001 (target audience and provider types), OQ-002 (language) and OQ-003 (scope interpretation) in `OPEN_QUESTIONS.md`.

**Clinical safety boundary** [SAD §1]: SANAD matches a stated condition or care need to an appropriate specialty or provider category. It **does not diagnose** and **does not replace emergency services**. Urgent or emergency rules override normal ranking.

## 3. Scope Statement

The approved scope is the **full product scope**: every feature in both documents is kept. Nothing has been removed to simplify implementation.

### 3.1 In scope

All 84 features in `FEATURE_MATRIX.md`, grouped into these modules:

1. Identity & Access, including MFA, RBAC/ABAC, sessions, and break-glass handling
2. Patient & Family: profiles, dependents, delegates, consent
3. Provider Directory: doctor, specialist and clinic/rehab-center profiles, affiliations, verification
4. Discovery & Matching: search, filters, condition→specialty smart matching, emergency override, comparison, taxonomy
5. Scheduling: availability, atomic booking, reschedule/cancel, reminders
6. Clinical Records: encounters, prescriptions, lab, imaging, documents, timeline, amendments, export
7. Payments through an external gateway: card and mobile wallet, webhooks, refunds, reconciliation
8. Reviews & Ratings: verified, moderated, Bayesian rating
9. Notifications: email, SMS, in-app, realtime
10. Advertising & Sponsored Placements, kept strictly separate from clinical matching
11. Community & Events: groups, group chat, social and educational events, event management *(UX only)*
12. AI Assistance: Ask SANAD, AI chatbot, AI-powered analysis and reports *(UX only)*
13. Accessibility & Localization
14. Administration, Audit, Reporting, Configuration, Monitoring & Support
15. Public website and legal or consent pages

### 3.2 Explicitly excluded by source (not built unless re-approved)

These exclusions come from [SAD §2.4]. A "full scope" mandate doesn't automatically override them; see **OQ-003**.

| Excluded item | Source | Interaction with UX features |
|---------------|--------|------------------------------|
| Autonomous diagnosis, prescribing, triage diagnosis, clinical decision support | SAD §2.4, §1 | **Conflicts with how UX "AI-Powered Analysis & Reports" could be read** → OQ-020 |
| Insurance claims adjudication / payer integrations | SAD §2.4 | — |
| Full PACS replacement / raw DICOM storage | SAD §2.4, §14.2 | Imaging stores reports and attachments only |
| Pharmacy dispensing / controlled-drug fulfilment | SAD §2.4 | — |
| Nationwide HIE interoperability | SAD §2.4 | — |
| In-house storage or processing of raw PAN/CVV | SAD §2.4, §13.2; CLAUDE.md | Absolute rule |

### 3.3 Future enhancements named by SAD (status needs confirmation, OQ-003)

Telemedicine/video [SAD §18.1], FHIR interoperability, PACS/DICOM integration, lab integrations for electronic orders and results, insurance eligibility, native iOS/Android apps, privacy-preserving recommendation analytics, and a clinically governed symptom-triage module.

The SAD still asks the MVP to be ready for some of these. The matching "Location" component includes a "telehealth option if enabled" [SAD §6.1], and the imaging page has a "PACS link if integrated" [SAD §12]. The architecture must leave room for them without building them.

## 4. Module Map (modular monolith — proposed for Phase 2 confirmation)

Laravel is the single source of business logic (CLAUDE.md). Each module owns its data and exposes contracts; other modules don't touch its tables directly.

| # | Module | Owns (features) | Depends on |
|---|--------|-----------------|------------|
| M01 | **Identity & Access (IAM)** | F-IAM-* | — (root) |
| M02 | **Audit & Compliance (AUD)** | F-AUD-* | IAM |
| M03 | **Localization & Accessibility (ACC)** | F-ACC-* (cross-cutting) | — |
| M04 | **Content & Public Site (PUB)** | F-PUB-* | IAM (admin), ACC |
| M05 | **Patient & Family (PAT)** | F-PAT-* incl. consent engine | IAM, AUD |
| M06 | **Provider Directory (PRV)** | F-PRV-* incl. verification | IAM, AUD, Files |
| M07 | **Taxonomy & Matching (DSC)** | F-DSC-* | PRV, SCH (availability), REV (rating) |
| M08 | **Scheduling (SCH)** | F-SCH-* | PRV, PAT, NOT |
| M09 | **Clinical Records (CLN)** | F-CLN-* | PAT (consent), SCH (relationship), Files, AUD |
| M10 | **Files (shared kernel)** | encrypted object storage, scanning, signed URLs | IAM, AUD |
| M11 | **Payments (PAY)** | F-PAY-* | SCH, gateway |
| M12 | **Reviews (REV)** | F-REV-* | SCH (completed appointment) |
| M13 | **Notifications (NOT)** | F-NOT-* | IAM (preferences) |
| M14 | **Advertising (ADS)** | F-ADS-* | PRV (sponsored), *never* CLN/PAT medical data |
| M15 | **Community & Events (COM)** | F-COM-* | IAM, PAT, PRV, NOT |
| M16 | **AI Assistance (AI)** | F-AI-* | depends on OQ-020/021 — data access must go through IAM/PAT consent |
| M17 | **Administration & Reporting (ADM)** | F-ADM-* | all (read models) |

**Hard boundaries the sources require:**
- ADS must never read from CLN or from medical fields in PAT [SAD FR-ADS-03, §6.4].
- Being an admin doesn't grant clinical visibility [SAD §3.1]. ADM can't bypass the CLN access policies.
- Matching scores must not take any input from ADS [SAD FR-ADS-02].
- Group membership in COM can reveal a condition. Treat it as sensitive and keep it away from ADS (see BR-COM-03 and OQ-024).

## 5. External Systems & Integrations

| System | Role | Source | Vendor decided? |
|--------|------|--------|-----------------|
| Payment gateway (hosted page / tokenization, card + mobile wallet) | Collect provider payments | SAD §13.2, FR-PAY-* | ❌ OQ-030 |
| Email provider | Notifications | FR-NOT-01 | ❌ OQ-034 |
| SMS / OTP provider | Contact verification, notifications | FR-AUTH-01, FR-NOT-01 | ❌ OQ-034 |
| Encrypted object storage (+ CDN, pre-signed URLs) | Medical files, images | SAD §9, §13.1 | ❌ OQ-035 |
| Malware scanning service | Uploaded files | FR-MED-05 | ❌ OQ-035 |
| Maps / geolocation | Clinic address/map, distance matching | FR-PROV-02, §6.1 | ❌ OQ-036 |
| AI / LLM provider | Ask SANAD, chatbot, analysis | UX §6 | ❌ OQ-021 |
| Lab / Imaging provider (external actor) | Submit results | SAD Fig.1 only | ❌ OQ-013 |
| PACS | Imaging link (future) | SAD §2.4, §18.1 | Deferred |

## 6. Approved Technology vs. SAD Baseline

The approved decisions (CLAUDE.md) **override** the SAD's technology baseline. The deviations below change how some SAD requirements are met. The requirements themselves stay.

| SAD baseline | Approved | Impact on requirements | Action |
|--------------|----------|------------------------|--------|
| **PostgreSQL** [SAD §8, §9] | **MySQL (InnoDB)** | SAD §3.1 and §13.1 require **PostgreSQL Row-Level Security** as a second database-level boundary on patient-scoped tables, and MySQL has no RLS. SAD §14.3 also suggests PostgreSQL full-text/trigram search. MySQL FULLTEXT supports Arabic through the ngram parser, but with different behavior. | OQ-040: approve a replacement for the defense-in-depth control, such as mandatory scoped repositories plus DB-user privilege separation and views. |
| **Node.js realtime/integration service** [SAD §8, §15] | No separate Node backend without explicit approval | WebSocket notifications, realtime presence, and **UX group chat** need a realtime transport. | OQ-041: approve a Laravel-native realtime option, such as a PHP WebSocket server, or explicitly approve Node. |
| React + **Bootstrap** [SAD §8, §11.1] | One React application | Bootstrap isn't in the approved decisions or prohibited by them. | OQ-042 |
| Docker, provider-neutral cloud [SAD §15] | Not addressed | `infrastructure/` exists. Cloud provider and region aren't decided, and data residency depends on jurisdiction. | OQ-043 |
| Redis (recommended) | **Redis — approved** | Aligned | — |

## 7. Non-Functional Requirements (from SAD §5, retained)

| ID | Category | Requirement | Notes for SANAD |
|----|----------|-------------|-----------------|
| NFR-SEC-01 | Security | OWASP ASVS 5.0.0 baseline; address OWASP Top 10:2025 | |
| NFR-SEC-02 | Encryption | TLS in transit; encryption at rest for DB backups & object storage; managed key rotation | |
| NFR-SEC-03 | Privacy | Data minimization, consent, purpose limitation, access logging, jurisdiction-specific retention/deletion | Jurisdiction → OQ-004 |
| NFR-SEC-04 | Payments | Cardholder data kept out of app (hosted/tokenized); PCI DSS v4.0.1 alignment | |
| NFR-PERF-01 | Latency | p95 API < 500 ms (CRUD/search, excl. 3rd party); p95 page interaction < 2.5 s | |
| NFR-PERF-02 | Capacity | Horizontally scalable stateless API & realtime; cache, queue, direct object-storage uploads | |
| NFR-AVL-01 | Availability | MVP 99.9%; growth 99.95% (multi-AZ DB, redundant app containers) | |
| NFR-DR-01 | Recovery | RPO ≤ 15 min, RTO ≤ 2 h (production target, subject to budget) | |
| NFR-A11Y-01 | Accessibility | WCAG 2.2 AA: keyboard, contrast, labels, error messaging | **Raised in importance by UX primary audience** (OQ-016) |
| NFR-UX-01 | Responsive | Desktop, tablet, modern mobile browsers | UX persona: mobile is primary for families [UX §9.1] |
| NFR-I18N-01 | Localization | Multi-language & RTL "if future Arabic UI required" | **Conflict with UX** → OQ-002 |
| NFR-MNT-01 | Maintainability | Modular domains, documented APIs, automated tests, versioned migrations, structured logs | |
| NFR-OBS-01 | Observability | Central logs, metrics, traces, security alerts, uptime, synthetic booking checks | |
| NFR-DATA-01 | Integrity | FKs, constraints, transactions, idempotency keys, immutable audit/event history | |
| NFR-SCL-01 | Scalability | Large files out of DB; object storage + CDN/pre-signed transfer | |

**Capacity planning baselines** [SAD §14]. These are planning examples, not commitments:

| Stage | Registered | DAU | Peak concurrent | Design RPS | Design API BW | Medical files/day | Storage reserve/yr |
|-------|-----------:|----:|----------------:|-----------:|--------------:|------------------:|-------------------:|
| MVP | 25,000 | 3,000 | 300 | 100 | 44 Mbps | 500 | 1.82 TB |
| Growth | 150,000 | 20,000 | 1,200 | 500 | 260 Mbps | 5,000 | 29.2 TB |
| Scale | 1,000,000 | 100,000 | 5,000 | 2,500 | 1.5 Gbps | 25,000 | 182.5 TB |

**Security release gate** [SAD §13.4, §19]: a weighted security coverage of at least 90%, no critical domain below 0.8, and no unresolved critical or high findings without documented risk approval.

## 8. Environments & Data Rules [SAD §15.3]

| Environment | Data | Purpose |
|-------------|------|---------|
| Local | Synthetic only | Development, automated tests |
| Development | Synthetic/anonymized only | Shared integration |
| Staging/UAT | Synthetic or formally de-identified | Release validation, perf & security tests |
| Production | Real data | Strict access, audit, backups, monitoring, incident response |

## 9. Key Assumptions (labelled, to be confirmed)

| # | Assumption | Basis | Confirm via |
|---|------------|-------|-------------|
| A1 | The product name is SANAD, and "Medical Platform" in the SAD is only a working title. | SAD cover note | — (low risk) |
| A2 | The UX Case Study is the authority on **what** the product offers and for **whom**. The SAD is the authority on **how** to build it safely and on clinical, privacy and payment rules. | The documents' respective purposes | OQ-005 |
| A3 | The operating country is likely Egypt, since all personas are in Cairo. **This hasn't been confirmed**, and the SAD keeps it jurisdiction-neutral. | UX §9 | OQ-004 |
| A4 | "Full product scope" means every feature in both documents. It doesn't automatically include the SAD's exclusions or future enhancements. | User instruction + SAD §2.4/§18.1 | OQ-003 |
