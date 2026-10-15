# BOQ Blindspot

> **Local-first, AI-assisted billing reconciliation for small construction subcontractors.**

---

## 1. Product Overview & Problem Statement

In the construction sector, small trade subcontractors (e.g., groundworks, concrete, drylining, MEP, masonry) routinely operate under tight margins and heavy administrative pressure. Project records are frequently fragmented across:
- Contractual Bills of Quantities (BOQ) and price schedules
- Formal and informal variation instructions
- Site measurement books and daywork sheets
- WhatsApp photos, field notes, and voice recordings
- Prior interim payment applications, claims, and valuation certificates

Reconciling these disparate records manually is time-consuming, tedious, and prone to oversight. Subcontractors routinely miss billable work, fail to substantiate variations with required site records, or make clerical errors that trigger client rejections and delayed payments.

**BOQ Blindspot** bridges this gap. It connects contractual BOQ items, variation records, site evidence, and payment histories within a single, locally operated desktop application. It uses local open-weight AI models to assist with document extraction and matching, paired with a transparent, deterministic calculation engine that highlights potential billing discrepancies, unbilled scope, and missing substantiation.

---

## 2. Target Customer & Intended Use

- **Target Customer**: Small-to-medium trade subcontractors, commercial managers, quantity surveyors (QS), and owner-operators in the construction industry.
- **Intended Use**: Preparing and validating monthly interim payment claims, verifying certified valuations against actual site measurements, checking variations for complete evidentiary substantiation, and identifying unbilled work before final account closure.
- **Explicit Human Role**: The software assists investigation; it does not replace the commercial judgment of the contractor. Every finding must be verified by a human reviewer before inclusion in formal billing.

---

## 3. Core Workflow

1. **Project Setup**: Initialize a local project profile with relevant trade conventions and currency/measurement units.
2. **Contract & BOQ Import**: Ingest contractual BOQs, schedules of rates, and contract clauses.
3. **Contract Review**: Verify parsed line items, units, contract rates, and baseline quantities.
4. **Billing History Ingestion**: Ingest prior interim claims, payment certificates, and certified quantities.
5. **Site Evidence Ingestion**: Add mixed evidence, including measurement sheets, delivery dockets, site photos, daily logs, and voice notes.
6. **Local AI Extraction & Candidate Matching**: Local parsers, OCR, and AI models extract structured facts and propose candidate associations between site records and BOQ items.
7. **Human Disambiguation**: Review and confirm ambiguous matches, extracted measurements, or unclassified items.
8. **Deterministic Reconciliation**: Execute the deterministic rules engine to calculate quantity variances, check unit compatibility, detect overlaps, and flag unapproved variations.
9. **Review Discrepancies**: Investigate generated candidate findings with direct links to underlying source records. Accept, reject, defer, or request further evidence.
10. **Audit Report Export**: Generate a verifiable audit report and evidence index for commercial meetings or valuation submissions.

---

## 4. Key Capabilities & Explicit Non-Goals

### Key Capabilities
- **Multi-Source Evidence Linking**: Link unstructured photos, audio notes, and measurement sheets directly to contractual BOQ lines.
- **Deterministic Math & Rule Validation**: Strict arithmetic validation across the seven defined quantity states (`Contract`, `Reported`, `Measured`, `Verified`, `Claimed`, `Certified`, `Paid`) with zero AI hallucination in calculations.
- **Variation & Scope Control**: Track approved vs. pending variations; flag claimed works lacking formal instruction or site substantiation.
- **Local OCR & Speech Processing**: On-device document scanning and audio transcription keeping proprietary commercial data offline.
- **Traceable Audit Findings**: Every discrepancy is backed by an auditable chain of custody linking extracted facts to immutable source files.

### Explicit Non-Goals
- **Not an Automated Billing/Entitlement Engine**: The software never automatically generates or submits legal claims, invoices, or binding adjudications.
- **Not a Cloud SaaS Platform**: The core application does not upload customer files, project data, or commercial rates to external cloud endpoints.
- **Not an AI Oracle**: AI models are never treated as the source of truth for arithmetic, quantity totals, or legal entitlement.
- **Not a General Accounting/ERP Suite**: The application does not manage payroll, general ledgers, tax compliance, or corporate invoicing.

---

## 5. Technology Stack (Proposed)

> **Notice**: The technology stack below represents the proposed architectural baseline for implementation in Phase 2 onward. In the current Phase 0, no code or dependencies are installed.

| Layer | Proposed Technology | Role / Responsibility |
|---|---|---|
| **Frontend** | React, TypeScript, Vite | Desktop-focused, accessible web interface running locally in the browser |
| **Backend** | Python 3.11+, FastAPI | Modular monolith backend exposing REST endpoints on `localhost` |
| **Database** | SQLite with WAL mode | Local structured data storage, foreign key constraints, and migration tracking |
| **File Storage** | Local Managed Storage | Content-addressed storage (SHA-256) for immutable source evidence |
| **Parsers** | `pdfplumber` / `pypdf`, `openpyxl`, Python `csv` | Structured parsing of tabular BOQs, rate schedules, and measurement sheets |
| **Local OCR** | Lightweight open-source OCR (e.g., Tesseract / PaddleOCR / Surya) | On-device text extraction from scanned tickets and paper sheets |
| **Local Audio** | `faster-whisper` / `whisper.cpp` | On-device transcription of field voice notes |
| **Local AI Engine** | Local inference runtime (`llama.cpp` / ONNX / Ollama) | Quantized open-weight model for schema-constrained extraction and matching |
| **Reconciliation Engine** | Pure Python (deterministic) | Rigorous arithmetic, unit validation, quantity progression, and rule checks |
| **Testing** | `pytest`, Playwright / Vitest | Automated unit, integration, and regression test suites |

---

## 6. Privacy & Local-First Principles

1. **Zero External Data Leakage**: By default, customer contracts, rates, site photos, and voice notes never leave the user's local machine.
2. **Localhost Binding**: All backend services bind exclusively to loopback interfaces (`127.0.0.1` / `localhost`).
3. **Immutable Source Preservation**: Original files are treated as read-only evidence. They are hashed upon ingestion and never altered or overwritten.
4. **Data Ownership**: Projects reside in local SQLite databases and local directories, allowing transparent local backup, archive, or deletion without vendor lock-in.

---

## 7. Current Project Status & Phased Roadmap

The project is currently in **Phase 0: Product Definition & Repository Foundation**. No application code has been implemented yet.

| Phase | Title | Status |
|---|---|---|
| **Phase 0** | Product Definition & Repository Foundation | **Current Phase** |
| **Phase 1** | Customer & Document-Format Discovery | Planned |
| **Phase 2** | Local Application Foundation | Planned |
| **Phase 3** | BOQ, Contract & Billing-Data Imports | Planned |
| **Phase 4** | Mixed Evidence Ingestion & Provenance | Planned |
| **Phase 5** | Local Open-Weight AI Integration & Extraction/Matching | Planned |
| **Phase 6** | Deterministic Reconciliation & Finding Generation | Planned |
| **Phase 7** | Human Review, Audit History & Reports | Planned |
| **Phase 8** | End-to-End Evaluation, Security, Packaging & Pilot Validation | Planned |

See [ROADMAP.md](docs/ROADMAP.md) for detailed deliverables, acceptance criteria, and exit conditions for each phase.

---

## 8. Development Setup

> **Note**: Development setup is **not yet available**.
> Application code, virtual environments, package manifests (`package.json`, `pyproject.toml`), and local database migrations will be introduced systematically starting in **Phase 2**.

---

## 9. Documentation Directory

- [Product Specification](docs/PRODUCT_SPEC.md) — Detailed requirements, user personas, functional workflows, and unresolved questions.
- [Architecture Blueprint](docs/ARCHITECTURE.md) — System boundaries, data flow, entity relationships, and trust zones.
- [Reconciliation Rules](docs/RECONCILIATION_RULES.md) — Deterministic math definitions, quantity types, and finding rules.
- [Engineering Standards](docs/ENGINEERING_STANDARDS.md) — Code quality, testing rigor, security, licensing, and workflow rules.
- [Project Roadmap](docs/ROADMAP.md) — Detailed phase-by-phase execution plan.
- [Contributing Guidelines](CONTRIBUTING.md) — Contribution practices, testing expectations, and Git workflow.
