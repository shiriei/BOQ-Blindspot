# BOQ Blindspot — Product Specification

> **Phase 0 Notice**: This specification defines the planned functional and technical requirements for BOQ Blindspot. In Phase 0, no application source code or dependencies are implemented.

---

## 1. Executive Summary & Vision

**BOQ Blindspot** is an offline-capable, local-first commercial software application designed to assist small construction subcontractors in discovering, investigating, and resolving billing discrepancies.

Subcontractors operate in an asymmetric commercial environment where main contractors possess dedicated commercial teams and software tools. Small subcontractors often rely on paper site notes, informal messages, fragmented spreadsheets, and memory. As a result, legitimate work executed on site frequently goes unbilled, approved variations are omitted from interim applications, measurement sheet discrepancies pass unnoticed, and claims are rejected due to missing substantiating evidence.

BOQ Blindspot unifies contractual data, recorded field work, supporting multimedia evidence, and billing histories into a single traceable workspace. It uses open-weight local AI for parsing, speech transcription, OCR, and candidate matching, coupled with a deterministic calculation engine that enforces rigorous arithmetic and commercial rules.

---

## 2. Target Customer & User Personas

### 2.1 Target Audience
- **Primary**: Small-to-medium trade subcontractors (subcontract package values typically between £50k and £2M / $50k to $2.5M).
- **Representative Trades**: Groundworks, reinforced concrete structures, drylining/ceilings, commercial masonry, mechanical and electrical (MEP), painting/finishing, and structural steel erection.
- **Key Users**:
  - Subcontractor Owner / Managing Director
  - Sole-trader Quantity Surveyor (QS) or Commercial Manager
  - Senior Project Engineer / Site Agent compiling interim billing

### 2.2 Core Problem Statement
Subcontractors face significant commercial leakage due to:
1. **Unbilled Completed Work**: Dayworks, extras, and site-measured quantities recorded in field books or photos that fail to make it onto the monthly interim payment application.
2. **Measurement Sheet vs. Billing Divergence**: Differences between actual site measurement books and what is submitted on interim applications.
3. **Overlapping or Duplicate Claims**: Accidentally re-claiming previously certified quantities, leading to payment delays and commercial friction with main contractors.
4. **Unsubstantiated Variations**: Variation instructions referenced in billing without attaching required supporting documentation (site instructions, delivery tickets, signed site sheets).
5. **Arithmetic and Rate Discrepancies**: Inaccurate rate applications, unit mismatches (e.g., linear metres claimed against square metres), or calculation mistakes across multiple revision spreadsheets.

---

## 3. End-to-End User Workflow

The application guides the user through a structured 12-step reconciliation workflow:

```
[1. Create Project]
        │
        ▼
[2. Ingest Contract & BOQ]
        │
        ▼
[3. Review Extracted Contract Scope] ──(Human Verification of Rates/Units/Qty)
        │
        ▼
[4. Ingest Historical Bills & Certs] ──(Baseline of Claimed/Certified to Date)
        │
        ▼
[5. Ingest Site Evidence] ────────────(Photos, Audio Notes, Measurement Sheets, Scans)
        │
        ▼
[6. Local Processing] ───────────────(Local OCR, Transcription, Table Parsing)
        │
        ▼
[7. Fact Extraction & Matching] ─────(Local AI identifies facts and candidate BOQ items)
        │
        ▼
[8. Human Disambiguation Gate] ──────(User resolves ambiguous matches & missing units)
        │
        ▼
[9. Deterministic Reconciliation] ───(Deterministic rules engine executes arithmetic checks)
        │
        ▼
[10. Review Candidate Findings] ─────(Interactive review: inspect evidence, math, rules)
        │
        ▼
[11. Human Disposition] ─────────────(Accept / Reject / Defer / Request Further Evidence)
        │
        ▼
[12. Export Audit Report] ───────────(Export traceable PDF/XLSX summary and evidence index)
```

---

## 4. Functional Requirements

### 4.1 Project & Workspace Management (`projects`)
- **FR-PRJ-01**: The system must allow users to create, view, update, archive, and delete local projects.
- **FR-PRJ-02**: Each project must define its currency, primary measurement standard (e.g., metric/imperial), project title, client (main contractor) name, and contract reference.
- **FR-PRJ-03**: Project data must reside in an isolated SQLite database and managed local file directory.

### 4.2 Document & File Management (`documents`)
- **FR-DOC-01**: Support import of common file formats: PDF (native and scanned), XLSX, CSV, PNG, JPG, JPEG, TIFF, and common audio formats (WAV, MP3, M4A).
- **FR-DOC-02**: All imported files must be verified, assigned a cryptographic SHA-256 hash upon ingestion, and stored immutably in content-addressed local storage.
- **FR-DOC-03**: The original file must never be modified or overwritten by parsing or extraction routines.
- **FR-DOC-04**: The system must detect duplicate file imports based on content hash.

### 4.3 Data Extraction Pipeline (`extraction`)
- **FR-EXT-01**: Native PDF & Spreadsheet Parsing: Extract tabular lines, descriptions, quantities, units, and rates from digital BOQs and spreadsheets.
- **FR-EXT-02**: Local OCR: Extract text and tabular data from scanned paper records, signed measurement sheets, and delivery tickets without sending images to external APIs.
- **FR-EXT-03**: Local Audio Transcription: Transcribe spoken field voice notes into timestamped text transcripts locally on-device.
- **FR-EXT-04**: Schema-Enforced Extraction: Extracted data must validate against strict Pydantic schemas (e.g., item number, description, quantity, unit, unit rate).

### 4.4 Evidence & Provenance (`evidence`)
- **FR-EVI-01**: Maintain a strict chain of custody: every extracted fact (e.g., "50 m² concrete poured on 12/04") must link directly to the source document ID, page/line number or bounding box, and file hash.
- **FR-EVI-02**: Provide manual inspection and correction tools allowing the user to view the original source file side-by-side with extracted values.
- **FR-EVI-03**: Mark extracted facts as `unverified`, `user_verified`, or `rejected`.

### 4.5 Matching Engine (`matching`)
- **FR-MAT-01**: Associate field evidence and measurement records with contractual BOQ items and variation items.
- **FR-MAT-02**: Propose candidate matches using a combination of deterministic identifier matching (e.g., matching item codes like "3.12") and semantic similarity (open-weight embeddings / local LLM).
- **FR-MAT-03**: Candidate matches must include a confidence score and plain-language explanation of why the match was proposed.
- **FR-MAT-04**: Matching is strictly advisory: matches require human confirmation before being consumed by the reconciliation engine.

### 4.6 Contract & Billing Ingestion (`billing`)
- **FR-BIL-01**: Ingest contractual BOQ line items including: Item ID, Reference/Code, Description, Unit, Contract Quantity, Contract Unit Rate, and Total Value.
- **FR-BIL-02**: Ingest Variations (Compensation Events / Change Orders): Variation ID, Description, Status (`draft`, `submitted`, `approved`, `rejected`), Approved Quantity, Approved Rate, and Supporting Reference.
- **FR-BIL-03**: Ingest Billing History (Interim Applications / Valuation Certificates): Period number, Claimed Quantity, Certified Quantity, Paid Quantity, and Period Dates.

### 4.7 Deterministic Reconciliation Engine (`reconciliation`)
- **FR-REC-01**: Execute purely deterministic, auditable rules in Python without relying on model hallucination.
- **FR-REC-02**: Reconcile cumulative quantities across all seven defined quantity states (`Contract`, `Reported`, `Measured`, `Verified`, `Claimed`, `Certified`, `Paid`), specifically enforcing bounds between verified site work ($Q_{verified}$) and cumulative billing claims ($Q_{claimed}$) against authorized contract limits ($Q_{authorized}$).
- **FR-REC-03**: Validate unit compatibility (e.g., flag attempts to reconcile `m²` against `m³` without an explicit conversion factor).
- **FR-REC-04**: Detect overlapping claims across consecutive billing periods.
- **FR-REC-05**: Identify unbilled work: verified measurement records or site notes that have no corresponding claim in the current or prior bills.
- **FR-REC-06**: Flag variations claimed without an approved status or without required substantiating documentation.

### 4.8 Review Interface & Decision Log (`review`)
- **FR-REV-01**: Present generated candidate findings in a dedicated triage interface.
- **FR-REV-02**: Allow the user to assign a disposition to each finding: `Accepted`, `Rejected`, `Deferred`, or `Requires More Evidence`.
- **FR-REV-03**: Capture reviewer notes and record an immutable audit entry with timestamp and user action for every decision.
- **FR-REV-04**: Allow manual override of quantity calculations with mandatory recorded rationale.

### 4.9 Reporting & Export (`reports`)
- **FR-REP-01**: Export structured reconciliation audit reports in PDF and Excel (XLSX) formats.
- **FR-REP-02**: Reports must include: Executive Summary of potential discrepancies, Itemized Findings Table, Rule Application Basis, and Complete Evidence Provenance Index.
- **FR-REP-03**: Any financial discrepancies must be clearly labeled as "Potential Discrepancies for Review" and include explicit calculation assumptions and limitations.

### 4.10 Local AI Runtime (`ai_runtime`)
- **FR-AIR-01**: Provide an abstraction layer over local model execution (e.g., `llama.cpp` / ONNX / Ollama).
- **FR-AIR-02**: Implement a system health check reporting available hardware (RAM, VRAM, CPU threads, model availability).
- **FR-AIR-03**: Ensure the application operates in "manual mode" (parsing and deterministic math) even if the AI runtime is unavailable or disabled.

---

## 5. Non-Functional Requirements

### 5.1 Privacy & Local-First Architecture
- **NFR-PRV-01**: All data storage, file processing, OCR, transcription, and model inference must execute locally on the host machine.
- **NFR-PRV-02**: Core workflows must function completely offline without internet connectivity.
- **NFR-PRV-03**: Network listeners must bind exclusively to `127.0.0.1` / `localhost`.

### 5.2 Determinism & Correctness
- **NFR-DET-01**: Financial arithmetic, quantity aggregations, and rule checks must be 100% deterministic, producing identical outputs for identical inputs.
- **NFR-DET-02**: Floating-point arithmetic for currency and measurements must use fixed-point or precise decimal representations (e.g., Python `Decimal`) to avoid rounding errors.

### 5.3 Security & Untrusted Input Handling
- **NFR-SEC-01**: Treat all imported files (PDFs, spreadsheets, images, audio) and extracted text as untrusted.
- **NFR-SEC-02**: Mitigate spreadsheet formula injection: sanitize CSV/XLSX exports and prevent execution of dynamic formulas (`=CMD`, `@SUM`, etc.) during parsing.
- **NFR-SEC-03**: Prevent path traversal vulnerabilities by using strict UUID-based or hash-based file naming in storage.
- **NFR-SEC-04**: Model prompts must treat extracted document text as data, preventing prompt injection from manipulating extraction schemas or system behavior.

### 5.4 Performance & Resource Constraints
- **NFR-PER-01**: The system must run on standard consumer hardware: 16 GB RAM, modern 4+ core CPU, with or without a dedicated consumer GPU.
- **NFR-PER-02**: Heavy extraction tasks (OCR, audio transcription, model inference) must run asynchronously in background workers, keeping the UI responsive.
- **NFR-PER-03**: Database queries for typical projects (up to 5,000 BOQ items and 1,000 evidence files) must respond in under 200ms.

---

## 6. MVP Scope vs. Future Exclusions

### 6.1 In Scope for MVP
- Single-user local desktop web application.
- Import of tabular BOQ files (Excel XLSX and CSV).
- Ingestion of measurement sheets (Excel/CSV) and digital payment certificates (PDF/Excel).
- Ingestion of site photos and audio notes.
- Local OCR for scanned documents and local transcription for voice recordings.
- Local open-weight model for fact extraction and candidate BOQ matching.
- Deterministic reconciliation engine covering:
  - Cumulative quantity vs. contract + approved variation limits.
  - Identification of recorded site work missing from current bill.
  - Rate divergence between contract and bill line.
  - Overlap with previously certified quantities.
  - Missing variation approval status.
- Interactive human review UI with accept/reject/defer triage.
- Export of PDF and Excel reconciliation audit reports.

### 6.2 Explicit Exclusions (Deferred to Future Versions)
- Direct cloud ERP / accounting integrations (Xero, QuickBooks, Sage, Procore).
- Multi-user collaborative real-time editing / cloud sync.
- Automated generation of legally binding contractual notices or adjudications.
- Complex 3D BIM model (IFC) quantity takeoff integration.
- Automated client billing submission or banking integration.

---

## 7. Success Criteria

1. **Deterministic Accuracy**: Zero calculation discrepancies between the engine's arithmetic and verified manual spreadsheets.
2. **End-to-End Provenance**: 100% of reported findings must link directly to an underlying source document hash and location.
3. **Local Operational Independence**: Complete verification of the core workflow without any internet connection or cloud service dependencies.
4. **Usability & Speed**: A user can ingest a standard 200-item BOQ and 20 site records, review proposed matches, and generate a reconciliation report in under 15 minutes.
5. **Safe Failure**: Any incomplete extraction, corrupt file, or unsupported unit must surface as an explicit unresolved warning; it must never produce an artificial clean audit.

---

## 8. Unresolved Decisions & Open Questions

The following decisions are deliberately open and must be evaluated through Phase 1 discovery and Phase 5 benchmarking:

1. **Target Trade for Initial Pilot Workflow**: Which subcontractor trade (e.g., groundworks vs. drylining vs. electrical) should provide the sample documents for the initial end-to-end prototype?
2. **Local Inference Runtime Engine**: Selection between `llama.cpp` (direct C++ bindings / `llama-cpp-python`), ONNX Runtime, or an external local manager such as Ollama.
3. **Specific Model Family & Quantization**: Selection of the extraction LLM (e.g., Qwen 2.5 7B/14B, Mistral 7B, Llama 3.2 3B) evaluated against commercial licensing (Apache 2.0 / MIT vs. custom commercial restrictions) and CPU-only performance.
4. **Local OCR Technology**: Selection between Tesseract (lightweight, mature), PaddleOCR (high accuracy on tables), or Docling/Surya.
5. **Local Speech-to-Text Engine**: Selection between `faster-whisper` (CTranslate2 backend) and `whisper.cpp` for local voice note transcription.
6. **Desktop Packaging Strategy**: Evaluation of delivery format (PyInstaller standalone backend + Vite frontend, or packaged via Tauri/Electron, or a local browser launcher script).
