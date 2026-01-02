# Tasks: Interactive Todo Application

**Input**: Design documents from `/specs/001-todo-app/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are OPTIONAL per specification - no test tasks included unless explicitly requested

**Organization**: Tasks are grouped by sub-agent ownership following the constitution's mandatory 8-step execution flow

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Single project**: `src/` at repository root (per constitution)
- All Python code MUST reside in `/src` directory
- Paths shown below follow constitution requirements

---

## Phase 1: Setup (Project Initialization)

**Purpose**: Initialize project structure and dependencies

- [X] T001 Create `src/` directory for all Python code
- [X] T002 Create `.gitignore` file with `.env` and `__pycache__` entries
- [X] T003 Create `requirements.txt` with psycopg2-binary>=2.9.0 and python-dotenv>=1.0.0
- [X] T004 Create `.env.example` template file with DATABASE_URL placeholder
- [X] T005 [P] Create `README.md` placeholder (will be completed by Documentation Agent)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Step 2: Database Agent - Schema and Connection Layer

**Constitutional Requirements**:
- Owner: Database Agent
- Deliverables: `src/db.py` with all database functions
- Forbidden: NO `input()`, NO `print()`, NO business logic
- Submit to Spec Guardian for review after completion

- [X] T006 [P] Implement `get_connection()` in src/db.py (load DATABASE_URL from .env, return psycopg2 connection)
- [X] T007 [P] Implement `initialize_schema()` in src/db.py (CREATE TABLE IF NOT EXISTS tasks with 6 fields per data-model.md)
- [X] T008 [P] Implement `create_task_db()` in src/db.py (parameterized INSERT query, return new task ID)
- [X] T009 [P] Implement `get_all_tasks_db()` in src/db.py (SELECT all tasks ORDER BY created_at DESC, return list of tuples)
- [X] T010 [P] Implement `update_task_db()` in src/db.py (dynamic UPDATE query with parameterized values, set updated_at=NOW())
- [X] T011 [P] Implement `toggle_task_status_db()` in src/db.py (UPDATE is_completed = NOT is_completed, set updated_at=NOW())
- [X] T012 [P] Implement `delete_task_db()` in src/db.py (parameterized DELETE query, return boolean)

**Checkpoint**: Database layer complete - Submit to Spec Guardian for review

### Step 3: Spec Guardian - Database Review

**Guardian Validation Checklist**:
- [X] T013 Validate NO `input()` or `print()` calls in src/db.py
- [X] T014 Validate NO business logic in src/db.py (no validation, no calculations)
- [X] T015 Validate ALL queries use parameterized format (no string interpolation)
- [X] T016 Validate proper error handling (psycopg2 exceptions caught and re-raised)
- [X] T017 Validate transaction management (commit/rollback on all write operations)

**Gate**: Service layer cannot proceed until Guardian approves database layer

---

## Phase 3: Service Logic Layer (Prerequisite for User Stories)

**Purpose**: Business logic implementation required by all user stories

**Constitutional Requirements**:
- Owner: Service Logic Agent
- Deliverables: `src/services.py`, `src/models.py`
- Forbidden: NO direct database connections, NO `input()`, NO `print()`
- Submit to Spec Guardian for review after completion

### Step 4: Service Logic Agent - Business Logic Implementation

- [X] T018 [P] Implement Task dataclass in src/models.py (6 fields: id, title, description, is_completed, created_at, updated_at)
- [X] T019 [P] Implement `Task.to_dict()` method in src/models.py (convert Task to dictionary)
- [X] T020 [P] Implement `Task.from_db_row()` classmethod in src/models.py (create Task from database tuple)
- [X] T021 Create global connection variable in src/services.py (will be set by main.py)
- [X] T022 [P] Implement `create_task()` in src/services.py (validate title 1-200 chars, description max 1000 chars, call db.create_task_db(), return dict)
- [X] T023 [P] Implement `list_tasks()` in src/services.py (call db.get_all_tasks_db(), convert tuples to Task objects, return dict)
- [X] T024 [P] Implement `update_task()` in src/services.py (validate at least one field provided, validate title/description if provided, call db.update_task_db(), return dict)
- [X] T025 [P] Implement `toggle_task_completion()` in src/services.py (call db.toggle_task_status_db(), return dict)
- [X] T026 [P] Implement `delete_task()` in src/services.py (call db.delete_task_db(), return dict)

**Checkpoint**: Service layer complete - Submit to Spec Guardian for review

### Step 5: Spec Guardian - Service Layer Review

**Guardian Validation Checklist**:
- [X] T027 Validate NO database connections in src/services.py (only calls to db.py functions)
- [X] T028 Validate NO `input()` or `print()` calls in src/services.py or src/models.py
- [X] T029 Validate proper separation from database layer (no SQL queries)
- [X] T030 Validate all validation logic in service layer (not database layer)
- [X] T031 Validate service functions return structured dicts (not raw tuples)

**Gate**: Menu/UX layer cannot proceed until Guardian approves service layer

---

## Phase 4: User Story 1 - Create and View Tasks (Priority: P1) 🎯 MVP

**Goal**: Enable users to create tasks and view their task list

**Independent Test**: Launch application, create multiple tasks with titles and descriptions, verify persistence after restart

**Constitutional Requirements**:
- Owner: Menu/UX Agent
- Deliverables: `src/menu.py`, `src/main.py`
- Forbidden: NO SQL, NO business logic, NO schema changes
- Submit to Spec Guardian for review after completion

### Step 6: Menu/UX Agent - User Interface Implementation (US1)

- [X] T032 [P] [US1] Implement `display_menu()` in src/menu.py (print menu options 1-6)
- [X] T033 [P] [US1] Implement `get_menu_choice()` in src/menu.py (get and validate user input 1-6)
- [X] T034 [US1] Implement `handle_create_task()` in src/menu.py (prompt for title and description, call services.create_task(), display result)
- [X] T035 [US1] Implement `format_task_display()` in src/menu.py (format Task object for display with status icon)
- [X] T036 [US1] Implement `handle_list_tasks()` in src/menu.py (call services.list_tasks(), format and display all tasks)
- [X] T037 [US1] Implement `main()` function in src/main.py (initialize connection, call db.initialize_schema(), run menu loop for options 1-2 and 6 only)

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently (Create + View + Exit)

---

## Phase 5: User Story 2 - Mark Tasks Complete/Incomplete (Priority: P2)

**Goal**: Enable users to toggle task completion status

**Independent Test**: Create tasks, mark them complete, verify status persists, toggle back to incomplete

### Implementation for User Story 2

- [X] T038 [US2] Implement `handle_toggle_task()` in src/menu.py (prompt for task ID, call services.toggle_task_completion(), display result)
- [X] T039 [US2] Update `main()` in src/main.py to include menu option 4 (Toggle Task Completion) in menu loop

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 6: User Story 3 - Update Task Details (Priority: P3)

**Goal**: Enable users to edit existing task titles and descriptions

**Independent Test**: Create task, edit title and description, verify changes persist

### Implementation for User Story 3

- [X] T040 [US3] Implement `handle_update_task()` in src/menu.py (prompt for task ID, new title, new description, call services.update_task(), display result)
- [X] T041 [US3] Update `main()` in src/main.py to include menu option 3 (Update Task) in menu loop

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently

---

## Phase 7: User Story 4 - Delete Tasks (Priority: P3)

**Goal**: Enable users to permanently remove tasks

**Independent Test**: Create tasks, delete specific tasks, verify they no longer appear after restart

### Implementation for User Story 4

- [X] T042 [US4] Implement `handle_delete_task()` in src/menu.py (prompt for task ID, call services.delete_task(), display result)
- [X] T043 [US4] Update `main()` in src/main.py to include menu option 5 (Delete Task) in menu loop

**Checkpoint**: All user stories should now be independently functional

---

## Phase 8: Final Compliance and Documentation

**Purpose**: Guardian audit and documentation finalization

### Step 7: Spec Guardian - Final Compliance Audit

**Guardian Comprehensive Validation**:
- [X] T044 Validate all scope boundaries respected (Database/Service/Menu separation)
- [X] T045 Validate NO interaction mixing (no print in db.py, no SQL in menu.py, no input in services.py)
- [X] T046 Validate NO schema drift (schema matches data-model.md exactly)
- [X] T047 Validate all code in correct locations (all Python in `/src`)
- [X] T048 Validate proper error handling throughout all layers
- [X] T049 Validate parameterized queries in all database operations

**Gate**: Documentation cannot proceed until Guardian approves entire implementation

### Step 8: Documentation Agent - Project Documentation

**Constitutional Requirements**:
- Owner: Documentation Agent
- Deliverables: README.md, CLAUDE.md updates
- Forbidden: NO code changes, NO specification changes

- [X] T050 [P] Create comprehensive README.md with Neon setup instructions, installation steps, usage guide
- [X] T051 [P] Update CLAUDE.md with implementation summary and agent responsibilities
- [X] T052 [P] Document DATABASE_URL configuration in README.md
- [X] T053 [P] Add troubleshooting section to README.md for common errors
- [X] T054 [P] Create quickstart section in README.md with example commands

**Checkpoint**: All documentation complete, project ready for delivery

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
  - Step 2: Database Agent implementation
  - Step 3: Spec Guardian database review (GATE)
- **Service Logic (Phase 3)**: Depends on Database layer approval - BLOCKS all user stories
  - Step 4: Service Logic Agent implementation
  - Step 5: Spec Guardian service review (GATE)
- **User Stories (Phase 4-7)**: All depend on Service Logic approval
  - User Story 1 (P1): Can start after Service Logic approved - MVP
  - User Story 2 (P2): Independent of US1, can start after Service Logic
  - User Story 3 (P3): Independent of US1/US2, can start after Service Logic
  - User Story 4 (P3): Independent of US1/US2/US3, can start after Service Logic
- **Final (Phase 8)**: Depends on all user stories being complete
  - Step 7: Spec Guardian final audit
  - Step 8: Documentation Agent

### Guardian Gates (CRITICAL)

1. **Gate 1** (After T012): Database layer review - Service layer BLOCKED until approval
2. **Gate 2** (After T026): Service layer review - Menu/UX layer BLOCKED until approval
3. **Gate 3** (After T043): Final audit - Documentation BLOCKED until approval

### User Story Dependencies

- **User Story 1 (P1)**: No dependencies on other stories - MVP ready after T037
- **User Story 2 (P2)**: Independent of US1 - adds toggle functionality
- **User Story 3 (P3)**: Independent of US1/US2 - adds update functionality
- **User Story 4 (P3)**: Independent of US1/US2/US3 - adds delete functionality

### Within Each Layer

- **Database Layer (T006-T012)**: All functions can be implemented in parallel [P]
- **Service Layer (T018-T026)**: Models and services can be implemented in parallel [P]
- **Menu Layer (T032-T043)**: Menu functions for different stories implemented sequentially

### Parallel Opportunities

- **Setup (T001-T005)**: All tasks independent, can run in parallel
- **Database Functions (T006-T012)**: All marked [P], can implement concurrently
- **Service Functions (T018-T026)**: All marked [P], can implement concurrently
- **Documentation (T050-T054)**: All marked [P], can write concurrently

---

## Parallel Example: Database Layer

```bash
# Launch all database functions together:
Task: "Implement get_connection() in src/db.py"
Task: "Implement initialize_schema() in src/db.py"
Task: "Implement create_task_db() in src/db.py"
Task: "Implement get_all_tasks_db() in src/db.py"
Task: "Implement update_task_db() in src/db.py"
Task: "Implement toggle_task_status_db() in src/db.py"
Task: "Implement delete_task_db() in src/db.py"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T005)
2. Complete Phase 2: Foundational - Database Layer (T006-T017) → **Guardian Gate 1**
3. Complete Phase 3: Service Logic Layer (T018-T031) → **Guardian Gate 2**
4. Complete Phase 4: User Story 1 (T032-T037)
5. **STOP and VALIDATE**: Test User Story 1 independently (create tasks, view tasks, restart app, verify persistence)
6. Deploy/demo if ready - **This is the MVP**

### Incremental Delivery

1. Complete Setup + Foundational + Service Logic → Foundation ready → **2 Guardian Gates passed**
2. Add User Story 1 (T032-T037) → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 (T038-T039) → Test independently → Deploy/Demo
4. Add User Story 3 (T040-T041) → Test independently → Deploy/Demo
5. Add User Story 4 (T042-T043) → Test independently → Deploy/Demo
6. Final audit (T044-T049) → **Guardian Gate 3** → Documentation (T050-T054)
7. Each story adds value without breaking previous stories

### Sequential Agent Strategy

With constitution's mandatory workflow:

1. **Database Agent** completes T006-T012
2. **Spec Guardian** reviews T013-T017 → **Gate 1**
3. **Service Logic Agent** completes T018-T026
4. **Spec Guardian** reviews T027-T031 → **Gate 2**
5. **Menu/UX Agent** completes T032-T043 (all user stories)
6. **Spec Guardian** audits T044-T049 → **Gate 3**
7. **Documentation Agent** completes T050-T054

---

## Task Summary

**Total Tasks**: 54
- Setup: 5 tasks
- Database Layer: 7 implementation + 5 Guardian review = 12 tasks
- Service Layer: 9 implementation + 5 Guardian review = 14 tasks
- User Story 1 (MVP): 6 tasks
- User Story 2: 2 tasks
- User Story 3: 2 tasks
- User Story 4: 2 tasks
- Final Audit: 6 Guardian tasks
- Documentation: 5 tasks

**Tasks by User Story**:
- US1 (P1 - MVP): 6 tasks (T032-T037)
- US2 (P2): 2 tasks (T038-T039)
- US3 (P3): 2 tasks (T040-T041)
- US4 (P3): 2 tasks (T042-T043)

**Parallel Opportunities**: 27 tasks marked [P] (50% of tasks)

**Guardian Gates**: 3 mandatory review gates
- Gate 1: After T012 (Database layer)
- Gate 2: After T026 (Service layer)
- Gate 3: After T043 (All user stories)

**MVP Scope**: Phases 1-4 (T001-T037) = Create and View Tasks only

**Independent Test Criteria**:
- US1: Create tasks, view tasks, verify persistence after restart
- US2: Toggle completion status, verify persistence
- US3: Update task details, verify persistence
- US4: Delete tasks, verify removal persistence

---

## Notes

- All tasks follow strict checklist format: `- [ ] [ID] [P?] [Story?] Description with file path`
- [P] tasks = different files or functions, no dependencies within layer
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Constitution requires Guardian approval at 3 gates before proceeding
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence

---

**Tasks Status**: ✅ COMPLETE - Ready for implementation
**Next Command**: Begin with T001 (Setup phase)
**Orchestrator**: Coordinate agents following constitution's mandatory 8-step workflow
