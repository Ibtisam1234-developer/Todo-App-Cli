# Menu/UX Layer Contract

**Files**: `src/menu.py`, `src/main.py`
**Owner**: Menu/UX Agent
**Purpose**: Interactive terminal user interface and application lifecycle

## Constitutional Constraints

**ALLOWED**:
- Menu display (`print()`)
- User input collection (`input()`)
- Input validation (format, type)
- Calls to service layer functions
- User-friendly error messages

**FORBIDDEN**:
- SQL queries or database connections
- Business logic (validation of business rules)
- Schema changes
- Direct database calls (must use service layer)

## Function Signatures (src/menu.py)

### Menu Display

```python
def display_menu() -> None:
    """Display main menu options to user."""
```

### Input Collection

```python
def get_menu_choice() -> str:
    """Get and validate menu selection from user."""
```

### Task Operations

```python
def handle_create_task() -> None:
    """Collect task details and create task."""

def handle_list_tasks() -> None:
    """Display all tasks in readable format."""

def handle_update_task() -> None:
    """Collect task ID and new details, update task."""

def handle_toggle_task() -> None:
    """Collect task ID and toggle completion."""

def handle_delete_task() -> None:
    """Collect task ID and delete task."""
```

### Formatting

```python
def format_task_display(task: Task) -> str:
    """Format task for terminal display."""
```

## Application Entry Point (src/main.py)

```python
def main() -> None:
    """
    Application entry point.
    - Initialize database connection
    - Run main menu loop
    - Handle cleanup
    """

if __name__ == "__main__":
    main()
```

## Constitutional Compliance Checklist

- [ ] NO SQL queries
- [ ] NO database connections (use service layer)
- [ ] NO business logic validation (display service errors only)
- [ ] ALL user interaction in this layer
- [ ] Clean error display (user-friendly messages)

---

**Contract Status**: ✅ DEFINED
**Owner**: Menu/UX Agent
**Reviewer**: Spec Guardian Agent
