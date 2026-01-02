# Quickstart Guide: Task Organization & Usability

**Feature**: 001-task-organization
**Purpose**: Get started with enhanced Todo CLI featuring priorities, categories, filtering, search, and Rich UI
**Date**: 2025-12-30

## Prerequisites

1. **Python 3.12+** installed
2. **Neon PostgreSQL** database with connection string
3. **Environment configured** with `DATABASE_URL` variable

## Installation

### 1. Clone Repository

```bash
git clone <repository-url>
cd todo-cli
```

### 2. Set Up Python Environment

```bash
# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Linux/macOS)
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

**Required packages**:
- `typer` - CLI framework
- `rich` - Terminal UI
- `pydantic` - Data validation
- `psycopg[pool]` - Async PostgreSQL connection pool
- `python-dotenv` - Environment configuration

### 4. Configure Database

```bash
# Copy example environment file
copy .env.example .env

# Edit .env and add your Neon database URL
DATABASE_URL=postgresql://user:password@host:port/database?sslmode=require
```

**Alternative**: Set environment variable directly

```bash
# Windows
set DATABASE_URL=postgresql://...

# Linux/macOS
export DATABASE_URL=postgresql://...
```

## Basic Usage

### Create Your First Task

```bash
# Simple task with default priority
add "Buy groceries"

# Task with high priority and category
add "Submit project report" --priority high --category work

# Task with due date
add "Pay rent" --priority high --category personal --due "2025-01-15"

# Task with all fields
add "Complete documentation" --priority medium --category work --due "2025-01-10" --description "Write API docs for new features"
```

### List Tasks

```bash
# List all tasks (sorted by due date by default)
list

# List only pending tasks
list --status pending

# List only high priority work tasks
list --category work --priority high

# List all tasks sorted by priority
list --sort priority

# List tasks alphabetically
list --sort alpha

# Combine multiple filters
list --category work --status pending --priority high --sort date
```

### Search Tasks

```bash
# Search for tasks containing "meeting"
search "meeting"

# Search within specific category
search "project" --category work

# Search pending tasks
search "report" --status pending
```

### Update Tasks

```bash
# Change task priority
update 42 --priority high

# Change task category
update 42 --category personal

# Mark task as completed
update 42 --status completed

# Update multiple fields at once
update 42 --priority medium --due "2025-02-01"
```

### Delete Tasks

```bash
# Delete task by ID (will prompt for confirmation)
delete 42
```

## Rich UI Features

### Color-Coded Priority

Tasks are displayed with color-coded priorities:
- **Red** = High priority
- **Yellow** = Medium priority
- **Green** = Low priority

### Progress Indicators

Database operations show a spinner while in progress:
```
⠋ Loading tasks...
```

### Formatted Tables

Tasks are displayed in a clean table format:
```
 ID │ Title                    │ Category │ Priority │ Status   │ Due Date   │
────┼──────────────────────────┼──────────┼──────────┼──────────┼────────────┤
 42 │ Complete project report   │ work      │ high     │ pending  │ 2025-01-15 │
 43 │ Buy groceries            │ personal  │ low      │ pending  │ None        │
```

## Common Workflows

### Daily Task Review

```bash
# List pending work tasks sorted by priority
list --category work --status pending --sort priority

# Update priority as needed
update 42 --priority high

# Mark completed tasks
update 43 --status completed
```

### Weekly Planning

```bash
# List all personal tasks
list --category personal --sort date

# Create new tasks for the week
add "Schedule dentist appointment" --category personal --due "2025-01-20" --priority high

# Search for overdue work tasks
search "deadline" --category work --status pending
```

### Focus Sessions

```bash
# List only high priority tasks
list --priority high --status pending

# Work on tasks...

# Mark as complete
update 42 --status completed
update 43 --status completed
```

## Reference

### Priority Values

| Value | Meaning | When to Use |
|-------|--------|---------------|
| low | Non-urgent tasks | Optional tasks, can wait |
| medium | Normal priority | Standard tasks with no special urgency |
| high | Urgent tasks | Deadline-driven, critical tasks |

### Status Values

| Value | Meaning |
|-------|--------|
| pending | Task not yet completed |
| completed | Task finished |

### Sort Options

| Option | Result | Best For |
|-------|--------|-----------|
| date | Sort by due date (earliest first) | Time-based prioritization |
| priority | Sort by priority (high → medium → low) | Urgency-based focus |
| alpha | Sort alphabetically by title | Finding specific tasks |

### Filter Flags

| Flag | Values | Example |
|------|---------|---------|
| `--status` | pending, completed, all | `list --status pending` |
| `--category` | Any text | `list --category work` |
| `--priority` | low, medium, high | `list --priority high` |

## Troubleshooting

### Database Connection Issues

**Error**: "Error connecting to database. Please try again later."

**Solutions**:
1. Verify `DATABASE_URL` is set correctly
2. Check Neon database is accessible
3. Verify network connectivity
4. Try again (connection pool will retry with exponential backoff)

### Validation Errors

**Error**: "Invalid priority: 'urgent'. Valid options are: low, medium, high"

**Solution**: Use correct priority value (low, medium, or high)

### Task Not Found

**Error**: "Task with ID 999 not found"

**Solutions**:
1. List tasks to find correct ID: `list`
2. Search by title: `search "task title"`
3. Check ID is correct number

### No Results Found

**Message**: "No tasks found matching criteria"

**Solutions**:
1. Verify filter values are correct (check spelling)
2. Try broader filters (e.g., `--status all` instead of `--status pending`)
3. List all tasks without filters: `list`

## Next Steps

1. Explore all commands: `--help`
2. Create your first task: `add "My first task"`
3. List all tasks: `list`
4. Learn about filtering: `list --help`
5. Try searching: `search "keyword"`

## Getting Help

View help for any command:

```bash
# General help
--help

# Help for specific command
add --help
list --help
update --help
delete --help
search --help
```
