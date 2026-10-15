# BOQ Blindspot — Phased Implementation Roadmap

> **Process Rule**: Development proceeds strictly phase-by-phase. Do not implement future phases prematurely. Each phase requires defined deliverables, automated tests, verified acceptance criteria, and explicit user signoff before moving to the next.

---

## Roadmap Overview

```
Phase 0 ──▶ Phase 1 ──▶ Phase 2 ──▶ Phase 3 ──▶ Phase 4 ──▶ Phase 5 ──▶ Phase 6 ──▶ Phase 7 ──▶ Phase 8
(Docs &     (Trade &    (Local      (BOQ &      (Evidence   (Local AI   (Determ.    (Review &   (E2E, Sec &
 Found.)     Formats)    Found.)     Billing)    Store)      Runtime)    Recon.)     Reports)    Packaging)
```

---

## Phase 0: Product Definition & Repository Foundation (Current Phase)

- **Objective**: Establish the master specification, architectural blueprints, reconciliation rules, engineering governance, repository hygiene, and Git ignore rules before writing code.
- **Deliverables**:
  - `README.md`
  - `docs/PRODUCT_SPEC.md`
  - `docs/ARCHITECTURE.md`
  - `docs/RECONCILIATION_RULES.md`
  - `docs/ENGINEERING_STANDARDS.md`
  - `docs/ROADMAP.md`
  - `.gitignore`
  - `CONTRIBUTING.md`
- **Dependencies**: Master project prompt / context.
- **Tests**: Internal documentation link verification, consistency checks across all specification documents, `.gitignore` coverage validation.
- **Acceptance Criteria**:
  - All 8 foundational documents created and cross-referenced.
  - Zero application code written or premature dependencies installed.
  - No Git commands executed by assistant; manual user workflow documented.
- **Exit Condition**: Explicit human review and approval of Phase 0 documentation baseline.

---

## Phase 1: Customer & Document-Format Discovery

- **Objective**: Define the specific subcontractor pilot trade (e.g., groundworks or drylining), assemble representative real-world document samples (BOQ, measurement sheet, interim application, payment certificate, site photos), and codify edge cases.
- **Deliverables**:
  - `docs/discovery/PILOT_TRADE_PROFILE.md`: Trade-specific measurement conventions, billing cadence, and standard contractual clauses.
  - `docs/discovery/SAMPLE_DOCUMENT_SPECS.md`: Analysis of real-world spreadsheet layouts, PDF structures, and field notes.
  - Anonymized sample fixture suite in `tests/fixtures/` covering:
    - 1x Baseline contractual BOQ (Excel & PDF)
    - 1x Interim payment application with prior certified quantities
    - 2x Site measurement sheets
    - 2x Unstructured field notes / photos with measurement claims
    - 1x Approved variation document
- **Dependencies**: Phase 0 signoff.
- **Tests**: Structural schema validation of sample fixtures against domain entities.
- **Acceptance Criteria**:
  - Concrete trade chosen and documented.
  - Realistic fixtures in place with zero synthetic or hallucinated financial claims.
  - Clearly articulated sample discrepancies to be detected by later phases.
- **Exit Condition**: Approval of pilot trade profile and test fixture suite.

---

## Phase 2: Local Application Foundation

- **Objective**: Establish the core local runtime: FastAPI backend skeleton, SQLite database layer with migration tooling, local file storage manager with content hashing, and minimal React + Vite frontend shell.
- **Deliverables**:
  - Backend project structure (`backend/` with `projects`, `documents`, and storage modules).
  - Database layer: SQLite configuration, connection management, schema migrations (`alembic`).
  - Storage engine: Content-addressed immutable file store (SHA-256 calculation and storage).
  - Frontend shell (`frontend/`): React, TypeScript, Vite, desktop layout, API client.
  - Health check endpoint (`GET /api/health`) reporting system status and storage integrity.
- **Dependencies**: Phase 1 approval.
- **Tests**:
  - Unit tests for SHA-256 hashing and file deduplication.
  - Integration tests for SQLite migration lifecycle (up/down).
  - Backend API endpoint tests for project CRUD and file upload.
- **Acceptance Criteria**:
  - Server binds strictly to `127.0.0.1`.
  - Can create a project, upload an arbitrary file, verify its hash in storage, and retrieve project metadata via frontend.
- **Exit Condition**: Passing test suite, clean local build, and user verification of local runtime.

---

## Phase 3: BOQ, Contract & Billing-Data Imports

- **Objective**: Implement parsers and data models for contractual BOQs, schedules of rates, approved variations, and historical billing applications.
- **Deliverables**:
  - `backend/modules/billing/`: Models for `BOQItem`, `Variation`, `Bill`, and `BillLine`.
  - Parsers for tabular XLSX, CSV, and digital PDF formats.
  - Import verification UI: Screen allowing user to view, edit, and confirm imported BOQ lines, units, and rates.
- **Dependencies**: Phase 2 foundation.
- **Tests**:
  - Unit tests parsing complex multi-sheet Excel BOQs with merged headers.
  - Unit tests rejecting invalid or non-numeric rates and quantities.
  - Test verifying formula sanitization on imported CSVs.
- **Acceptance Criteria**:
  - Ingestion of sample Phase 1 BOQ achieves 100% line item fidelity.
  - Quantities and rates stored as exact decimals without floating-point inaccuracies.
  - User can review and adjust parsed lines in UI before committing.
- **Exit Condition**: Successful import and human verification of Phase 1 contractual fixtures.

---

## Phase 4: Mixed Evidence Ingestion & Provenance

- **Objective**: Ingest unstructured site evidence (photos, voice recordings, signed paper scans, daily logs), establish immutable provenance records, and provide human inspection/verification tools.
- **Deliverables**:
  - `backend/modules/evidence/`: Models for `EvidenceRecord`, `MeasurementRecord`, and `ExtractedFact`.
  - File viewer in frontend: Side-by-side inspection showing original PDF/image/audio alongside extracted text.
  - Manual fact entry/correction interface for site measurements.
- **Dependencies**: Phase 3 completion.
- **Tests**:
  - Provenance integrity test: Assert every fact points to a valid file hash and location.
  - Test upload of image, PDF, and audio formats; assert original file bytes remain identical on disk.
- **Acceptance Criteria**:
  - User can upload mixed evidence files, preview them in the browser, and record verified measurements manually.
- **Exit Condition**: Provenance verification test suite passing.

---

## Phase 5: Local Open-Weight AI Integration & Extraction/Matching

- **Objective**: Integrate local inference runtime (OCR, STT, and quantized open-weight LLM) to automate fact extraction from documents/audio and propose candidate matches to BOQ items.
- **Deliverables**:
  - `backend/modules/ai_runtime/`: Model loader, hardware resource check, and inference abstraction.
  - Local OCR worker (e.g., Tesseract / PaddleOCR) and speech worker (`faster-whisper`).
  - Structured extraction prompt pipeline validating outputs against Pydantic schemas.
  - Candidate matching engine proposing `BOQMatch` records with confidence scores.
  - Disambiguation UI: Screen for user to accept, reassign, or reject candidate matches.
- **Dependencies**: Phase 4 completion, model license evaluation.
- **Tests**:
  - Mocked and live tests for structured JSON output validation.
  - Schema error handling tests (graceful recovery when model returns malformed text).
  - Offline execution test (assert zero outbound network traffic during inference).
- **Acceptance Criteria**:
  - Core pipeline operates without internet connection.
  - Model extraction achieves target accuracy on Phase 1 sample fixtures.
  - Candidate matches are strictly advisory and require human confirmation.
- **Exit Condition**: Verified local inference benchmark and human match signoff.

---

## Phase 6: Deterministic Reconciliation & Finding Generation

- **Objective**: Implement the pure Python deterministic reconciliation engine executing the full rules catalog (REC-01 through REC-07) to generate traceable findings.
- **Deliverables**:
  - `backend/modules/reconciliation/`: Pure mathematical engine independent of AI and web framework.
  - Implementation of rules: Over-claim, Unbilled Work, Rate Mismatch, Certification Deductions, Unit Incompatibility, Duplicate Claims, and Missing Approvals.
  - Strict reconciliation across all seven (7) defined quantity states (`Contract`, `Reported`, `Measured`, `Verified`, `Claimed`, `Certified`, `Paid`).
  - Finding generator producing structured `Finding` and `FindingEvidence` records.
- **Dependencies**: Phase 3 (Billing data) and Phase 4/5 (Verified facts & matches).
- **Tests**:
  - 100% coverage unit test suite for all rules using synthetic and real trade fixtures.
  - Strict decimal arithmetic tests verifying zero rounding errors.
  - Negative tests: Incompatible units trigger warnings, missing rates flag unquantified status, never invent missing numbers.
- **Acceptance Criteria**:
  - Engine correctly identifies all intentional discrepancies embedded in Phase 1 fixtures.
  - Engine produces zero false clean audits on incomplete input data.
- **Exit Condition**: 100% passing reconciliation test suite.

---

## Phase 7: Human Review, Audit History & Reports

- **Objective**: Build the interactive triage review interface, immutable audit decision logging, and exportable PDF/Excel reconciliation reports.
- **Deliverables**:
  - Review UI: Triage dashboard showing candidate findings, source evidence previews, calculation breakdowns, and accept/reject/defer actions.
  - `ReviewDecision` audit logging: Persist reviewer notes, timestamps, and manual overrides.
  - Report generator (`backend/modules/reports/`): Compile professional, traceable PDF audit reports and Excel workbooks with hyperlinked evidence indices.
- **Dependencies**: Phase 6 reconciliation engine.
- **Tests**:
  - End-to-end test of decision workflow: finding triage → status transition → audit log verification.
  - Report generation tests verifying exported totals match database records exactly.
- **Acceptance Criteria**:
  - Exported reports clearly mark all findings as "Candidate Discrepancies for Review" with documented calculation assumptions.
  - Report evidence index allows an auditor to trace every line item back to original source file hashes.
- **Exit Condition**: Approval of sample exported PDF and Excel audit reports.

---

## Phase 8: End-to-End Evaluation, Security, Packaging & Pilot Validation

- **Objective**: Conduct comprehensive security hardening, resource optimization, end-to-end integration testing, local desktop packaging, and validation against a complete real-world pilot scenario.
- **Deliverables**:
  - Security audit report: Verification of localhost binding, path traversal prevention, formula injection sanitization, and untrusted input isolation.
  - Resource benchmark: Verification of CPU/RAM/VRAM consumption on consumer hardware.
  - Packaging: Portable local distribution bundle or installer.
  - Pilot validation report documenting full execution of a realistic subcontractor billing cycle.
- **Dependencies**: Phases 0 through 7.
- **Tests**:
  - Full end-to-end automated regression suite (Playwright + Pytest).
  - Performance and memory leak tests across repeated audit runs.
  - Complete air-gapped run on an offline workstation.
- **Acceptance Criteria**:
  - Application runs completely offline on target hardware without performance degradation.
  - Zero critical or high-severity security vulnerabilities.
  - Successful pilot workflow execution with verified user signoff.
- **Exit Condition**: Final acceptance and project signoff for commercial packaging.
