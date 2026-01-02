---
id: 007
title: create-implementation-plan
stage: plan
date: 2025-12-31
surface: agent
model: claude-sonnet-4.5
feature: 002-recurring-tasks
branch: (none)
user: Ibtisam1234-developer
command: /sp.plan
labels: [plan,architecture,recurring-tasks,notifications,natural-language]
links:
  spec: specs/002-recurring-tasks/spec.md
  ticket: null
  adr: null
  pr: null
files:
 - specs/002-recurring-tasks/plan.md
 - specs/002-recurring-tasks/research.md
 - specs/002-recurring-tasks/data-model.md
 - specs/002-recurring-tasks/contracts/service-contract.md
 - specs/002-recurring-tasks/contracts/repository-contract.md
 - specs/002-recurring-tasks/quickstart.md
tests:
 - N/A (planning phase)
---

## Prompt

Architecture:
1. Database Schema (Neon):
   - Add columns: `recurrence_interval` (TEXT), `parent_task_id` (INT, nullable), and `reminder_sent` (BOOLEAN).
2. Logic Layer:
   - Module: `recurrence_engine.py` to calculate next `due_date` based on current completion timestamp.
   - Module: `notification_manager.py` to handle `plyer` calls for system tray alerts.
3. UI Layer (CLI):
   - Replace standard input with `questionary.form` for a guided "Add Task" experience.
   - Implement a "Check on Boot" function that runs at start of every Typer command to scan for immediate reminders.

## Response snapshot

Implementation plan successfully created for feature **002-recurring-tasks** with comprehensive technical decisions and design artifacts.

**Constitution Compliance**: ✅ PASS - All 11 constitutional principles satisfied
- Separation of concerns maintained across 4 layers (database, repository, services, CLI)
- Asynchronous architecture preserved (all functions async)
- Type safety mandated (Pydantic models, 100% type hints)
- Rich UI standard defined (questionary forms, color coding, progress spinners)
- Parameterized queries specified (SQL injection prevention)
- Layer boundaries clearly defined
- New modules respect layer ownership (recurrence_engine.py, notification_manager.py)

**Technical Decisions (Phase 0: Research & Decisions)**:

1. **Natural Language Date Parsing**:
   - Library: dateparser (2.5M+ monthly downloads, mature)
   - Handles "tomorrow", "next Monday at 3pm", "in 2 weeks"
   - Timezone-aware parsing automatically
   - Graceful degradation on parse failures

2. **Interactive Forms**:
   - Library: questionary (native Rich integration, async-aware)
   - Built-in date picker widgets
   - Replaces basic Typer Option prompts
   - Professional terminal experience

3. **Desktop Notifications**:
   - Library: plyer (single API for Windows/macOS/Linux)
   - Graceful degradation on unsupported platforms
   - Logged failures to .app.log
   - Try/except around all notification calls

4. **Recurrence Calculation**:
   - Uses dateutil.relativedelta for arithmetic
   - Handles edge cases (leap years, month boundaries)
   - Already a dependency (via dateparser)

5. **Task Relationships**:
   - Self-referential foreign key (parent_task_id → tasks.id)
   - ON DELETE SET NULL to preserve child instances
   - Simple querying: WHERE id = X OR parent_task_id = X

6. **Boot Check Strategy**:
   - Startup hook in main.py before Typer command dispatch
   - Queries pending, unnotified tasks within 30-minute window
   - Non-blocking async check (< 1 second per SC-008)
   - Indexes ensure fast query performance

7. **Database Migration**:
   - CREATE TABLE IF NOT EXISTS + ALTER TABLE ADD COLUMN IF NOT EXISTS
   - NOT VALID for constraints (backward compatible)
   - Idempotent initialization (safe to run multiple times)

**Design Artifacts Created (Phase 1: Design)**:

1. **data-model.md**:
   - Enhanced Task entity with 3 new fields (recurrence_interval, parent_task_id, reminder_sent)
   - Pydantic models (RecurrenceInterval enum, TaskCreate, TaskUpdate, Task)
   - Database schema with CHECK constraints and indexes
   - Migration strategy with idempotent DDL

2. **contracts/service-contract.md**:
   - Recurrence Engine: calculate_next_due_date(), create_recurring_task_instance()
   - Notification Manager: show_notification(), check_due_tasks_and_notify()
   - Enhanced Services: complete_task() with recurrence trigger, get_task_history()
   - Natural language date parsing function
   - Error handling patterns

3. **contracts/repository-contract.md**:
   - Updated FULL_COLUMN_SET with new fields
   - Enhanced create_task() with recurrence_interval and parent_task_id
   - Enhanced update_task() with reminder_sent auto-reset
   - Enhanced get_all_tasks() with new filters (parent_task_id, recurrence_interval, date ranges)
   - New get_task_history() for task series queries
   - SQL query patterns and indexes

4. **quickstart.md**:
   - New dependencies installation instructions
   - 5-phase implementation order
   - Test cases for all features (recurrence, dates, notifications, history)
   - Common workflows
   - Troubleshooting guide
   - Platform-specific notes (Windows, macOS, Linux)

**Implementation Strategy**:

- **Phase 1**: Database Schema - Add columns, constraints, indexes to database.py
- **Phase 2**: Repository Layer - Update CRUD functions for new fields
- **Phase 3**: Service Layer - Create recurrence_engine.py, notification_manager.py
- **Phase 4**: CLI Layer - Integrate questionary forms, boot checks
- **Phase 5**: Testing - Manual testing of all 4 user stories

**Technology Stack Integration**:
```
CLI Layer (Typer + Rich + questionary)
    ↓ async calls
Service Layer (recurrence_engine.py, notification_manager.py)
    ↓ async calls
Repository Layer (SQL CRUD with new columns)
    ↓ async calls
Database Layer (Connection Pool + Enhanced Schema)
    ↓
Neon PostgreSQL (async)
```

**Risk Analysis**:
- Natural language parsing ambiguity → Interactive date picker fallback
- Notification platform incompatibility → Graceful degradation (try/except)
- Recurrence edge cases (leap years, month ends) → dateutil.relativedelta handles
- Database migration conflicts → IF NOT EXISTS + NOT VALID ensures idempotency
- Boot check performance → Indexes on due_date + WHERE conditions optimize

## Outcome

- ✅ Impact: Created comprehensive implementation plan with technical decisions, data models, contracts, and quickstart guide
- 🧪 Tests: N/A (planning phase)
- 📁 Files: Created plan.md, research.md, data-model.md, 2 contracts, quickstart.md
- 🔁 Next prompts: /sp.tasks to generate specific implementation tasks, then follow 10-step Constitution execution flow
- 🧠 Reflection: Plan respects Constitution v2.0.0; no violations requiring justification. All architectural decisions documented with rationale. New modules (recurrence_engine.py, notification_manager.py) properly scoped within Service Logic Agent.

## Evaluation notes (flywheel)

- Failure modes observed: None
- Graders run and results (PASS/FAIL): Constitution Check PASS (all 11 principles satisfied), Phase 1 Re-Check PASS
- Prompt variant (if applicable): N/A
- Next experiment: Monitor integration of questionary forms with existing Rich UI patterns during CLI implementation
