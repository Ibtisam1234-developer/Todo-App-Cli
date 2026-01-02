# Research Report: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature**: 002-recurring-tasks
**Date**: 2025-12-31
**Purpose**: Resolve technical decisions for implementation

---

## Decision 1: Date Parsing Library

**Decision**: Use `dateparser` for natural language date parsing

**Rationale**:
- Widely adopted, mature library with 2.5M+ monthly downloads
- Supports extensive natural language expressions ("tomorrow", "next Monday at 3pm", "in 2 weeks")
- Handles timezone-aware parsing automatically
- Active maintenance and comprehensive documentation
- Python 3.12+ compatible

**Alternatives Considered**:
1. **python-dateutil** (Rejected)
   - Less intuitive natural language support
   - Requires more manual configuration for common expressions
   - Not as feature-rich for relative dates

2. **mayavi** (Rejected)
   - Primarily for visualization, not date parsing
   - Inappropriate for this use case

**Integration Pattern**:
```python
import dateparser

# Synchronous operation - isolated from async path
def parse_date(input_str: str) -> Optional[datetime]:
    """Parse natural language date string to datetime."""
    return dateparser.parse(input_str)
```

**Edge Case Handling**:
- Ambiguous dates ("Friday") → Interprets as upcoming Friday
- Invalid expressions → Returns None, triggers interactive picker
- Timezone awareness → Uses system local timezone by default

---

## Decision 2: Interactive Forms Library

**Decision**: Use `questionary` for CLI wizard and date pickers

**Rationale**:
- Native support for rich, interactive forms with autocomplete
- Built-in date picker widgets (`questionary.form` with calendar integration)
- Compatible with Rich library (already in use)
- Async-aware, can be integrated with existing async workflow
- Provides professional terminal experience (similar to `inquirer.js` for Node.js)

**Alternatives Considered**:
1. **inquirer** (Rejected)
   - Less actively maintained
   - Fewer widget options
   - No built-in date picker

2. **promp-toolkit** (Rejected)
   - Lower-level, requires more boilerplate
   - Less polished UI
   - Not as feature-rich for forms

**Integration Pattern**:
```python
from questionary import form, form_question, select, confirm

# Synchronous input collection in CLI layer
async def interactive_task_form() -> dict:
    """Guided task creation wizard."""
    answers = await to_threadsafe(form(
        form_question("title", "Task title:", validate=...),
        form_question("due_date", "Due date (or leave blank):"),
        # ... more questions
    ))
    return answers
```

**Boot Check Integration**:
- Questionary forms replace basic `typer.Option` prompts
- Date picker fallback when natural language parsing fails
- Maintains consistency with existing Rich UI patterns

---

## Decision 3: Desktop Notifications Library

**Decision**: Use `plyer` for cross-platform desktop notifications

**Rationale**:
- Single API for Windows, macOS, Linux
- Mature, battle-tested library
- Simple notification API with title/message/app_name fields
- Handles platform-specific notification quirks internally
- No platform-specific code required

**Alternatives Considered**:
1. **py-notifier** (Rejected)
   - Less mature, smaller community
   - Fewer platform backends
   - Less reliable on Windows

2. **Platform-specific libraries** (Rejected)
   - Would require conditional imports (win10toast, notify2, etc.)
   - Violates single-responsibility principle
   - Increases maintenance burden

**Integration Pattern**:
```python
from plyer import notification
from plyer.platforms.win10toast import toast as win_toast

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

**Graceful Degradation**:
- Try/except around all notification calls
- No system notification capability → Application continues normally
- Logged to `.app.log` for debugging

**Boot Check Implementation**:
```python
async def check_due_tasks_and_notify():
    """Run at startup to check for tasks due within 30 minutes."""
    tasks = await get_tasks_due_within(minutes=30)
    if tasks:
        await send_due_tasks_notification(tasks)
```

---

## Decision 4: Recurrence Calculation Strategy

**Decision**: Implement `recurrence_engine.py` with interval-based arithmetic using `dateutil.relativedelta`

**Rationale**:
- `dateutil.relativedelta` handles month/year boundaries correctly (leap years, end-of-month)
- Reliable arithmetic for "next month", "next week", etc.
- Already a dependency (via `dateparser`)
- Handles edge cases like February 29th → February 28th (non-leap year)

**Alternatives Considered**:
1. **Custom datetime arithmetic** (Rejected)
   - Vulnerable to edge cases (leap years, month boundaries)
   - More code to maintain
   - Reinventing wheel

2. **cron-like scheduling** (Rejected)
   - Over-engineered for this use case
   - Requires persistent scheduler or daemon
   - CLI application runs on-demand, not continuously

**Integration Pattern**:
```python
from dateutil.relativedelta import relativedelta
from datetime import datetime, timedelta

async def calculate_next_due_date(
    current_due: datetime,
    interval: str  # "daily", "weekly", "monthly"
) -> Optional[datetime]:
    """Calculate next occurrence based on recurrence interval."""
    if interval == "daily":
        return current_due + timedelta(days=1)
    elif interval == "weekly":
        return current_due + relativedelta(weeks=1)
    elif interval == "monthly":
        return current_due + relativedelta(months=1)
    else:  # "none" or invalid
        return None
```

**Edge Case Handling**:
- Leap year February 29th → Next year February 28th (relativedelta handles this)
- January 31st monthly → February 28th/29th (last day of month)
- Recurrence without due date → No calculation, copy attributes only

---

## Decision 5: Parent-Child Task Relationship Storage

**Decision**: Add `parent_task_id` and `recurrence_interval` columns to existing `tasks` table

**Rationale**:
- Simple foreign key relationship (self-referential)
- Enables querying entire task series: `WHERE id = X OR parent_task_id = X`
- Preserves historical context without separate table
- No schema explosion (avoids `task_instances` table)

**Alternatives Considered**:
1. **Separate `task_instances` table** (Rejected)
   - Over-engineered for current requirements
   - Adds join complexity for simple queries
   - No additional metadata required beyond what task already has

2. **JSON array for instances** (Rejected)
   - Not queryable
   - Violates relational principles
   - Harder to maintain integrity

**Schema Changes**:
```sql
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS recurrence_interval TEXT DEFAULT 'none';
ALTER TABLE tasks ADD CONSTRAINT recurrence_check CHECK (recurrence_interval IN ('daily', 'weekly', 'monthly', 'none'));

ALTER TABLE tasks ADD COLUMN IF NOT EXISTS parent_task_id INT REFERENCES tasks(id) ON DELETE SET NULL;
CREATE INDEX IF NOT EXISTS idx_tasks_parent ON tasks(parent_task_id);

ALTER TABLE tasks ADD COLUMN IF NOT EXISTS reminder_sent BOOLEAN DEFAULT FALSE;
CREATE INDEX IF NOT EXISTS idx_tasks_reminder ON tasks(due_date) WHERE is_completed = FALSE AND reminder_sent = FALSE;
```

---

## Decision 6: Boot Check Execution Strategy

**Decision**: Add startup hook in `main.py` that runs before Typer command dispatch

**Rationale**:
- Executes for every CLI command (add, list, update, etc.)
- Non-blocking async check (< 1 second per SC-008)
- Logged to `.app.log` for observability
- Gracefully fails if database unavailable (existing retry logic)

**Alternatives Considered**:
1. **Daemon process** (Rejected)
   - CLI application runs on-demand, not continuously
   - Daemon architecture is over-engineered
   - Cross-platform complexity (systemd, launchd, Windows services)

2. **Scheduled job library** (Rejected)
   - Requires persistent background process
   - Not compatible with CLI-on-demand model

**Integration Pattern**:
```python
# src/main.py
async def main():
    """Main application entry point with boot check."""
    # Boot check: Notifications for due tasks
    await notification_manager.check_due_tasks_and_notify()

    # Dispatch to Typer command
    await app()

if __name__ == "__main__":
    asyncio.run(main())
```

**Performance Considerations**:
- Index on `due_date` ensures fast query (< 200ms)
- Only queries pending, unnotified tasks (`is_completed = FALSE AND reminder_sent = FALSE`)
- Limits to tasks within 30-minute window in WHERE clause

---

## Decision 7: Database Migration Strategy

**Decision**: Use `CREATE TABLE IF NOT EXISTS` + `ALTER TABLE ADD COLUMN IF NOT EXISTS` with `NOT VALID` for constraints

**Rationale**:
- Idempotent initialization (safe to run multiple times)
- Backward compatible with existing data
- Handles migration from beginner version (001-todo-app) to advanced (002-recurring-tasks)
- NOT VALID allows constraint addition without validating existing rows

**Alternatives Considered**:
1. **Schema versioning/migration table** (Rejected)
   - Over-engineered for single-developer project
   - Adds complexity without significant benefit
   - Spec-driven development suffices

2. **Drop and recreate table** (Rejected)
   - Loses existing user data
   - Violates data retention requirement (SC-002)

**Migration Steps**:
```sql
-- Add recurrence column with default
ALTER TABLE tasks ADD COLUMN IF NOT EXISTS recurrence_interval TEXT DEFAULT 'none';

-- Add constraint in non-validating mode (existing rows OK)
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

-- Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX IF NOT EXISTS idx_tasks_reminder
ON tasks(due_date) WHERE is_completed = FALSE AND reminder_sent = FALSE;
```

---

## Summary of Technical Decisions

| Component | Library/Approach | Rationale |
|-----------|-----------------|-----------|
| Date parsing | `dateparser` | Natural language support, mature, Python 3.12+ compatible |
| Interactive forms | `questionary` | Rich UI, async-aware, built-in date picker |
| Notifications | `plyer` | Cross-platform, simple API, graceful degradation |
| Recurrence logic | `dateutil.relativedelta` | Handles edge cases (leap years, month boundaries) |
| Task relationships | Self-referential FK on `tasks` table | Simple, queryable, preserves history |
| Boot check | Startup hook in `main.py` | On-demand execution, non-blocking |
| Migration strategy | `ALTER TABLE IF NOT EXISTS` + `NOT VALID` | Idempotent, backward compatible |

**Next Phase**: See [data-model.md](./data-model.md) for entity definitions and relationships.
