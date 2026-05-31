<!--
  Sync Impact Report
  ==================
  Version change: 1.0.0 → 1.1.0 (MINOR — new principle VI + Project Structure section + clarifications)
  Modified principles: I (coverage metric clarified), II (deps justification), V added, old V→VI
  Added sections: "Error Handling & Security" principle (V), "Project Structure" section
  Removed sections: N/A
  Amended sections: Tech Stack (npm, Node, path), Development Workflow (pre-commit→pre-merge, API contract format), Governance (approval rule)
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

Tests MUST be written before implementation code. Minimum line coverage MUST be
≥ 80%. Trivially uncovered code (config files, entry points) MUST be listed in
a `.coverage-exclusions` file. Strict Red-Green-Refactor cycle enforced —
tests fail first, then pass after implementation, then refactor safely.

Rationale: TDD guarantees correctness from the start and prevents regressions
in a codebase with no QA pipeline.

### II. Simplicity First

YAGNI — no features beyond current requirements. Minimal dependencies — justify
each addition. No dependency added without documented reason. Premature
abstraction forbidden. Flat structure preferred over deep nesting.

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

### V. Error Handling & Security

All user input MUST be validated server-side before processing. HTML-escape all
dynamic content output to prevent XSS. Graceful degradation on persistence
failure — if JSON read fails, reset to empty state and log error.

Rationale: No auth layer means input validation is the only defense. Single
page app with file storage needs explicit failure handling.

### VI. Maintainability

Small functions (ideally < 20 lines). Self-documenting code — comments only
when intent is not obvious from code. Consistent naming conventions throughout
codebase. Linting enforced.

Rationale: Simple code is easier to test, review, and modify. Team size is
small.

## Technology Stack

- **Backend**: Node.js + Express
- **Frontend**: Vanilla HTML, CSS, JavaScript (no framework)
- **Persistence**: JSON file on disk
- **Testing**: Jest + Supertest
- **Linting**: ESLint
- **Package manager**: npm
- **Node version**: >= 18 LTS
- **JSON file path**: `data/games.json` at project root
- **No authentication**: explicitly excluded
- **No database**: explicitly excluded

## Project Structure

```
backend/
├── src/
│   ├── game/        # Game logic (pure functions, testable without HTTP)
│   ├── routes/      # Express route handlers
│   └── storage/     # JSON file persistence
├── tests/
│   ├── unit/        # Game logic tests (no HTTP — Jest)
│   └── integration/ # API tests (Supertest)
└── data/            # JSON persistence files

frontend/
├── index.html
├── css/
├── js/
└── tests/           # Frontend tests (if applicable)
```

**Test type rules**:
- Game logic: unit tests only — no HTTP, fast, pure function calls
- API routes: integration tests — Supertest against Express app
- Frontend: manual or DOM-level tests (decided per feature)

## Development Workflow

1. **TDD cycle**: Write failing test → verify failure → implement → verify
   pass → refactor
2. **Coverage gate**: Run full test suite before merging to main. Coverage MUST
   be ≥ 80%. WIP commits allowed with `--no-verify` and documented
   justification.
3. **API-first**: Define API contract (Markdown in
   `specs/<feature>/contracts/`) before implementing backend or frontend
4. **Feature branch**: Each feature/spec on its own branch
5. **Pre-merge**: Lint + test must pass on CI or before merge

## Governance

This constitution governs all development decisions. Any deviation from these
principles requires documented justification and amendment.

**Amendment Process**:
1. Propose change in writing with rationale
2. Document in constitution with version bump
3. MAJOR: backward-incompatible principle removal/redefinition
4. MINOR: new principle or materially expanded guidance
5. PATCH: clarifications, typos, non-semantic refinements
6. Amendment approved by committer + reviewer
7. All PRs/reviews MUST verify constitution compliance

**Version**: 1.1.0 | **Ratified**: 2026-05-31 | **Last Amended**: 2026-05-31
