# Quickstart Guide: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature**: 002-recurring-tasks
**Date**: 2025-12-31
**Purpose**: Implementation guide for recurring tasks, natural language dates, and notifications

---

## Prerequisites

- Python 3.12+ installed
- Neon Serverless PostgreSQL account
- Existing todo-cli application (feature 001-task-organization implemented)
- All existing dependencies installed (Typer, Rich, Pydantic, psycopg[pool])

---

## New Dependencies

### Install Required Packages

```bash
# Natural language date parsing
pip install dateparser

# Interactive CLI forms and date pickers
pip install questionary

# Cross-platform desktop notifications
pip install plyer

# Date arithmetic (for recurrence)
pip install python-dateutil
```

### Update requirements.txt

```text
# Existing dependencies
typer>=0.9.0
rich>=13.7.0
pydantic>=2.0.0
psycopg[pool]>=3.1.0
python-dotenv>=1.0.0

# New dependencies for 002-recurring-tasks
dateparser>=1.2.0
questionary>=2.0.0
plyer>=2.1.0
python-dateutil>=2.8.2
```

---

## Implementation Order

### Phase 1: Database Schema Enhancement

1. **Update `src/database.py`**
   - Add migration to init_db() function
   - Add new columns: `recurrence_interval`, `parent_task_id`, `reminder_sent`
   - Add constraints and indexes

2. **Test migration**
   ```bash
   python -m src.main  # Should run existing init_db()
   # Verify new columns exist in Neon database
   ```

### Phase 2: Repository Layer Updates

1. **Update `src/repository.py`**
   - Update FULL_COLUMN_SET constant
   - Modify create_task() to accept new parameters
   - Modify update_task() to handle new fields
   - Add get_task_history() function
   - Update get_all_tasks() with new filters

2. **Test repository functions**
   ```python
   # Test creating task with recurrence
   task = await create_task(
       title="Daily standup",
       recurrence_interval="daily",
       due_date=datetime(2025, 1, 1)
   )
   ```

### Phase 3: Service Logic Layer

1. **Create `src/recurrence_engine.py`**
   - Implement calculate_next_due_date()
   - Implement create_recurring_task_instance()
   - All functions async
   - Use repository layer for data access

2. **Create `src/notification_manager.py`**
   - Implement show_notification()
   - Implement check_due_tasks_and_notify()
   - All functions async
   - Graceful degradation on unsupported platforms

3. **Update `src/services.py`**
   - Add natural language date parsing function
   - Enhance complete_task() to trigger recurrence
   - Add get_task_history() orchestration

4. **Update `src/models.py`**
   - Add RecurrenceInterval enum
   - Add reminder_sent field to Task model
   - Add parent_task_id field to Task model
   - Update TaskCreate, TaskUpdate schemas

### Phase 4: CLI/UX Layer Integration

1. **Update `src/cli.py`**
   - Replace basic prompts with questionary forms
   - Add recurrence_interval option to add command
   - Add parent_task_id display to list command
   - Add history command to view task series
   - Integrate natural language date parsing
   - Show Rich error messages for date parsing failures

2. **Update `src/main.py`**
   - Add boot check for notifications
   - Call check_due_tasks_and_notify() before Typer command dispatch
   - Handle async properly

### Phase 5: Testing

#### Test 1: Recurring Task Creation

```bash
# Add a daily recurring task
python -m src.main add --title "Daily standup" \
    --recurrence daily \
    --due-date "2025-01-01 09:00"
```

#### Test 2: Task Completion and Regeneration

```bash
# Mark task complete
python -m src.main complete --task-id 1

# Verify new task created with due date "2025-01-02 09:00"
python -m src.main list
```

#### Test 3: Natural Language Dates

```bash
# Test various date expressions
python -m src.main add --title "Meeting" \
    --due-date "next Monday at 3pm"

python -m src.main add --title "Review" \
    --due-date "tomorrow morning"

python -m src.main add --title "Deadline" \
    --due-date "in 2 weeks"
```

#### Test 4: Desktop Notifications

```bash
# Create task due in 15 minutes
python -m src.main add --title "Urgent task" \
    --due-date "$(date -u -d '+15 minutes' +%Y-%m-%dT%H:%M:%S)"

# Launch app - should show notification
python -m src.main list
```

#### Test 5: Task History

```bash
# View entire recurring task series
python -m src.main history --root-task-id 1
```

---

## Common Workflows

### Creating a Recurring Task

**CLI Command**:
```bash
python -m src.main add --title "Weekly report" \
    --description "Prepare weekly status report" \
    --priority high \
    --category work \
    --due-date "next Friday at 5pm" \
    --recurrence weekly
```

**Questionary Form** (if using interactive mode):
```
Task title: Weekly report
Description: Prepare weekly status report
Priority [low/medium/high] (medium): high
Category: work
Due Date (natural language): next Friday at 5pm
Recurrence [none/daily/weekly/monthly] (none): weekly
```

### Completing a Recurring Task

```bash
# Mark complete - automatically creates next instance
python -m src.main complete --task-id 5

# List tasks to see new instance
python -m src.main list
```

### Viewing Task History

```bash
# View entire recurring task series
python -m src.main history --root-task-id 5

# Output:
# #5: Weekly report (completed) - parent: null
# #12: Weekly report (pending) - parent: 5
# #19: Weekly report (pending) - parent: 12
```

---

## Troubleshooting

### Issue: Date parsing fails

**Error**: "Could not parse date 'next monday'"

**Solution**:
1. Check for typos in date expression
2. Try alternative expressions ("next Monday", "this coming Monday")
3. If still fails, interactive date picker will be offered
4. Verify dateparser is installed: `pip list | grep dateparser`

### Issue: No desktop notifications appear

**Symptom**: Tasks due within 30 minutes but no notification

**Solution**:
1. Check `.app.log` for notification errors
2. Verify plyer is installed: `pip list | grep plyer`
3. On Windows: Check notification permissions in Settings
4. On macOS: Check notification permissions in System Preferences
5. On Linux: Ensure notification daemon is running (`notify-send --version`)

**Graceful Behavior**: Application continues normally regardless of notification success

### Issue: Recurring task not regenerating

**Symptom**: Complete recurring task but no new instance created

**Solution**:
1. Check task has `recurrence_interval` != "none"
2. Check task has a `due_date` (recurrence requires due date)
3. Verify recurrence_engine.py is creating instance (check logs)
4. Check database for new task with correct `parent_task_id`

### Issue: Migration fails on existing database

**Error**: "Column already exists" or "Constraint already exists"

**Solution**:
- Migration uses `ADD COLUMN IF NOT EXISTS` and `NOT VALID` for constraints
- Should be idempotent - safe to run multiple times
- If still failing, check constraint name in Neon console and drop manually

---

## Performance Considerations

### Recurring Task Regeneration

- **Target**: < 2 seconds after task completion
- **Optimization**: Calculate next due date in-memory, single INSERT query
- **Bottleneck**: Database connection (handled by existing connection pool)

### Notification Boot Check

- **Target**: < 1 second from app launch
- **Optimization**: Indexed query on `due_date` with WHERE conditions
- **Bottleneck**: Query for tasks + plyer notification call
- **Mitigation**: Limited to 30-minute window, only queries pending tasks

### Date Parsing

- **Target**: < 100ms per input
- **Optimization**: Single dateparser.parse() call
- **Bottleneck**: None (synchronous but isolated from async path)

---

## Platform-Specific Notes

### Windows

- Desktop notifications via plyer (win10toast)
- Requires Python 3.8+
- Notification permissions: Settings > System > Notifications & actions

### macOS

- Desktop notifications via plyer (apnotifier)
- Requires Python 3.8+
- Notification permissions: System Preferences > Notifications

### Linux

- Desktop notifications via plyer (libnotify)
- Requires notification daemon (`libnotify-bin` or `notify-osd`)
- Test with: `notify-send "Test" "Notification works"`

---

## Next Steps

1. Implement following phases in order (1-5)
2. Test each phase before proceeding
3. Refer to constitution for layer boundaries
4. Run Guardian review after each layer
5. Update CLAUDE.md and README.md after completion

---

## References

- [Constitution](../../.specify/memory/constitution.md)
- [Specification](./spec.md)
- [Implementation Plan](./plan.md)
- [Data Model](./data-model.md)
- [Service Contract](./contracts/service-contract.md)
- [Repository Contract](./contracts/repository-contract.md)
