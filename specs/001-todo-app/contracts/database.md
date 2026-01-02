# Database Layer Contract

**File**: `src/db.py`
**Owner**: Database Agent
**Purpose**: Neon PostgreSQL connection and CRUD query execution

## Constitutional Constraints

**ALLOWED**:
- Database connection management
- SQL query execution
- Schema initialization
- Parameterized query construction
- Database error handling

**FORBIDDEN**:
- User input collection (`input()` calls)
- User output rendering (`print()` calls for menus/messages)
- Business logic (validation, state management)
- Direct calls to service layer

## Function Signatures

### Connection Management

```python
def get_connection() -> psycopg2.extensions.connection:
    """
    Establish connection to Neon PostgreSQL using DATABASE_URL from environment.

    Returns:
        Active database connection with autocommit disabled

    Raises:
        ConnectionError: If DATABASE_URL environment variable is missing
        ConnectionError: If connection to database fails

    Implementation Notes:
        - Load DATABASE_URL from environment (python-dotenv)
        - Use psycopg2.connect() with connection string
        - Set autocommit=False for transaction control
        - Connection must support context manager (with statement)

    Constitutional Compliance:
        ✅ NO user interaction
        ✅ NO business logic
        ✅ Database-only concern
    """
```

### Schema Initialization

```python
def initialize_schema(conn: psycopg2.extensions.connection) -> None:
    """
    Create tasks table if it doesn't exist.

    Args:
        conn: Active database connection

    Raises:
        DatabaseError: If schema creation fails

    Implementation Notes:
        - Execute CREATE TABLE IF NOT EXISTS (idempotent)
        - Commit transaction on success
        - Rollback transaction on failure
        - Table definition from data-model.md

    Constitutional Compliance:
        ✅ NO user interaction
        ✅ NO business logic
        ✅ Database-only concern
    """
```

### Create Operations

```python
def create_task_db(conn: psycopg2.extensions.connection,
                   title: str,
                   description: str | None) -> int:
    """
    Insert new task into database.

    Args:
        conn: Active database connection
        title: Task title (assumed pre-validated by service layer)
        description: Task description (optional, can be None)

    Returns:
        ID of newly created task (SERIAL auto-generated)

    Raises:
        DatabaseError: If insertion fails

    Implementation Notes:
        - Use parameterized query: INSERT INTO tasks (title, description) VALUES (%s, %s) RETURNING id
        - NO validation (service layer responsibility)
        - is_completed defaults to FALSE (database default)
        - created_at and updated_at default to NOW() (database default)
        - Commit transaction on success
        - Rollback transaction on failure

    Constitutional Compliance:
        ✅ NO validation (service layer responsibility)
        ✅ NO user interaction
        ✅ Parameterized queries (SQL injection prevention)
    """
```

### Read Operations

```python
def get_all_tasks_db(conn: psycopg2.extensions.connection) -> list[tuple]:
    """
    Retrieve all tasks from database ordered by creation date (newest first).

    Args:
        conn: Active database connection

    Returns:
        List of tuples, each tuple is:
        (id, title, description, is_completed, created_at, updated_at)

        Empty list if no tasks exist

    Raises:
        DatabaseError: If query fails

    Implementation Notes:
        - Query: SELECT id, title, description, is_completed, created_at, updated_at
                 FROM tasks
                 ORDER BY created_at DESC
        - Return raw tuples (service layer converts to Task objects)
        - No pagination (scope supports < 1000 tasks)

    Constitutional Compliance:
        ✅ NO business logic (no filtering, no transformation)
        ✅ Returns raw data for service layer processing
    """
```

### Update Operations

```python
def update_task_db(conn: psycopg2.extensions.connection,
                   task_id: int,
                   title: str | None,
                   description: str | None) -> bool:
    """
    Update task title and/or description.

    Args:
        conn: Active database connection
        task_id: ID of task to update
        title: New title (None = no change)
        description: New description (None = no change)

    Returns:
        True if task was updated (task existed)
        False if task was not found

    Raises:
        DatabaseError: If update fails

    Implementation Notes:
        - Build dynamic UPDATE query based on which fields are not None
        - Always set updated_at = NOW()
        - Use parameterized query for all values
        - Example: UPDATE tasks SET title = %s, updated_at = NOW() WHERE id = %s
        - Check rowcount to determine if task was found
        - Commit transaction on success
        - Rollback transaction on failure

    Constitutional Compliance:
        ✅ NO validation (service layer validates before calling)
        ✅ Parameterized queries
        ✅ NO business logic (no field value processing)
    """
```

```python
def toggle_task_status_db(conn: psycopg2.extensions.connection,
                          task_id: int) -> bool:
    """
    Toggle task completion status (complete <-> incomplete).

    Args:
        conn: Active database connection
        task_id: ID of task to toggle

    Returns:
        True if task was toggled (task existed)
        False if task was not found

    Raises:
        DatabaseError: If toggle fails

    Implementation Notes:
        - Query: UPDATE tasks
                 SET is_completed = NOT is_completed, updated_at = NOW()
                 WHERE id = %s
        - Use parameterized query
        - Check rowcount to determine if task was found
        - Commit transaction on success
        - Rollback transaction on failure

    Constitutional Compliance:
        ✅ Simple boolean toggle (no conditional logic)
        ✅ Parameterized query
        ✅ NO validation needed (toggle is idempotent)
    """
```

### Delete Operations

```python
def delete_task_db(conn: psycopg2.extensions.connection,
                   task_id: int) -> bool:
    """
    Permanently delete task from database.

    Args:
        conn: Active database connection
        task_id: ID of task to delete

    Returns:
        True if task was deleted (task existed)
        False if task was not found

    Raises:
        DatabaseError: If deletion fails

    Implementation Notes:
        - Query: DELETE FROM tasks WHERE id = %s
        - Use parameterized query
        - Check rowcount to determine if task was found
        - Commit transaction on success
        - Rollback transaction on failure
        - No CASCADE needed (no foreign keys)

    Constitutional Compliance:
        ✅ Simple DELETE operation
        ✅ NO validation (service layer confirms intent)
        ✅ Parameterized query
    """
```

## Error Handling

### Database Exceptions

```python
try:
    # Database operation
except psycopg2.OperationalError as e:
    # Connection errors, network issues
    conn.rollback()
    raise ConnectionError(f"Database connection failed: {e}")

except psycopg2.IntegrityError as e:
    # Constraint violations (unlikely with our schema)
    conn.rollback()
    raise DatabaseError(f"Data integrity error: {e}")

except psycopg2.DatabaseError as e:
    # General database errors
    conn.rollback()
    raise DatabaseError(f"Database operation failed: {e}")
```

### Transaction Management

- All write operations (INSERT, UPDATE, DELETE) must commit or rollback
- Use `try-except-finally` for proper cleanup
- Connection rollback on any error
- Connection commit only on success

## Constitutional Compliance Checklist

- [ ] NO `input()` calls (user interaction forbidden)
- [ ] NO `print()` calls for menus or user messages (logging acceptable)
- [ ] NO validation logic (service layer responsibility)
- [ ] NO business rules (service layer responsibility)
- [ ] ALL queries use parameterized format (SQL injection prevention)
- [ ] ALL database errors caught and re-raised with context
- [ ] ALL write operations commit or rollback explicitly
- [ ] Connection passed as parameter (no global state)

## Testing Criteria (for Spec Guardian Review)

1. **NO User Interaction**: Search code for `input()` and `print()` - must be ZERO occurrences
2. **NO Business Logic**: Check for validation, calculations, or business rules - must be ZERO
3. **Parameterized Queries**: All SQL uses `%s` placeholders, NO string interpolation
4. **Error Handling**: All psycopg2 exceptions caught and handled
5. **Transaction Management**: All writes have explicit commit/rollback
6. **Return Types**: Functions return only raw data (tuples, ints, bools)

## Dependencies

```python
import os
import psycopg2
import psycopg2.extensions
from dotenv import load_dotenv
```

---

**Contract Status**: ✅ DEFINED
**Owner**: Database Agent
**Reviewer**: Spec Guardian Agent
**Next**: Service Layer Contract
