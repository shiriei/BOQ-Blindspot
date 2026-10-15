# BOQ Blindspot — Engineering & Development Standards

> **Phase 0 Notice**: This document outlines the engineering, security, and quality governance for BOQ Blindspot. In Phase 0, no code or test suites are implemented.

---

## 1. Core Engineering Principles

1. **Correctness Over Convenience**: In construction billing, an incorrect formula or silent data truncation can lead to contractual disputes, false claims, or severe financial loss. Accuracy and verification always take precedence over superficial automation.
2. **Explicit Over Implicit**: Missing values, unverified OCR, and incompatible units must be raised explicitly. Never coerce `None` or missing rates to `0.0` or guess units.
3. **Deterministic Math Isolation**: Financial and arithmetic logic must remain completely independent of external dependencies, network access, and probabilistic AI models.
4. **Strict Quantity State Segregation**: Preserve and distinguish between all seven (7) defined quantity states (`Contract`, `Reported`, `Measured`, `Verified`, `Claimed`, `Certified`, `Paid`). Arithmetic functions and schema models must explicitly declare the quantity states they process.
5. **Step-by-Step Approval**: Do not start any new implementation phase without explicit review and approval from the project lead.

---

## 2. Testing Expectations & Acceptance Criteria

Every functional module must be supported by automated tests before merging.

### 2.1 Backend Testing (Python / Pytest)
- **Unit Testing**:
  - Pure unit tests for the reconciliation engine, dimensional analysis, currency calculations, and data parsing.
  - Reconciliation rules must be 100% testable without initiating a database connection or loading an AI model.
  - Floating-point financial calculations must use Python's `Decimal` module, tested against exact penny/cent precision.
- **Integration Testing**:
  - Test SQLite transactions, foreign key constraints, file ingestion pipelines, and schema migrations.
  - Verify that rolling back an aborted audit run leaves the database in a consistent state.
- **Regression Testing**:
  - Every resolved bug or extraction error must have a dedicated test case reproducing the failure before the fix is applied.

### 2.2 Frontend Testing (Vitest / Playwright)
- Unit tests for data transformation utilities, currency/unit formatters, and table components.
- End-to-end integration tests (via Playwright) for critical user journeys: Project creation → File import → Review triage → Report export.

### 2.3 Acceptance Criteria Gates
- 100% passing test suite across all active modules.
- Zero unhandled exceptions on malformed, corrupted, or password-protected test files.
- Test runs must execute locally in under 60 seconds for unit test suites.

---

## 3. Code Quality, Modularity & Error Handling

### 3.1 Python Backend Standards
- **Typing**: Strict type annotations (`mypy` compliant) on all function signatures and domain models.
- **Data Validation**: Use `Pydantic` v2 for all API request/response payloads and internal boundary contracts.
- **Linting & Formatting**: Follow `ruff` and `black` standards (PEP 8, line length 100).
- **Error Handling**: Custom domain exceptions (e.g., `UnitMismatchError`, `MissingContractRateError`, `EvidenceHashMismatchError`). Never use bare `except:` clauses.
- **Logging**: Structured logging using standard Python `logging`. Log event context, execution time, and file hashes. **Never log proprietary financial rates, subcontractor personal data, or private contract text in log files.**

### 3.2 Frontend Standards
- **TypeScript**: Strict mode enabled (`noImplicitAny: true`).
- **State Management**: Predictable, localized state; server state synchronized via clear REST hooks.
- **Component Design**: Accessible, responsive, desktop-optimized UI designed for dense data tables and side-by-side evidence inspection.

---

## 4. Security, Privacy & Local-First Governance

### 4.1 Localhost Isolation
- Backend services must bind exclusively to `127.0.0.1` (loopback). Listening on `0.0.0.0` is strictly forbidden.
- Cross-Origin Resource Sharing (CORS) must be restricted exclusively to the local Vite development or local desktop origin.
- No outbound telemetry, analytics, or automated error reporting to third-party servers.

### 4.2 Handling Untrusted Uploads & Extracted Text
- **File Ingestion**:
  - Validate MIME types and magic bytes, not just file extensions.
  - Store files under their SHA-256 digest in a managed directory isolated from system executable paths.
- **Spreadsheet Formula Injection**:
  - Treat all cell values in uploaded CSVs and Excel files as untrusted text.
  - When parsing or exporting to CSV/XLSX, escape leading formula characters (`=`, `+`, `-`, `@`, `\t`, `\r`) to prevent DDE/formula execution in Excel.
- **Prompt Injection Defense**:
  - When passing extracted OCR or document text to a local LLM for structured extraction, wrap the content in explicit data boundaries (e.g., `<untrusted_document_content>`).
  - Instruct the model strictly that text inside data boundaries must never be interpreted as system instructions or override schema formatting.

---

## 5. Dependency & Model Licensing Governance

### 5.1 Permissible Open-Source Licenses
Because BOQ Blindspot is intended as a commercial software product, all dependencies and third-party libraries must carry commercially permissive licenses:
- **Allowed**: MIT, Apache 2.0, BSD-2-Clause, BSD-3-Clause, ISC.
- **Conditional / Review Required**: LGPL (only dynamic linking; subject to explicit review), MPL-2.0.
- **Strictly Prohibited**: GPL v2/v3, AGPL v3, or licenses with non-commercial restrictions (e.g., CC-BY-NC) that taint proprietary distribution.

### 5.2 AI Model Weight Licenses
Open weights do not guarantee commercial distribution rights. Before selecting any local model (LLM, OCR, or STT):
- Verify the exact model license (e.g., Apache 2.0, MIT, Llama 3 Community License, Mistral Research vs Commercial terms).
- Confirm absence of non-commercial clauses, revenue thresholds, or field-of-use restrictions that would prevent distribution to subcontractors.

---

## 6. Git, Branching & Commit Workflow

The repository is managed **manually by the user**. AI assistants must not execute Git commands, create branches, or push code.

### 6.1 Branching Strategy
- `main`: Production-stable, fully tested documentation and code. Protected branch.
- `feature/<phase-number>-<short-description>`: Active development branches for individual phases (e.g., `feature/phase-2-local-foundation`).
- `fix/<issue-description>`: Targeted bug fix branches.

### 6.2 Commit Message Standards
Follow Conventional Commits:
```
<type>(<scope>): <concise description in imperative mood>

[optional body explaining rationale and context]

[optional footer with reference issue or phase tag]
```
- Examples:
  - `docs(phase-0): create core product spec and architecture documentation`
  - `feat(billing): add excel parser for contractual boq tables`
  - `test(reconciliation): add unit tests for cumulative overclaim rule`

### 6.3 Test-Before-Merge Policy
A branch must not be merged into `main` until:
1. All unit and integration tests pass cleanly.
2. The working tree is clean and uncommitted files are resolved.
3. Acceptance criteria for that phase are explicitly verified and approved.
