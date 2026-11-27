# CLAUDE.md

Guidance for Claude Code sessions in this repository.

## What this is

**CredAIble** is an AI-assisted loan origination system for Australian lenders. It starts with non-bank lenders and personal loans, with credit cards and secured lending later. AI agents do the document and analysis work, and people make every credit decision. Each lender runs its own installation in its own cluster.

The product name is CredAIble (D-66). The repo folder is still called `CreditmAIte`. Do not use the old working names CreditMate or CreditmAIte in new material. Identifiers use `credaible` or `CREDAIBLE`.

## Status

Design is complete to v0.1 and **no code exists yet**. The next step is a release 1 build plan, then a walking skeleton. Plan before coding and agree the plan with the user.

## Source documents (read before designing or coding)

| Document | What it is the source of truth for |
|---|---|
| `PROJECT_BRIEF.md` | Product summary, non-negotiables (section 3), the **binding decisions log D-01..D-66** (section 5), assumptions A-xx, open questions OP-xx, glossary |
| `DOMAIN_MODEL.md` | Entities, relationships, state models, invariants, audit integrity |
| `ARCHITECTURE_DRAFT.md` | Logical architecture, agents, Model Gateway, evidence ledger, rules, security, tenancy, ADR index (section 22), channels and intake (section 27) |
| `ARCHITECTURE_DIAGRAMS.md` | Diagrams, deployables and allowed connections (section 3A) |
| `TECHNICAL_DESIGN.md` (TDD) | Technology stack, deployables, Temporal design, extraction pipeline, data, APIs, identity, security, delivery, install profiles, testing, **repository layout (section 22)**, open items TD-01..TD-10 |
| `REFERENCE_POLICY.md` | Draft Australian personal loan credit policy that the rules will encode |
| `diagrams/` | Rendered PNGs. Mermaid sources are inline in the md files |
| `*.docx` | Word exports generated from the md files. Never edit them by hand |

When the documents disagree, the decisions log in `PROJECT_BRIEF.md` wins, then the ADRs, then the TDD.

## Non-negotiables

- **A person decides.** No code path approves or declines credit without a named human action (ADR-002). Never describe the product as making automated approvals or instant decisions.
- **Deterministic route, agentic steps.** Product configuration fixes the workflow. AI works inside steps and may add work, never remove it (D-58).
- **Evidence or rejection.** Every AI output is schema-validated and grounded in a source document region, or it is rejected.
- **AI extracts, rules decide.** Keep AI reasoning separate from deterministic rules (DMN).
- **The lender's environment is the product's environment.** No customer data leaves the lender's installation or passes through a CredAIble-operated service. Telemetry is stripped of personal data.
- **Pluggable dependencies** (AI providers, OCR, storage, identity, bureaus, KYC, notifications), without reducing models to the lowest common denominator.
- **Permissive licences only** in what ships (ADR-037).
- **Contracts first.** Write OpenAPI, AsyncAPI and JSON Schema before code, and generate from them.

## Stack (summary; details in TDD sections 4 and 5)

- **Core:** Java 25, Spring Boot, Spring Modulith (core-api, core-worker, ledger-service, ai-gateway, tool-broker, adapters). Gradle Kotlin DSL with a version catalogue.
- **Workflow:** Temporal on PostgreSQL. KIE DMN for rules. Transactional outbox; no message broker.
- **Data:** PostgreSQL, a schema per module, Flyway. TD-02 (jOOQ or Spring Data JDBC) is still open.
- **AI and documents:** Python only, in sandboxed workers (agent-worker, document-sandbox) with no data credentials. uv, Ruff, mypy strict, pytest. Layered extraction, layers 0 to 5: text layer before OCR, OCR before vision, small models before large ones.
- **Web:** TypeScript and React with Vite (portal-web, backoffice-web). pnpm. No Node.js on the server.
- **Platform:** Keycloak (staff, external and partner realms), OPA, Envoy Gateway, Tink with the lender's KMS, OpenTelemetry. Helm charts with Enterprise, Standard and Evaluation install profiles, plus a preflight check.

## Conventions

- **IDs:** decisions `D-xx`, assumptions `A-xx`, open questions `OP-xx`, ADRs `ADR-xxx`, technical open items `TD-xx`. Never renumber. Add new ones at the end.
- **When a decision changes or is added:** update the decisions log in `PROJECT_BRIEF.md` and the ADR index. Then update any affected document so they stay consistent.
- **Writing style:** Australian English (licence, organisation), plain words, short sentences, tables for comparisons.
- **Git:** work on a branch named `en-N`, then merge to `main`. Commit only when the user asks.
- **Secrets:** never commit them. `.gitignore` covers the common cases.

## Open items that may block coding

See TDD section 24: TD-01 (versions), TD-02 (data access), TD-03 (static analysis tool), TD-04 (approved AU-region models), TD-05 (on-premises OCR), TD-07 (Istio ambient), TD-08 (evaluation thresholds), TD-10 (support policy). Raise the relevant ones when planning a slice. Do not decide them silently.
