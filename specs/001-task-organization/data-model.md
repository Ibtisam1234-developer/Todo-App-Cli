# Data Model: Task Organization & Usability

**Feature**: 001-task-organization
**Purpose**: Define data structures for tasks with priorities, categories, and timestamps
**Date**: 2025-12-30

## Entities

### Task

**Description**: Represents a to-do item with organization attributes (priority, category, timestamps)

**Fields**:

| Field | Type | Required | Default | Validation | Description |
|-------|--------|-----------|------------|-------------|
| id | INTEGER | Yes | Auto-generated | Primary key, auto-increment |
| title | TEXT | Yes | - | Task title, max 255 chars (enforced in Pydantic) |
| description | TEXT | No | NULL | Optional task description |
| priority | TEXT | Yes | "medium" | One of: low, medium, high (enum in Pydantic) |
| category | TEXT | No | NULL | Free-form category label (e.g., "work", "personal") |
| status | TEXT | Yes | "pending" | One of: pending, completed (existing field) |
| created_at | TIMESTAMP | Yes | NOW() | Auto-generated timestamp on creation |
| due_date | TIMESTAMP | No | NULL | Optional due date for the task |

**Constraints**:
- Title must be non-empty
- Priority must be one of: "low", "medium", "high"
- Status must be one of: "pending", "completed"
- created_at is auto-populated by database
- category and due_date are optional (NULL allowed)

**State Transitions**:
- `pending` → `completed` (task marked as done)
- `completed` → `pending` (task reactivated)

## Relationships

### Task to Category

**Relationship**: Weak association (category stored as TEXT on task)
- Task has zero or one category (stored as free-form text)
- Category is not a separate entity in this feature scope
- No foreign key constraint (category is just a text label)
- Future enhancement: Separate categories table with FK per constitution X

## Pydantic Model Definition

```python
from pydantic import BaseModel, Field, field_validator
from enum import Enum
from datetime import datetime
from typing import Optional

class Priority(str, Enum):
    """Task priority levels"""
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

class TaskStatus(str, Enum):
    """Task status values"""
    PENDING = "pending"
    COMPLETED = "completed"

class TaskCreate(BaseModel):
    """Model for creating a new task"""
    title: str = Field(..., min_length=1, max_length=255)
    description: Optional[str] = Field(None, max_length=1000)
    priority: Priority = Field(default=Priority.MEDIUM)
    category: Optional[str] = Field(None, max_length=100)
    due_date: Optional[datetime] = None

class TaskUpdate(BaseModel):
    """Model for updating an existing task"""
    title: Optional[str] = Field(None, min_length=1, max_length=255)
    description: Optional[str] = Field(None, max_length=1000)
    priority: Optional[Priority] = None
    category: Optional[str] = Field(None, max_length=100)
    due_date: Optional[datetime] = None
    status: Optional[TaskStatus] = None

class Task(BaseModel):
    """Full task model with all fields"""
    id: int
    title: str
    description: Optional[str]
    priority: Priority
    category: Optional[str]
    status: TaskStatus
    created_at: datetime
    due_date: Optional[datetime]

    class Config:
        from_attributes = True
```

## Index Strategy

### Database Indexes for Performance

**Priority Sort Index**:
```sql
CREATE INDEX idx_tasks_priority ON tasks(priority);
```
*Rationale*: Speed up sorting by priority (common filter/sort combination)

**Due Date Sort Index**:
```sql
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
```
*Rationale*: Speed up sorting by due date; tasks without due dates (NULL) handled separately

**Composite Indexes** (optional, if query patterns emerge):
```sql
CREATE INDEX idx_tasks_status_priority ON tasks(status, priority);
CREATE INDEX idx_tasks_category_status ON tasks(category, status);
```

## Data Migration Strategy

### Schema Update Approach

**Current State**: Tasks table exists with basic fields (id, title, description, status)

**Desired State**: Tasks table enhanced with priority, category, created_at, due_date

**Migration Strategy**: ALTER TABLE with IF NOT EXISTS checks
```sql
-- Add priority column with default
ALTER TABLE tasks
ADD COLUMN IF NOT EXISTS priority TEXT DEFAULT 'medium'
CHECK (priority IN ('low', 'medium', 'high'));

-- Add category column (optional)
ALTER TABLE tasks
ADD COLUMN IF NOT EXISTS category TEXT;

-- Add created_at column
ALTER TABLE tasks
ADD COLUMN IF NOT EXISTS created_at TIMESTAMP DEFAULT NOW();

-- Add due_date column (optional)
ALTER TABLE tasks
ADD COLUMN IF NOT EXISTS due_date TIMESTAMP;

-- Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_tasks_priority ON tasks(priority);
CREATE INDEX IF NOT EXISTS idx_tasks_due_date ON tasks(due_date);
```

**Rollback Strategy**:
- `ALTER TABLE tasks DROP COLUMN IF EXISTS priority;`
- `ALTER TABLE tasks DROP COLUMN IF EXISTS category;`
- `ALTER TABLE tasks DROP COLUMN IF EXISTS created_at;`
- `ALTER TABLE tasks DROP COLUMN IF EXISTS due_date;`
- `DROP INDEX IF EXISTS idx_tasks_priority;`
- `DROP INDEX IF EXISTS idx_tasks_due_date;`

## Query Patterns

### Dynamic Filter Builder Pattern

**Purpose**: Build WHERE clause based on optional filter parameters

**Pattern**:
```python
def build_where_clause(filters: Dict[str, Any]) -> tuple[str, List[Any]]:
    """Build dynamic WHERE clause with parameterized values"""
    conditions = []
    params = []

    if filters.get('status'):
        conditions.append("status = %s")
        params.append(filters['status'])

    if filters.get('category'):
        conditions.append("category = %s")
        params.append(filters['category'])

    if filters.get('priority'):
        conditions.append("priority = %s")
        params.append(filters['priority'])

    if filters.get('search'):
        conditions.append("(title LIKE %s OR description LIKE %s)")
        term = f"%{filters['search']}%"
        params.extend([term, term])

    where_clause = " AND ".join(conditions) if conditions else "1=1"
    return where_clause, params
```

### Dynamic Order By Pattern

**Purpose**: Build ORDER BY clause based on sort preference

**Pattern**:
```python
def build_order_clause(sort_option: Optional[str]) -> str:
    """Build ORDER BY clause based on sort preference"""
    sort_map = {
        'priority': "CASE priority "
                   "WHEN 'high' THEN 1 "
                   "WHEN 'medium' THEN 2 "
                   "WHEN 'low' THEN 3 END",
        'date': "due_date ASC NULLS LAST",
        'alpha': "title COLLATE C",
    }
    return sort_map.get(sort_option, "created_at DESC")
```
