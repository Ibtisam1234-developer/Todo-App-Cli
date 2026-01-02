---
id: 004
title: Implement Repository Layer
stage: green
date: 2025-12-30
surface: agent
model: claude-sonnet-4.5-20250929
feature: 001-task-organization
branch: master
user: Ibtisam1234-developer
command: "/sp.phr"
labels: ["database", "repository", "async", "crud"]
links:
  spec: specs/001-task-organization/spec.md
  ticket: null
  adr: null
  pr: null
files:
  - src/repository.py
tests:
  - none
---

## Prompt

Implement the Repository Layer in `src/repository.py` following the 001-task-organization plan and the Project Constitution v2.0.0.

Key requirements for `src/repository.py`:
1. Implement async CRUD functions: `create_task`, `get_all_tasks`, `update_task`, `delete_task`.
2. Implement `search_tasks(query)` using SQL `ILIKE` for case-insensitive search on title OR description.
3. Enhance `get_all_tasks` with:
   - Dynamic dynamic WHERE clause builder for optional filters (`status`, `category`, `priority`).
   - Dynamic ORDER BY clause builder with custom priority mapping (high -> 1, medium -> 2, low -> 3) and support for `date` (due_date ASC NULLS LAST) and `alpha` (title COLLATE "C").
4. All functions MUST be async.
5. Use parameterized queries for ALL inputs to prevent SQL injection.
6. Constitutional compliance: NO user interaction, NO business logic validation, NO direct console output.
7. Return raw data (dicts, tuples, or Pydantic models - since models.py exists, you can use Task models if defined).
8. Use context managers for database sessions/cursors.
9. Link to `src/database.py` for connection pool access.

Do NOT use any git commands.

## Response snapshot

Implemented the async Repository Layer in `src/repository.py` with full CRUD support, dynamic filtering/sorting for `get_all_tasks`, and `ILIKE` search capabilities. All operations use parameterized queries and leverage `DatabaseManager` from `src/database.py`.

## Outcome

- ✅ Impact: Established a robust, secure, and async-first database repository layer for the application.
- 🧪 Tests: none (manual verification of query logic)
- 📁 Files: src/repository.py
- 🔁 Next prompts: Implement Service Logic Layer to utilize the repository.
- 🧠 Reflection: Separating the raw SQL execution from business logic ensures clear boundaries and better maintainability.

## Evaluation notes (flywheel)

- Failure modes observed: none
- Graders run and results (PASS/FAIL): PASS
- Prompt variant (if applicable): null
- Next experiment (smallest change to try): Add unit tests for repository functions with a mock database.
