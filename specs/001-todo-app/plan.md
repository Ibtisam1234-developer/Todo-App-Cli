# Implementation Plan: Interactive Todo Application

**Branch**: `001-todo-app` | **Date**: 2025-12-17 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-todo-app/spec.md`

## Summary

Build an interactive, terminal-based Todo application with full CRUD operations that persists tasks using Neon Serverless PostgreSQL. The application follows a strict agent-oriented architecture with three distinct layers: Database (Neon connection and SQL), Service Logic (business rules), and Menu/UX (terminal interaction). All layers are coordinated through a mandatory review workflow enforced by the Spec Guardian Agent.

## Technical Context

**Language/Version**: Python 3.11+
**Primary Dependencies**: psycopg2-binary (Neon PostgreSQL driver), python-dotenv (environment configuration)
**Storage**: Neon Serverless PostgreSQL (connection via DATABASE_URL environment variable)
**Testing**: pytest (unit and integration tests)
**Target Platform**: Cross-platform terminal (Windows, Linux, macOS)
**Project Type**: Single project with strict `/src` code location
**Performance Goals**: Task operations complete under 5 seconds, immediate menu response
**Constraints**: No external UI frameworks, terminal-only interaction, all Python code in `/src`
**Scale/Scope**: Single-user local application, support 1000+ tasks

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Principle I: Separation of Concerns
✅ **PASS** - Three distinct agents with clear boundaries:
- Database Agent (`src/db.py`) - Database operations only
- Service Logic Agent (`src/services.py`, `src/models.py`) - Business logic only
- Menu/UX Agent (`src/menu.py`, `src/main.py`) - User interaction only

### Principle II: Orchestrator-Controlled Workflow
✅ **PASS** - Mandatory 8-step execution flow defined:
1. Spec Guardian validates specs
2. Database Agent implements layer
3. Spec Guardian reviews
4. Service Logic Agent implements layer
5. Spec Guardian reviews
6. Menu/UX Agent implements layer
7. Spec Guardian final audit
8. Documentation Agent finalizes

### Principle III: Guardian Gate System
✅ **PASS** - Review gates enforced between all phases. No agent proceeds without approval.

### Principle IV: Code Location Discipline
✅ **PASS** - All Python code resides in `/src`:
- `src/db.py` - Database layer
- `src/services.py` - Service layer
- `src/models.py` - Data models
- `src/menu.py` - Menu functions
- `src/main.py` - Entry point

### Principle V: No Interaction Mixing
✅ **PASS** - Strict layer separation enforced:
- Database: NO `input()`, NO `print()`, NO business logic
- Service: NO DB connections, NO `input()`, NO `print()`
- Menu/UX: NO SQL, NO business logic

### Principle VI: Neon DB Integration Standard
✅ **PASS** - Neon requirements met:
- DATABASE_URL from environment variable
- Connection pooling via psycopg2
- Parameterized queries (no string interpolation)
- Transaction management
- Error handling for connection failures

**Constitution Compliance**: ✅ ALL GATES PASS - Proceed to implementation

## Project Structure

### Documentation (this feature)

```text
specs/001-todo-app/
├── spec.md              # Feature specification
├── plan.md              # This file (/sp.plan command output)
├── research.md          # Phase 0 output (technology decisions)
├── data-model.md        # Phase 1 output (database schema)
├── quickstart.md        # Phase 1 output (setup and run instructions)
├── contracts/           # Phase 1 output (function signatures)
│   ├── database.md      # Database layer contract
│   ├── service.md       # Service layer contract
│   └── menu.md          # Menu layer contract
└── tasks.md             # Phase 2 output (/sp.tasks command - NOT created by /sp.plan)
```

### Source Code (repository root)

```text
src/
├── db.py                # Database Agent: Neon connection + CRUD queries
├── models.py            # Service Agent: Task data model
├── services.py          # Service Agent: Business logic orchestration
├── menu.py              # Menu Agent: Interactive terminal UI
└── main.py              # Menu Agent: Application entry point

.env                     # Environment configuration (DATABASE_URL)
requirements.txt         # Python dependencies
```

**Structure Decision**: Single project structure selected per constitution requirement. All Python code in `/src` with clear file-level agent ownership. No tests directory initially (testing optional per spec).

## Complexity Tracking

No constitution violations. Complexity tracking not required.

## Phase 0: Research & Technology Decisions

### Research Areas

1. **Neon Serverless PostgreSQL Integration**
   - **Decision**: Use psycopg2-binary for connection
   - **Rationale**: Official PostgreSQL adapter, compatible with Neon, supports connection pooling
   - **Alternatives Considered**:
     - psycopg3: Newer but less mature
     - SQLAlchemy: Too heavy for simple CRUD operations

2. **Environment Configuration**
   - **Decision**: python-dotenv for DATABASE_URL management
   - **Rationale**: Industry standard, simple `.env` file support
   - **Alternatives Considered**:
     - Manual os.environ: No `.env` file support
     - configparser: Overkill for single variable

3. **Terminal Menu Pattern**
   - **Decision**: Simple numbered menu with input() loops
   - **Rationale**: Cross-platform, no external dependencies
   - **Alternatives Considered**:
     - curses: Platform compatibility issues on Windows
     - prompt_toolkit: Additional dependency

4. **Error Handling Strategy**
   - **Decision**: Try-except at layer boundaries with descriptive messages
   - **Rationale**: Clear error propagation, user-friendly messages
   - **Alternatives Considered**:
     - Custom exception hierarchy: Over-engineered for scope

5. **Connection Pooling Strategy**
   - **Decision**: Single persistent connection per session
   - **Rationale**: Single-user local app, simple lifecycle
   - **Alternatives Considered**:
     - Connection pool library: Unnecessary for single-user

### Technology Stack Summary

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Language | Python | 3.11+ | Application runtime |
| Database | Neon PostgreSQL | Latest | Serverless task storage |
| DB Driver | psycopg2-binary | Latest | PostgreSQL connection |
| Config | python-dotenv | Latest | Environment variables |
| Terminal | Built-in input() | N/A | User interaction |

## Phase 1: Design & Contracts

### Data Model (data-model.md)

**Task Entity**:

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | SERIAL | PRIMARY KEY | Auto-increment unique identifier |
| title | VARCHAR(200) | NOT NULL | Task title (required) |
| description | TEXT | NULL | Optional task description |
| is_completed | BOOLEAN | NOT NULL, DEFAULT FALSE | Completion status |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Creation timestamp |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT NOW() | Last modification timestamp |

**Database Schema**:

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    is_completed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### Layer Contracts (contracts/)

**Database Layer Contract** (`contracts/database.md`):

```python
# src/db.py - Database Agent Ownership

def get_connection() -> psycopg2.extensions.connection:
    """
    Establish connection to Neon PostgreSQL using DATABASE_URL from environment.
    Returns: Active database connection
    Raises: ConnectionError if DATABASE_URL missing or connection fails
    """

def initialize_schema(conn: connection) -> None:
    """
    Create tasks table if it doesn't exist.
    Args: conn - Active database connection
    Raises: DatabaseError if schema creation fails
    """

def create_task_db(conn: connection, title: str, description: str | None) -> int:
    """
    Insert new task into database.
    Args:
        conn - Active database connection
        title - Task title (required)
        description - Task description (optional)
    Returns: ID of created task
    Raises: DatabaseError if insertion fails
    """

def get_all_tasks_db(conn: connection) -> list[tuple]:
    """
    Retrieve all tasks from database.
    Args: conn - Active database connection
    Returns: List of tuples (id, title, description, is_completed, created_at, updated_at)
    Raises: DatabaseError if query fails
    """

def update_task_db(conn: connection, task_id: int, title: str | None, description: str | None) -> bool:
    """
    Update task title and/or description.
    Args:
        conn - Active database connection
        task_id - ID of task to update
        title - New title (None = no change)
        description - New description (None = no change)
    Returns: True if task was updated, False if task not found
    Raises: DatabaseError if update fails
    """

def toggle_task_status_db(conn: connection, task_id: int) -> bool:
    """
    Toggle task completion status (complete <-> incomplete).
    Args:
        conn - Active database connection
        task_id - ID of task to toggle
    Returns: True if task was toggled, False if task not found
    Raises: DatabaseError if toggle fails
    """

def delete_task_db(conn: connection, task_id: int) -> bool:
    """
    Permanently delete task from database.
    Args:
        conn - Active database connection
        task_id - ID of task to delete
    Returns: True if task was deleted, False if task not found
    Raises: DatabaseError if deletion fails
    """
```

**Service Layer Contract** (`contracts/service.md`):

```python
# src/services.py - Service Logic Agent Ownership

def create_task(title: str, description: str | None = None) -> dict:
    """
    Create a new task with validation.
    Args:
        title - Task title (required, 1-200 chars)
        description - Task description (optional, max 1000 chars)
    Returns: dict with 'success': bool, 'task_id': int, 'message': str
    Validation: Title required, non-empty after strip
    """

def list_tasks() -> dict:
    """
    Retrieve all tasks.
    Returns: dict with 'success': bool, 'tasks': list[Task], 'message': str
    """

def update_task(task_id: int, title: str | None = None, description: str | None = None) -> dict:
    """
    Update existing task title and/or description with validation.
    Args:
        task_id - ID of task to update
        title - New title (optional, 1-200 chars if provided)
        description - New description (optional, max 1000 chars if provided)
    Returns: dict with 'success': bool, 'message': str
    Validation: Task exists, at least one field provided, title non-empty if provided
    """

def toggle_task_completion(task_id: int) -> dict:
    """
    Toggle task completion status.
    Args: task_id - ID of task to toggle
    Returns: dict with 'success': bool, 'message': str
    Validation: Task exists
    """

def delete_task(task_id: int) -> dict:
    """
    Permanently delete a task.
    Args: task_id - ID of task to delete
    Returns: dict with 'success': bool, 'message': str
    Validation: Task exists
    """
```

```python
# src/models.py - Service Logic Agent Ownership

@dataclass
class Task:
    """Task data model."""
    id: int
    title: str
    description: str | None
    is_completed: bool
    created_at: datetime
    updated_at: datetime

    def to_dict(self) -> dict:
        """Convert Task to dictionary representation."""

    @classmethod
    def from_db_row(cls, row: tuple) -> 'Task':
        """Create Task from database row tuple."""
```

**Menu/UX Layer Contract** (`contracts/menu.md`):

```python
# src/menu.py - Menu/UX Agent Ownership

def display_menu() -> None:
    """Display main menu options to user."""

def get_menu_choice() -> str:
    """
    Get and validate menu selection from user.
    Returns: Validated menu choice as string
    """

def handle_create_task() -> None:
    """
    Collect task details from user and create task.
    Displays success or error message.
    """

def handle_list_tasks() -> None:
    """
    Display all tasks in readable format.
    Shows ID, title, description, status for each task.
    """

def handle_update_task() -> None:
    """
    Collect task ID and new details from user and update task.
    Displays success or error message.
    """

def handle_toggle_task() -> None:
    """
    Collect task ID from user and toggle completion status.
    Displays success or error message.
    """

def handle_delete_task() -> None:
    """
    Collect task ID from user and delete task.
    Displays success or error message.
    """

def format_task_display(task: Task) -> str:
    """
    Format task for terminal display.
    Args: task - Task object
    Returns: Formatted string representation
    """
```

```python
# src/main.py - Menu/UX Agent Ownership

def main() -> None:
    """
    Application entry point.
    Initializes database connection, runs main menu loop, handles cleanup.
    """

if __name__ == "__main__":
    main()
```

### Quickstart (quickstart.md)

**Setup and Run Instructions**:

1. **Prerequisites**:
   - Python 3.11 or higher
   - Neon PostgreSQL account and database created
   - Git (for cloning repository)

2. **Installation**:
   ```bash
   # Clone repository
   git clone <repository-url>
   cd todo-hackathon

   # Install dependencies
   pip install -r requirements.txt
   ```

3. **Configuration**:
   ```bash
   # Create .env file in project root
   echo "DATABASE_URL=postgresql://user:password@host/database?sslmode=require" > .env

   # Replace with your Neon connection string from Neon dashboard
   ```

4. **Run Application**:
   ```bash
   python src/main.py
   ```

5. **Usage**:
   - Select menu options using numbers (1-6)
   - Follow prompts to create, view, update, toggle, or delete tasks
   - Select option 6 to exit

6. **Troubleshooting**:
   - **Connection Error**: Verify DATABASE_URL in `.env` file
   - **Module Not Found**: Run `pip install -r requirements.txt`
   - **Permission Error**: Check file permissions on `.env`

### Sub-Agent Implementation Order

Following the constitution's mandatory execution flow:

**Step 1: Spec Guardian - Initial Validation** ✅ COMPLETE
- Constitution check passed
- Specification validated
- Plan approved

**Step 2: Database Agent - Schema and Connection Layer**
- Implement `src/db.py` with all database functions
- Deliverables: `src/db.py`, schema documentation
- Submit to Spec Guardian for review

**Step 3: Spec Guardian - Database Review**
- Validate NO user interaction code (`input()`, `print()`)
- Validate NO business logic in database layer
- Validate proper error handling and parameterized queries
- Approve or reject with specific violations

**Step 4: Service Logic Agent - Business Logic Implementation**
- Implement `src/services.py` with all service functions
- Implement `src/models.py` with Task dataclass
- Ensure NO direct database connections
- Ensure NO user interaction
- Deliverables: `src/services.py`, `src/models.py`
- Submit to Spec Guardian for review

**Step 5: Spec Guardian - Service Layer Review**
- Validate NO database connections in service code
- Validate NO user interaction (`input()`, `print()`)
- Validate proper separation from database layer
- Approve or reject with specific violations

**Step 6: Menu/UX Agent - User Interface Implementation**
- Implement `src/menu.py` with all menu functions
- Implement `src/main.py` with application loop
- Ensure all user interaction in this layer
- Ensure NO SQL or business logic
- Deliverables: `src/menu.py`, `src/main.py`
- Submit to Spec Guardian for review

**Step 7: Spec Guardian - Final Compliance Audit**
- Validate all scope boundaries respected
- Validate NO interaction mixing
- Validate NO schema drift
- Validate all code in correct locations (`/src`)
- Approve or reject entire implementation

**Step 8: Documentation Agent - Project Documentation**
- Create `README.md` with setup instructions
- Update `CLAUDE.md` with implementation guidance
- Document Neon DB configuration
- Deliverables: `README.md`, `CLAUDE.md` updates

## Dependencies & Execution Order

1. **Phase 0 (Research)**: Complete - Technology decisions finalized
2. **Phase 1 (Design)**: Complete - Contracts and data model defined
3. **Phase 2 (Tasks)**: Next - Run `/sp.tasks` to generate implementation tasks
4. **Implementation**: Follow sub-agent order (Database → Service → Menu → Documentation)
5. **Guardian Reviews**: Between each implementation phase

## Risk Analysis

| Risk | Impact | Mitigation |
|------|--------|------------|
| Neon connection failure | Application unusable | Graceful error handling, clear error messages, connection retry logic |
| Agent scope violation | Architecture compromise | Spec Guardian mandatory review gates, strict contract enforcement |
| Database schema drift | Data integrity issues | Single schema definition, migration tracking, Guardian validation |
| Invalid user input | Application crash | Input validation at menu layer, error handling throughout |
| Long-running operations | Poor UX | Connection timeout configuration, user feedback during operations |

## Success Metrics

Implementation will be validated against specification success criteria:

- ✅ Task creation under 30 seconds
- ✅ 100% data retention across restarts
- ✅ Task viewing under 5 seconds
- ✅ Immediate status change reflection
- ✅ No data loss with network interruptions
- ✅ 100% clear error messages
- ✅ Intuitive first-attempt operation completion

## Next Steps

1. Run `/sp.tasks` to generate detailed implementation tasks organized by sub-agent
2. Begin implementation with Database Agent (`src/db.py`)
3. Submit to Spec Guardian for review after each layer
4. Complete all Guardian reviews before final delivery
5. Run `/sp.git.commit_pr` to commit and create pull request

---

**Plan Status**: ✅ COMPLETE - Ready for task generation (`/sp.tasks`)
**Constitution Compliance**: ✅ ALL GATES PASS
**Phase 0 Research**: ✅ COMPLETE
**Phase 1 Design**: ✅ COMPLETE
**Next Command**: `/sp.tasks`
