# Repository Layer Contract: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature**: 002-recurring-tasks
**Date**: 2025-12-31
**Purpose**: Define repository layer CRUD operations for enhanced task management

---

## Database Schema Enhancement

### New Columns (Migration)

```sql
-- Recurrence interval
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS recurrence_interval TEXT DEFAULT 'none';
ALTER TABLE tasks ADD CONSTRAINT recurrence_check
CHECK (recurrence_interval IN ('daily', 'weekly', 'monthly', 'none')) NOT VALID;
ALTER TABLE tasks VALIDATE CONSTRAINT recurrence_check;

-- Parent task relationship
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS parent_task_id INT;
ALTER TABLE tasks ADD CONSTRAINT fk_tasks_parent
FOREIGN KEY (parent_task_id) REFERENCES tasks(id) ON DELETE SET NULL;

-- Reminder tracking
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS reminder_sent BOOLEAN DEFAULT FALSE;

-- Indexes
CREATE INDEX IF NOT EXISTS idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX IF NOT EXISTS idx_tasks_reminder
ON tasks(due_date) WHERE is_completed = FALSE AND reminder_sent = FALSE;
CREATE INDEX IF NOT EXISTS idx_tasks_recurrence
ON tasks(recurrence_interval) WHERE recurrence_interval != 'none';
```

---

## CRUD Operations (Updates)

### Full Column Set

```python
FULL_COLUMN_SET = (
    "id, title, description, priority, category, due_date, "
    "is_completed, recurrence_interval, parent_task_id, "
    "reminder_sent, created_at, updated_at"
)
```

### Function: `create_task` (UPDATED)

**Signature**:
```python
async def create_task(
    title: str,
    description: Optional[str] = None,
    priority: str = 'medium',
    category: Optional[str] = None,
    due_date: Optional[datetime] = None,
    recurrence_interval: str = 'none',
    parent_task_id: Optional[int] = None
) -> dict:
```

**Changes from existing**:
- Added `recurrence_interval` parameter (default: 'none')
- Added `parent_task_id` parameter (default: None)
- INSERT statement includes new columns
- RETURNING clause uses FULL_COLUMN_SET

**Implementation**:
```python
async def create_task(
    title: str,
    description: Optional[str] = None,
    priority: str = 'medium',
    category: Optional[str] = None,
    due_date: Optional[datetime] = None,
    recurrence_interval: str = 'none',
    parent_task_id: Optional[int] = None
) -> dict:
    async with await db_manager.get_connection() as conn:
        async with conn.cursor() as cur:
            await cur.execute(f"""
                INSERT INTO tasks (title, description, priority, category, due_date,
                                recurrence_interval, parent_task_id)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
                RETURNING {FULL_COLUMN_SET}
                """,
                (title, description, priority, category, due_date,
                 recurrence_interval, parent_task_id)
            )
            row = await cur.fetchone()
            await conn.commit()
            return _row_to_dict(row)
```

---

### Function: `update_task` (UPDATED)

**Signature**:
```python
async def update_task(task_id: int, **kwargs) -> Optional[dict]:
```

**Changes from existing**:
- Supports updating `recurrence_interval`
- Supports updating `parent_task_id`
- Supports updating `reminder_sent`
- Supports resetting `reminder_sent` when `due_date` changes

**Behavior**:
- If `due_date` is updated → Automatically set `reminder_sent = FALSE`
- If `recurrence_interval` updated → No constraint validation (handled in service layer)
- If `parent_task_id` updated → Validated by FK constraint in database

**Implementation**:
```python
async def update_task(task_id: int, **kwargs) -> Optional[dict]:
    # Handle reminder_sent reset on due_date change
    if 'due_date' in kwargs:
        kwargs['reminder_sent'] = False

    # Filter None values
    kwargs = {k: v for k, v in kwargs.items() if v is not None}

    if not kwargs:
        return await get_task_by_id(task_id)

    kwargs['updated_at'] = datetime.now()

    # Build SET clause
    columns = []
    params = []
    for k, v in kwargs.items():
        columns.append(f"{k} = %s")
        params.append(v)
    params.append(task_id)

    query = f"UPDATE tasks SET {', '.join(columns)} WHERE id = %s RETURNING {FULL_COLUMN_SET}"

    async with await db_manager.get_connection() as conn:
        async with conn.cursor() as cur:
            await cur.execute(query, params)
            row = await cur.fetchone()
            await conn.commit()
            if row:
                return _row_to_dict(row)
            return None
```

---

### Function: `get_task_by_id` (UPDATED)

**Signature**:
```python
async def get_task_by_id(task_id: int) -> Optional[dict]:
```

**Changes from existing**:
- Uses FULL_COLUMN_SET (includes new columns)

**Implementation**:
```python
async def get_task_by_id(task_id: int) -> Optional[dict]:
    async with await db_manager.get_connection() as conn:
        async with conn.cursor() as cur:
            await cur.execute(
                f"SELECT {FULL_COLUMN_SET} FROM tasks WHERE id = %s",
                (task_id,)
            )
            row = await cur.fetchone()
            if row:
                return _row_to_dict(row)
            return None
```

---

### Function: `get_all_tasks` (UPDATED)

**Signature**:
```python
async def get_all_tasks(
    status: Optional[str] = None,
    category: Optional[str] = None,
    priority: Optional[str] = None,
    parent_task_id: Optional[int] = None,
    recurrence_interval: Optional[str] = None,
    due_before: Optional[datetime] = None,
    due_after: Optional[datetime] = None,
    sort_by: str = 'date'
) -> List[dict]:
```

**Changes from existing**:
- Added `parent_task_id` filter (for task history queries)
- Added `recurrence_interval` filter
- Added `due_before` and `due_after` filters (for notification checks)
- Uses FULL_COLUMN_SET

**Implementation**:
```python
async def get_all_tasks(
    status: Optional[str] = None,
    category: Optional[str] = None,
    priority: Optional[str] = None,
    parent_task_id: Optional[int] = None,
    recurrence_interval: Optional[str] = None,
    due_before: Optional[datetime] = None,
    due_after: Optional[datetime] = None,
    sort_by: str = 'date'
) -> List[dict]:
    query = f"SELECT {FULL_COLUMN_SET} FROM tasks"
    conditions = []
    params = []

    if status == 'completed':
        conditions.append("is_completed = TRUE")
    elif status == 'pending':
        conditions.append("is_completed = FALSE")

    if category:
        conditions.append("category = %s")
        params.append(category)

    if priority:
        conditions.append("priority = %s")
        params.append(priority)

    if parent_task_id:
        conditions.append("parent_task_id = %s")
        params.append(parent_task_id)

    if recurrence_interval:
        conditions.append("recurrence_interval = %s")
        params.append(recurrence_interval)

    if due_before:
        conditions.append("due_date <= %s")
        params.append(due_before)

    if due_after:
        conditions.append("due_date >= %s")
        params.append(due_after)

    if conditions:
        query += " WHERE " + " AND ".join(conditions)

    # Sorting (same as existing)
    if sort_by == 'priority':
        query += """ ORDER BY CASE
            WHEN priority = 'high' THEN 1
            WHEN priority = 'medium' THEN 2
            WHEN priority = 'low' THEN 3
            ELSE 4 END, created_at DESC"""
    elif sort_by == 'alpha':
        query += ' ORDER BY title COLLATE "C" ASC'
    else:  # default 'date'
        query += " ORDER BY due_date ASC NULLS LAST, created_at DESC"

    async with await db_manager.get_connection() as conn:
        async with conn.cursor() as cur:
            await cur.execute(query, params)
            rows = await cur.fetchall()
            return [_row_to_dict(row) for row in rows]
```

---

### Function: `get_task_history` (NEW)

**Signature**:
```python
async def get_task_history(root_task_id: int) -> List[dict]:
```

**Purpose**: Retrieve all tasks in a recurring series (root + descendants)

**Parameters**:
- `root_task_id`: ID of root task (original recurring task)

**Returns**:
- `List[dict]`: All tasks in series (root + all descendants)
- Empty list if root_task_id invalid

**Implementation**:
```python
async def get_task_history(root_task_id: int) -> List[dict]:
    """Get all instances of a recurring task series."""
    async with await db_manager.get_connection() as conn:
        async with conn.cursor() as cur:
            # Get root task + all descendants
            query = f"""
                SELECT {FULL_COLUMN_SET}
                FROM tasks
                WHERE id = %s OR parent_task_id = %s
                ORDER BY created_at ASC
            """
            await cur.execute(query, (root_task_id, root_task_id))
            rows = await cur.fetchall()
            return [_row_to_dict(row) for row in rows]
```

---

## Helper Function

### Function: `_row_to_dict` (UPDATED)

**Signature**:
```python
def _row_to_dict(row: tuple) -> dict:
```

**Purpose**: Convert database row tuple to dictionary

**Implementation**:
```python
def _row_to_dict(row: tuple) -> dict:
    """Convert database row tuple to dictionary."""
    return {
        'id': row[0],
        'title': row[1],
        'description': row[2],
        'priority': row[3],
        'category': row[4],
        'due_date': row[5],
        'is_completed': row[6],
        'recurrence_interval': row[7],
        'parent_task_id': row[8],
        'reminder_sent': row[9],
        'created_at': row[10],
        'updated_at': row[11]
    }
```

---

## SQL Query Patterns

### Check Constraint Addition (Idempotent)

```sql
-- Step 1: Add column
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS recurrence_interval TEXT DEFAULT 'none';

-- Step 2: Add constraint in non-validating mode
DO $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM pg_constraint
        WHERE conname = 'recurrence_check'
    ) THEN
        ALTER TABLE tasks
        ADD CONSTRAINT recurrence_check
        CHECK (recurrence_interval IN ('daily', 'weekly', 'monthly', 'none'))
        NOT VALID;
    END IF;
END $$;

-- Step 3: Validate constraint
ALTER TABLE tasks VALIDATE CONSTRAINT recurrence_check;
```

### Foreign Key Addition

```sql
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS parent_task_id INT;

DO $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM pg_constraint
        WHERE conname = 'fk_tasks_parent'
    ) THEN
        ALTER TABLE tasks
        ADD CONSTRAINT fk_tasks_parent
        FOREIGN KEY (parent_task_id) REFERENCES tasks(id)
        ON DELETE SET NULL;
    END IF;
END $$;
```

---

## Query Performance

### Indexes for Notification Check

```sql
-- Composite index for fast pending, unnotified tasks query
CREATE INDEX IF NOT EXISTS idx_tasks_reminder
ON tasks(due_date)
WHERE is_completed = FALSE AND reminder_sent = FALSE;
```

**Query**:
```sql
SELECT * FROM tasks
WHERE is_completed = FALSE
  AND reminder_sent = FALSE
  AND due_date <= NOW() + INTERVAL '30 minutes'
ORDER BY due_date ASC;
```

**Performance**: < 200ms for 1000+ tasks (with index)

---

## Summary of Repository Contracts

| Function | Purpose | Changes | Async |
|----------|---------|---------|-------|
| `create_task` | Create new task | Added `recurrence_interval`, `parent_task_id` | ✅ |
| `update_task` | Update existing task | Supports new fields, auto-resets `reminder_sent` | ✅ |
| `get_task_by_id` | Get single task | Uses FULL_COLUMN_SET | ✅ |
| `get_all_tasks` | List tasks with filters | Added `parent_task_id`, `recurrence_interval`, date range filters | ✅ |
| `get_task_history` (NEW) | Get recurring task series | Queries root + descendants | ✅ |
| `delete_task` | Delete task | Uses CASCADE delete via FK | ✅ |
| `search_tasks` | Search by keyword | No changes | ✅ |

---

**Next Phase**: See [quickstart.md](../quickstart.md) for implementation guide.
