# Service Layer Contract: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature**: 002-recurring-tasks
**Date**: 2025-12-31
**Purpose**: Define service layer business logic contracts for recurrence, notifications, and enhanced task management

---

## Recurrence Engine Contract

### Module: `src/recurrence_engine.py`

**Responsibility**: Calculate next due dates, create recurring task instances, handle recurrence logic

**Constitutional Compliance**:
- ✅ Service Logic Agent ownership
- ✅ NO database connections (must use repository layer)
- ✅ NO user interaction (`input()`, `print()`)
- ✅ NO Rich UI (business logic only)
- ✅ All functions async

---

### Function: `calculate_next_due_date`

**Signature**:
```python
async def calculate_next_due_date(
    current_due_date: datetime,
    recurrence_interval: str  # "daily", "weekly", "monthly", "none"
) -> Optional[datetime]:
```

**Purpose**: Calculate next occurrence date based on recurrence interval

**Parameters**:
- `current_due_date`: Current task's due date (datetime)
- `recurrence_interval`: Interval string from enum ("daily", "weekly", "monthly", "none")

**Returns**:
- `datetime`: Next occurrence due date
- `None`: If recurrence_interval is "none" or invalid

**Raises**: None (gracefully handles invalid intervals)

**Edge Cases**:
- Leap year February 29th → February 28th in non-leap year (relativedelta handles)
- January 31st monthly → February 28th/29th (last day of month)
- Recurrence without due date → None (cannot calculate)

**Implementation**:
```python
from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta
from typing import Optional

async def calculate_next_due_date(
    current_due_date: datetime,
    recurrence_interval: str
) -> Optional[datetime]:
    """Calculate next occurrence based on recurrence interval."""
    if recurrence_interval == "daily":
        return current_due_date + timedelta(days=1)
    elif recurrence_interval == "weekly":
        return current_due_date + relativedelta(weeks=1)
    elif recurrence_interval == "monthly":
        return current_due_date + relativedelta(months=1)
    else:  # "none" or invalid
        return None
```

---

### Function: `create_recurring_task_instance`

**Signature**:
```python
async def create_recurring_task_instance(
    completed_task_id: int,
    current_due_date: Optional[datetime],
    recurrence_interval: str
) -> Optional[int]:
```

**Purpose**: Create new pending task instance when a recurring task is completed

**Parameters**:
- `completed_task_id`: ID of the just-completed task (becomes parent)
- `current_due_date`: Due date of completed task (None if task had no due date)
- `recurrence_interval`: Recurrence interval string

**Returns**:
- `int`: ID of newly created task
- `None`: If recurrence_interval is "none" or creation fails

**Raises**:
- `ValueError`: If completed_task_id is invalid (task not found)
- `RepositoryError`: If database operation fails

**Behavior**:
1. Fetch completed task details (title, description, priority, category)
2. Calculate next due date using `calculate_next_due_date()`
3. Create new task with:
   - Same title, description, priority, category, recurrence_interval
   - `parent_task_id` = completed_task_id
   - `due_date` = calculated next date (or None)
   - `is_completed` = FALSE
   - `reminder_sent` = FALSE
   - `created_at` = now()
   - `updated_at` = now()
4. Return new task ID

**Implementation**:
```python
from src.repository import get_task_by_id, create_task
from src.models import TaskCreate, RecurrenceInterval

async def create_recurring_task_instance(
    completed_task_id: int,
    current_due_date: Optional[datetime],
    recurrence_interval: str
) -> Optional[int]:
    """Create new pending instance when recurring task completed."""
    # Skip if no recurrence
    if recurrence_interval == "none" or recurrence_interval is None:
        return None

    # Fetch completed task details
    completed_task = await get_task_by_id(completed_task_id)
    if not completed_task:
        return None

    # Calculate next due date
    next_due_date = None
    if current_due_date:
        next_due_date = await calculate_next_due_date(
            current_due_date,
            recurrence_interval
        )

    # Create new instance
    new_task = TaskCreate(
        title=completed_task.title,
        description=completed_task.description,
        priority=completed_task.priority,
        category=completed_task.category,
        due_date=next_due_date,
        recurrence_interval=RecurrenceInterval(recurrence_interval)
    )

    created_task = await create_task(
        title=new_task.title,
        description=new_task.description,
        priority=new_task.priority.value,
        category=new_task.category,
        due_date=new_task.due_date
    )

    # Set parent_task_id (requires separate update after creation)
    if created_task:
        from src.repository import update_task
        await update_task(created_task.id, parent_task_id=completed_task_id)
        return created_task.id

    return None
```

---

## Notification Manager Contract

### Module: `src/notification_manager.py`

**Responsibility**: Handle cross-platform desktop notifications, boot checks for due tasks

**Constitutional Compliance**:
- ✅ Service Logic Agent ownership
- ✅ NO database connections (must use repository layer)
- ✅ NO user interaction (`input()`, `print()`)
- ✅ NO Rich UI (system notifications only)
- ✅ All functions async
- ✅ Graceful degradation if notifications unsupported

---

### Function: `show_notification`

**Signature**:
```python
async def show_notification(title: str, message: str) -> bool:
```

**Purpose**: Display cross-platform desktop notification

**Parameters**:
- `title`: Notification title
- `message`: Notification message body

**Returns**:
- `True`: Notification displayed successfully
- `False`: Notification failed or unsupported

**Raises**: None (gracefully handles all errors)

**Behavior**:
1. Try to send notification using plyer
2. If success, return True
3. If exception, log error and return False
4. Application continues normally regardless of notification outcome

**Implementation**:
```python
from plyer import notification
import logging

logger = logging.getLogger("notification_manager")

async def show_notification(title: str, message: str) -> bool:
    """Display cross-platform desktop notification."""
    try:
        notification.notify(
            title=title,
            message=message,
            app_name="Todo CLI",
            timeout=5  # seconds
        )
        return True
    except Exception as e:
        logger.error(f"Notification failed: {e}")
        return False  # Gracefully fail
```

---

### Function: `check_due_tasks_and_notify`

**Signature**:
```python
async def check_due_tasks_and_notify() -> int:
```

**Purpose**: Boot check: find tasks due within 30 minutes and send notifications

**Parameters**: None

**Returns**:
- `int`: Number of tasks notified

**Raises**: None (gracefully handles database errors)

**Behavior**:
1. Query tasks where:
   - `is_completed = FALSE`
   - `reminder_sent = FALSE`
   - `due_date` between NOW and NOW + 30 minutes
2. If tasks found:
   - Group into single notification (or separate if > 5)
   - Send notification with task titles and due times
3. Mark tasks as `reminder_sent = TRUE`
4. Return count of notified tasks

**Implementation**:
```python
from datetime import datetime, timedelta
from src.repository import get_all_tasks
from typing import List

async def check_due_tasks_and_notify() -> int:
    """Check for due tasks and send notifications."""
    now = datetime.now()
    time_threshold = now + timedelta(minutes=30)

    # Query tasks due within 30 minutes
    tasks = await get_all_tasks(
        status='pending'  # is_completed = FALSE
    )

    # Filter by due date and reminder_sent
    due_tasks = [
        t for t in tasks
        if t.due_date
        and t.due_date <= time_threshold
        and not t.reminder_sent
    ]

    if not due_tasks:
        return 0

    # Build notification message
    if len(due_tasks) == 1:
        title = "Task Due Soon"
        message = f"'{due_tasks[0].title}' is due at {due_tasks[0].due_date.strftime('%H:%M')}"
    elif len(due_tasks) <= 5:
        title = f"{len(due_tasks)} Tasks Due Soon"
        task_list = "\n".join([
            f"- {t.title} at {t.due_date.strftime('%H:%M')}"
            for t in due_tasks
        ])
        message = f"{task_list}"
    else:
        title = f"{len(due_tasks)} Tasks Due Soon"
        message = f"You have {len(due_tasks)} tasks due within 30 minutes."

    # Send notification
    await show_notification(title, message)

    # Mark tasks as notified
    from src.repository import update_task
    for task in due_tasks:
        await update_task(task.id, reminder_sent=True)

    return len(due_tasks)
```

---

## Enhanced Services Contract (Existing Layer Updates)

### Function: `complete_task` (Enhanced)

**Signature**:
```python
async def complete_task(task_id: int) -> Optional[dict]:
```

**Enhancement**: Add recurrence logic after marking task complete

**New Behavior**:
1. Mark task as completed (existing logic)
2. If task has `recurrence_interval != "none"`:
   - Call `create_recurring_task_instance()`
   - Log new instance creation

**Implementation Update**:
```python
from src.recurrence_engine import create_recurring_task_instance

async def complete_task(task_id: int) -> Optional[dict]:
    """Mark task as complete and create recurring instance if needed."""
    # Existing: mark task complete
    updated_task = await update_task(task_id, is_completed=True)

    if updated_task and updated_task.get('recurrence_interval') != "none":
        # Create recurring instance
        new_task_id = await create_recurring_task_instance(
            completed_task_id=task_id,
            current_due_date=updated_task.get('due_date'),
            recurrence_interval=updated_task.get('recurrence_interval')
        )
        if new_task_id:
            logger.info(f"Created recurring task instance: {new_task_id}")

    return updated_task
```

---

### Function: `get_task_history` (NEW)

**Signature**:
```python
async def get_task_history(root_task_id: int) -> List[dict]:
```

**Purpose**: Retrieve all instances of a recurring task series

**Parameters**:
- `root_task_id`: ID of root task (original recurring task)

**Returns**:
- `List[dict]`: All tasks in series (root + all descendants)
- Empty list if root_task_id invalid

**Implementation**:
```python
from src.repository import get_all_tasks, get_task_by_id
from typing import List

async def get_task_history(root_task_id: int) -> List[dict]:
    """Get all instances of a recurring task series."""
    # Verify root task exists
    root = await get_task_by_id(root_task_id)
    if not root:
        return []

    # Query: root task OR any task with parent_task_id = root
    # This requires new repository function with custom WHERE clause
    all_tasks = await get_all_tasks()  # Full list for now (optimize with repo filter)

    # Filter to get series
    series = [t for t in all_tasks if t.id == root_task_id or t.parent_task_id == root_task_id]

    # Sort by created_at
    series.sort(key=lambda t: t.created_at)

    return [t.to_dict() for t in series]
```

---

## Data Validation Contract

### Natural Language Date Parsing

**Function**: `parse_natural_date`

**Signature**:
```python
def parse_natural_date(input_str: str) -> Optional[datetime]:
```

**Purpose**: Parse natural language date string to datetime

**Parameters**:
- `input_str`: Natural language date expression ("tomorrow", "next Monday at 3pm")

**Returns**:
- `datetime`: Parsed date/time
- `None`: If cannot parse

**Implementation**:
```python
import dateparser
from datetime import datetime
from typing import Optional

def parse_natural_date(input_str: str) -> Optional[datetime]:
    """Parse natural language date string to datetime."""
    if not input_str or not input_str.strip():
        return None

    parsed = dateparser.parse(
        input_str,
        languages=['en'],
        settings={
            'RETURN_AS_TIMEZONE_AWARE': True,
            'PREFER_DATES_FROM': 'future'
        }
    )

    return parsed
```

**Usage in CLI Layer**:
```python
# User enters: "next Monday at 3pm"
due_date_str = answers['due_date']

# Validate with dateparser
if due_date_str:
    parsed_date = parse_natural_date(due_date_str)
    if not parsed_date:
        # Show error and offer interactive date picker
        ...
    else:
        task_data['due_date'] = parsed_date
```

---

## Error Handling Contract

All service functions must follow this error handling pattern:

```python
import logging

logger = logging.getLogger("services")

try:
    # Operation
    result = await repository_operation()
except RepositoryError as e:
    logger.error(f"Repository error: {e}")
    raise ValueError(f"Database error: {user_friendly_message}")
except ValueError as e:
    logger.warning(f"Validation error: {e}")
    raise  # Re-raise to CLI layer
except Exception as e:
    logger.error(f"Unexpected error: {e}", exc_info=True)
    raise RuntimeError(f"Unexpected error occurred")
```

---

## Summary of Service Contracts

| Function | Module | Purpose | Async |
|----------|--------|---------|-------|
| `calculate_next_due_date` | recurrence_engine.py | Calculate next occurrence date | ✅ |
| `create_recurring_task_instance` | recurrence_engine.py | Create recurring instance on completion | ✅ |
| `show_notification` | notification_manager.py | Display desktop notification | ✅ |
| `check_due_tasks_and_notify` | notification_manager.py | Boot check for due tasks | ✅ |
| `complete_task` (enhanced) | services.py | Mark complete + trigger recurrence | ✅ |
| `get_task_history` (new) | services.py | Get recurring task series | ✅ |
| `parse_natural_date` | services.py | Parse natural language dates | ❌ (sync) |

---

**Next Phase**: See [quickstart.md](./quickstart.md) for implementation guide.
