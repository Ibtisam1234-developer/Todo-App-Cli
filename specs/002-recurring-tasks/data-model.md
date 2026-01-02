# Data Model: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature**: 002-recurring-tasks
**Date**: 2025-12-31
**Purpose**: Define entity structure, attributes, relationships, and validation rules

---

## Task Entity (Enhanced)

### Attributes

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | INT | Yes | Auto-increment | Unique task identifier (PRIMARY KEY) |
| `title` | TEXT | Yes | - | Task title (1-255 characters) |
| `description` | TEXT | No | NULL | Optional detailed description |
| `priority` | TEXT | No | 'medium' | Priority level: 'low', 'medium', 'high' |
| `category` | TEXT | No | NULL | Category name for grouping |
| `due_date` | TIMESTAMP | No | NULL | Due date/time for task (optional) |
| `is_completed` | BOOLEAN | No | FALSE | Completion status (TRUE/FALSE) |
| `recurrence_interval` | TEXT | No | 'none' | Recurrence: 'daily', 'weekly', 'monthly', 'none' |
| `parent_task_id` | INT | No | NULL | Reference to parent task (self-referential FK) |
| `reminder_sent` | BOOLEAN | No | FALSE | Whether notification sent for due task |
| `created_at` | TIMESTAMP | Yes | NOW() | Creation timestamp |
| `updated_at` | TIMESTAMP | Yes | NOW() | Last modification timestamp |

### Validation Rules

**Title**:
- Required (NOT NULL)
- Length: 1-255 characters
- Must be non-empty after trimming whitespace

**Priority**:
- Optional (default: 'medium')
- Enum values: 'low', 'medium', 'high'
- CHECK constraint: `priority IN ('low', 'medium', 'high')`

**Category**:
- Optional (NULL allowed)
- Length: 1-100 characters if provided
- No constraint on uniqueness (can reuse categories)

**Due Date**:
- Optional (NULL allowed)
- Must be valid TIMESTAMP
- Can be any future or past date (no validation logic)

**Recurrence Interval**:
- Optional (default: 'none')
- Enum values: 'daily', 'weekly', 'monthly', 'none'
- CHECK constraint: `recurrence_interval IN ('daily', 'weekly', 'monthly', 'none')`
- Only meaningful when `due_date` is set

**Parent Task ID**:
- Optional (NULL allowed)
- Foreign key: `parent_task_id REFERENCES tasks(id) ON DELETE SET NULL`
- Self-referential relationship (task can reference another task)
- NULL for original tasks, non-NULL for child instances

**Reminder Sent**:
- Optional (default: FALSE)
- Tracks whether desktop notification displayed for due task
- Reset to FALSE when `due_date` changes or task uncompleted

### State Transitions

**Task Creation**:
- `id`: Auto-assigned by database
- `created_at`, `updated_at`: Set to current timestamp
- `is_completed`: FALSE (new tasks are pending)
- `reminder_sent`: FALSE (no notification yet)
- `recurrence_interval`: User-specified or 'none' (default)
- `parent_task_id`: NULL for new tasks, or parent ID when creating child instance

**Task Completion**:
- `is_completed`: Set to TRUE
- `updated_at`: Updated to current timestamp
- **Trigger**: If `recurrence_interval != 'none'`, create new task with:
  - Same `title`, `description`, `priority`, `category`, `recurrence_interval`
  - `parent_task_id` = current task's `id`
  - `due_date` = calculated next occurrence (if original had `due_date`)
  - `is_completed` = FALSE
  - `reminder_sent` = FALSE
  - `created_at` = current timestamp

**Task Update**:
- Any field modified → `updated_at` = current timestamp
- If `due_date` changed → `reminder_sent` = FALSE (reset notification flag)

**Task Deletion**:
- Cascade delete: Child tasks (`WHERE parent_task_id = deleted_id`) have `parent_task_id` set to NULL (FK constraint)
- Original deletion removes task permanently

### Relationships

**Self-Referential Parent-Child**:
```
tasks.id (parent)
    ↓ 1-to-many
tasks.parent_task_id (child)
```

**Example Recurring Task Series**:
```
Task 1 (id: 1, parent_task_id: NULL, recurrence_interval: "weekly", due_date: "2025-01-06")
  ↓ completed → creates child
Task 2 (id: 2, parent_task_id: 1, recurrence_interval: "weekly", due_date: "2025-01-13")
  ↓ completed → creates child
Task 3 (id: 3, parent_task_id: 2, recurrence_interval: "weekly", due_date: "2025-01-20")
```

**Querying Task History**:
- Get all instances of a recurring series: `WHERE id = X OR parent_task_id = X`
- Get root task: `WHERE id = X AND parent_task_id IS NULL`
- Get direct children: `WHERE parent_task_id = X`

---

## Pydantic Models

### RecurrenceInterval (Enum)

```python
from enum import Enum

class RecurrenceInterval(str, Enum):
    """Recurrence interval for tasks."""
    NONE = 'none'
    DAILY = 'daily'
    WEEKLY = 'weekly'
    MONTHLY = 'monthly'
```

### TaskCreate (Schema for Task Creation)

```python
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, Field, ConfigDict

class TaskCreate(BaseModel):
    """Schema for creating a new task with enhanced fields."""
    title: str = Field(min_length=1, max_length=255)
    description: Optional[str] = None
    priority: Priority = Priority.MEDIUM
    category: Optional[str] = Field(default=None, max_length=100)
    due_date: Optional[datetime] = None
    recurrence_interval: RecurrenceInterval = RecurrenceInterval.NONE

    model_config = ConfigDict(use_enum_values=True)
```

### TaskUpdate (Schema for Task Updates)

```python
class TaskUpdate(BaseModel):
    """Schema for updating an existing task."""
    title: Optional[str] = Field(default=None, min_length=1, max_length=255)
    description: Optional[str] = None
    priority: Optional[Priority] = None
    category: Optional[str] = Field(default=None, max_length=100)
    due_date: Optional[datetime] = None
    is_completed: Optional[bool] = None
    recurrence_interval: Optional[RecurrenceInterval] = None

    model_config = ConfigDict(use_enum_values=True)
```

### Task (Enhanced Task Model)

```python
class Task(BaseModel):
    """
    Enhanced Task entity with recurrence and history tracking.

    Attributes:
        id: Unique task identifier
        title: Task title (1-255 characters)
        description: Optional detailed description
        priority: Priority level (low, medium, high)
        category: Optional category name
        due_date: Optional due date timestamp
        is_completed: Completion status (mapped to TaskStatus)
        recurrence_interval: Recurrence interval (daily, weekly, monthly, none)
        parent_task_id: Reference to parent task (for recurring task instances)
        reminder_sent: Whether notification sent
        created_at: Creation timestamp
        updated_at: Last modification timestamp
    """
    id: int
    title: str = Field(min_length=1, max_length=255)
    description: Optional[str] = None
    priority: Priority = Priority.MEDIUM
    category: Optional[str] = None
    due_date: Optional[datetime] = None
    status: TaskStatus = TaskStatus.PENDING  # Computed from is_completed
    recurrence_interval: RecurrenceInterval = RecurrenceInterval.NONE
    parent_task_id: Optional[int] = None
    reminder_sent: bool = False
    created_at: datetime
    updated_at: datetime

    model_config = ConfigDict(use_enum_values=True, from_attributes=True)
```

### DueTaskSummary (For Notifications)

```python
class DueTaskSummary(BaseModel):
    """Simplified model for notification display."""
    id: int
    title: str
    due_date: datetime
    priority: Priority
    minutes_until_due: int  # Calculated: (due_date - now).total_seconds() / 60
```

---

## Database Schema (DDL)

### Table Definition

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,
    priority TEXT DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high')),
    category TEXT,
    due_date TIMESTAMP,
    is_completed BOOLEAN DEFAULT FALSE,
    recurrence_interval TEXT DEFAULT 'none' CHECK (recurrence_interval IN ('daily', 'weekly', 'monthly', 'none')),
    parent_task_id INT REFERENCES tasks(id) ON DELETE SET NULL,
    reminder_sent BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Indexes

```sql
-- Index for parent-child queries (task history)
CREATE INDEX IF NOT EXISTS idx_tasks_parent ON tasks(parent_task_id);

-- Index for reminder check (fast query of pending, unnotified due tasks)
CREATE INDEX IF NOT EXISTS idx_tasks_reminder
ON tasks(due_date) WHERE is_completed = FALSE AND reminder_sent = FALSE;

-- Index for recurrence filtering (find all recurring tasks)
CREATE INDEX IF NOT EXISTS idx_tasks_recurrence
ON tasks(recurrence_interval) WHERE recurrence_interval != 'none';

-- Existing indexes (from 001-task-organization)
CREATE INDEX IF NOT EXISTS idx_tasks_priority ON tasks(priority);
CREATE INDEX IF NOT EXISTS idx_tasks_due_date ON tasks(due_date);
```

### Migration (Idempotent)

```sql
-- Add recurrence_interval column (backward compatible)
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS recurrence_interval TEXT DEFAULT 'none';

-- Add check constraint (non-validating for existing data)
ALTER TABLE tasks ADD CONSTRAINT recurrence_check
CHECK (recurrence_interval IN ('daily', 'weekly', 'monthly', 'none')) NOT VALID;

-- Validate constraint for future rows
ALTER TABLE tasks VALIDATE CONSTRAINT recurrence_check;

-- Add parent_task_id foreign key
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS parent_task_id INT;
ALTER TABLE tasks ADD CONSTRAINT fk_tasks_parent
FOREIGN KEY (parent_task_id) REFERENCES tasks(id) ON DELETE SET NULL;

-- Add reminder_sent flag
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS reminder_sent BOOLEAN DEFAULT FALSE;

-- Create indexes
CREATE INDEX IF NOT EXISTS idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX IF NOT EXISTS idx_tasks_reminder
ON tasks(due_date) WHERE is_completed = FALSE AND reminder_sent = FALSE;
CREATE INDEX IF NOT EXISTS idx_tasks_recurrence
ON tasks(recurrence_interval) WHERE recurrence_interval != 'none';
```

---

## Edge Case Handling

### Recurrence Without Due Date

**Scenario**: Task with `recurrence_interval = 'daily'` but `due_date = NULL`

**Behavior**:
- Task creates new instance on completion
- New instance has same attributes (title, description, priority, category, recurrence_interval)
- New instance's `due_date` = NULL (cannot calculate next date)
- New instance's `parent_task_id` = completed task's ID

**Rationale**: Maintains recurrence pattern even when due dates are not set. User can add due dates later.

### Leap Year Handling

**Scenario**: Monthly recurring task due on February 29th in a leap year

**Behavior**:
- Using `dateutil.relativedelta(months=1)`
- Next occurrence in non-leap year → February 28th (last day of month)
- `relativedelta` automatically adjusts to valid date

### End-of-Month Handling

**Scenario**: Monthly recurring task due on January 31st

**Behavior**:
- February occurrence → February 28th (last day of month)
- March occurrence → March 31st
- `relativedelta(months=1)` handles this automatically

### Ambiguous Date Expressions

**Scenario**: User enters "Friday" and today is Friday 10:00 AM

**Behavior** (via `dateparser`):
- Interprets as today (Friday) if current time < due time
- Interprets as next Friday if current time > due time
- Returns None if ambiguous and cannot parse → triggers interactive picker

---

**Next Phase**: See [contracts/](./contracts/) for API and service layer contracts.
