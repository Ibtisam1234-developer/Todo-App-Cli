# Data Model: Interactive Todo Application

**Feature**: 001-todo-app
**Date**: 2025-12-17
**Purpose**: Database schema and entity definitions

## Entity: Task

### Description

Represents a single todo item with title, optional description, completion status, and timestamps.

### Fields

| Field | Type | Constraints | Description | Validation Rules |
|-------|------|-------------|-------------|------------------|
| id | SERIAL | PRIMARY KEY | Auto-incrementing unique identifier | System-generated, read-only |
| title | VARCHAR(200) | NOT NULL | Task title (required) | 1-200 characters, non-empty after strip |
| description | TEXT | NULL | Optional detailed task description | Max 1000 characters (service layer validation) |
| is_completed | BOOLEAN | NOT NULL, DEFAULT FALSE | Task completion status | true (complete) or false (incomplete) |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Task creation timestamp | System-generated, immutable |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last modification timestamp | System-updated on each change |

### Relationships

None. Task is a standalone entity with no foreign key relationships.

### State Transitions

```
┌─────────────┐
│  is_completed│
│   = false    │  (Default state on creation)
│  (Incomplete)│
└──────┬───────┘
       │
       │ toggle_task_status()
       ├──────────────────────┐
       │                      │
       ▼                      ▼
┌─────────────┐         ┌─────────────┐
│ is_completed│◄────────┤ is_completed│
│   = true    │         │   = false   │
│ (Complete)  │────────►│ (Incomplete)│
└─────────────┘         └─────────────┘
    toggle_task_status()
```

**State Rules**:
- New tasks default to `is_completed = false`
- Status toggles between true and false
- No other status values permitted
- Status changes update `updated_at` timestamp

---

## Database Schema

### SQL DDL

```sql
-- Table: tasks
-- Purpose: Store all todo tasks with completion tracking
-- Owner: Database Agent

CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    is_completed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Index on completion status for filtering (optional optimization)
CREATE INDEX IF NOT EXISTS idx_tasks_completed ON tasks(is_completed);

-- Index on created_at for sorting (optional optimization)
CREATE INDEX IF NOT EXISTS idx_tasks_created ON tasks(created_at DESC);
```

### Schema Notes

- **SERIAL type**: PostgreSQL auto-increment, generates unique IDs
- **VARCHAR(200)**: Title length matches FR specification (1-200 chars)
- **TEXT type**: Unlimited description length (service layer enforces 1000 char limit)
- **TIMESTAMP**: Neon stores in UTC, application should handle timezone conversion if needed
- **Indexes**: Optional for performance, may be added later if needed

---

## Python Data Model

### Task Dataclass

```python
# src/models.py
# Owner: Service Logic Agent

from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class Task:
    """
    Task entity representing a todo item.

    Attributes:
        id: Unique task identifier
        title: Task title (1-200 characters)
        description: Optional detailed description
        is_completed: Completion status (True = complete, False = incomplete)
        created_at: Creation timestamp
        updated_at: Last modification timestamp
    """
    id: int
    title: str
    description: Optional[str]
    is_completed: bool
    created_at: datetime
    updated_at: datetime

    def to_dict(self) -> dict:
        """
        Convert Task to dictionary representation.

        Returns:
            Dictionary with all task fields
        """
        return {
            'id': self.id,
            'title': self.title,
            'description': self.description,
            'is_completed': self.is_completed,
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat()
        }

    @classmethod
    def from_db_row(cls, row: tuple) -> 'Task':
        """
        Create Task from database row tuple.

        Args:
            row: Database row as tuple (id, title, description, is_completed, created_at, updated_at)

        Returns:
            Task instance
        """
        return cls(
            id=row[0],
            title=row[1],
            description=row[2],
            is_completed=row[3],
            created_at=row[4],
            updated_at=row[5]
        )

    def __str__(self) -> str:
        """String representation for display."""
        status = "✓" if self.is_completed else "○"
        desc_preview = f" - {self.description[:50]}..." if self.description else ""
        return f"[{status}] #{self.id}: {self.title}{desc_preview}"
```

---

## Validation Rules

### Database Layer (src/db.py)

- **No validation**: Database layer performs NO validation, only executes queries
- **Constraint enforcement**: PostgreSQL enforces NOT NULL and type constraints
- **Error handling**: Catch psycopg2 exceptions, raise with context

### Service Layer (src/services.py)

| Field | Validation | Error Message |
|-------|-----------|---------------|
| title | Required, 1-200 chars, non-empty after strip | "Title is required and must be 1-200 characters" |
| description | Optional, max 1000 chars if provided | "Description must be 1000 characters or less" |
| task_id | Must exist in database for update/delete/toggle | "Task with ID {id} not found" |
| create | At least title must be provided | "Title is required to create a task" |
| update | At least one field (title or description) must be provided | "Must provide title or description to update" |

### Menu Layer (src/menu.py)

- **Input collection**: Prompt user for required fields
- **Type conversion**: Convert string input to appropriate types
- **Error display**: Show validation errors from service layer to user
- **Retry logic**: Allow user to retry after validation errors

---

## Data Flow

### Create Task Flow

```
1. Menu Layer (src/menu.py):
   - Prompt user for title and description
   - Call services.create_task(title, description)

2. Service Layer (src/services.py):
   - Validate title (required, 1-200 chars)
   - Validate description (optional, max 1000 chars)
   - Call db.create_task_db(conn, title, description)
   - Return success/failure response

3. Database Layer (src/db.py):
   - Execute parameterized INSERT query
   - Return new task ID
   - PostgreSQL sets is_completed=false, created_at=NOW(), updated_at=NOW()
```

### Read Tasks Flow

```
1. Menu Layer (src/menu.py):
   - Call services.list_tasks()

2. Service Layer (src/services.py):
   - Call db.get_all_tasks_db(conn)
   - Convert row tuples to Task objects using Task.from_db_row()
   - Return list of Task objects

3. Database Layer (src/db.py):
   - Execute SELECT * FROM tasks ORDER BY created_at DESC
   - Return list of row tuples
```

### Update Task Flow

```
1. Menu Layer (src/menu.py):
   - Prompt user for task ID, new title, new description
   - Call services.update_task(task_id, title, description)

2. Service Layer (src/services.py):
   - Validate at least one field provided
   - Validate title if provided (1-200 chars, non-empty)
   - Validate description if provided (max 1000 chars)
   - Call db.update_task_db(conn, task_id, title, description)
   - Return success/failure response

3. Database Layer (src/db.py):
   - Build dynamic UPDATE query (only update provided fields)
   - Set updated_at = NOW()
   - Execute parameterized UPDATE query
   - Return true if updated, false if not found
```

### Toggle Completion Flow

```
1. Menu Layer (src/menu.py):
   - Prompt user for task ID
   - Call services.toggle_task_completion(task_id)

2. Service Layer (src/services.py):
   - Validate task_id (no other validation needed)
   - Call db.toggle_task_status_db(conn, task_id)
   - Return success/failure response

3. Database Layer (src/db.py):
   - Execute UPDATE tasks SET is_completed = NOT is_completed, updated_at = NOW()
   - Return true if toggled, false if not found
```

### Delete Task Flow

```
1. Menu Layer (src/menu.py):
   - Prompt user for task ID
   - Call services.delete_task(task_id)

2. Service Layer (src/services.py):
   - Validate task_id (no other validation needed)
   - Call db.delete_task_db(conn, task_id)
   - Return success/failure response

3. Database Layer (src/db.py):
   - Execute DELETE FROM tasks WHERE id = %s
   - Return true if deleted, false if not found
```

---

## Migration Strategy

### Initial Schema Creation

- Run `initialize_schema()` on first application start
- CREATE TABLE IF NOT EXISTS ensures idempotent execution
- No migration framework needed for initial version

### Future Schema Changes

If schema changes required:
1. Create new migration SQL file
2. Track applied migrations (future: add migrations table)
3. Run migrations on application start
4. Document breaking changes

---

## Performance Considerations

### Expected Load

- Single-user local application
- < 10 operations per minute typical
- < 1000 total tasks expected

### Optimization Strategy

- **No premature optimization**: Simple queries sufficient for expected load
- **Optional indexes**: Can add indexes if performance issues observed
- **Query patterns**: All queries are simple single-table operations (no joins)

### Neon Serverless Considerations

- Neon auto-scales compute
- Cold start latency possible after inactivity
- Connection pooling handled server-side
- No client-side tuning required

---

## Data Integrity

### ACID Guarantees

- PostgreSQL provides full ACID compliance
- Each operation is a single transaction
- No multi-statement transactions required

### Referential Integrity

- No foreign keys (single entity application)
- No cascade deletes needed

### Data Loss Prevention

- Database-level constraints prevent invalid data
- Application-level validation provides user-friendly errors
- Neon provides automatic backups

---

## Summary

**Entity**: Task (id, title, description, is_completed, created_at, updated_at)
**Schema**: Single table, simple structure, no relationships
**Validation**: Service layer validates, database enforces constraints
**State**: Binary completion status with toggle capability
**Performance**: Simple queries sufficient for single-user scope

**Constitution Compliance**: ✅ Data model respects layer separation (database structure, service validation, menu display)

**Next**: Proceed to contract definitions for each layer
