<!--
  Sync Impact Report
  ==================
  Version change: (template) → 1.0.0
  Modified principles: N/A (first fill)
  Added sections: Core Principles (5 principles), Technology Stack, Development Workflow, Governance
  Removed sections: N/A
  Templates requiring updates:
    - .specify/templates/constitution-template.md: ✅ (template preserved)
    - .specify/templates/plan-template.md: ✅ (no change needed)
    - .specify/templates/spec-template.md: ✅ (no change needed)
    - .specify/templates/tasks-template.md: ✅ (no change needed)
    - .specify/templates/checklist-template.md: ✅ (no change needed)
  Follow-up TODOs: None
-->

# Hangman Web App Constitution

## Core Principles

### I. Test-Driven Development (NON-NEGOTIABLE)

Tests MUST be written before implementation code. Minimum test coverage MUST be
≥ 80%. Strict Red-Green-Refactor cycle enforced — tests fail first, then pass
after implementation, then refactor safely.

Rationale: TDD guarantees correctness from the start and prevents regressions in
a codebase with no QA pipeline.

### II. Simplicity First

YAGNI — no features beyond current requirements. Minimal dependencies: prefer
vanilla solutions over frameworks. Premature abstraction forbidden. Flat
structure preferred over deep nesting.

Rationale: Small project with clear boundary. Overengineering wastes time and
increases maintenance burden.

### III. Separation of Concerns

Game logic MUST be independent of HTTP/transport layer. Backend (Express API)
and Frontend (HTML/CSS/JS) are distinct layers. API contract between them MUST
be explicit and documented.

Rationale: Isolated game logic is testable without HTTP. Frontend can change
independently of backend.

### IV. File-based Persistence

JSON file as single persistence store. Atomic read/write operations required.
No database dependency allowed.

Rationale: Requirement explicitly forbids databases. JSON file is sufficient for
MVP scale.

### V. Maintainability

Small functions (ideally < 20 lines). Self-documenting code — comments only
when intent is not obvious from code. Consistent naming conventions throughout
codebase. Linting enforced.

Rationale: Simple code is easier to test, review, and modify. Team size is
small.

## Technology Stack

- **Backend**: Node.js + Express
- **Frontend**: Vanilla HTML, CSS, JavaScript (no framework)
- **Persistence**: JSON file on disk
- **Testing**: Jest + Supertest (or equivalent test runner with HTTP assertion)
- **Linting**: ESLint
- **No authentication**: explicitly excluded
- **No database**: explicitly excluded

## Development Workflow

1. **TDD cycle**: Write failing test → verify failure → implement → verify
   pass → refactor
2. **Coverage gate**: Run full test suite before any commit. Coverage MUST be
   ≥ 80%
3. **API-first**: Define API contract before implementing backend or frontend
4. **Feature branch**: Each feature/spec on its own branch
5. **Pre-commit**: Lint + test must pass

## Governance

This constitution governs all development decisions. Any deviation from these
principles requires documented justification and amendment.

**Amendment Process**:
1. Propose change in writing with rationale
2. Document in constitution with version bump
3. MAJOR: backward-incompatible principle removal/redefinition
4. MINOR: new principle or materially expanded guidance
5. PATCH: clarifications, typos, non-semantic refinements
6. All PRs/reviews MUST verify constitution compliance

**Version**: 1.0.0 | **Ratified**: 2026-05-31 | **Last Amended**: 2026-05-31
