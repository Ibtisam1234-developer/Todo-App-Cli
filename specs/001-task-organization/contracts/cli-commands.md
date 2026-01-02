# CLI Commands: Task Organization & Usability

**Feature**: 001-task-organization
**Purpose**: Define command interface for Typer CLI with new flags and parameters
**Date**: 2025-12-30

## Command Contracts

### add - Create New Task

**Command**: `add <title> [options]`

**Arguments**:
- `title` (required): Task title (string)

**Options**:
- `--description, -d` (optional): Task description (string)
- `--priority, -p` (optional): Task priority (enum: low, medium, high; default: medium)
- `--category, -c` (optional): Task category (string)
- `--due, -t` (optional): Due date (datetime format)

**Behavior**:
- Creates task in database with specified attributes
- Shows Rich-formatted success message with task ID
- Shows Rich progress spinner during database operation
- Validates inputs via Pydantic before calling service layer

**Errors**:
- Invalid priority: Display valid options (low, medium, high) with red text
- Invalid date format: Display expected format example
- Empty title: Display error requiring non-empty title

**Example**:
```bash
add "Complete project report" --priority high --category work --due "2025-01-15"
```

### list - List Tasks

**Command**: `list [options]`

**Arguments**: None

**Options**:
- `--status, -s` (optional): Filter by status (enum: pending, completed, all)
- `--priority, -p` (optional): Filter by priority (enum: low, medium, high)
- `--category, -c` (optional): Filter by category (string)
- `--sort` (optional): Sort results (enum: date, priority, alpha; default: date)

**Behavior**:
- Displays tasks in Rich Table with columns: ID, Title, Category, Priority, Status, Due Date
- Applies all specified filters (composable)
- Sorts results based on specified option
- Shows color-coded priority (red=high, yellow=medium, green=low)
- Shows Rich progress spinner during database operation
- Displays "No tasks found matching criteria" if no results

**Example**:
```bash
# List all pending work tasks sorted by priority
list --status pending --category work --sort priority

# List all tasks with high priority sorted by due date
list --priority high --sort date

# List all tasks alphabetically
list --sort alpha
```

### search - Search Tasks

**Command**: `search <keyword> [options]`

**Arguments**:
- `keyword` (required): Search term (string)

**Options**:
- `--status, -s` (optional): Filter by status (enum: pending, completed, all)
- `--priority, -p` (optional): Filter by priority (enum: low, medium, high)
- `--category, -c` (optional): Filter by category (string)

**Behavior**:
- Searches tasks where keyword appears in title OR description
- Applies specified filters in addition to keyword search
- Displays matching tasks in Rich Table (same format as list command)
- Shows Rich progress spinner during database operation
- Displays "No tasks found matching criteria" if no results

**Example**:
```bash
# Search for "meeting" in pending work tasks
search "meeting" --status pending --category work
```

### update - Update Task

**Command**: `update <id> [options]`

**Arguments**:
- `id` (required): Task ID (integer)

**Options**:
- `--title, -t` (optional): New task title (string)
- `--description, -d` (optional): New task description (string)
- `--priority, -p` (optional): New task priority (enum: low, medium, high)
- `--category, -c` (optional): New task category (string)
- `--due` (optional): New due date (datetime format)
- `--status, -s` (optional): Toggle task status (enum: pending, completed)

**Behavior**:
- Updates specified task with provided options
- Shows Rich-formatted success message
- Shows Rich progress spinner during database operation
- Validates inputs via Pydantic before calling service layer
- Shows error if task ID does not exist

**Errors**:
- Task not found: Display error with task ID
- Invalid priority: Display valid options with red text
- Invalid date format: Display expected format example

**Example**:
```bash
# Update task priority and category
update 42 --priority high --category work

# Mark task as completed
update 42 --status completed

# Update task due date
update 42 --due "2025-01-20"
```

### delete - Delete Task

**Command**: `delete <id>`

**Arguments**:
- `id` (required): Task ID (integer)

**Options**: None

**Behavior**:
- Deletes specified task from database
- Shows Rich-formatted confirmation prompt before deletion
- Shows Rich progress spinner during database operation
- Shows success message after deletion
- Shows error if task ID does not exist

**Example**:
```bash
delete 42
```

## Rich UI Components

### Table Columns

| Column | Width | Alignment | Color Logic |
|-------|---------|-----------|------------|
| ID | 6 | right | none |
| Title | 30 | left | none |
| Category | 15 | left | none |
| Priority | 10 | center | red (high), yellow (medium), green (low) |
| Status | 10 | center | yellow (pending), green (completed) |
| Due Date | 20 | left | red if overdue (due_date < NOW) |

### Progress Spinner

**Usage**:
```python
from rich.progress import Progress, SpinnerColumn

with Progress(
    SpinnerColumn(),
    TextColumn("[progress.description]"),
    transient=True,
) as progress:
    progress.add_task("database", total=None, started=True)
    # Database operation here
    progress.update("database", completed=True)
```

### Color Scheme

| Element | Color | Purpose |
|---------|--------|---------|
| High priority | red | Indicates urgent tasks |
| Medium priority | yellow | Indicates normal priority tasks |
| Low priority | green | Indicates non-urgent tasks |
| Pending status | yellow | Indicates incomplete tasks |
| Completed status | green | Indicates finished tasks |
| Overdue due date | red | Indicates missed deadline |
| Success messages | green | Indicates successful operations |
| Error messages | red | Indicates operation failures |
| Warning messages | yellow | Indicates validation warnings |

## Input Validation

### Priority Enum

**Values**: `low`, `medium`, `high`
**Default**: `medium`
**Validation**: Pydantic enum with clear error message

### Status Enum

**Values**: `pending`, `completed`, `all`
**Default**: `pending` (for create), `all` (for list filter)
**Validation**: Pydantic enum with clear error message

### Sort Enum

**Values**: `date`, `priority`, `alpha`
**Default**: `date`
**Validation**: Pydantic enum with clear error message

### Date Format

**Accepted formats**:
- ISO 8601: `2025-01-15` or `2025-01-15T14:30:00`
- Human-readable: `Jan 15, 2025` (requires date parser)
- Relative: `tomorrow`, `next week` (future enhancement, not in scope)

## Error Messages

### User-Friendly Error Format

```python
from rich.console import Console
from rich.text import Text

console = Console()

# Error example
error_msg = Text(
    "Invalid priority: 'urgent'. Valid options are: low, medium, high",
    style="bold red"
)
console.print(error_msg)

# Success example
success_msg = Text(
    "Task created successfully with ID: 42",
    style="bold green"
)
console.print(success_msg)
```

### Common Error Messages

| Error Scenario | Message | Color |
|---------------|---------|--------|
| Invalid priority | "Invalid priority: '<value>'. Valid options are: low, medium, high" | red |
| Invalid status | "Invalid status: '<value>'. Valid options are: pending, completed" | red |
| Invalid sort | "Invalid sort: '<value>'. Valid options are: date, priority, alpha" | red |
| Task not found | "Task with ID <id> not found" | red |
| Empty title | "Title cannot be empty" | red |
| No tasks found | "No tasks found matching criteria" | yellow |
| Database error | "Error connecting to database. Please try again later." | red |
