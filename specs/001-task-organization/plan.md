# Implementation Plan: Task Organization & Usability

**Branch**: `001-task-organization` | **Date**: 2025-12-30 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-task-organization/spec.md`

**Note**: This template is filled in by the `/sp.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Enhance the Todo CLI with advanced organization features: task priorities, categories, timestamps, smart filtering, search capabilities, and Rich UI with color-coded display. The implementation follows the constitution's async architecture with repository pattern layers, updating database schema, Pydantic models, repository queries, service logic, and CLI commands.

## Technical Context

**Language/Version**: Python 3.12+
**Primary Dependencies**: Typer (CLI framework), Rich (terminal UI), Pydantic (data validation), psycopg[pool] (async PostgreSQL), python-dotenv (environment configuration)
**Storage**: Neon Serverless PostgreSQL (PostgreSQL 15+ compatible)
**Testing**: Not specified in requirements (test implementation optional per constitution)
**Target Platform**: CLI application running on Windows/Linux/macOS
**Project Type**: Single project with async architecture
**Performance Goals**: Filtered task lists display within 1 second; search returns results for 50+ task database in under 1 second
**Constraints**: 100% async/await patterns required; all Python code must have complete type hints; Rich UI required for all user output; parameterized queries mandatory
**Scale/Scope**: Single-user local CLI with persistent storage; tasks table with priority/category/timestamps

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Pre-Phase Constitution Compliance

**Status**: ✅ PASS - No violations identified

| Constitution Principle | Compliance Check | Notes |
|---------------------|-------------------|--------|
| I. Separation of Concerns | ✅ PASS | Plan respects layer boundaries: database (schema), repository (SQL), services (orchestration), CLI (UI) |
| II. Orchestrator-Controlled Workflow | ✅ PASS | Follows 10-step execution flow with Guardian gates |
| III. Guardian Gate System | ✅ PASS | Guardian reviews at each implementation phase |
| IV. Code Location Discipline | ✅ PASS | All code in `/src`; database.py, repository.py, services.py, models.py, cli.py |
| V. No Interaction Mixing | ✅ PASS | No cross-layer concerns (e.g., no UI in database layer) |
| VI. Asynchronous Architecture | ✅ PASS | All database operations, repository methods, and services are async |
| VII. Type Safety Mandate | ✅ PASS | Pydantic models used; 100% type annotations required |
| VIII. Rich UI Standard | ✅ PASS | Rich library for tables, panels, progress spinners |
| IX. Logging and Observability | ✅ PASS | .app.log for debugging (implementation detail) |
| X. Neon DB Integration Standard | ✅ PASS | Uses psycopg[pool] with async connection pool |
| XI. Input Validation Standard | ✅ PASS | Pydantic validates before database |

**Complexity Tracking**: None - No constitutional violations requiring justification

## Project Structure

### Documentation (this feature)

```text
specs/001-task-organization/
├── plan.md              # This file (/sp.plan command output)
├── spec.md              # Feature specification (already created)
├── data-model.md         # Phase 1 output (below)
├── quickstart.md         # Phase 1 output (below)
├── contracts/            # Phase 1 output (CLI contracts below)
└── tasks.md             # Phase 2 output (/sp.tasks command - NOT created by /sp.plan)
```

### Source Code (repository root)

```text
src/
├── database.py          # Database Agent: Connection pool, schema, retry logic
├── repository.py        # Repository Agent: CRUD SQL queries (async)
├── services.py          # Service Logic Agent: Business orchestration (async)
├── models.py            # Service Logic Agent: Pydantic schemas
└── cli.py              # CLI/UX Agent: Typer commands, Rich UI (async)

.app.log              # Hidden logging file (created at runtime)
.env.example           # Environment configuration template
requirements.txt        # Python dependencies
```

**Structure Decision**: Single project with async architecture as defined in Constitution v2.0.0. Four-layer repository pattern (database → repository → services → CLI) with strict separation of concerns.

## Complexity Tracking

> No constitutional violations requiring justification. All architectural decisions align with Constitution v2.0.0.

## Phase 0: Research & Decisions

### 0.1 Schema Enhancement Decisions

**Decision**: Add columns to existing tasks table rather than creating new tables
- **Rationale**: Feature spec references "Update 'tasks' table" - priority and category are stored as text fields, not separate entities
- **Alternatives considered**:
  - Create separate `categories` table with foreign key (per constitution X)
  - Keep as free-form text in tasks table (chosen)
- **Reason**: Category management (create/edit/delete categories) is not in scope per FR-002 and Assumptions

**Decision**: Priority values stored as text with constraint check
- **Rationale**: Simple, no lookup table needed, validation via Pydantic enum
- **Alternatives considered**: Integer mapping (1=low, 2=medium, 3=high)
- **Reason**: Text is more readable in database queries and Rich UI

**Decision**: Timestamps stored as PostgreSQL TIMESTAMP
- **Rationale**: Standard type for date/time data; supports due_date and created_at
- **Alternatives considered**: Unix epoch integers, ISO 8601 strings
- **Reason**: TIMESTAMP provides built-in date arithmetic and sorting

### 0.2 Query Builder Approach

**Decision**: Dynamic SQL builder using parameterized query construction
- **Rationale**: Flexible handling of optional filters (priority, category, status, search)
- **Alternatives considered**:
  - Hardcoded queries for each combination (impractical: 2^4 = 16 combinations)
  - ORM library (adds dependency, violates constitution's "direct SQL" principle)
- **Reason**: Dynamic construction with `%s` parameters prevents SQL injection while maintaining flexibility

### 0.3 Sorting Strategy

**Decision**: Database-level ORDER BY with priority mapping
- **Rationale**: Efficient sorting at database level; PostgreSQL handles optimization
- **Priority order**: High → Medium → Low (custom order, not alphabetical)
- **Alternatives considered**:
  - Application-level sorting (load all, sort in Python) - rejected: inefficient for large datasets
  - Database CASE statement (chosen) - handles custom sort order
- **Date sorting**: Due date ascending (earliest first); NULL values last
- **Alpha sorting**: Title column COLLATE "C" for case-insensitive comparison

### 0.4 Search Implementation

**Decision**: PostgreSQL `LIKE` with `%term%` pattern
- **Rationale**: Simple, efficient for text search; no additional indexes needed
- **Alternatives considered**:
  - Full-text search with tsvector (more complex, requiresGIN indexes)
  - Regular expressions (overkill for keyword search)
- **Reason**: `LIKE` sufficient for spec requirement; tsvector can be future enhancement
- **Search scope**: Both `title` and `description` fields (OR condition)

### 0.5 Rich UI Design

**Decision**: Color-coded priority in table cells
- **High priority**: Red (`color="bold red"`)
- **Medium priority**: Yellow (`color="bold yellow"`)
- **Low priority**: Green (`color="bold green"`)
- **Rationale**: Clear visual distinction per FR-017-019
- **Alternatives considered**: Icons (●/■/▲), background colors
- **Reason**: Colored text most compatible across terminal emulators; background colors may have contrast issues

**Decision**: Rich Table for listings, Panel for details
- **Rationale**: Per constitution VIII - Rich UI Standard
- **Table columns**: ID, Title, Category, Priority, Status, Due Date
- **Progress indicator**: `rich.progress.Progress` with `Spinner()` for database operations

## Phase 1: Data Model & Contracts

### 1.1 Data Model

See `data-model.md` for complete entity definitions and relationships.

### 1.2 CLI Contracts

See `contracts/` directory for command interface definitions.

### 1.3 Quickstart Guide

See `quickstart.md` for setup and usage instructions.

## Phase 2: Implementation Tasks

*Note: This phase is NOT executed by `/sp.plan`. Run `/sp.tasks` to generate `tasks.md` with specific implementation tasks.*

### Task Grouping by Constitution Layers

**Phase 2a: Database Layer (Step 2-3)**
- Update `init_db()` schema with new columns
- Add index on priority for sorting optimization
- Add index on due_date for sorting optimization

**Phase 2b: Repository Layer (Step 4-5)**
- Update Task model in `models.py` with new fields
- Create `create_task()` with priority, category, due_date parameters
- Update `list_tasks()` with dynamic WHERE clause builder
- Add `search_tasks()` function
- Update sort options in repository methods

**Phase 2c: Service Layer (Step 6-7)**
- Update Pydantic Task model with validation for priority enum
- Create `Priority` enum (LOW, MEDIUM, HIGH) for validation
- Update service functions to handle new fields
- Add filter parameter to service methods
- Implement search service method

**Phase 2d: CLI/UX Layer (Step 8-9)**
- Add `--priority`, `--category`, `--status` flags to `list` command
- Add `--sort` flag with enum options (date, priority, alpha)
- Implement `search` command with keyword argument
- Update Rich Table display with new columns
- Implement color-coded priority display
- Add Rich progress spinners for database operations
- Update `add` and `update` commands to accept new fields

**Phase 2e: Documentation (Step 10)**
- Update README.md with new command examples
- Document filter and sort flags
- Document Rich UI features

## Dependencies & Integration

### Layer Dependencies

1. **Database Layer** (src/database.py)
   - No dependencies (foundation layer)
   - Delivers: Async connection pool, updated schema

2. **Repository Layer** (src/repository.py)
   - Depends on: Database layer (connection pool)
   - Delivers: CRUD functions with filtering/sorting
   - Uses: Parameterized queries, async/await patterns

3. **Service Layer** (src/services.py, src/models.py)
   - Depends on: Repository layer (async functions)
   - Delivers: Business orchestration, Pydantic validation
   - Uses: Pydantic models, type hints

4. **CLI Layer** (src/cli.py)
   - Depends on: Service layer (business methods)
   - Delivers: Typer commands, Rich UI
   - Uses: Rich library, Typer CLI framework

### Technology Stack Integration

```
┌─────────────────────────────────────────────────────────┐
│                   CLI Layer                     │
│  (Typer + Rich: src/cli.py)               │
└──────────────────────┬──────────────────────────┘
                       │ async calls
┌──────────────────────┴──────────────────────────┐
│                 Service Layer                  │
│  (Pydantic + Business: src/services.py)      │
└──────────────────────┬──────────────────────────┘
                       │ async calls
┌──────────────────────┴──────────────────────────┐
│               Repository Layer                   │
│   (SQL CRUD: src/repository.py)              │
└──────────────────────┬──────────────────────────┘
                       │ async calls
┌──────────────────────┴──────────────────────────┐
│              Database Layer                     │
│ (Connection Pool: src/database.py)              │
└──────────────────────┬──────────────────────────┘
                       │
              Neon PostgreSQL (async)
```

## Risk Analysis

### Top Risks

| Risk | Impact | Mitigation |
|-------|---------|------------|
| Dynamic SQL builder complexity | Medium | Use parameterized queries; unit test all filter combinations |
| Sorting NULL due dates | Low | Add CASE statement to ORDER BY: tasks with dates first, NULL last |
| Rich color compatibility | Low | Use standard terminal colors; avoid background colors |
| Async/await errors | Medium | Follow constitution patterns; all functions async; use `anyio.run` in CLI |
| Database connection retry logic | Medium | Implement exponential backoff per constitution X (already in database.py) |

### Blast Radius

- Database schema changes: Single table (tasks); isolated impact
- Repository layer changes: Only affects query functions; backward compatible if defaults used
- CLI flag additions: Additive; existing commands unchanged unless flags provided
- Rich UI updates: Visual only; functional behavior unchanged

## Re-Check Constitution After Phase 1

### Post-Design Constitution Compliance

**Status**: ✅ PASS - No new violations

| Check | Result | Notes |
|--------|----------|--------|
| Async architecture maintained | ✅ | All design decisions preserve async patterns |
| Layer boundaries respected | ✅ | No cross-layer concerns identified |
| Rich UI standard | ✅ | Rich table, colors, progress defined |
| Type safety | ✅ | Pydantic models for all data |
| Pydantic validation | ✅ | Input validation at CLI layer |

## Next Steps

1. Run `/sp.tasks` to generate `tasks.md` with specific implementation tasks
2. Follow Constitution Execution Flow (10 steps with Guardian reviews)
3. Implement in order: Database → Repository → Service → CLI
4. Submit each layer to Spec Guardian for approval
5. Create PHR after `/sp.plan` completion
