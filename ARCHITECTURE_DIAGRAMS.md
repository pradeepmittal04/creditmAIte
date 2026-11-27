# CredAIble - Architecture Diagrams v0.3

> Status: DRAFT, 2026-11-27. For architects, engineers, and lender infrastructure and security teams.
> Based on `ARCHITECTURE_DRAFT.md` v0.5, `DOMAIN_MODEL.md` v0.3 and the decisions (D-xx) and assumptions (A-xx) in `PROJECT_BRIEF.md`.
> v0.2 corrects the external integration route: Tool Broker, provider adapters, callbacks and lender gateways (DG-09..DG-11).
> v0.3 adds origination channels, applicant links, customer resolution and internet trust zones (2E–2J, 3D; D-45..D-56).
> The Mermaid source in this file is the master. The images in `diagrams/` are exports of it.

| Level | Question it answers | Diagrams |
|---|---|---|
| 1. High-level | What is the product, who uses it and what does it connect to? | [HLD](#1-high-level-architecture) |
| 2. Low-level | What are the components and how do they work together? | [2A components](#2a-component-map), [2B intake and processing](#2b-intake-and-processing), [2C AI call path](#2c-ai-call-path), [2D decision and handoff](#2d-decision-and-handoff), [2E channel intake](#2e-channel-intake-overview), [2F broker](#2f-broker-route), [2G branch](#2g-branch-and-relationship-manager-route), [2H direct](#2h-direct-route), [2I secure link](#2i-secure-link-lifecycle), [2J customer resolution](#2j-customer-resolution-by-registry-mode) |
| 3. Deployment | What runs where in the lender's infrastructure? | [3A topology](#3a-deployment-topology), [3B availability and recovery](#3b-availability-and-recovery), [3C release path](#3c-release-path), [3D trust zones](#3d-trust-zones-and-internet-boundaries) |

How to read all diagrams:

| Style | Meaning |
|---|---|
| Solid border | In the draft design |
| Dashed orange border | Proposed technology, placement or control. Not yet approved |
| Dotted red border | Open question. Not designed |
| Blue | Product component |
| Purple | AI plane |
| Green | Human action or user |
| Yellow | Data store |
| Grey | Outside the product |

---

## 1. High-level architecture

AI produces evidence and recommendations, rules apply lender policy, and a human decides. The whole product runs in one lender's infrastructure.

[Open as image](diagrams/1-high-level.png)

<!-- diagram: 1-high-level -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  EXT["Applicants and brokers<br/>web and mobile"]:::human
  PART["Partner systems<br/>broker CRMs, aggregators, LIXI2"]:::ext
  STAFF["Lender staff and branch RMs<br/>Indexer, Assessor, Checker,<br/>BA Admin, Auditor, Support"]:::human

  subgraph LENDER["Lender infrastructure"]
    subgraph PRODUCT["CredAIble"]
      PUB["Public channels<br/>applicant and broker portals"]:::core
      PARTG["Partner API<br/>mTLS, separate gateway"]:::prop
      INT["Back-office application<br/>branch capture, SAML or OIDC"]:::core
      ORIG["Intake and origination<br/>canonical intake, secure links,<br/>consent, documents, customers"]:::core
      STORE[("Application database and<br/>document repository")]:::data
      FLOW["Workflow orchestrator"]:::core
      AI["AI plane<br/>6 agents, harness, model gateway"]:::ai
      TOOLS["Tool Broker<br/>OCR, forensics, bureau, KYC"]:::ai
      RULES["Rules engine<br/>lender policy, no LLM"]:::core
      HUMAN["Human review and decision<br/>configurable maker-checker"]:::human
      HAND["Downstream handoff"]:::core
      GOV["Configuration and governance"]:::core
      LEDGER["Evidence Ledger<br/>decision trace"]:::data
    end
    IDP["Lender identity providers<br/>staff, and customers optional"]:::ext
    DOWN["Lender downstream systems<br/>contract, funding, servicing"]:::ext
    MASTER["Customer master<br/>core banking or CRM, optional"]:::ext
  end
  MSGP["Email and SMS providers"]:::ext

  MODELS["Model providers<br/>AU processing by default"]:::ext
  PROV["OCR, credit bureau,<br/>DVS and KYC providers"]:::ext

  EXT --> PUB
  PART --> PARTG
  PARTG --> ORIG
  STAFF --> INT
  INT --> ORIG
  ORIG -.->|"customer lookup"| MASTER
  ORIG -->|"links, reminders"| MSGP
  MSGP -.-> EXT
  INT -.->|"federated sign-in"| IDP
  PUB --> ORIG
  INT --> HUMAN
  INT --> GOV
  ORIG -->|"1 submit"| FLOW
  ORIG --> STORE
  FLOW -->|"2 analyse"| AI
  FLOW -->|"3 verify"| TOOLS
  AI --> TOOLS
  FLOW -->|"4 evaluate"| RULES
  FLOW -->|"5 recommendation and evidence"| HUMAN
  GOV -.->|"approved versions"| RULES
  AI --> MODELS
  TOOLS -->|"through adapters, direct or<br/>via lender gateway"| PROV
  HUMAN -->|"6 human decision"| HAND
  HAND -->|"7 REST, events, webhook"| DOWN
  AI -.-> LEDGER
  RULES -.-> LEDGER
  HUMAN -.-> LEDGER
  HAND -.-> LEDGER

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef ai fill:#ede9fe,stroke:#6d28d9,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
```

Lender infrastructure is on-premises or any cloud, with one installation per lender. Dotted arrows are sign-in, configuration, lookups and audit writes. Section 2E shows the channel intake and 3D the internet trust zones.

---

## 2. Low-level architecture

### 2A. Component map

All modules, their main parts and their stores. Arrows between layers show allowed dependencies. 2B to 2D show the detailed calls.

[Open as image](diagrams/2a-components.png)

<!-- diagram: 2a-components -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  subgraph L1["Channels and edge"]
    direction TB
    C1["Applicant and Broker Portals<br/>links, consent, upload, status"]:::core
    C2["Back-office Web App<br/>queues, decision trace,<br/>branch capture, admin"]:::core
    C3["Partner API, LIXI2 later<br/>versioned, idempotent"]:::core
    C4["API Gateway<br/>authentication, scopes, rate limits,<br/>schema validation"]:::core
    C5["Identity broker<br/>staff SAML and OIDC,<br/>separate external realm"]:::prop
    C6["Authorization<br/>RBAC, ABAC, separation of duties"]:::prop
    C1 ~~~ C4
    C2 ~~~ C5
    C3 ~~~ C6
  end

  subgraph L2["Business modules"]
    direction LR
    subgraph L2A["Origination"]
      direction TB
      B0["Intake Service<br/>canonical submission, mapping,<br/>checklist, duplicates"]:::core
      B0A["Applicant Access Service<br/>secure links, one-time codes"]:::core
      B0C["Customer Registry<br/>external, built-in, hybrid"]:::core
      B1["Application and Case Service<br/>parties, credit request,<br/>collateral, document requests"]:::core
      B2["Disclosure and Consent Service<br/>versions, consent checks, notices"]:::core
      B3["Document Service<br/>intake, short-lived access,<br/>repository adapter"]:::core
      B4["Metadata Service<br/>document IDs, regions, lineage"]:::core
    end
    subgraph L2B["Assessment"]
      direction TB
      B5["Workflow Orchestrator<br/>retries, timers, human tasks"]:::core
      B6["Rules and Decision Engine<br/>eligibility, affordability,<br/>referral and checker triggers"]:::core
      B7["Human Review Service<br/>verdict, checker, decision,<br/>unsuitability assessment"]:::core
      B8["Downstream Handoff Service<br/>mapping, channels, reconciliation"]:::core
    end
    subgraph L2C["Control"]
      direction TB
      B9["Configuration and Governance<br/>versions, approval, snapshots,<br/>product catalogue"]:::core
      B10["Evidence Ledger Service<br/>append, hash chain, trace"]:::core
      B11["Metering and Cost Service<br/>usage, budgets, alerts"]:::core
      B12["Retention Hooks<br/>label, export, hold, purge"]:::core
    end
  end

  subgraph L3["AI and integration"]
    direction LR
    subgraph L3A["Agents"]
      direction TB
      A1["Agent Registry<br/>contracts, versions, allow-lists"]:::ai
      A2["Agent Runtime<br/>Classification, Extraction,<br/>Income and Bank Statement Analysis,<br/>Cross-Document Validation,<br/>Fraud Screening, Assessment Summary"]:::ai
    end
    subgraph L3B["Model control"]
      direction TB
      A3["Prompt and Model Registry<br/>approved versions, evaluations"]:::ai
      A4["AI Harness<br/>minimise, guard, validate, ground"]:::ai
      A5["Model Gateway<br/>routing, residency, fallback"]:::ai
    end
    subgraph L3C["Tools"]
      direction TB
      A6["Tool Broker<br/>allow-lists, residency, audit"]:::ai
      A7["Provider adapters, one per provider<br/>own credentials and destinations<br/>OCR, bureau, KYC, notification,<br/>customer master, LIXI2"]:::ai
      A8["Document sandbox<br/>malware scan, render, forensics"]:::ai
    end
  end

  subgraph L4["Data"]
    direction TB
    D1[("Relational database<br/>module schemas, configuration,<br/>outbox")]:::data
    D2[("Ledger<br/>references and keyed hashes")]:::data
    D3[("Erasable personal<br/>data store")]:::data
    D4[("Document repository<br/>S3-compatible")]:::data
    D5[("Trace and large<br/>payload objects")]:::data
    D6[("Encrypted working<br/>copies with TTL")]:::data
    D7[("Workflow history<br/>references only")]:::prop
    D8[("Upload staging<br/>and quarantine")]:::prop
    D1 ~~~ D5
    D2 ~~~ D6
    D3 ~~~ D7
    D4 ~~~ D8
  end

  L1 --> L2
  L2 --> L3
  L2 --> L4
  L3 -->|"scoped, through owning services"| L4

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef ai fill:#ede9fe,stroke:#6d28d9,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
```

Component rules:

- A module writes only its own data and refers to other modules' data by ID.
- Agents never call each other, hold provider or repository credentials, or write business data.
- Only the Model Gateway calls models. Tools are called only through the Tool Broker.
- Only the Tool Broker calls provider adapters. An adapter holds the credentials for its own provider and no other.
- `tenantId` travels in every call, event, row and trace (D-36).

### 2B. Intake and processing

[Open as image](diagrams/2b-intake-processing.png)

<!-- diagram: 2b-intake-processing -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  subgraph INTAKE["Intake"]
    U["Upload through portal,<br/>API or back office"]:::human
    V["Size and type checks"]:::core
    Q[("Durable staging<br/>and quarantine")]:::prop
    ACK["Acknowledge after write to<br/>second failure domain, A-28"]:::core
    MAL["Malware scan and hash"]:::ai
    REJ["Reject and request re-upload"]:::core
    ORI[("Immutable original<br/>in repository")]:::data
    U --> V --> Q
    Q --> ACK
    Q --> MAL
    MAL -->|"unsafe"| REJ
    MAL -->|"clean"| ORI
  end

  subgraph PROCESS["Assessment workflow"]
    SNAP["Start or signal workflow,<br/>snapshot configuration"]:::core
    CON{"Required disclosure<br/>and consent present?"}:::core
    WAIT["Hold and request consent"]:::core
    TB["Tool Broker<br/>allow-list, residency, audit"]:::ai
    FOR["Forensics sandbox<br/>original file, no internet"]:::ai
    OCR["OCR adapter"]:::ai
    CLS["Classification agent"]:::ai
    CC{"Type confident?"}:::core
    IDC["Indexer classifies"]:::human
    EXT["Extraction agent"]:::ai
    CONF{"Confidence and<br/>validation pass?"}:::core
    IDX["Indexer corrects"]:::human
    INC["Income and Bank<br/>Statement agent"]:::ai
    XDOC["Cross-Document<br/>Validation agent"]:::ai
    BUR["Bureau adapter"]:::ai
    KYC["DVS or KYC adapter"]:::ai
    PEP["PEP and sanctions<br/>scope open, PQ-01"]:::open
    PRICE["Rate and fees<br/>source open, DM-01"]:::open
    FRA["Fraud Screening agent<br/>cannot lower a tool signal"]:::ai
    RUL["Rules engine<br/>outcome and reason codes"]:::core
    SUM["Assessment Summary agent<br/>draft only"]:::ai
    REV["Assessor queue"]:::human

    SNAP --> CON
    CON -->|"no"| WAIT
    CON -->|"yes"| TB
    TB --> FOR
    TB --> OCR
    TB --> BUR
    TB --> KYC
    OCR --> CLS --> CC
    CC -->|"no"| IDC
    CC -->|"yes"| EXT
    IDC --> EXT
    EXT --> CONF
    CONF -->|"no"| IDX
    IDX -->|"re-run dependent steps"| INC
    CONF -->|"yes"| INC
    INC --> XDOC --> FRA
    FOR --> FRA
    FRA --> RUL
    BUR --> RUL
    KYC --> RUL
    PEP -.-> RUL
    PRICE -.-> RUL
    RUL --> SUM --> REV
  end

  XP["External provider, reached directly<br/>or through lender ESB or API gateway"]:::ext
  ORI -->|"DocumentReceived"| SNAP
  OCR <-.->|"if external"| XP
  BUR <-.-> XP
  KYC <-.-> XP

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef ai fill:#ede9fe,stroke:#6d28d9,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef open fill:#fee2e2,stroke:#b91c1c,color:#111827,stroke-dasharray:2 3
```

- Every tool request and result passes through the Tool Broker, including calls made by agents. Arrows out of the tools show where results are used.
- Dotted two-way arrows are external calls through the proposed egress proxy (DG-05).
- KYC can be a manual check where lender policy says so (D-26).
- Submission starts the workflow. A new document signals the running one. Bureau and KYC run once per party, not per document.
- Joint and self-employed cases go to manual assessment without an AI recommendation (A-25).
- A failed fraud check is `NOT_ASSESSABLE` and refers the case. It is never treated as clean.
- Any step that exhausts its retries goes to manual review.

### 2C. AI call path

[Open as image](diagrams/2c-ai-call-path.png)

<!-- diagram: 2c-ai-call-path -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  ORC["Workflow Orchestrator<br/>AgentTask with pinned versions"]:::core
  RT["Agent Runtime"]:::ai
  AREG["Agent Registry"]:::ai
  DOC["Document Service<br/>short-lived, scoped access"]:::core

  subgraph HPRE["AI Harness: before"]
    PRE["Pre-call<br/>minimise, mask, injection screen,<br/>delimit untrusted content"]:::ai
  end
  subgraph HPOST["AI Harness: after"]
    POST["Post-call<br/>schema, domain and grounding checks,<br/>confidence"]:::ai
  end

  subgraph MG["Model Gateway"]
    POL["Policy<br/>capability, AU residency, budget"]:::ai
    ROU["Route<br/>timeout, circuit breaker,<br/>approved fallback"]:::ai
    ADP["Provider adapters<br/>hold the model credentials"]:::ai
  end

  PREG["Prompt and Model Registry"]:::ai
  CUST["Customer or product-provided<br/>accounts on Azure, AWS or GCP"]:::ext
  SELF["Self-hosted model<br/>optional, needs GPU"]:::prop
  TB["Tool Broker<br/>per-agent allow-list"]:::ai
  TOOL["OCR adapter,<br/>forensics sandbox"]:::ai
  LED[("Evidence Ledger")]:::data
  MET["Metering"]:::core
  RES["Validated AgentResult<br/>returned to orchestrator"]:::core
  MAN["Manual review"]:::human

  ORC --> RT
  AREG --> RT
  DOC --> RT
  RT --> TB --> TOOL
  RT --> PRE
  PREG --> PRE
  PRE --> POL --> ROU --> ADP
  POL -->|"not allowed or over budget"| MAN
  ADP <-->|"request, response"| CUST
  ADP <-->|"request, response"| SELF
  ADP -->|"response"| POST
  POST -->|"pass"| RES
  POST -->|"fail after bounded retry"| MAN
  POST -.-> LED
  TB -.-> LED
  ADP -.->|"tokens, latency, cost"| MET

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef ai fill:#ede9fe,stroke:#6d28d9,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
```

### 2D. Decision and handoff

[Open as image](diagrams/2d-decision-handoff.png)

<!-- diagram: 2d-decision-handoff -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  REC["Rule outcome, evidence<br/>and draft summary"]:::core
  MK["Maker records verdict,<br/>rationale and assessment"]:::human
  TRG{"Lender rules<br/>require a checker?"}:::core
  CHK["Checker reviews<br/>never the same person as maker"]:::human

  subgraph TX["Single database transaction"]
    GATE{"If approving: assessment current<br/>and not unsuitable?"}:::core
    DEC["Immutable human Decision"]:::core
    OUTB[("Outbox")]:::data
    GATE -->|"yes, or not an approval"| DEC
    DEC --> OUTB
    DEC --> NOT
    NOT["Refusal notice task<br/>if bureau data was relied on"]:::core
  end
  HS["Handoff Service<br/>expiry check, field allow-list,<br/>mapping, idempotency key"]:::core
  REST["REST push"]:::core
  KAF["Kafka-compatible event"]:::core
  WH["Signed webhook"]:::core
  SYS["Lender downstream systems<br/>enforce assessment validUntil"]:::ext
  DLV["DELIVERED<br/>transport confirmed"]:::core
  ACC["ACCEPTED<br/>business confirmed"]:::core
  DLQ["Dead-letter and operations task<br/>decision unchanged"]:::human
  LED[("Evidence Ledger")]:::data

  REC --> MK --> TRG
  TRG -->|"no"| GATE
  TRG -->|"yes"| CHK
  CHK -->|"approve"| GATE
  CHK -->|"return"| MK
  GATE -->|"no: re-assess"| MK
  DEC -.-> LED
  OUTB --> HS
  HS --> REST --> SYS
  HS --> KAF --> SYS
  HS --> WH --> SYS
  HS -->|"expired or retries exhausted"| DLQ
  SYS --> DLV --> ACC
  ACC -.-> LED

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
```

A change to data, documents or verdict after submission invalidates the checker's approval (D-42).

### 2E. Channel intake overview

Every route ends in the same canonical intake (D-45). Draft section 27.3 lists the steps.

[Open as image](diagrams/2e-channel-intake.png)

<!-- diagram: 2e-channel-intake -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart LR
  subgraph ROUTES["Origination routes"]
    direction TB
    BR["Broker portal"]:::human
    BCRM["Broker CRM or aggregator<br/>partner API"]:::ext
    LIXI["Broker platform<br/>LIXI2, later"]:::open
    RM["Branch RM<br/>back-office capture"]:::human
    WEB["Applicant web<br/>and mobile"]:::human
  end

  subgraph EDGE["Gateways"]
    direction TB
    PUBG["public-gateway"]:::prop
    PARG["partner-gateway"]:::prop
    STG["staff-gateway"]:::prop
  end

  subgraph INTAKE["Intake Service"]
    direction TB
    MAP["Translate to canonical<br/>IntakeSubmission"]:::core
    VAL["Validate against pinned<br/>product version"]:::core
    CRE["Create application, parties,<br/>credit request, number"]:::core
    CHK["Document checklist<br/>DocumentRequests"]:::core
    CON{"Every party<br/>consented?"}:::core
  end

  WAITC["AwaitingApplicantConsent<br/>secure link per party"]:::core
  PROC["Processing<br/>section 2B"]:::core
  CUST["Customer Registry<br/>resolve, propose"]:::core
  DUP["Duplicate detection<br/>human review"]:::core

  BR --> PUBG
  WEB --> PUBG
  BCRM --> PARG
  LIXI -.-> PARG
  RM --> STG
  PUBG --> MAP
  PARG --> MAP
  STG --> MAP
  MAP --> VAL --> CRE --> CHK --> CON
  CON -->|"no"| WAITC
  WAITC -->|"consent given"| PROC
  CON -->|"yes"| PROC
  CRE -.-> CUST
  CRE -.-> DUP

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
  classDef open fill:#fef2f2,stroke:#b91c1c,color:#111827,stroke-dasharray:2 3
```

Dotted arrows run in the background and only propose. No bureau, identity or AI processing starts before `Processing` (D-46).

### 2F. Broker route

[Open as image](diagrams/2f-broker-route.png)

<!-- diagram: 2f-broker-route -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'actorBkg':'#dcfce7','actorBorder':'#15803d','noteBkgColor':'#fff7ed','noteBorderColor':'#c2410c','lineColor':'#475569'}}}%%
sequenceDiagram
  autonumber
  actor B as Broker
  participant BP as Broker portal or partner system
  participant G as public or partner gateway
  participant I as Intake Service
  participant A as Applicant Access Service
  participant N as Notification Service
  actor P as Applicant
  B->>BP: Enter application, attest privacy notice given
  BP->>G: Submit with idempotency key
  G->>G: Authenticate broker or partner, check originator
  G->>I: IntakeSubmission
  I->>I: Map, validate, create application, checklist
  I-->>BP: Application number, status AwaitingApplicantConsent
  I->>N: Broker acknowledgement
  I->>A: Issue secure link per party
  A->>N: Link message, no personal data
  N-->>P: Email or SMS with link
  P->>A: Open link
  A->>N: One-time code to verified mobile
  P->>A: Enter code
  A-->>P: Session for this application only
  P->>A: Read disclosures, give consent
  P->>A: Upload outstanding documents
  A->>I: Consent recorded, requests fulfilled
  I->>I: Start processing (2B)
  opt Broker holds documents
    B->>BP: Upload documents
    BP->>G: Stream to quarantine, uploader BROKER
  end
  opt Link expired
    B->>BP: Reissue link
    BP->>A: Supersede old link, issue new
  end
  BP->>G: Poll or receive webhook
  G-->>BP: Status only, no findings
```

The broker's attestation is recorded but is not consent (D-46). Partner systems use the same flow through the partner-gateway (A-37).

### 2G. Branch and relationship manager route

[Open as image](diagrams/2g-branch-route.png)

<!-- diagram: 2g-branch-route -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'actorBkg':'#dcfce7','actorBorder':'#15803d','noteBkgColor':'#fff7ed','noteBorderColor':'#c2410c','lineColor':'#475569'}}}%%
sequenceDiagram
  autonumber
  actor P as Applicant at branch
  actor R as Relationship manager
  participant BO as Back-office app
  participant SG as staff-gateway
  participant CR as Customer Registry
  participant I as Intake Service
  participant A as Applicant Access Service
  R->>BO: Sign in through staff realm, lender network
  R->>BO: Search customer
  BO->>SG: Search
  SG->>CR: Search by name, DOB, customer number
  CR-->>BO: Existing customer or none, prospect
  R->>BO: Capture application, scan documents
  R->>BO: Record identity document sighted
  BO->>SG: Submit, uploader RM
  SG->>I: IntakeSubmission, channel BRANCH
  I->>I: Map, validate, create, checklist
  alt Applicant consents in branch
    BO->>A: Request branch-assisted consent
    A-->>P: One-time code to applicant's mobile
    P->>BO: Applicant enters own code, reads, consents
    A->>I: Consent, capturedVia BRANCH_ASSISTED
  else Applicant consents later
    I->>A: Issue secure link
    A-->>P: Link by email or SMS
  end
  I-->>BO: Application number
  Note over R,I: The RM is the originator and can never be maker or checker (INV-27)
```

The sighting is supporting evidence. A verified identity still needs the electronic check or a document review (D-49).

### 2H. Direct route

[Open as image](diagrams/2h-direct-route.png)

<!-- diagram: 2h-direct-route -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'actorBkg':'#dcfce7','actorBorder':'#15803d','noteBkgColor':'#fff7ed','noteBorderColor':'#c2410c','lineColor':'#475569'}}}%%
sequenceDiagram
  autonumber
  actor P as Applicant
  participant W as Applicant portal
  participant G as public-gateway
  participant ID as identity-broker, external realm
  participant CIDP as Lender customer IdP
  participant A as Applicant Access Service
  participant I as Intake Service
  P->>W: Choose product, start
  alt New applicant
    W->>A: Verify email or mobile
    A-->>P: One-time code
    P->>W: Enter code
  else Existing customer
    W->>ID: Sign in with internet banking
    ID->>CIDP: OIDC federation
    CIDP-->>ID: Customer claims
    ID-->>W: Customer session
    W->>G: Request pre-fill
    G-->>W: Known details, marked PREFILLED
  end
  P->>W: Confirm details, complete form
  P->>W: Read disclosures, consent
  P->>W: Upload documents
  W->>G: Save draft, account created at first save
  W->>G: Submit with idempotency key
  G->>I: IntakeSubmission, channel DIRECT
  I-->>W: Application number
  I->>I: Consent present, start processing (2B)
  opt Later requests
    I->>A: Issue secure link for new request
  end
```

Customer sign-in needs OIDC support at the lender (OQ-17). Without it, existing customers follow the new-applicant path and are matched later (2J).

### 2I. Secure link lifecycle

[Open as image](diagrams/2i-secure-link.png)

<!-- diagram: 2i-secure-link -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'actorBkg':'#dcfce7','actorBorder':'#15803d','noteBkgColor':'#fff7ed','noteBorderColor':'#c2410c','lineColor':'#475569'}}}%%
sequenceDiagram
  autonumber
  participant S as Issuer: intake, broker, RM or staff
  participant A as Applicant Access Service
  participant DB as Link store, token hash only
  participant N as Notification Service
  actor P as Applicant
  participant G as public-gateway
  S->>A: Issue link for application and party
  A->>A: Random token, at least 128 bits
  A->>DB: Save hash, scope, expiry, purposes
  A->>N: Send link, first name only
  N-->>P: Email or SMS
  P->>G: GET link
  G->>A: Exchange token, not logged
  A->>DB: Check hash, expiry, revoked
  A-->>P: Token removed from URL, ask for code
  A->>N: One-time code to verified mobile
  P->>G: Enter code
  G->>A: Verify, count attempts
  alt Code correct
    A-->>P: Short session, one application and party
  else Attempts exhausted
    A->>DB: Lock link
    A-->>P: Ask issuer or self-service reissue
  end
  Note over A,N: Reminders on days 2, 5 and 7. Expiry after 7 days (A-39)
  opt Reissue or revoke
    S->>A: Reissue, withdraw, decide, contact change
    A->>DB: Supersede or revoke at once
  end
```

States are in `DOMAIN_MODEL.md` section 5.3. The application number never opens a session (A-38).

### 2J. Customer resolution by registry mode

[Open as image](diagrams/2j-customer-resolution.png)

<!-- diagram: 2j-customer-resolution -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  P["Party created as prospect<br/>at intake"]:::core
  MODE{"Registry mode"}:::core
  P --> MODE

  subgraph EXTM["EXTERNAL: core banking or CRM is master"]
    E1["Search through<br/>customer-master-adapter"]:::core
    E2["Store external customer ref<br/>on CustomerLink"]:::core
    E3["New customer created by master<br/>at handoff, ID returned"]:::ext
  end

  subgraph BLT["BUILT_IN: CredAIble is master"]
    B1["Search built-in registry"]:::core
    B2["Create Customer and<br/>CustomerIdentifier"]:::core
  end

  subgraph HYB["HYBRID"]
    H1["Search both"]:::core
    H2["Mapping table and sync"]:::core
  end

  MODE -->|"external"| E1
  MODE -->|"built-in"| B1
  MODE -->|"hybrid"| H1

  RULE{"Deterministic rule match?<br/>customer number, verified ID,<br/>verified contact and DOB"}:::core
  E1 --> RULE
  B1 --> RULE
  H1 --> RULE
  RULE -->|"yes"| CONF["Link CONFIRMED<br/>ledgered"]:::core
  RULE -->|"probable"| PROP["Link PROPOSED"]:::core
  RULE -->|"none"| NEW["No match: new customer<br/>at decision or handoff"]:::core
  PROP --> STAFF["Staff confirm or reject<br/>matched fields shown"]:::human
  STAFF -->|"confirm"| CONF
  STAFF -->|"reject"| NEW
  CONF --> E2
  NEW --> E3
  NEW --> B2
  CONF --> H2

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
```

Weak matches are never merged automatically (INV-29). Which branch applies is set once per installation (D-53).

---

## 3. Deployment architecture

The target is generic Kubernetes or OpenShift with S3-compatible storage (D-30, D-35). **The namespaces, deployable grouping and replica counts are proposals.**

### 3A. Deployment topology

[Open as image](diagrams/3a-deployment.png)

<!-- diagram: 3a-deployment -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  NET["Internet<br/>applicants, brokers"]:::ext
  PNET["Partner systems<br/>mTLS"]:::ext
  CBK["Provider callbacks<br/>only where required"]:::ext
  CORP["Lender private network<br/>staff and administrators"]:::ext

  subgraph INFRA["Lender environment"]
    WAF["Public load balancer<br/>and WAF"]:::ext
    PLB["Private load balancer"]:::ext
    PTLB["Partner load balancer"]:::prop

    subgraph K8S["Kubernetes cluster"]
      subgraph NSP["cm-public"]
        PW["portal-web<br/>applicant and broker"]:::prop
        PGW["public-gateway"]:::prop
      end
      subgraph NSB["cm-partner"]
        PTG["partner-gateway"]:::prop
      end
      subgraph NSS["cm-staff"]
        BW["backoffice-web"]:::prop
        SGW["staff-gateway"]:::prop
      end
      subgraph NSI["cm-identity"]
        IDB["identity-broker"]:::prop
      end
      subgraph NSC["cm-core"]
        API["core-api"]:::prop
        WRK["core-worker"]:::prop
        WFE["workflow-engine"]:::prop
        LDS["ledger-service"]:::prop
      end
      subgraph NSA["cm-ai"]
        AGW["agent-worker"]:::prop
        MGW["ai-gateway"]:::prop
        TBK["tool-broker"]:::prop
      end
      subgraph NSD["cm-adapters: one deployment per provider"]
        OAD["ocr-adapter"]:::prop
        BAD["bureau-adapter"]:::prop
        KAD["kyc-adapter"]:::prop
        NAD["notification-adapter<br/>email, SMS"]:::prop
        CMA["customer-master-adapter<br/>optional"]:::prop
        LXA["lixi-adapter, later"]:::prop
      end
      subgraph NSM["cm-models, optional"]
        GPU["self-hosted-model<br/>GPU nodes"]:::prop
      end
      subgraph NSX["cm-sandbox"]
        SBX["document-sandbox<br/>no internet or database"]:::prop
      end
      subgraph NSE["cm-egress"]
        EGR["egress-proxy<br/>destination allow-list"]:::prop
      end
      subgraph NSO["cm-ops"]
        OTL["telemetry-collector<br/>scrubs personal data"]:::prop
      end
    end

    subgraph DATA["Private data zone"]
      PG[("PostgreSQL<br/>proposed, ADR-003")]:::prop
      S3[("S3-compatible storage<br/>documents, payloads")]:::data
      STG[("Staging store<br/>independent of repository")]:::prop
    end

    subgraph PLAT["Lender platform services"]
      IDP["Staff identity provider"]:::ext
      CIDP["Customer identity provider<br/>optional"]:::ext
      CBS["Core banking or CRM<br/>optional"]:::ext
      KMS["KMS or HSM,<br/>secrets manager"]:::ext
      OBS["Monitoring and SIEM"]:::ext
      DST["Downstream systems<br/>and event platform"]:::ext
      LGW["Lender ESB or API gateway<br/>optional"]:::ext
    end
  end

  MOD["Approved models<br/>AU processing"]:::ext
  EXP["OCR, bureau, KYC,<br/>notification providers"]:::ext

  NET --> WAF
  PNET --> PTLB --> PTG
  PTG --> API
  PTG -->|"LIXI2"| LXA
  IDB -.-> CIDP
  CBK -.->|"signed or mTLS"| WAF
  WAF --> PW --> PGW
  WAF --> PGW
  WAF -->|"external realm"| IDB
  CORP --> PLB
  PLB --> BW --> SGW
  PLB -->|"staff realm"| IDB
  PGW --> API
  SGW --> API
  IDB --> IDP
  API --> WFE
  WFE -->|"tasks, polled by workers"| WRK
  API --> LDS
  WRK --> AGW
  WRK --> TBK
  AGW --> MGW
  AGW --> TBK
  MGW --> GPU
  TBK --> SBX
  NSC -->|"private connections"| DATA
  MGW --> EGR
  TBK <-->|"calls, results"| NSD
  NSD --> EGR
  PGW -.->|"callback route,<br/>off by default"| NSD
  WRK --> EGR
  EGR --> MOD
  EGR -->|"direct"| EXP
  EGR -.->|"or via lender"| LGW
  LGW -.-> EXP
  EGR --> DST
  EGR -.->|"customer master"| CBS
  OTL -.-> OBS

  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
```

The environment can be on-premises, AWS, Azure, GCP or another platform. Network policies deny by default. The data zone can be in the cluster or lender-managed. Only the main connections are drawn: key, telemetry and other secondary connections are in the table below.

#### What each deployable contains

A-16 calls for a few deployables with enforced module boundaries, rather than one service per module.

| Deployable | Contains | Scales on | Proposed minimum replicas |
|---|---|---|---|
| portal-web | Applicant and Broker Portals, on separate host names | Requests | 2 |
| backoffice-web | Back-office Web App | Requests | 2 |
| public-gateway | API Gateway for the applicant and broker portals, including the applicant access API and upload streaming | Requests | 2 |
| partner-gateway | API Gateway for partner systems: mTLS, client credentials, partner quotas (A-37) | Requests | 2 |
| staff-gateway | API Gateway for staff | Requests | 2 |
| identity-broker | Staff federation, external realm, optional customer IdP federation | Sign-ins | 2 |
| core-api | Intake, Applicant Access, Customer Registry, Application and Case, Disclosure and Consent, Document, Metadata, Human Review, Configuration and Governance, Agent Registry, Prompt and Model Registry, Metering, Retention Hooks API, authorization | Requests | 3 |
| core-worker | Workflow activities, Rules engine, outbox relay, Handoff, purge and export jobs | Queue depth | 3 |
| workflow-engine | Workflow Orchestrator (ADR-004) | Workflow rate | 3 |
| ledger-service | Evidence Ledger, with its own database role | Writes | 2 |
| agent-worker | Agent Runtime and the six agents | Queue depth, provider quota | 3 |
| ai-gateway | AI Harness, Model Gateway | Calls in flight | 3 |
| tool-broker | Tool Broker. Holds no provider credentials | Calls in flight | 2 |
| ocr-adapter | Adapter for one OCR provider or engine | Calls in flight | 2 |
| bureau-adapter | Adapter for one credit bureau | Calls in flight | 2 |
| kyc-adapter | Adapter for one DVS or KYC provider. Omitted if checks are manual | Calls in flight | 2 |
| notification-adapter | Adapter for one email or SMS provider | Calls in flight | 2 |
| customer-master-adapter | Adapter for the lender's core banking or CRM. Omitted in built-in mode | Calls in flight | 2 if used |
| lixi-adapter | LIXI2 parsing and mapping, isolated (T25). Later phase | Messages | 2 if used |
| document-sandbox | Malware scan, rendering, forensics | Queue depth | 2 |
| self-hosted-model | Optional local model serving | GPU capacity | 2 if used |
| egress-proxy | Outbound allow-list | Connections | 2 |
| telemetry-collector | OpenTelemetry collection and scrubbing | Volume | 2 |

The adapters share one chart template. Deploy one instance per provider in use, including any later third-party fraud provider (D-18). They are extensions (draft section 20), not new core services.

Replica counts provide redundancy only. They are not sizing, which needs the 5x load test (A-33).

#### Allowed connections

Anything not listed is denied, apart from cluster DNS.

| From | To | Proposed protocol | Purpose |
|---|---|---|---|
| Public load balancer | portal-web, public-gateway, identity-broker (external realm) | HTTPS | External traffic and sign-in |
| Partner load balancer | partner-gateway | HTTPS with client mTLS | Partner submissions, documents and status |
| partner-gateway | core-api, identity-broker, lixi-adapter | HTTPS with mTLS | Canonical intake, token keys, LIXI2 mapping |
| lixi-adapter | core-api | HTTPS with mTLS | Mapped canonical submissions |
| public-gateway, partner-gateway | Staging store | S3 API over HTTPS, write-only | Upload streaming to quarantine (A-40) |
| Private load balancer | backoffice-web, staff-gateway, identity-broker (staff realm) | HTTPS | Staff traffic and sign-in |
| portal-web | public-gateway | HTTPS | Portal API |
| backoffice-web | staff-gateway | HTTPS | Staff API |
| public-gateway, staff-gateway | core-api, identity-broker | HTTPS with mTLS | APIs and token keys |
| identity-broker | Staff identity provider, customer identity provider (optional) | SAML 2.0 or OIDC over HTTPS | Federation |
| identity-broker, workflow-engine | Their own databases | PostgreSQL with TLS | State |
| core-api | workflow-engine, ledger-service | gRPC or HTTPS with mTLS | Business operations |
| core-api, core-worker | PostgreSQL | PostgreSQL with TLS | Module schemas, outbox |
| core-api, core-worker | Object and staging stores | S3 API over HTTPS | Documents, export, purge |
| core-worker | workflow-engine, core-api, ledger-service, agent-worker, tool-broker | gRPC or HTTPS with mTLS | Workflow steps. Workers poll the engine |
| core-worker | egress-proxy | HTTPS, Kafka protocol with TLS | Handoff, signed partner webhooks |
| ledger-service | PostgreSQL, object store | PostgreSQL with TLS, S3 API | Ledger and anchors |
| agent-worker | ai-gateway, tool-broker, core-api, ledger-service | gRPC or HTTPS with mTLS | Agent tasks |
| ai-gateway | core-api, ledger-service, self-hosted-model | gRPC or HTTPS with mTLS | Registries, metering, local models |
| ai-gateway | egress-proxy | HTTPS | Models |
| tool-broker | ocr-adapter, bureau-adapter, kyc-adapter, notification-adapter | gRPC or HTTPS with mTLS | Provider calls |
| core-api, core-worker | notification-adapter, customer-master-adapter | gRPC or HTTPS with mTLS | Links, reminders, customer search and handoff acknowledgement |
| ocr-adapter, bureau-adapter, kyc-adapter, notification-adapter, customer-master-adapter | egress-proxy | HTTPS, or the protocol the provider requires, with encryption | Each adapter reaches only its own destination |
| Public load balancer | public-gateway, separate callback route | HTTPS, signed or mTLS | Provider callbacks, off by default |
| Private load balancer | staff-gateway, separate callback route | HTTPS, signed or mTLS | Callbacks relayed by a lender gateway, off by default |
| public-gateway, staff-gateway | Adapters with callbacks enabled | HTTPS with mTLS | Callback delivery |
| Adapters with callbacks enabled | tool-broker | gRPC or HTTPS with mTLS | Verified callback results |
| tool-broker | document-sandbox, core-api, ledger-service | gRPC or HTTPS with mTLS | Tools |
| document-sandbox | core-api | HTTPS with mTLS | Scoped document access and results only |
| egress-proxy | Approved providers, lender ESB or API gateway, downstream systems | HTTPS, Kafka protocol with TLS, approved provider protocols | Outbound traffic |
| Workloads that need keys or secrets | KMS, secrets manager | HTTPS | Workload identity |
| All workloads | telemetry-collector | OTLP with TLS | Telemetry |
| telemetry-collector | Lender monitoring and SIEM | OTLP or lender's protocol | Export |

#### External integration rules

- **Route:** workflow or agent, Tool Broker, provider adapter, egress proxy, then the provider or the lender's gateway. The API Gateway is not on the outbound path.
- **Results:** a synchronous response or polling is the default. Callbacks are enabled for a provider only where it requires them.
- **Callbacks:** the adapter verifies the sender and rejects replays. The Tool Broker matches the result to an outstanding request through core-api, which saves it and signals the workflow. The provider is acknowledged only after the save. A missing callback is reconciled, not treated as a clean result.
- **Retries:** no layer automatically resends a bureau enquiry unless A-30 is met. This covers the workflow engine, adapter, service mesh, egress proxy and lender gateway.
- **Lender gateway:** it does not replace the Tool Broker. The lender confirms the final provider, processing region and payload logging, because CredAIble cannot observe them.

Vendor support has no standing path. Access is time-limited, lender-approved break-glass (A-08).

### 3B. Availability and recovery

[Open as image](diagrams/3b-availability-recovery.png)

<!-- diagram: 3b-availability-recovery -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  LB["Redundant load balancers"]:::ext

  subgraph SITE["Primary site in Australia"]
    subgraph ZA["Failure domain A"]
      PA["Application replicas"]:::core
      DA[("Database primary,<br/>object and staging copy")]:::data
      PA --> DA
    end
    subgraph ZB["Failure domain B"]
      PB["Application replicas"]:::core
      DB[("Database standby,<br/>object and staging copy")]:::data
      PB ~~~ DB
    end
    subgraph ZC["Failure domain C"]
      PC["Application replicas<br/>and quorum member"]:::prop
      DC[("Database standby,<br/>object and staging copy")]:::prop
      PC ~~~ DC
    end
    SYN["Acknowledge only after a synchronous<br/>write in a second domain, A-28"]:::core
  end

  subgraph DR["Recovery site in Australia"]
    BK[("Encrypted backups,<br/>database logs, object replica")]:::data
    KB[("Recoverable keys<br/>and secrets")]:::data
    RS["Redeploy from release,<br/>restore, reconcile, reprocess<br/>RTO 8 hours"]:::core
    SL["Loss of whole primary site<br/>zero-loss method open, DG-01"]:::open
    BK --> RS
    KB --> RS
    RS -.-> SL
  end

  LB --> PA
  LB --> PB
  LB --> PC
  DA --> SYN
  SYN --> DB
  SYN -.->|"or"| DC
  DB -.->|"asynchronous"| BK

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
  classDef open fill:#fee2e2,stroke:#b91c1c,color:#111827,stroke-dasharray:2 3
```

All replicas write to the database primary. Only an up-to-date standby may be promoted. The lender provides the recovery site (A-11).

Failure domains are zones in a cloud, or separately powered rooms or racks on-premises.

### 3C. Release path

[Open as image](diagrams/3c-release.png)

<!-- diagram: 3c-release -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart LR
  subgraph VENDOR["Product vendor"]
    CI["Build, test,<br/>scan"]:::core
    REL["Signed images,<br/>Helm charts, SBOM"]:::core
    CI --> REL
  end
  subgraph CUSTOMER["Lender-controlled"]
    REG["Private registry<br/>verify signatures"]:::ext
    GIT["Configuration repository<br/>lender approval"]:::ext
    GO["GitOps<br/>Argo CD or Flux"]:::prop
    NP["Non-production<br/>tests, agent evaluation"]:::core
    PR["Production<br/>rolling or blue/green,<br/>rollback"]:::core
    TF["Optional Terraform<br/>prerequisite modules"]:::prop
    REG --> GO
    GIT --> GO
    TF -.-> NP
    GO --> NP
    NP -->|"approve"| PR
  end
  REL -->|"lender pulls"| REG

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
```

The vendor does not push into lender environments.

### What changes by target

| Capability | On-premises | AWS | Azure | GCP | Portability |
|---|---|---|---|---|---|
| Kubernetes | Kubernetes or OpenShift | EKS | AKS | GKE | High, with differences in ingress and security policy |
| Database | Self-operated PostgreSQL | RDS or self-operated | Flexible Server or self-operated | Cloud SQL or self-operated | Medium. Failover and replication differ |
| Object storage | Supported S3-compatible store | S3 | Blob has no native S3 API: needs an adapter or separate store | Cloud Storage has partial S3 interoperability: test it | Low to high, by target |
| Keys | HSM or Vault | KMS | Key Vault | Cloud KMS | Medium |
| Edge | Lender WAF and load balancer | AWS WAF, load balancer | Application Gateway WAF | Cloud Armor | Low |
| Models | Approved AU egress, or self-hosted with GPU | Bedrock | Azure OpenAI | Vertex AI | Medium. AU processing is checked per model |

Managed service names are examples and have not been validated for this product.

### 3D. Trust zones and internet boundaries

Each audience enters through its own edge, gateway and identity realm (A-36). Draft section 27.15 has the controls per zone.

[Open as image](diagrams/3d-trust-zones.png)

<!-- diagram: 3d-trust-zones -->
```mermaid
%%{init: {'theme':'base','themeVariables':{'clusterBkg':'#f8fafc','clusterBorder':'#94a3b8','lineColor':'#475569','edgeLabelBackground':'#ffffff'},'flowchart':{'subGraphTitleMargin':{'top':6,'bottom':22}}}}%%
flowchart TB
  subgraph Z1["Z1 Internet: applicants"]
    AP["Applicant browser<br/>or mobile app"]:::human
  end
  subgraph Z2["Z2 Internet: brokers"]
    BRW["Broker browser<br/>MFA"]:::human
  end
  subgraph Z3["Z3 Partner B2B"]
    PS["Broker CRM, aggregator,<br/>LIXI platform, dealer"]:::ext
  end
  subgraph Z4["Z4 Lender network or ZTNA"]
    ST["Staff and branch RMs"]:::human
  end

  subgraph EDGEZ["Edge, lender environment"]
    WAF["WAF and bot protection<br/>public load balancer"]:::ext
    PLB2["Partner load balancer<br/>mTLS termination"]:::prop
    PRV["Private load balancer"]:::ext
  end

  subgraph GWZ["Gateways and identity, no data credentials"]
    PUBG["public-gateway<br/>applicant and broker hosts"]:::prop
    PARG["partner-gateway<br/>OAuth client credentials"]:::prop
    STG["staff-gateway"]:::prop
    IDX["identity-broker<br/>external realm"]:::prop
    IDS["identity-broker<br/>staff realm"]:::prop
  end

  subgraph Z5["Z5 Application core"]
    CORE["core-api: intake, access,<br/>registry, application"]:::core
    WORK["core-worker, agents,<br/>tool-broker, adapters"]:::core
  end

  subgraph Z6["Z6 Data, never internet-reachable"]
    QS[("Staging and quarantine")]:::data
    DBS[("Database, ledger,<br/>repository")]:::data
  end

  subgraph Z7["Z7 Outbound"]
    EGR["egress-proxy<br/>allow-list"]:::prop
    MSG["Email and SMS providers"]:::ext
    PRV2["Bureau, DVS/KYC, OCR,<br/>model providers"]:::ext
    CBS["Core banking or CRM<br/>customer master"]:::ext
    HOOK["Partner webhooks<br/>signed"]:::ext
  end

  subgraph Z8["Z8 Federation"]
    SIDP["Lender staff IdP"]:::ext
    CIDP["Lender customer IdP<br/>internet banking"]:::open
  end

  AP --> WAF
  BRW --> WAF
  PS --> PLB2
  ST --> PRV
  WAF --> PUBG
  WAF --> IDX
  PLB2 --> PARG
  PRV --> STG
  PRV --> IDS
  IDX -.-> CIDP
  IDS -.-> SIDP
  PUBG -->|"mTLS"| CORE
  PARG -->|"mTLS"| CORE
  STG -->|"mTLS"| CORE
  PUBG -->|"upload stream"| QS
  PARG -->|"upload stream"| QS
  CORE --> DBS
  CORE --> QS
  WORK --> DBS
  WORK --> EGR
  EGR --> MSG
  EGR --> PRV2
  EGR --> CBS
  EGR --> HOOK

  classDef core fill:#dbeafe,stroke:#1d4ed8,color:#111827
  classDef human fill:#dcfce7,stroke:#15803d,color:#111827
  classDef data fill:#fef9c3,stroke:#a16207,color:#111827
  classDef ext fill:#e5e7eb,stroke:#4b5563,color:#111827
  classDef prop fill:#fff7ed,stroke:#c2410c,color:#111827,stroke-dasharray:6 4
  classDef open fill:#fee2e2,stroke:#b91c1c,color:#111827,stroke-dasharray:2 3
```

The gateways write uploads to quarantine through a write-only, size-limited path; they cannot read the repository or database (A-40). Customer IdP federation is open until OQ-17 is answered.

| Boundary | Crossing | Control |
|---|---|---|
| Z1 or Z2 to edge | HTTPS from the internet | WAF, bot protection, TLS 1.2+, HSTS, per-host rate limits |
| Z3 to edge | HTTPS from registered partners | mTLS client certificate, IP allow-list where possible, per-client quotas |
| Z4 to edge | Lender network or zero-trust access | Private load balancer, lender device and network policy |
| Edge to gateways | Inside the cluster | Separate host names and gateway policies per zone |
| Gateways to core | mTLS | Token validated, principal type checked against route (applicant, broker, partner, staff) |
| Gateways to quarantine | Write-only upload stream | Size and type limits, quota per link and application |
| Core to data | Private connections | Workload identity, encryption with lender keys |
| Core to outbound | egress-proxy only | Destination allow-list, minimum data, AU processing |
| Identity to federation | Redirects and back-channel | Signed assertions or tokens, audience and replay checks |

---

## 4. Design gaps found while drawing

These are not decided. They need an answer or an ADR.

| ID | Gap | Recommendation | Affects |
|---|---|---|---|
| DG-01 | "No document loss" (D-11) and A-28 cover a second failure domain, but do not say whether losing a whole site or region is included | Promise zero loss for a single failure domain. Offer cross-site zero loss only where the lender funds synchronous replication | 3B, contract, infrastructure minimum |
| DG-02 | A durable workflow engine keeps payload history and may apply its own retention period, which conflicts with A-27 and D-39 | Put references only in history, make its retention customer-set and include it in purge | 2A, ADR-004 |
| DG-03 | The draft scans before storing, but also needs a durable acknowledgement and must accept uploads during a repository outage | Add a replicated, quarantined staging store that is independent of the repository | 2B, 3A, ADR-015 |
| DG-04 | Two failure domains cannot keep quorum and accept synchronous writes through every single-domain failure | Make three failure domains the recommended production minimum. Document a reduced guarantee for two | 3B, A-11 |
| DG-05 | Standard Kubernetes NetworkPolicy cannot allow-list by host name | Route all outbound traffic through an egress proxy, either bundled or the lender's | 3A, security design |
| DG-06 | The default S3-compatible store (D-30) is not native on Azure | Decide between a Blob adapter in Phase 1 and a lender-supplied S3-compatible store | 3A, OP-04 |
| DG-07 | Public and staff traffic share one gateway in the draft's section 3 | Use separate public and private gateways from the same component | 3A, security design |
| DG-08 | The draft names no product for the applicant and broker realm | Select it with ADR-021, with a separate realm and preferably a separate instance | 3A, ADR-021 |
| DG-09 | v0.1 put all adapters in the Tool Broker. Draft section 20 packages them separately or in an adapter host, while section 5 has the broker inject secrets | Deploy one adapter per provider, holding only its own credentials. Shown in v0.2. Confirm against A-16 by ADR and align draft section 5 | 2A, 3A, draft sections 5 and 20 |
| DG-10 | The draft has no inbound path for asynchronous provider results | Poll by default, with a verified callback route only where required. Shown in v0.2. Confirm by ADR | 3A, security design |
| DG-11 | A lender may require provider calls to use its ESB or API gateway, which hides retries, final destination and logging | Support it as an adapter destination behind the Tool Broker, with lender confirmation of all three. Shown in v0.2 | 1, 2B, 3A, A-30, D-31 |
| DG-12 | The draft had one public gateway for applicants, brokers and partner systems | Add a separate partner-gateway and partner load balancer with mTLS. Shown in v0.3. Confirm by ADR-030 | 3A, 3D |
| DG-13 | Gateways need to stream uploads to quarantine, but A-36 forbids storage credentials at the edge | Give gateways a write-only, size-limited credential for the staging prefix only, or a small upload-receiver service in Z5 | 3A, 3D, A-40 |
| DG-14 | Branch-assisted consent needs the applicant's own code on a staff device | Show a separate applicant screen or the applicant's own phone. Confirm the user experience with the lender | 2G, D-46 |
| DG-15 | Customer IdP federation and pre-fill depend on what the lender's internet banking supports | Make it optional and fall back to the new-applicant path (2H) | 2H, OQ-17 |
| DG-16 | LIXI2 support for attachments and status is unconfirmed | Keep the lixi-adapter optional; brokers use the portal until OQ-15 is answered | 2E, 3A, D-55 |
| DG-17 | In external mode the customer ID often exists only after handoff, so pre-decision checks cannot use it | Use the CustomerLink and external reference before handoff, and the returned ID after | 2J, D-53 |
