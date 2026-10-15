# Contributing to BOQ Blindspot

Welcome to **BOQ Blindspot**. This document outlines our development principles, collaboration workflow, and coding expectations.

---

## 1. Core Principles

1. **Correctness Over Convenience**: In construction financial reconciliation, accuracy and auditable proof always supersede quick shortcuts.
2. **Deterministic Arithmetic**: Financial calculations, quantity comparisons, and rule evaluations must be implemented in pure, deterministic, and independently testable Python code. Never delegate math or rate calculations to probabilistic AI.
3. **Local-First & Privacy Preserving**: Customer documents, commercial rates, site photos, and voice notes must remain on the user's local machine. Never introduce external API calls for core processing.
4. **Phase-by-Phase Execution**: Development proceeds in strict, controlled phases. Do not build features from future phases until the current phase is tested, reviewed, and approved.

---

## 2. Collaboration & Git Workflow

All Git operations (branching, commits, merges, pushes) are managed **manually by the project lead**. AI assistants and contributors must not execute Git commands directly.

### 2.1 Branching Strategy
- `main`: Production-stable, fully tested documentation and code.
- `feature/<phase-number>-<short-description>`: Active development branches for individual phases (e.g., `feature/phase-1-trade-discovery`).
- `fix/<issue-description>`: Targeted bug fixes.

### 2.2 Standard Development Flow
For each implementation phase:
1. Confirm a clean working tree on `main`.
2. Create and switch to the assigned feature branch.
3. Implement the phase deliverables accompanied by automated tests.
4. Verify all tests pass locally.
5. Commit work using structured, meaningful commit messages.
6. Switch back to `main` and merge the verified feature branch.
7. Push to the remote repository.

### 2.3 Commit Message Convention
We adhere to Conventional Commits:
```
<type>(<scope>): <short description in imperative mood>

[optional body providing technical context and rationale]

[optional footer with phase or issue reference]
```
- **Allowed Types**: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`, `perf`.
- **Examples**:
  - `docs(spec): add dimensional analysis requirements to reconciliation rules`
  - `feat(documents): implement sha256 content-addressable storage engine`
  - `test(billing): add unit tests for multi-period claim progression`

---

## 3. Test-Before-Merge Policy

No branch may be merged into `main` without:
- 100% passing test suite across all active modules.
- New functionality backed by comprehensive unit and integration tests.
- Zero regressions in previously established calculation or extraction rules.
- Explicit verification against the phase acceptance criteria.

---

## 4. Proposing Changes & Reporting Bugs

- **Proposing Architectural Changes**: Significant changes to data models, AI runtimes, or system boundaries must be documented as an Architecture Decision Record (ADR) in `docs/` and cross-referenced with `docs/ARCHITECTURE.md` before implementation.
- **Reporting Bugs or Calculation Anomalies**: Bug reports must include:
  - The specific rule ID or component involved.
  - Anonymized sample input data reproducing the discrepancy.
  - Expected deterministic output vs. actual observed output.

---

## 5. Development Environment Setup

> **Note**: Detailed implementation instructions, virtual environment setup, package manifests (`package.json`, `pyproject.toml`), and local database migrations will be introduced systematically when **Phase 2 (Local Application Foundation)** begins.
