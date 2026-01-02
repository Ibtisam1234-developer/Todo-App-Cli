# Service Layer Contract

**Files**: `src/services.py`, `src/models.py`
**Owner**: Service Logic Agent
**Purpose**: Business logic orchestration and data model definitions

## Constitutional Constraints

**ALLOWED**:
- Business rule validation
- CRUD operation orchestration
- Data model definitions
- Calls to database layer functions
- Error context enrichment

**FORBIDDEN**:
- Direct database connections (must call `db.py` functions)
- User input collection (`input()` calls)
- User output rendering (`print()` calls)
- SQL queries or schema modifications

## Function Signatures (src/services.py)

### Create Task

```python
def create_task(title: str, description: str | None = None) -> dict:
    """
    Create a new task with validation.

    Args:
        title: Task title (required)
        description: Task description (optional)

    Returns:
        dict: {'success': bool, 'task_id': int | None, 'message': str}

    Validation:
        - Title required, non-empty after strip, 1-200 chars
        - Description optional, max 1000 chars if provided
    """
```

### List Tasks

```python
def list_tasks() -> dict:
    """
    Retrieve all tasks.

    Returns:
        dict: {'success': bool, 'tasks': list[Task], 'message': str}
    """
```

### Update Task

```python
def update_task(task_id: int, title: str | None = None,
                description: str | None = None) -> dict:
    """
    Update task with validation.

    Args:
        task_id: ID of task to update
        title: New title (optional)
        description: New description (optional)

    Returns:
        dict: {'success': bool, 'message': str}

    Validation:
        - At least one field must be provided
        - Title if provided: non-empty, 1-200 chars
        - Description if provided: max 1000 chars
    """
```

### Toggle Task Completion

```python
def toggle_task_completion(task_id: int) -> dict:
    """
    Toggle task completion status.

    Args:
        task_id: ID of task to toggle

    Returns:
        dict: {'success': bool, 'message': str}
    """
```

### Delete Task

```python
def delete_task(task_id: int) -> dict:
    """
    Permanently delete a task.

    Args:
        task_id: ID of task to delete

    Returns:
        dict: {'success': bool, 'message': str}
    """
```

## Data Model (src/models.py)

```python
from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class Task:
    """Task entity."""
    id: int
    title: str
    description: Optional[str]
    is_completed: bool
    created_at: datetime
    updated_at: datetime

    def to_dict(self) -> dict:
        """Convert to dictionary."""

    @classmethod
    def from_db_row(cls, row: tuple) -> 'Task':
        """Create from database row."""
```

## Constitutional Compliance Checklist

- [ ] NO direct database connections (only calls to `db.py`)
- [ ] NO `input()` calls
- [ ] NO `print()` calls
- [ ] ALL validation in service layer (not database)
- [ ] ALL business rules in service layer
- [ ] Returns structured dicts (not raw tuples)

---

**Contract Status**: ✅ DEFINED
**Owner**: Service Logic Agent
**Reviewer**: Spec Guardian Agent
