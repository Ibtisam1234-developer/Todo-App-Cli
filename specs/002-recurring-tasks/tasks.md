# Implementation Tasks: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature**: 002-recurring-tasks
**Date**: 2025-12-31
**Status**: Implementation Complete - Auditing
**Spec**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)

---

## Phase 1: Setup

- [x] T001 [P] Install new dependencies (dateparser, questionary, plyer, python-dateutil) in requirements.txt
- [x] T002 [P] Verify existing database connection and schema are working

---

## Phase 2: Foundational - Database Schema Enhancement

- [x] T003 Update FULL_COLUMN_SET constant in src/repository.py to include new columns
- [x] T004 [US1] Add migration to src/database.py init_db()
- [x] T005 [US1] Add recurrence_interval CHECK constraint
- [x] T006 [US1] Add parent_task_id INT column with FK
- [x] T007 [US1] Add reminder_sent BOOLEAN column
- [x] T008 [US1] Create idx_tasks_parent index
- [x] T009 [US1] Create idx_tasks_reminder composite index
- [x] T010 [US1] Create idx_tasks_recurrence index

---

## Phase 3: Foundational - Repository Layer Updates

- [x] T011 [P] Update _row_to_dict() helper function
- [x] T012 [P] Update create_task() parameters
- [x] T013 [P] Update create_task() INSERT statement
- [x] T014 [P] Update update_task() reminder_sent reset
- [x] T015 [P] Update get_all_tasks() parent_task_id filter
- [x] T016 [P] Update get_all_tasks() recurrence_interval filter
- [x] T017 [P] Update get_all_tasks() due date filters
- [x] T018 [US4] Implement get_task_history() in src/repository.py
- [x] T019 [US4] Add SQL query for root + descendants

---

## Phase 4: Foundational - Data Model Enhancement

- [x] T020 [P] Add RecurrenceInterval enum to src/models.py
- [x] T021 [P] Add recurrence_interval to TaskCreate
- [x] T022 [P] Add recurrence_interval to TaskUpdate
- [x] T023 [P] Add recurrence_interval to Task model
- [x] T024 [P] Add parent_task_id to Task model
- [x] T025 [P] Add reminder_sent to Task model

---

## Phase 5: User Story 1 - Recurring Task Management (Priority: P1)

- [x] T026 [P] [US1] Create src/recurrence_engine.py module
- [x] T027 [US1] Implement calculate_next_due_date()
- [x] T028 [P] [US1] Add edge case handling for leap years
- [x] T029 [US1] Implement create_recurring_task_instance()
- [x] T030 [US1] Add task detail fetching logic
- [x] T031 [US1] Add parent_task_id setting
- [x] T032 [P] [US1] Add parse_natural_date() function to src/services.py
- [x] T033 [US1] Configure dateparser settings
- [x] T034 [US1] Update complete_task() trigger logic
- [x] T035 [US1] Add create_recurring_task_instance() call
- [x] T036 [US1] Add recurrence_interval option to add command
- [x] T037 [US1] Add recurrence_interval validation
- [x] T038 [US1] Update TaskCreate schema usage
- [x] T039 [US1] Add recurrence_interval display to list
- [x] T040 [US1] Add recurrence_interval display to details
- [x] T041 [US1] Add recurrence_interval to update command
- [x] T042 [US1] Add Success message for recurring instance

---

## Phase 6: User Story 2 - Smart Date Input (Priority: P2)

- [x] T043 [US2] Add questionary import to src/cli.py
- [x] T044 [US2] Implement interactive task form
- [x] T045 [US2] Add natural language date prompt
- [x] T046 [US2] Add error message for parsing failure
- [x] T047 [US2] Add fallback to interactive date picker
- [x] T048 [US2] Update add command to use parse_natural_date()
- [x] T049 [US2] Add parsing progress spinner
- [x] T050 [US2] Add due_date to update command
- [x] T051 [US2] Update update command for interactive mode
- [x] T052 [US2] Add example help text

---

## Phase 7: User Story 3 - Desktop Notifications (Priority: P3)

- [x] T053 [P] [US3] Create src/notification_manager.py
- [x] T054 [US3] Implement show_notification() with plyer
- [x] T055 [US3] Add try/except for graceful failure
- [x] T056 [US3] Implement check_due_tasks_and_notify()
- [x] T057 [US3] Add query logic for 30-min window
- [x] T058 [US3] Add notification grouping logic
- [x] T059 [US3] Add reminder_sent update
- [x] T060 [P] [US3] Add boot check to src/main.py
- [x] T061 [US3] Add async/await handling in main.py
- [x] T062 [US3] Add progress indicator during boot check
- [x] T063 [US3] Add detailed logging to .app.log
- [x] T064 [US3] Verify graceful OS degradation

---

## Phase 8: User Story 4 - Recurring Task History (Priority: P4)

- [x] T065 [P] [US4] Implement get_task_history() service
- [x] T066 [US4] Add root task validation
- [x] T067 [US4] Add sorting by creation date
- [x] T068 [US4] Add history command to CLI
- [x] T069 [US4] Add Rich Table for history
- [x] T070 [US4] Add parent_task_id column
- [x] T071 [US4] Add visual indicators for lineage
- [x] T072 [US4] Add Rich Panel series details
- [x] T073 [US4] Validate root_task_id existence

---

## Phase 9: Polish & Cross-Cutting Concerns

- [x] T074 [P] Detailed logging audit
- [x] T075 [P] Final type hint verification
- [x] T076 [P] Backward compatibility verification
- [x] T077 [P] Performance stress test
- [x] T078 [P] Final constitutional audit by Spec Guardian
- [x] T079 [P] Update root documentation (README/CLAUDE)
