# Tasks: Task Organization & Usability

**Input**: Design documents from `/specs/001-task-organization/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: The examples below include test tasks. Tests are OPTIONAL - only include them if explicitly requested in the feature specification.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Layer] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Layer]**: Which constitution layer this task belongs to (DB, Repo, Service, CLI, Docs)
- Include exact file paths in descriptions

## Path Conventions

- **Single project**: `src/`, `tests/` at repository root
- **Web app**: `backend/src/`, `frontend/src/`
- **Mobile**: `api/src/`, `ios/src/` or `android/src/`
- Paths shown below assume single project - adjust based on plan.md structure

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [x] T001 Create feature directory structure per implementation plan (COMPLETED)
- [x] T002 Verify Python 3.12+ environment and dependencies (COMPLETED)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T003 [P] [DB] Verify `src/database.py` has async Neon PostgreSQL connection pool with `init_db()` function (COMPLETED)
- [x] T004 [P] [Repo] Verify `src/repository.py` exists with async CRUD function signatures (COMPLETED)
- [x] T005 [P] [Service] Verify `src/services.py` exists with business orchestration functions (COMPLETED)
- [x] T006 [P] [Service] Verify `src/models.py` exists with Pydantic models (COMPLETED)
- [x] T007 [P] [CLI] Verify `src/cli.py` exists with Typer CLI commands (COMPLETED)

**Checkpoint**: Foundation ready - task implementation can now begin

---

## Phase 3: Database Layer - Schema Enhancement (Step 2-3)

**Goal**: Update database schema to support priorities, categories, and timestamps

**Independent Test**: Can verify `tasks` table has new columns and indexes via database inspection

### Implementation for Database Layer

- [x] T008 [DB] Modify `src/database.py` to update table schema with new columns (priority, category, created_at, due_date) in `init_db()` function (COMPLETED)
- [x] T009 [DB] Add `priority` column with default value 'medium' and CHECK constraint (low, medium, high) in `init_db()` (COMPLETED)
- [x] T010 [DB] Add `category` column as optional TEXT field in `init_db()` (COMPLETED)
- [x] T011 [DB] Add `created_at` column with TIMESTAMP and default NOW() in `init_db()` (COMPLETED)
- [x] T012 [DB] Add `due_date` column as optional TIMESTAMP field in `init_db()` (COMPLETED)
- [x] T013 [DB] Create index `idx_tasks_priority` on priority column in `init_db()` (COMPLETED)
- [x] T014 [DB] Create index `idx_tasks_due_date` on due_date column in `init_db()` (COMPLETED)
- [x] T015 [DB] Ensure all database operations use async/await patterns per constitution VI (COMPLETED)

**Checkpoint**: Database schema updated and ready for repository layer

---

## Phase 4: Repository Layer - CRUD Enhancement (Step 4-5)

**Goal**: Implement SQL CRUD logic with filtering, sorting, and search capabilities

**Independent Test**: Can create task with new fields and query with filters/sort

### Implementation for Repository Layer

- [x] T016 [Repo] Update `src/repository.py` `create_task()` function to accept priority, category, and due_date parameters (COMPLETED)
- [x] T017 [Repo] Update `src/repository.py` `create_task()` to use parameterized INSERT query with all new fields (COMPLETED)
- [x] T018 [Repo] Create `search_tasks(query)` function in `src/repository.py` using SQL `ILIKE` for case-insensitive search (COMPLETED)
- [x] T019 [Repo] Implement `search_tasks()` to search both title and description fields with OR condition (COMPLETED)
- [x] T020 [Repo] Enhance `get_all_tasks()` function in `src/repository.py` to accept optional filter parameters (status, category, priority) (COMPLETED)
- [x] T021 [Repo] Enhance `get_all_tasks()` function in `src/repository.py` to accept optional sort parameter (date, priority, alpha) (COMPLETED)
- [x] T022 [Repo] Implement dynamic WHERE clause builder in `src/repository.py` for compositional filters (COMPLETED)
- [x] T023 [Repo] Implement dynamic ORDER BY clause builder in `src/repository.py` with priority mapping (CASE statement) (COMPLETED)
- [x] T024 [Repo] Update `get_all_tasks()` to handle NULL due_date values in sorting (NULLS LAST) (COMPLETED)
- [x] T025 [Repo] Ensure all repository functions are async and use parameterized queries per constitution X (COMPLETED)

**Checkpoint**: Repository layer complete with filtering, sorting, and search capabilities

---

## Phase 5: Service Layer - Model Updates (Step 6-7)

**Goal**: Update Pydantic models with validation and business orchestration

**Independent Test**: Pydantic validation rejects invalid priority values

### Implementation for Service Layer

- [x] T026 [Service] Create `Priority` enum in `src/models.py` with values: LOW, MEDIUM, HIGH (COMPLETED)
- [x] T027 [Service] Create `TaskStatus` enum in `src/models.py` with values: PENDING, COMPLETED (COMPLETED)
- [x] T028 [Service] Update `src/models.py` `TaskCreate` model to include priority (default: MEDIUM), category, and due_date fields (COMPLETED)
- [x] T029 [Service] Update `src/models.py` `TaskCreate` model with field validators (title max_length 255, priority enum) (COMPLETED)
- [x] T030 [Service] Update `src/models.py` `TaskUpdate` model to include priority, category, due_date, and status fields (COMPLETED)
- [x] T031 [Service] Update `src/models.py` `Task` model to include all new fields with complete type hints (COMPLETED)
- [x] T032 [Service] Update `src/services.py` `create_task()` function to handle new fields with Pydantic validation (COMPLETED)
- [x] T033 [Service] Update `src/services.py` `list_tasks()` function to accept filter and sort parameters (COMPLETED)
- [x] T034 [Service] Update `src/services.py` to call repository layer with new parameters (COMPLETED)
- [x] T035 [Service] Ensure all service functions are async and have complete type hints per constitution VII (COMPLETED)

**Checkpoint**: Service layer complete with Pydantic validation and business logic

---

## Phase 6: CLI Layer - Rich UI & Commands (Step 8-9)

**Goal**: Implement Typer commands with Rich UI, filtering, sorting, and search

**Independent Test**: Can run `add` with new flags, `list` with filters/sort, `search` with keyword

### Implementation for CLI Layer

- [x] T036 [CLI] Update `add` command in `src/cli.py` to accept `--priority` option (default: medium) (COMPLETED)
- [x] T037 [CLI] Update `add` command in `src/cli.py` to accept `--category` option (COMPLETED)
- [x] T038 [CLI] Update `add` command in `src/cli.py` to accept `--due` option for due date (COMPLETED)
- [x] T039 [CLI] Refactor `list` command in `src/cli.py` to render Rich table with columns: ID, Title, Category, Priority, Status, Due Date (COMPLETED)
- [x] T040 [CLI] Implement conditional formatting in `list` command for priorities: red (high), yellow (medium), green (low) (COMPLETED)
- [x] T041 [CLI] Add `--priority` flag to `list` command in `src/cli.py` for filtering by priority level (COMPLETED)
- [x] T042 [CLI] Add `--category` flag to `list` command in `src/cli.py` for filtering by category name (COMPLETED)
- [x] T043 [CLI] Add `--status` flag to `list` command in `src/cli.py` for filtering by task status (COMPLETED)
- [x] T044 [CLI] Add `--sort` flag to `list` command in `src/cli.py` with options: date, priority, alpha (COMPLETED)
- [x] T045 [CLI] Implement `search` command in `src/cli.py` accepting keyword argument (COMPLETED)
- [x] T046 [CLI] Add optional filter flags (`--status`, `--priority`, `--category`) to `search` command in `src/cli.py` (COMPLETED)
- [x] T047 [CLI] Add `update` command in `src/cli.py` to allow changing priority or status of existing tasks (COMPLETED)
- [x] T048 [CLI] Add optional flags to `update` command: `--priority`, `--category`, `--due`, `--status`, `--title`, `--description` (COMPLETED)
- [x] T049 [CLI] Implement Rich progress spinners for database operations in `src/cli.py` (COMPLETED)
- [x] T050 [CLI] Show user-friendly error messages with Rich formatting for invalid inputs in `src/cli.py` (COMPLETED)
- [x] T051 [CLI] Display "No tasks found matching criteria" message when filters return no results in `src/cli.py` (COMPLETED)
- [x] T052 [CLI] Ensure all CLI commands handle async execution properly per constitution VI (COMPLETED)

**Checkpoint**: CLI layer complete with Rich UI, filtering, sorting, and search

---

## Phase 7: Documentation (Step 10)

**Purpose**: Update project documentation with new features

**Independent Test**: README documents all new commands and flags

### Documentation Tasks

- [x] T053 [Docs] Update `README.md` with examples of `--priority`, `--category`, `--status`, `--sort` flags (COMPLETED)
- [x] T054 [Docs] Document `search` command usage in `README.md` (COMPLETED)
- [x] T055 [Docs] Document `update` command usage in `README.md` (COMPLETED)
- [x] T056 [Docs] Document Rich UI features and color scheme in `README.md` (COMPLETED)
- [x] T057 [Docs] Update `CLAUDE.md` with new async architecture implementation guidance if needed (COMPLETED)

**Checkpoint**: Documentation complete

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Final improvements and validation

- [x] T058 [P] [All] Verify all functions have complete type hints (no `Any` types) per constitution VII (COMPLETED)
- [x] T059 [P] [All] Verify all database operations use async/await patterns per constitution VI (COMPLETED)
- [x] T060 [P] [All] Verify Rich UI used for all user output per constitution VIII (COMPLETED)
- [x] T061 [P] [All] Verify Pydantic validation used for all user inputs per constitution XI (COMPLETED)
- [x] T062 [P] [All] Verify parameterized queries used throughout repository layer per constitution X (COMPLETED)
- [x] T063 [P] [All] Verify no interaction mixing (e.g., no SQL in CLI layer) per constitution V (COMPLETED)
- [x] T064 [CLI] Test color-coded priority display in Rich table (COMPLETED)
- [x] T065 [CLI] Test compositional filters (combine `--priority`, `--category`, `--status`) (COMPLETED)
- [x] T066 [CLI] Test search with keyword matches in title and description (COMPLETED)
- [x] T067 [CLI] Test sort options (date, priority, alpha) with edge cases (NULL due dates) (COMPLETED)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1**: Setup - COMPLETED
- **Phase 2**: Foundational verification (T003-T007) - COMPLETED
- **Phase 3**: Database Layer (T008-T015) - COMPLETED
- **Phase 4**: Repository Layer (T016-T025) - COMPLETED
- **Phase 5**: Service Layer (T026-T035) - COMPLETED
- **Phase 6**: CLI Layer (T036-T052) - COMPLETED
- **Phase 7**: Documentation (T053-T057) - COMPLETED
- **Phase 8**: Polish (T058-T067) - COMPLETED

### Layer Dependencies (Constitution Execution Flow)

```
Database Agent (Phase 3) → Guardian Review → Approval
    ↓
Repository Agent (Phase 4) → Guardian Review → Approval
    ↓
Service Logic Agent (Phase 5) → Guardian Review → Approval
    ↓
CLI/UX Agent (Phase 6) → Guardian Review → Approval
    ↓
Final Guardian Audit (Phase 8) → Approval
    ↓
Documentation Agent (Phase 7)
```
