---
id: 002
title: Async Architecture Upgrade
stage: constitution
date: 2025-12-30
surface: agent
model: claude-sonnet-4-5
feature: none
branch: master
user: Ibtisam1234-developer
command: /sp.constitution
labels: ["constitution", "async", "repository-pattern", "rich-ui", "pydantic", "psycopg-pool"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - .specify/memory/constitution.md
tests:
 - N/A (documentation only)
---

## Prompt

# TODO CLI CONSTITUTION: INTERMEDIATE & ADVANCED (NEON + ASYNC)

## 1. ARCHITECTURAL STANDARDS
- **Framework:** Python 3.12+ with `Typer` for CLI and `Pydantic` for data schemas.
- **Async Protocol:** The core must be 100% asynchronous using `asyncio` and `psycopg[pool]`. No blocking I/O allowed.
- **Repository Pattern:** Logic must be decoupled into distinct layers:
    - `models.py`: Pydantic schemas for tasks and categories.
    - `database.py`: Neon connection pool logic and `init_db()` using `CREATE TABLE IF NOT EXISTS`.
    - `repository.py`: CRUD logic (SQL queries) isolated from the CLI interface.
    - `cli.py`: User-facing Typer commands and Rich UI rendering.

## 2. NEON DATABASE REQUIREMENTS
- **Schema Design:** Beyond a single list—implement a relational schema with `tasks` and `categories` linked by Foreign Keys.
- **Connection Management:** Implement robust retry logic for Neon "cold starts" and use environment variables for the connection string.
- **SQL Best Practices:** Use prepared statements and parameterized queries to prevent SQL injection.

## 3. ADVANCED FEATURES & UX
- **Rich UI:** All outputs must use the `Rich` library. Use `Table` for listings, `Panel` for task details, and `Progress` (spinners) for database operations.
- **Smart Filtering:** Support CLI flags for `--status`, `--category`, and `--priority`.
- **Search:** Implement full-text search across task descriptions using Postgres `LIKE` or `tsvector`.
- **Validation:** Use Pydantic to validate all user inputs before they ever touch the database.

## 4. QUALITY & RELIABILITY
- **Error Handling:** Catch specific Postgres errors (Connection, Integrity) and show human-readable "Rich" errors instead of tracebacks.
- **Type Safety:** 100% Type Hinting is mandatory for every function signature.
- **Logging:** Maintain a hidden `.app.log` for debugging database transactions without cluttering the CLI.

## 5. TOOLCHAIN WORKFLOW
- **Spec-Driven:** Claude Code must always check the latest `.md` specifications created by SpecKit Plus before implementation.
- **Atomic Implementation:** Changes must be implemented in small, testable chunks (e.g., 'Setup DB' -> 'Add Task' -> 'List Task').

## Response snapshot

Constitution successfully amended to version 2.0.0 with major architectural upgrades:

**Version Change**: 1.0.0 → 2.0.0 (MAJOR)
Rationale: Backward-incompatible changes including async I/O transition, repository pattern restructuring, and technology stack upgrades

**New Core Principles Added**:
- VI. Asynchronous Architecture - 100% async/await patterns with psycopg[pool]
- VII. Type Safety Mandate - 100% type hinting requirement for all functions
- VIII. Rich UI Standard - Rich library for all terminal output (Table, Panel, Progress)
- IX. Logging and Observability - .app.log for debugging without CLI clutter
- X. Neon DB Integration Standard - Enhanced with retry logic, prepared statements, relational schema
- XI. Input Validation Standard - Pydantic validation before database

**Layer Restructuring**:
- Database Agent: Now owns connection pool and schema only (`database.py`)
- Repository Agent (NEW): Handles CRUD SQL logic (`repository.py`)
- Service Logic Agent: Business orchestration only (`services.py`, `models.py`)
- CLI/UX Agent (renamed): Typer commands with Rich UI (`cli.py`)

**Advanced Features Section Added**:
- Rich UI Components (Table, Panel, Progress, colored errors)
- Smart Filtering (--status, --category, --priority flags)
- Full-text search (LIKE or tsvector)
- Validation via Pydantic models
- Error handling with Rich-formatted messages

**Schema Requirements**:
- Relational with `tasks` and `categories` tables
- Foreign key: `tasks.category_id` → `categories.id`
- Support for priorities, statuses, timestamps

**Execution Flow Expanded**:
Now includes 10 steps (was 8), adding Repository Agent layer between Database and Service layers.

## Outcome

- ✅ Impact: Upgraded constitution to async-first architecture with repository pattern, Rich UI, and Pydantic validation
- 🧪 Tests: N/A (documentation artifact)
- 📁 Files: Updated `.specify/memory/constitution.md` (v2.0.0, 606 lines, includes Sync Impact Report)
- 🔁 Next prompts: Update CLAUDE.md to reflect new async architecture; migrate existing code when implementing features
- 🧠 Reflection: Major version bump justified by backward-incompatible async transition and layer restructuring. Templates remain compatible; CLAUDE.md and existing code need migration.

## Evaluation notes (flywheel)

- Failure modes observed: None
- Graders run and results (PASS/FAIL): Manual validation PASS (no placeholders, version updated, new principles added)
- Prompt variant (if applicable): N/A
- Next experiment (smallest change to try): Update CLAUDE.md Implementation Summary section to reflect async architecture and new file structure
