# Implementation Plan: Advanced Todo CLI - Recurring Tasks & Smart Features

**Branch**: `002-recurring-tasks` | **Date**: 2025-12-31 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/002-recurring-tasks/spec.md`

## Summary

Implementation of advanced Todo CLI features including recurring task management, natural language date parsing, desktop notifications, and task history tracking. The feature extends the existing async repository architecture with three new modules (recurrence_engine.py, notification_manager.py) and enhances the CLI with questionary interactive forms. All additions must comply with the v2.0.0 Constitution's async/await patterns, type safety, and Rich UI standards.

**Core capabilities:**
1. Automatic task regeneration on completion with configurable intervals (daily, weekly, monthly)
2. Natural language date parsing using dateparser library
3. Cross-platform desktop notifications via plyer
4. Parent-child task relationship tracking for recurring task series

## Technical Context

**Language/Version**: Python 3.12+
**Primary Dependencies**:
- dateparser (natural language date parsing)
- questionary (interactive CLI forms and date pickers)
- plyer (cross-platform desktop notifications)
- Existing stack: Typer, Rich, Pydantic, psycopg[pool]

**Storage**: Neon Serverless PostgreSQL (existing)
**Testing**: pytest (existing)
**Target Platform**: Windows/Linux/macOS (cross-platform CLI)
**Project Type**: single (CLI application)
**Performance Goals**:
- Recurring task regeneration: < 2 seconds after completion
- Natural language date parsing: < 100ms per input
- Desktop notification trigger: < 1 second from app launch
- Database query for due tasks: < 200ms

**Constraints**:
- Must maintain 100% async/await patterns
- All new code must have 100% type annotations
- Rich UI required for all user output
- No blocking I/O in async path
- Desktop notifications must gracefully fail on unsupported systems

**Scale/Scope**:
- Support 100+ recurring task completions without degradation
- Handle 20+ tasks due within 30 minutes for notifications
- Track unlimited task history chains via parent_task_id

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Initial Assessment

**All 11 Constitutional Principles**: ✅ PASS

**I. Separation of Concerns** - ✅ PASS
- New modules respect layer boundaries:
  - `recurrence_engine.py`: Service Logic Agent (business logic, no database/CLI)
  - `notification_manager.py`: Service Logic Agent (cross-platform notification logic)
  - `questionary` integration: CLI/UX Agent (user interaction only)

**II. Orchestrator-Controlled Workflow** - ✅ PASS
- New features follow 10-step execution flow
- No shortcuts or skipped reviews

**III. Guardian Gate System** - ✅ PASS
- All new modules subject to Guardian review
- No exceptions to gate requirements

**IV. Code Location Discipline** - ✅ PASS
- All new code in `/src`
- New modules: `src/recurrence_engine.py`, `src/notification_manager.py`
- Existing files extended: `src/models.py`, `src/repository.py`, `src/cli.py`

**V. No Interaction Mixing** - ✅ PASS
- `recurrence_engine.py`: No database connections, no user input
- `notification_manager.py`: No user input, no Rich UI (system notifications only)
- Questionary integration: Only in CLI layer, no business logic

**VI. Asynchronous Architecture** - ✅ PASS
- All recurrence logic functions are async
- Notification check on boot is async
- Date parsing is synchronous but isolated from async path (acceptable)

**VII. Type Safety Mandate** - ✅ PASS
- All new functions have return type annotations
- All parameters have type annotations
- Pydantic models used for all data structures

**VIII. Rich UI Standard** - ✅ PASS
- Questionary forms replace basic input()
- Rich error messages for date parsing failures
- Rich progress indicators for notification checks

**IX. Logging and Observability** - ✅ PASS
- Recurrence engine operations logged to `.app.log`
- Notification attempts and failures logged
- Date parsing errors logged

**X. Neon DB Integration Standard (ENHANCED)** - ✅ PASS
- New columns (recurrence_interval, parent_task_id, reminder_sent) added via ALTER TABLE
- Parameterized queries for all new operations
- Connection pooling used (existing)

**XI. Input Validation Standard** - ✅ PASS
- Natural language dates validated via dateparser before database
- Questionary forms validate before Pydantic models
- Recurrence interval enum validation

### Phase 1 Re-Check (After Design)

**Status**: ✅ PASS - No violations introduced during design phase

**Verification Summary**:
- All new modules respect layer boundaries
- No cross-layer dependencies introduced
- Async patterns maintained throughout
- Type annotations specified in all contracts
- Rich UI integration isolated to CLI layer
- Date parsing isolated from async path
- Notification logic gracefully degrades
- Database migration is idempotent and backward compatible

## Project Structure

### Documentation (this feature)

```text
specs/002-recurring-tasks/
├── plan.md              # This file (/sp.plan command output)
├── research.md          # Phase 0 output (/sp.plan command)
├── data-model.md        # Phase 1 output (/sp.plan command)
├── quickstart.md        # Phase 1 output (/sp.plan command)
├── contracts/           # Phase 1 output (/sp.plan command)
│   ├── service-contract.md
│   └── repository-contract.md
└── tasks.md             # Phase 2 output (/sp.tasks command - NOT created by /sp.plan)
```

### Source Code (repository root)

```text
src/
├── database.py         # Existing - connection pool, schema (enhanced for new columns)
├── repository.py       # Existing - CRUD SQL (enhanced for recurrence/history queries)
├── services.py         # Existing - business orchestration (enhanced for recurrence)
├── models.py           # Existing - Pydantic schemas (enhanced for new fields)
├── recurrence_engine.py    # NEW - calculates next due dates, creates recurring instances
├── notification_manager.py  # NEW - handles plyer notifications
├── cli.py              # Existing - Typer commands (enhanced with questionary, boot check)
└── main.py             # Existing - application entry point (enhanced for notification boot check)
```

**Structure Decision**: Single project structure maintained. New modules added to `/src` for recurrence logic and notifications, respecting existing async repository architecture. No restructuring required - all extensions are additive.

## Complexity Tracking

> **No constitution violations requiring justification. All principles pass.**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | N/A |

---

**Next Phase**: See [research.md](./research.md) for Phase 0 technical research and decision-making.
