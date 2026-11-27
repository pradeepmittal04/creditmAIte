# CredAIble – Project Brief and Handoff Document

> Status: Phase 1 (Requirement Interrogation) complete for round 1; round 2 and architecture-review questions answered or deferred; assumptions A-01..A-35 accepted on 2026-11-22 (D-44). Origination channels, customer identity and product extensibility decided on 2026-11-27 (D-45..D-56, A-36..A-44). Technology stack, process model, packaging and value dashboard decided on 2026-11-27 (D-57..D-65), and the product named CredAIble (D-66); see `TECHNICAL_DESIGN.md`. Ready for detailed design.
> Last updated: 2026-11-27
> Audience: any AI agent or human architect continuing this work. This file is self-contained.

---

## 0. How to use this document (instructions for the receiving AI agent)

1. Read sections 1–9 fully before producing anything. Decisions (section 5) are binding. Assumptions (section 6) are binding unless the user overrides them. Open questions (section 8) must be raised with the user, not silently resolved.
2. Do not claim 100% cloud agnosticism, and do not let an LLM make or be the sole basis of a lending decision.
3. When you make a design choice that depends on an unresolved question, record it as an assumption with an ID and flag it.
4. Keep every design decision traceable: add an ADR entry (section 11) for each significant choice.
5. Regulatory notes in this document are not legal advice and are unverified. Mark any regulatory claim as "to be confirmed with counsel".
6. Current deliverable pointers: `ARCHITECTURE_DRAFT.md` (v0.5 draft, built on this brief; section 27 covers channels and intake), `ARCHITECTURE_DIAGRAMS.md` (v0.3) and `DOMAIN_MODEL.md` (v0.3).
7. Style expected by the user: concise, challenge assumptions, explain why, call out vendor lock-in risk.

### Role to adopt

Principal Enterprise Solution Architect and Product Architect specialising in: enterprise product architecture, agentic AI and multi-agent systems, banking and financial-services technology, loan origination systems, cloud-native and on-premises architecture, Zero Trust security, LLM gateways and model orchestration, document intelligence/OCR/content management, API and integration architecture, AI governance/observability/auditability, and configurable enterprise products.

This is a **commercial software product**, not a one-off application. Optimise for: Configurability, Extensibility, Security, Portability, Model Independence, Vendor Independence, Observability, Auditability, Scalability, Cost Control.

---

## 1. Product summary

| Item | Value |
|---|---|
| Product name | CredAIble (D-66; earlier working names CreditMate and CreditmAIte) |
| Product | Agentic-AI Credit Origination and Assessment Platform, launching with unsecured personal loans |
| Scope | Complete Loan Origination System (LOS). Personal loans first, credit cards next, then vehicle and home loans, all through product configuration (D-54, D-56) |
| Origination channels | Broker (portal, partner API, later LIXI2), lender branch or relationship manager (RM), and direct web and mobile (D-45) |
| Target customers | Australian non-bank lenders (small to medium) at launch; banks later (D-43) |
| Primary users | Back-office loan staff: Indexers, Assessors, Business (BA) Admins, Technical Support |
| Goals | Faster decisions, lower operating cost, better auditability |
| Core principle | AI agents only **recommend**, with recorded reasons and evidence. A **human always makes the final decision.** |
| Deployment | Single-tenant deployment per customer, in the customer's infrastructure: on-premises or any cloud. Designed so a future SaaS offering does not need major rework |
| Jurisdiction | Australia (AU) |

---

## 2. Business problem and context

Design an enterprise-grade Unsecured Loan Application Processing Platform. Specialised AI agents independently perform parts of the loan assessment and produce evidence and recommendations. Humans review and decide. Rules are deterministic and customer-configured; AI does not set policy outcomes.

Candidate agents (not every customer needs every agent; they must be enable/disable-able, configurable, replaceable, independently versioned, assignable to different models, run sequentially or in parallel, and governed by customer lending rules):

Document Classification, OCR/Document Extraction, Document Validation, Identity/KYC Verification, Income Verification, Employment Verification, Bank Statement Analysis, Spending Behaviour Analysis, Fraud Detection, Credit Bureau/Credit Score, Affordability Assessment, Existing Debt/Liability Analysis, Metadata Extraction, Data Consistency/Cross-Document Validation, Risk Assessment, Policy/Eligibility, Explainability, Financial Assessment Report Generation.

---

## 3. Product philosophy and non-negotiables

- Every major dependency is pluggable: AI providers, OCR engines, document stores, identity providers, KYC providers, credit bureaus, fraud services, notification providers, rules engines.
- Model abstraction must **not** reduce everything to the lowest common denominator. Provider-specific capabilities stay accessible through explicit, versioned capability flags and escape hatches.
- AI reasoning is separated from deterministic business rules (e.g. AI extracts monthly salary; a rule checks it against the lender's threshold).
- The final lending decision requires human review and never depends solely on an opaque LLM response.
- Do not assume an LLM recommendation can legally make the final decision.
- Be honest about cloud portability: identify what is practically portable versus where managed services force compromises.

---

## 4. Original requirement areas (19 areas)

Each area is tagged with its status for this product. "P1" = Phase 1, "P2" = Phase 2, "TBD" = not decided.

| # | Area | Key requirements | Status |
|---|---|---|---|
| 1 | Business problem | Agent set above; pluggable, versioned, configurable agents | P1 |
| 2 | Product philosophy | Pluggable dependencies, model abstraction without lowest-common-denominator | P1 |
| 3 | Identity and SSO | Entra ID, Okta, Ping, Keycloak, generic OIDC/OAuth2, SAML; separate authn and authz; RBAC, ABAC, tenant boundaries, machine and workload identities, service-to-service auth | P1 (OIDC and SAML 2.0 for staff, D-41) |
| 4 | API and integration | Gateway, authn/z, rate limiting, throttling, versioning, quotas, validation, audit, idempotency, webhooks/events, partner integrations; sync vs async guidance | P1 |
| 5 | Document storage abstraction | Repository Provider/Adapter; FileNet, S3, Azure Blob/ADLS, GCS, SharePoint, ECM/DMS, on-prem object storage; encryption, retention, versioning, immutability, legal hold, malware scanning, signed URLs, lineage, residency | Adapter interface P1; repository choice TBD (Q13); retention policy is the customer's; export, purge and hold hooks P1 (D-39) |
| 6 | Metadata | Metadata separate from documents (IDs, type, repository object ID, hash, OCR/validation status, agent results, model/prompt versions, lineage, trace IDs, retention class, evidence refs) | P1 |
| 7 | Agentic architecture | Orchestrator, registry, runtime, contracts, workflow state, queues, retries, timeouts, checkpointing, parallelism, dependencies, human-in-the-loop, A2A comms, tool invocation, structured output, deterministic validation, confidence thresholds, escalation, dead-letter, versioning. Differentiate Agent vs Tool vs Workflow vs Model vs Business Rule | P1 |
| 8 | Model Gateway | Per-agent provider/model selection, fallback, timeouts, health, prompt versions, parameters, token limits, structured-output validation, retries, tenant provider config, cost limits, sensitive-data restrictions. Challenge whether universal abstraction is appropriate | P1 |
| 9 | AI Harness / governance | Prompt and model registries, I/O logging, PII redaction, evaluation, hallucination detection, guardrails, prompt-injection protection, drift, regression tests, human overrides, lineage, replay, audit. Define what must NOT be logged | P1 |
| 10 | Audit and explainability | For every AI conclusion: agent, evidence, model/version, prompt version, config, timestamp, confidence, validation result, lending rule, human override, final decision. Evidence Ledger / Decision Trace | P1 |
| 11 | Token and cost management | Tenant → Application → Workflow → Agent → Model; tokens (in, out, cached), cost, latency, retries, failures; quotas, budgets, alerts, cost-based routing, caching; per-application cost example | P1 (tracking), budgets behaviour TBD |
| 12 | Security | Zero Trust and defence-in-depth; encryption, KMS/HSM, secrets, isolation, segmentation, private endpoints, workload identity, WAF, DLP, PII, malware, supply chain, containers, prompt injection, malicious documents, exfiltration, insider threats; lightweight threat model | P1 |
| 13 | Observability | Platform, application, agent, AI levels; correlation Loan Application → Workflow → Agent → Tool → Model/API; logs, metrics, traces, audit events | P1 |
| 14 | Cloud and on-prem | AWS, Azure, GCP; Kubernetes/OpenShift, private networks, proxies, restricted/disconnected; Helm, GitOps, Terraform. No unrealistic claims | On-premises and any customer-managed cloud P1 (D-35); disconnected TBD |
| 15 | Multi-tenancy | Evaluate shared app/DB, schema-per-tenant, DB-per-tenant, dedicated deployment, hybrid | Decision: dedicated deployment per customer (D-12); SaaS-ready design (D-36) |
| 16 | Workflow and lending rules | Configurable Rules/Decision Engine, no source change for policy | P1 |
| 17 | Resilience | Provider outage, OCR failure, corrupt documents, repository outage, bureau timeout, fraud API outage, agent timeout, invalid LLM JSON, token limit, duplicates, interrupted workflow | P1 |
| 18 | Regulatory and responsible AI | Privacy, minimisation, audit, retention, explainability, adverse-decision reasoning, bias monitoring, model risk, consent, human oversight; jurisdiction variance | P1 (AU) |
| 19 | Extensibility | Plugin/provider frameworks for all pluggable categories; customer-specific integration without modifying product core | P1 |

---

## 5. Decisions log (binding)

| ID | Decision | Source |
|---|---|---|
| D-01 | Target market: non-bank lenders | User Q1 |
| D-02 | Product goals: faster decisions, lower operating cost, better auditability | User Q1 |
| D-03 | Product is a complete LOS (not only an assessment add-on) | User Q2 |
| D-04 | Launch loan product: personal loans | User Q3 |
| D-05 | Policy is owned, configured and approved by Business Admins | User Q4 |
| D-06 | AI agents only recommend and never take final decisions; they record why a recommendation is made | User Q4, Q6 |
| D-07 | Jurisdiction: Australia | User Q5 |
| D-08 | Decline/referral outputs include reasons and evidence; final decision is always human | User Q6 |
| D-09 | Personas: Indexers, Assessors, BA Admin, Technical Support Team | User Q7 |
| D-10 | Scale target: ~1,000 applications/day, ~8,000 documents/day | User Q8 |
| D-11 | RTO ~8 hours; RPO: no document loss, system reprocesses after recovery | User Q9 |
| D-12 | Tenancy: start with the simplest model, a dedicated single-tenant deployment per customer | User Q17 |
| D-13 | Deployment: customer-managed cloud or customer-managed Kubernetes | User Q18 |
| D-14 | Model accounts: both customer-provided and product-provided supported | User Q11 |
| D-15 | Human review is mandatory in all scenarios | User Q12 |
| D-16 | ~~Retention, legal hold and deletion are Phase 2~~ **Superseded by D-39** | User Q14 |
| D-17 | Credit scoring uses AU credit bureau integration | User Q15 |
| D-18 | Fraud detection uses internally defined rules (data anomalies, date checks, font inconsistencies, etc.) | User Q15 |
| D-19 | Cross-border data transfer only with approval (interpretation of "Approval"; confirm, see OQ-13) | User Q19 |
| D-20 | Tenant administrators can configure: agents, prompts, models, rules, workflows, integrations, document types ("all") – subject to governance tiers in A-07 | User Q21 |
| D-21 | Document repository question (Q13) deferred by the user; a default is assumed in A-05 | User Q13 |
| D-22 | Phase 1 includes the applicant/broker portal and disclosures; contract/e-signature, disbursement and servicing handoff remain deferred | Round 2 |
| D-23 | No fixed loan amount or term limits are hard-coded in Phase 1; they are customer-configurable lending rules | Round 2 |
| D-24 | Joint and self-employed applicant handling is deferred until lender policy is available | Round 2 |
| D-25 | Indexers handle AI classification/extraction exceptions rather than retaining first-pass manual classification | Round 2 |
| D-26 | Identity verification supports customer-policy selection between electronic DVS/KYC and manual review | Round 2 |
| D-27 | Bank statements use uploaded PDFs in Phase 1; CDR/open-banking feeds are deferred | Round 2 |
| D-28 | Credit-bureau selection is deferred; Phase 1 provides a bureau-neutral adapter contract | Round 2 |
| D-29 | Azure, AWS and GCP model providers must be supportable through the Model Gateway; no single provider is mandatory | Round 2 |
| D-30 | Initial portability target is generic Kubernetes with S3-compatible storage, without requiring one cloud-specific managed stack | Round 2 |
| D-31 | Applicant AI processing is AU-region-only by default; cross-border processing requires explicit customer approval | Round 2 |
| D-32 | ~~Phase 1 audit and assessment records use a 7-year retention classification~~ **Superseded by D-39** | Round 2 |
| D-33 | Small- and medium-amount credit-contract rules are configurable in Phase 1; the product must not silently apply assumptions | Round 2 |
| D-34 | Funding and loan servicing are outside the initial product scope | Round 2 |
| D-35 | The product is deployed in the lender's own infrastructure, either on-premises or on any cloud platform. SaaS is not currently planned (extends D-13) | Review Q-01 |
| D-36 | The D-10 volume is a design target for one lender's deployment. The design must scale and allow a later SaaS offering without major rework; D-12 still applies today | Review Q-01 |
| D-37 | After approval, the information is sent to the customer's downstream systems through selectable integration options, including REST calls and events to Kafka-like platforms. This supersedes the handoff deferral in D-22; funding and servicing themselves remain outside scope (D-34) | Review Q-02 |
| D-38 | With no pilot lender available, the product starts from publicly known, typical lender policies as defaults, held as configurable policy and not hard-coded (D-23) | Review Q-03 |
| D-39 | Retention policy is defined and managed by the customer and is outside the product. The product provides only hooks: customer-set labels, lineage, and audited export, purge and hold APIs. Supersedes D-16 and D-32 | Assumption review, Q-05 |
| D-40 | Fraud detection includes internal AI screening of documents as well as tools: data mismatches, wrong calculations, font inconsistencies, implausible creation dates (e.g. a PDF created long before the pay date it refers to) and similar. The result is a signal, never a verdict. Extends D-18 | Assumption review |
| D-41 | SAML 2.0 SSO for internal users (including Assessors and maker-checker roles) is in Phase 1 alongside OIDC | Assumption review |
| D-42 | Maker-checker on lending decisions is configurable by lender rule (e.g. amount, risk, fraud signal, override). A maker can never be their own checker | Assumption review, Q-06 |
| D-43 | Launch market remains non-bank lenders (D-01); the design must not block APRA-regulated banks later | Assumption review, Q-07 |
| D-44 | Assumptions A-01..A-35 are accepted with the user's updates and the recommended amendments | Assumption review, Q-04 |
| D-45 | Every origination route (broker, branch or RM, direct web and mobile, and later dealer or partner) feeds one canonical intake. Each application records its channel, originating organisation and originating user | Channel review, 2026-11-27 |
| D-46 | The applicant gives privacy and credit-check consent personally, never through a broker or RM. No bureau enquiry, DVS/KYC check or AI processing of applicant data runs before it. Until then the application is `AwaitingApplicantConsent`. Extends A-29 | Channel review |
| D-47 | A secure applicant link opens only after a one-time code sent to the applicant's mobile. It is scoped to one application and one party, expires, can be reissued by the broker or RM, is cancelled on withdrawal or decision, and triggers reminders. An application with outstanding items is proposed for lapse after a configurable period, and a human confirms it (A-26) | Channel review |
| D-48 | Required documents come from a policy-driven checklist per product and applicant. Assessors can raise further document requests at any time through the same link. Each joint applicant gets their own link and gives their own consent | Channel review |
| D-49 | Identity verification is document-based on every route. At a branch the RM records which original document they sighted, and its details still pass the electronic check that the lender's AML/CTF program requires. A sighting alone never sets a verified outcome. Refines D-26 | Channel review |
| D-50 | The person who originates an application (broker user or RM) can never be its maker or checker. Extends D-42 | Channel review |
| D-51 | Direct applicants can start without an account. Email or mobile is verified by one-time code, and the account is created at save or submit. Existing lender customers may sign in through the lender's customer identity provider (internet banking) and receive pre-filled data | Channel review |
| D-52 | Possible duplicate applications across routes are detected and sent for human review. They are never merged or closed automatically | Channel review |
| D-53 | Customer identity is a pluggable Customer Registry with three installation modes: external master (core banking or CRM), built-in registry, and hybrid. Intake creates an application-scoped prospect, and a weak match is never merged automatically. Resolves DM-07 | Channel review |
| D-54 | Credit products are defined as versioned configuration: type, secured or unsecured, term or revolving credit, collateral schema, document checklist, rule set, workflow, agent set, affordability method and handoff mapping. The data model includes Originator and Collateral from the first release | Channel review |
| D-55 | Broker documents arrive in stages: broker portal upload in Phase 1, then a partner API document endpoint, then a LIXI2 inbound adapter at the edge. LIXI is mapped to the canonical intake and never becomes the internal model | Channel review |
| D-56 | Phase 1 delivers personal loans only. Credit cards are the next product and must be addable mainly by configuration, plus card-specific rules and handoff. Vehicle and home loans follow | Channel review |
| D-57 | Two server languages: Java (Spring Boot) for core services, Python only in sandboxed AI and document workers with no data credentials. TypeScript is used in browsers only | Tech stack review, 2026-11-27 |
| D-58 | The route an application takes is deterministic and comes from the versioned product definition. AI agents work inside steps and may add steps from an allowed catalogue, but never skip, reorder or bypass a required step | Tech stack review |
| D-59 | Temporal, self-hosted on PostgreSQL, is the workflow engine. Resolves ADR-004 | Tech stack review |
| D-60 | Installation uses three profiles (Enterprise, Standard, Evaluation) with a defined split of lender-provided, bundled and either-side services, and a preflight check before install and every upgrade | Tech stack review |
| D-61 | CredAIble does not resell model, OCR, bureau or identity providers for now. The lender contracts them directly; CredAIble supplies adapters and evaluation | Tech stack review |
| D-62 | Document extraction is layered: text layer first, OCR only for scans, LLM understanding over text (vision only when needed), then grounding and arithmetic checks, a vision second read and an indexer | Tech stack review |
| D-63 | Consumer Data Right bank data and disconnected installs with self-hosted models are later phases | Tech stack review |
| D-64 | The value dashboard (turnaround, staff effort, cost per application, extraction accuracy against a baseline) is in the first release | Pitch review, 2026-11-27 |
| D-65 | Positioning: an AI-assisted credit origination tool that makes decisions faster, saves staff time and lowers cost, with a person always deciding | Pitch review |
| D-66 | The product name is **CredAIble**. It replaces the working names CreditMate and CreditmAIte in all documents, diagrams and pitch material | Brand review, 2026-11-28 |

---

## 6. Working assumptions (apply unless the user overrides)

Status key: **Accepted** = confirmed by the user (D-44). **Amended** = accepted with a change, and the text shown is the amended text. **Superseded** = replaced by a later decision. **Proposed** = recommended by the architect, not yet confirmed.

| ID | Assumption | Rationale | Status |
|---|---|---|---|
| A-01 | The launch agent set is six: Classification, Extraction, Income and Bank Statement Analysis, Cross-Document Validation, Fraud Screening, Assessment Summary. Other listed agents are later or implemented as tools or rules | Not every capability needs an agent; D-40 | Amended |
| A-02 | Affordability, policy/eligibility and risk-threshold logic live in the rules engine, not in LLMs | Auditability; D-06 | Accepted |
| A-03 | Deterministic fraud checks and credit-bureau calls are **tools**. The Fraud Screening agent uses the fraud tools and adds screening of the document content; it cannot remove or lower a tool's signal | Repeatable checks plus AI screening; D-40 | Amended |
| A-04 | Fraud output is a scored signal for human review, never a verdict | False positives from scans and re-saved PDFs | Accepted |
| A-05 | Default document repository is S3-compatible object storage behind the Document Repository Provider interface. MinIO community edition is not the default, as it is archived | D-30; covers cloud and on-premises stores | Amended |
| A-06 | Audit contract is a recorded **decision trace**, not byte-exact LLM replay | LLMs are non-deterministic; provider versions retire | Accepted |
| A-07 | Configuration governance tiers: (1) self-service: document types and notifications; outcome-affecting items in this tier (rules, thresholds, workflow parameters) also need maker-checker and tests; (2) governed self-service: prompts and model selection, needing golden-dataset regression pass, maker-checker approval, versioning, effective dates; (3) vendor/professional services: new agent types and new integration adapters | Prevent untestable or silently outcome-changing changes | Amended |
| A-08 | Vendor has no standing access to customer environments; support is time-limited, customer-approved, audited break-glass with masked data by default. Offshore access also needs cross-border approval (D-19) | Insider and third-party risk | Amended |
| A-09 | Customer-owned encryption keys (BYOK) | Customer-managed deployment | Accepted |
| A-10 | By default applicant data is sent only to AI services that process it within Australia; an AU endpoint alone is not sufficient | AU residency; D-19, D-31 | Amended |
| A-11 | Customer is responsible for DR infrastructure. The product states the minimum infrastructure needed for no document loss and provides backup, restore and reprocessing capability | Contract clarity; D-11 | Amended |
| A-12 | Customer-managed operations: customer runs production; vendor provides L3 support under A-08 | Operating model | Accepted |
| A-13 | Stored artefacts carry an owner, lineage and an optional customer-set retention label. The product does not interpret the label | Hooks for customer-managed retention; D-39 | Amended |
| A-14 | ~~Phase 1 audit retention period is 7 years~~ The customer defines retention; the product has no default period | D-39 | Superseded |
| A-15 | Bank statements are uploaded PDFs in Phase 1; Consumer Data Right (open banking) feeds are a later pluggable source | Scope control | Accepted |
| A-16 | A modular architecture sized for the stated scale is preferred over a many-microservice platform | ~0.1 docs/sec average; constraint is LLM latency and cost | Accepted |
| A-17 | Product-provided model accounts use AU processing only and require explicit customer approval. Credential custody is documented; only usage counts are exported for bill-back, never applicant data | Resolves tension between D-14 and D-19 | Amended |
| A-18 | Reviewer-oversight metrics (agreement rate, override rate, time-on-case) are captured to detect automation bias | Meaningful human oversight | Accepted |
| A-19 | Staff SSO supports both OIDC and SAML 2.0 in Phase 1 (Keycloak, Entra ID, Okta, Ping, generic) | Internal enterprise deployment; D-41 | Amended |
| A-20 | At budget, alert and require approval; cases continue by manual review while approval is pending; no silent model downgrade | Cost routing must not change policy outcomes | Amended |
| A-21 | The planning baseline is 10–15 minutes of assessor review per application. 21–31 assessors at 1,000 cases/day is a floor, before productive-time allowance, leave and checker effort | D-10, D-25, D-42 | Amended |
| A-22 | Affordability expense policy is configurable for declared, HEM or max(declared, HEM); no fixed expense method is selected at this stage | Lending-policy parameterisation | Accepted |
| A-23 | Regulatory responsibility for ACL, NCCP/RG 209, AML/CTF, privacy and small-credit-contract interpretation remains customer/counsel-led until a RACI is approved | Australia-specific legal advice requires confirmation | Accepted |
| A-24 | Applicants and brokers use a separate external identity realm; brokers see only their own submissions | D-22 adds external users | Accepted |
| A-25 | The schema supports multiple applicants; until policy exists, joint and self-employed cases go to manual assessment without an AI recommendation | D-24 gives no interim behaviour | Accepted |
| A-26 | No adverse outcome is automatic; knock-outs, expiry and system-initiated withdrawal need human confirmation | D-15 | Accepted |
| A-27 | Personal data sits in an erasable store; the audit hash chain holds references and keyed hashes | Allows customer purge without breaking audit integrity | Accepted |
| A-28 | Uploads and decisions are acknowledged only after a durable write replicated to a second failure domain | D-11 | Accepted |
| A-29 | Versioned disclosure and consent records are checked before bureau, DVS/KYC and AI processing where policy requires; wording comes from counsel | D-22 | Accepted |
| A-30 | Bureau enquiries are retried only where provider idempotency or a status check prevents duplicates | Protects the applicant's credit file | Accepted |
| A-31 | Design as if automated-decision transparency applies; provide an inventory of automated steps and the personal data they use, subject to counsel | Obligations commence 10 December 2026 | Accepted |
| A-32 | Build tenant-aware seams now, but no SaaS control plane | D-36 without over-engineering | Accepted |
| A-33 | Load-test at five times the average working-hour rate | D-36 | Accepted |
| A-34 | The first release ships REST, Kafka-compatible and webhook handoff channels and can send every final outcome (approved, declined, withdrawn) | D-37 | Accepted |
| A-35 | The reference policy is inactive in production until the lender's BA Admins review and approve it | Reconciles D-38 with D-05 and D-33 | Accepted |
| A-36 | Internet trust zones are separated. Applicant and broker browsers, partner systems (B2B), staff and branch networks, and outbound providers each enter or leave through their own edge, gateway and identity realm. No internet-facing component holds repository, database or provider credentials | Channel review; extends DG-07 | Proposed |
| A-37 | Partner systems (broker CRMs, aggregators, LIXI gateways) authenticate as machine clients with mTLS and OAuth 2.0 client credentials, registered per originator organisation. A partner can act only for the brokers it is registered for | D-45, D-55 | Proposed |
| A-38 | Emails and SMS carry no personal or application data beyond the applicant's first name, a non-guessable link and the lender's contact details. The application number shown to brokers and applicants is a reference, never an access credential | D-47; email is an untrusted channel | Proposed |
| A-39 | Link and lapse settings are configuration with reference values: link valid for 7 days, reminders on days 2, 5 and 7, one-time code valid for 10 minutes with 5 attempts, lapse proposed 30 days after the last outstanding request. The lender approves them before production (A-35) | D-47 | Proposed |
| A-40 | Applicant and broker uploads stream through the public gateway into the quarantined staging store. The document repository is never reachable from the internet, and no pre-signed repository URL is issued to an external party | D-47, A-28, DG-03 | Proposed |
| A-41 | Branch and RM users are lender staff in the staff realm with an Originator role. Branch devices reach the back office over the lender's private network or zero-trust access, never through the public portal | D-45, D-50 | Proposed |
| A-42 | In built-in registry mode CredAIble is the system of record for customer identity. It must support access and correction requests (APPs 12 and 13) and data-quality controls. It is not a full CRM | D-53 | Proposed |
| A-43 | Matching uses configurable deterministic rules first: lender customer number, verified identity document, or verified contact plus date of birth. Probabilistic scores only propose a match for human confirmation | D-52, D-53 | Proposed |
| A-44 | The canonical credit request supports term credit (amount and term) and revolving credit (limit) from the first release, so credit cards need no schema migration | D-54, D-56 | Proposed |

---

## 7. Challenges raised to the user and their status

| ID | Challenge | Status |
|---|---|---|
| C-01 | "Complete LOS" scope is split: portal and disclosures are in; contracts/e-sign, disbursement and servicing are deferred or excluded | Resolved for Phase 1 by D-22, D-34 and D-37 |
| C-02 | Mandatory human review means 1,000 apps/day requires roughly 21–31 full-time assessors at 10–15 min each; value depends on reducing review time | Planning baseline accepted; validate in a pilot |
| C-03 | Automation bias: reviewers may rubber-stamp AI recommendations | Mitigated by A-18 |
| C-04 | Product-provided model accounts vs customer-managed deployment and AU residency | Mitigated by A-10, A-17; needs confirmation |
| C-05 | "Tenant admins configure everything" is risky in regulated lending | Mitigated by A-07; needs confirmation |
| C-06 | Retention hooks must exist in Phase 1 even though the product owns no retention policy | Resolved by D-39, A-13, A-27 |
| C-07 | Deferring the repository decision is only workable with a named default | Mitigated by A-05 |
| C-08 | Document-forensics fraud checks produce false positives; fabricated documents are increasing | Mitigated by A-04 and D-40; third-party provider kept pluggable |
| C-09 | The stated scale is small; avoid over-engineering | Mitigated by A-16; scale and SaaS readiness are required by D-36 |
| C-10 | Single-tenant per customer means many installations; release/upgrade tooling becomes critical | Open (design) |
| C-11 | Brokers submit on the applicant's behalf but cannot consent for them, so bureau and AI work must wait for the applicant | Resolved by D-46 |
| C-12 | An emailed link is a bearer credential that can be forwarded or phished | Mitigated by D-47 and A-38 |
| C-13 | "Create a customer ID" assumes CredAIble is the customer master, which is not true for lenders with core banking | Resolved by D-53 |
| C-14 | Broker platforms vary in what they send, and LIXI2 attachment support is unconfirmed | Staged by D-55; open as OQ-15 |

---

## 8. Open questions

### 8.1 Round-1 questions not fully answered

| ID | Question | Current handling |
|---|---|---|
| OQ-R13 | Which document sources/repositories must work at launch; who owns originals; are customer-managed and on-prem stores required day one? | User said hold; assume A-05 |
| OQ-R20 | Who pays for model calls and external verification services; what happens at a budget limit (stop, switch model, require approval)? | Not answered; assume A-17, A-20 |
| OQ-R22 | What must an auditor reconstruct later, for how long, and must replay be exact or trace-only? | Trace-only (A-06); retention period is the customer's (D-39) |
| OQ-R23 | Who operates the service and handles incidents; what support access is acceptable? | Not answered; assume A-08, A-12 |
| OQ-R16 | Threat and trust boundaries | User asked for help; assume A-08 to A-10 |

### 8.2 Round-2 questions and decisions (asked on 2026-11-12)

| ID | Area | Question | Decision/status |
|---|---|---|---|
| OQ-01 | Business | Which LOS components are launch scope? | Resolved for Phase 1: portal and disclosures; contracts/e-sign, disbursement and servicing deferred (D-22, D-34) |
| OQ-02 | Lending | Loan range, term and applicant types? | Loan range/term is customer-configurable (D-23); joint/self-employed policy is deferred (D-24); small-credit rules are configurable (D-33) |
| OQ-03 | Business | Current review time and cost baseline? | Planning baseline accepted as 10–15 minutes/application; validate with pilot data (A-21) |
| OQ-04 | Personas | What do Indexers do after AI classification? | Indexers handle classification/extraction exceptions (D-25) |
| OQ-05 | Regulatory | ACL status and regulatory ownership? | Deferred pending customer/counsel RACI (A-23) |
| OQ-06 | Regulatory | KYC/DVS scope and method? | Electronic DVS/KYC and manual review are both supported according to customer policy (D-26) |
| OQ-07 | Lending | Affordability expense benchmark? | Keep deferred; implement declared, HEM and max methods as configurable rules (A-22) |
| OQ-08 | Integration | Which bureau is first? | Deferred; implement bureau-neutral adapter contract first (D-28) |
| OQ-09 | Integration | PDF or CDR bank data? | Uploaded PDFs in Phase 1; CDR deferred (D-27) |
| OQ-10 | Cost | Model billing and budget behaviour? | Customer/product-account support remains; alert and require approval at budget (A-17, A-20) |
| OQ-11 | AI | Which model providers are mandatory? | No single mandatory provider; gateway supports Azure, AWS and GCP (D-29) |
| OQ-12 | Platform | First cloud/Kubernetes target? | Generic Kubernetes with S3-compatible storage (D-30) |
| OQ-13 | Residency | What does approval mean? | AU-only AI processing by default; cross-border requires explicit customer approval (D-31) |
| OQ-14 | Retention | Minimum audit retention? | Superseded: defined and managed by the customer (D-39) |

### 8.3 Architecture-review questions (answered on 2026-11-22)

| ID | Area | Question | Decision/status |
|---|---|---|---|
| Q-01 | Scale | Is 1,000 applications/day per customer or across all customers? | Per lender, as a design target; customer-hosted now, SaaS-ready design (D-35, D-36) |
| Q-02 | Integration | How does an approved loan reach contract and funding? | Sent to downstream systems through selectable options such as REST or events (D-37) |
| Q-03 | Policy | Is a pilot lender available to supply a policy and documents? | No; start from publicly known typical policies (D-38). A source of test documents is still needed |

### 8.4 Assumption review (answered on 2026-11-22)

| ID | Area | Question | Decision/status |
|---|---|---|---|
| Q-04 | Governance | Which assumptions are accepted? | All 35, with the user's updates and the recommended amendments (D-44) |
| Q-05 | Retention | What does the product provide? | Hooks only; the customer owns the policy (D-39) |
| Q-06 | Decision | How many people act on a verdict? | Configurable maker-checker (D-42) |
| Q-07 | Market | Non-banks or banks? | Non-banks now, banks later (D-43) |

### 8.5 Still open

| ID | Item | Needed by |
|---|---|---|
| OP-01 | Name the owner of regulatory responsibility (A-23) | Before build |
| OP-02 | Counsel review of automated-decision transparency (A-31) and consent wording (A-29) | Before launch |
| OP-03 | Source of realistic test documents, including tampered samples for fraud screening | Before agent evaluation |
| OP-04 | Document sources and ownership of originals (OQ-R13) | Before repository adapters beyond S3 |

### 8.6 Channel review (answered on 2026-11-27)

| ID | Area | Question | Decision/status |
|---|---|---|---|
| Q-08 | Channels | Which origination routes must be supported? | Broker, branch or RM, and direct web and mobile; dealer and partner later (D-45) |
| Q-09 | Consent | Can a broker consent for the applicant? | No; the applicant consents personally (D-46) |
| Q-10 | Identity | Does sighting at a branch verify identity? | No. The sighting is recorded, but verification is document-based (D-49) |
| Q-11 | Customer | Who is the customer master? | Configurable: external, built-in or hybrid (D-53) |
| Q-12 | Broker | How do broker documents arrive? | Portal first, then partner API, then LIXI2 (D-55) |
| Q-13 | Product | Are credit cards in Phase 1? | Personal loans first, cards next, by configuration (D-56) |

### 8.7 New open questions

| ID | Item | Needed by |
|---|---|---|
| OQ-15 | Confirm the LIXI2 version, message types and supporting-document support with the main broker platforms and aggregators, and confirm LIXI licensing | Before the LIXI adapter |
| OQ-16 | Which email and SMS providers process data in Australia, and does the lender need its own sender domain and SMS sender ID? | Before secure links go live |
| OQ-17 | Does the lender's customer identity provider (internet banking) support OIDC federation, and which claims can be used for pre-fill? | Before existing-customer sign-in |
| OQ-18 | Who maintains the broker register and accreditation, and is it fed from the lender's CRM or an aggregator? | Before broker onboarding |
| OQ-19 | Under the lender's AML/CTF program, which electronic and document-based methods are acceptable for each channel? | Before identity rules are activated |

---

## 9. Regulatory notes (Australia; unverified, confirm with counsel)

- National Consumer Credit Protection Act 2009 (NCCP): responsible-lending obligations for credit licensees (reasonable inquiries about requirements and objectives, verifying financial situation, assessing unsuitability). ASIC Regulatory Guide 209 provides guidance.
- Credit licensing (ACL), AFCA membership and complaint handling for consumer lending.
- AML/CTF Act 2006 (AUSTRAC): customer identification and record keeping. Recent reforms affect scope; confirm applicability. The lender's AML/CTF program decides which document-based and electronic verification methods apply to each channel (D-49).
- Brokers (credit assistance providers) hold their own ACL or are credit representatives, with their own responsible-lending duties. Mortgage brokers also owe a best-interests duty. None of this transfers the lender's own obligations to the broker, and RG 209 expects lenders to monitor applications from third parties (RL-13 in `REFERENCE_POLICY.md`).
- Credit cards: affordability is assessed on the consumer's ability to repay the credit limit within a period set by regulation (understood to be 3 years), and unsolicited limit-increase offers are prohibited. Confirm before the card product is configured.
- Secured products add security registration (PPSR for motor vehicles), valuation and, for home loans, mortgage registration and settlement. These are later products (D-56).
- Privacy Act 1988 and Australian Privacy Principles; Privacy (Credit Reporting) Code for bureau data; Notifiable Data Breaches scheme.
- Automated decision-making transparency obligations introduced by the Privacy and Other Legislation Amendment Act 2024, expected to commence December 2026 (confirm date and applicability).
- Small-amount and medium-amount credit contract rules may apply depending on loan size and term.
- Consumer Data Right (open banking) is relevant if bank data feeds are used later.
- Record retention: commonly cited 7 years for credit assessment and AML records (confirm). The customer defines and manages retention (D-39).
- Model-risk and fairness practices (bias monitoring, human oversight) are good practice even where not mandated; APRA CPS 230 applies to APRA-regulated entities and may flow down contractually from bank funders.

---

## 10. Glossary

| Term | Meaning |
|---|---|
| Agent | Autonomous unit that reasons over inputs, may call tools, and produces structured evidence and a recommendation |
| Tool | Deterministic or external capability an agent or workflow calls (bureau API, forensics check, calculator) |
| Workflow | Durable, orchestrated sequence/graph of steps with state, retries and human tasks |
| Model | A specific AI/ML model reachable through the Model Gateway |
| Business rule | Deterministic, customer-configured lending policy logic executed by the rules engine |
| Evidence Ledger | Append-only record linking every AI conclusion to its evidence, versions and rule outcomes |
| Decision Trace | Reconstructable record of how a recommendation and a human decision were reached |
| Model Gateway | Layer routing agent requests to providers/models with policy, fallback and metering |
| AI Harness | Control plane around every AI call (prompt registry, guardrails, validation, logging, evaluation) |
| RTO / RPO | Recovery time objective / recovery point objective |
| HEM | Household Expenditure Measure, a published expense benchmark used in AU affordability assessment |
| DVS | Document Verification Service (AU identity document verification) |
| CDR | Consumer Data Right (AU open banking framework) |
| Maker-checker | Control where one person submits (maker) and a different person approves (checker) |
| Originator | The organisation and user that brought an application in: broker firm and broker, lender branch and RM, the direct channel, or later a dealer or partner |
| RM | Relationship manager: lender staff who capture applications for customers, usually at a branch |
| Aggregator | Organisation that provides accreditation, software and lender panels to many broker firms |
| LIXI2 | Lending Industry XML Initiative data standard for Australian lending, used by broker platforms |
| Customer master | The system of record for customer identity, usually core banking or a CRM |
| Prospect | An applicant not yet confirmed as an existing or new customer of the lender |
| Secure link | Single-application, expiring link that opens the applicant's consent and upload pages after a one-time code |
| OTP | One-time passcode sent to a verified mobile or email |
| PPSR | Personal Property Securities Register, where security over vehicles is registered |

---

## 11. Architecture decision records (ADR index)

ADRs are tracked in `ARCHITECTURE_DRAFT.md` section 22. Add new records there and reference decision/assumption IDs from this file.

---

## 12. Work plan for the next agent

1. **Track open items.** Sections 8.1 to 8.4 are answered or assumed; section 8.5 lists what is still open.
2. **Review and refine** `ARCHITECTURE_DRAFT.md` (v0.5). Check it and `TECHNICAL_DESIGN.md` (v0.1) against decisions D-01..D-66 and assumptions A-01..A-44, and confirm or amend the proposed assumptions A-36..A-44.
3. **Produce detailed design artefacts**, in this order:
   - Domain model and metadata schema (ERD)
   - Agent contract and registry specification
   - Model Gateway and AI Harness specification (including logging policy)
   - Evidence Ledger / Decision Trace schema
   - Rules/Decision Engine selection and rule authoring model
   - Document Repository Provider interface
   - API catalogue and event catalogue
   - Security architecture and full threat model
   - Deployment architecture (Helm/GitOps/Terraform layout)
   - Observability and cost model
   - Phase 1 / Phase 2 roadmap and backlog
4. **Validate** the design with a proof-of-concept plan: golden dataset of AU personal-loan documents (payslips, bank statements, ID), accuracy targets per agent, and reviewer-time measurements.
5. **Record** any new decision or assumption with an ID in this file.

---

## 13. Appendix A – Round 1 interrogation questions (summary) and user answers

| # | Question (topic) | User answer (as given, cleaned) |
|---|---|---|
| 1 | Target institutions and differentiator | Non-bank lenders; faster decisions, lower operating cost, better auditability |
| 2 | Assessment assistant, full LOS, or component | Complete LOS |
| 3 | Loan products and applicant types at launch | Starting with personal loans |
| 4 | Deterministic vs AI steps; policy ownership | Business Admins own/configure/approve policy; AI only recommends, never decides, and records why |
| 5 | Jurisdictions | Australia |
| 6 | Content of adverse/referral outputs and accountability | Reasons and evidence; always a human decision |
| 7 | Personas | Indexers, Assessors, BA Admin, Technical Support Team |
| 8 | Scale | ~1,000 applications/day; ~8,000 documents/day |
| 9 | Availability, RTO, RPO | RTO ~8 hours; no document loss; reprocess after recovery |
| 10 | Which tasks need generative AI | User asked the architect to recommend (see A-01 to A-04) |
| 11 | Customer vs product-managed model accounts | Both |
| 12 | Human review points | Mandatory in all scenarios |
| 13 | Document repositories and ownership | On hold |
| 14 | Retention, deletion, legal hold, residency of derived data | Required but Phase 2 |
| 15 | Integrations | AU credit-score bureau; fraud via internal rules (anomalies, date checks, font inconsistencies) |
| 16 | Threat and trust boundaries | Needs architect help |
| 17 | Isolation options | Start with the simplest: dedicated deployment |
| 18 | Deployment targets | Customer-managed cloud or customer-managed Kubernetes |
| 19 | Data residency | "Approval" (interpreted as cross-border transfer with approval) |
| 20 (user numbering) | Configurability | Tenant administrators configure all of: agents, prompts, models, rules, workflows, integrations, document types |
| 21 (user numbering) | Audit and explainability | Needs architect help |
| – | Cost/FinOps, support model | Not answered |

---

## 14. Appendix B – Original prompt, condensed

The user's original prompt asked for a two-phase engagement. **Phase 1:** interrogate requirements (20–30 hard questions grouped under 18 topics) without designing. **Phase 2:** produce the enterprise architecture covering: pluggable AI providers and model abstraction; SSO and authz; API gateway and sync/async guidance; document storage adapters; metadata persistence; multi-agent architecture (orchestrator, registry, runtime, contracts, workflow state, queues, retries, checkpoints, human-in-the-loop, dead-letter, versioning) with clear Agent/Tool/Workflow/Model/Rule distinctions; Model Gateway; AI Harness and governance (including what not to log); Evidence Ledger/Decision Trace; token and cost management with a per-application cost example; Zero Trust security and a lightweight threat model; four-level observability with end-to-end correlation; cloud and on-prem deployment with honest portability limits; multi-tenancy options; rules/decision engine; resilience; regulatory and responsible-AI controls; and a plugin/extensibility framework allowing customer-specific integrations without changing the product core.