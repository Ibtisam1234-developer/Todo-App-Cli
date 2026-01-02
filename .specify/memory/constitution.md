<!--
SYNC IMPACT REPORT
==================
Version Change: 1.0.0 → 2.0.0 (MAJOR)
  Rationale: Backward-incompatible changes including async I/O transition, repository pattern restructuring, and technology stack upgrades

Modified Principles:
  - VI. Neon DB Integration Standard → VII. Neon DB Integration Standard (enhanced with async, retry logic, prepared statements)
  - All file references updated (db.py → database.py, services.py → repository.py, menu.py → cli.py)

Added Sections:
  - VII. Asynchronous Architecture (NEW - core principle)
  - VIII. Type Safety Mandate (NEW - 100% type hinting requirement)
  - IX. Rich UI Standard (NEW - Rich library for all user output)
  - X. Logging and Observability (NEW - .app.log for debugging)
  - 4.1-4.5 Sub-Agent System restructured to match new layer architecture
  - Advanced Features section added (filtering, search, categories, validation)

Removed Sections:
  - None (all principles preserved, enhanced, or restructured)

Templates Status:
  ✅ plan-template.md - Constitution Check section compatible (no changes needed)
  ✅ spec-template.md - Scope/requirements compatible (no changes needed)
  ✅ tasks-template.md - Task categorization compatible (no changes needed)
  ✅ phr-template.prompt.md - Compatible (no changes needed)
  ⚠️ CLAUDE.md - REQUIRES UPDATE: Implementation Summary section still references old synchronous architecture and file names
  ⚠️ src/ - EXISTING CODE: Synchronous implementation (db.py, services.py, menu.py) will need migration to async architecture

Follow-up TODOs:
  - Update CLAUDE.md Implementation Summary to reflect async architecture and new file structure
  - Migrate existing synchronous code to async architecture when implementing features
  - Ensure all database operations use psycopg[pool] with async/await patterns
-->

# TODO CLI Constitution: Advanced Async Architecture

## 1. Objective

Build an interactive, terminal-based Todo application using Python 3.12+ with Typer, Rich, and Neon Serverless PostgreSQL. The application features a 100% asynchronous architecture using the repository pattern, with advanced capabilities including categories, smart filtering, full-text search, and a rich terminal UI. Development follows Spec-Kit Plus and Claude Code with controlled sub-agent orchestration.

## 2. Scope

The system is a local, asynchronous CLI application. All business logic, database access, and user interaction are handled through structured Python modules. All Python code resides strictly in `/src`.

## 3. Development Methodology

This project MUST follow **spec-driven development** using **Spec-Kit Plus** and **Claude Code**.

Work is executed through **specialized sub-agents** operating under strict authority boundaries and an enforced execution order.

No agent may violate this constitution.

## Core Principles

### I. Separation of Concerns (NON-NEGOTIABLE)

Each sub-agent operates within a clearly defined boundary:
- **Database Agent**: Owns connection pool and schema initialization (`database.py`)
- **Repository Agent**: Implements CRUD SQL logic (`repository.py`)
- **Service Logic Agent**: Orchestrates business operations (`services.py`, `models.py`)
- **CLI/UX Agent**: Manages user interaction and Rich UI (`cli.py`)
- **Spec Guardian Agent**: Enforces compliance (no code, no features)
- **Documentation Agent**: Maintains documentation (no code changes)

Violations of scope boundaries MUST be rejected immediately. No agent may perform work assigned to another agent.

### II. Orchestrator-Controlled Workflow (NON-NEGOTIABLE)

All work MUST proceed through the mandatory execution flow defined in Section 5. No phase may begin until the previous phase is approved by the Spec Guardian Agent.

The Orchestrator coordinates all sub-agents and enforces the workflow order. Direct agent-to-agent communication is forbidden.

### III. Guardian Gate System (NON-NEGOTIABLE)

Every implementation phase MUST be reviewed and approved by the Spec Guardian Agent before progression:
1. Database implementation → Guardian review → Approval/Rejection
2. Repository implementation → Guardian review → Approval/Rejection
3. Service logic implementation → Guardian review → Approval/Rejection
4. CLI/UX implementation → Guardian review → Approval/Rejection
5. Final audit → Guardian review → Approval/Rejection

Rejections MUST include specific violations and required corrections.

### IV. Code Location Discipline

All Python code MUST reside in `/src`. No production code outside this directory.

File placement is non-negotiable:
- Database: `src/database.py` (connection pool, schema, retry logic)
- Repository: `src/repository.py` (CRUD SQL queries)
- Business logic: `src/services.py` (business orchestration), `src/models.py` (Pydantic schemas)
- CLI interface: `src/cli.py` (Typer commands, Rich UI)

### V. No Interaction Mixing

Strict separation enforced:
- Database Agent: NO `input()`, NO `print()`, NO Rich UI, NO business logic
- Repository Agent: NO user input, NO Rich UI, NO business orchestration
- Service Logic Agent: NO database connections, NO `input()`, NO Rich UI, NO SQL
- CLI/UX Agent: NO SQL queries, NO business logic, NO schema changes, NO direct database access

Each agent communicates only through defined interfaces (async function calls, return values).

### VI. Asynchronous Architecture (NON-NEGOTIABLE)

The core application MUST be 100% asynchronous:
- All database operations use `asyncio` and `async/await` patterns
- Database connections via `psycopg[pool]` async connection pool
- NO blocking I/O operations in the async path
- All repository methods are async functions
- All service methods are async functions
- CLI commands must handle async properly (e.g., using Typer's async support or `anyio.run`)

Rationale: Asynchronous architecture enables efficient handling of database operations, especially important for Neon's cold start behavior and concurrent operations.

### VII. Type Safety Mandate (NON-NEGOTIABLE)

100% Type Hinting is mandatory for every function signature:
- All functions MUST have return type annotations
- All parameters MUST have type annotations
- Use Pydantic models for all data schemas and validation
- Use `typing` module for complex types (Optional, List, Dict, etc.)
- Type checkers (mypy) must pass without errors

Rationale: Type safety catches errors at development time, improves IDE autocomplete, and documents interfaces clearly.

### VIII. Rich UI Standard (NON-NEGOTIABLE)

All user-facing output MUST use the `Rich` library:
- Listings MUST use `rich.table.Table`
- Task details MUST use `rich.panel.Panel`
- Database operations MUST show `rich.progress.Progress` (spinners)
- Error messages MUST use `rich.console.Console` with colors/formatting
- NO plain `print()` statements for user output

Rationale: Rich provides professional terminal UI with tables, panels, progress indicators, and formatted output.

### IX. Logging and Observability (NON-NEGOTIABLE)

Maintain a hidden `.app.log` file for debugging:
- Log database transactions and SQL queries
- Log connection pool events (acquire, release, retry)
- Log async operation lifecycle
- Log errors with full tracebacks (only to file, not CLI)
- CLI must remain clean and user-friendly

Rationale: Detailed logging enables debugging without cluttering the user's terminal experience.

### X. Neon DB Integration Standard (ENHANCED)

Database connection MUST use Neon Serverless PostgreSQL with:
- Connection string from environment variable (`DATABASE_URL`)
- Async connection pool via `psycopg[pool]`
- Robust retry logic for Neon "cold starts" (exponential backoff)
- Parameterized queries (NO string interpolation)
- Prepared statements where appropriate
- Transaction management (async context managers)
- Proper error handling for Connection, Integrity, and Operational errors

**Schema Requirements**:
- Relational schema with `tasks` and `categories` tables
- Foreign key linking: `tasks.category_id` → `categories.id`
- `CREATE TABLE IF NOT EXISTS` for idempotent initialization
- Support for priorities, statuses, and timestamps

Rationale: Neon's serverless nature requires connection pooling and retry logic. Relational schema with categories enables advanced features like filtering and organization.

### XI. Input Validation Standard (NON-NEGOTIABLE)

All user inputs MUST be validated via Pydantic models before reaching the database:
- Define Pydantic schemas for all entities (Task, Category)
- Validate inputs at CLI layer before calling services
- Show Rich-formatted validation errors to users
- Never pass invalid data to repository layer

Rationale: Pydantic ensures data integrity, provides clear validation errors, and serves as self-documenting schema definition.

## Sub-Agent System (Authoritative)

### 4.1 Spec Guardian Agent (PRIMARY AUTHORITY)

**Role**: Enforce compliance with this constitution, `sp.specify`, and `sp.plan`

**Responsibilities**:
- Review outputs from all other agents
- Block any violation of scope or constraints
- Approve progression between phases
- Validate compliance with separation of concerns
- Ensure no schema drift or interaction mixing
- Verify async patterns are followed
- Check type annotations completeness
- Validate Rich UI usage
- Verify Pydantic validation is applied

**Restrictions**:
- Cannot write production code
- Cannot introduce features
- Cannot modify specifications

**Authority Level**: ABSOLUTE - All other agents must receive Guardian approval to proceed

### 4.2 Database Agent (Neon DB)

**Role**: Own database connection pool and schema initialization

**Responsibilities**:
- Implement async Neon PostgreSQL connection pool (`psycopg[pool]`)
- Create `init_db()` function using `CREATE TABLE IF NOT EXISTS`
- Define relational schema: `tasks` and `categories` tables with foreign keys
- Implement retry logic for Neon cold starts (exponential backoff)
- Configure connection pool parameters (min_size, max_size, timeout)
- Handle database errors (Connection, Integrity, Operational)
- Document schema and connection configuration

**Allowed Scope**:
- `src/database.py` (connection pool, schema initialization, retry logic)
- Schema documentation

**Forbidden**:
- User input handling (`input()` calls)
- Rich UI or `print()` statements
- CRUD business logic (belongs in repository)
- SQL query execution (belongs in repository)
- Business rule validation

**Submission**: All output MUST be submitted to Spec Guardian for review

### 4.3 Repository Agent (NEW LAYER)

**Role**: Implement CRUD SQL logic isolated from CLI interface

**Responsibilities**:
- Implement all CRUD SQL queries (Create, Read, Update, Delete)
- Use parameterized queries to prevent SQL injection
- Implement full-text search using Postgres `LIKE` or `tsvector`
- Implement filtering by status, category, priority
- All functions MUST be async (return Coroutines)
- Receive Pydantic models or primitives as parameters
- Return structured data (dicts, Pydantic models, or lists)

**Allowed Scope**:
- `src/repository.py` (SQL CRUD functions)
- Optional: `src/queries.py` (SQL query constants if needed)

**Forbidden**:
- User input handling (`input()` calls)
- Rich UI or `print()` statements
- Business logic validation (belongs in services)
- Database connections (must use `database.py` pool)
- CLI-specific concerns

**Submission**: All output MUST be submitted to Spec Guardian for review

### 4.4 Service Logic Agent

**Role**: Orchestrate business operations and manage entity lifecycles

**Responsibilities**:
- Implement business orchestration for tasks and categories
- Coordinate calls to repository layer
- Define Pydantic data models (Task, Category)
- Validate business rules (task existence, category validity, etc.)
- Implement filtering and search logic orchestration
- Handle error translation from repository to user-friendly messages
- All functions MUST be async

**Allowed Scope**:
- `src/services.py` (business orchestration functions)
- `src/models.py` (Pydantic schemas: Task, Category)

**Forbidden**:
- Direct database connections (MUST call repository functions)
- User input collection (`input()` calls)
- Rich UI output (must return data structures for CLI layer)
- SQL queries (belongs in repository)
- Schema modifications (belongs in database agent)

**Submission**: All output MUST be submitted to Spec Guardian for review

### 4.5 CLI / UX Agent (NEW NAME - was Menu/UX)

**Role**: Manage interactive user experience with Rich UI

**Responsibilities**:
- Implement Typer CLI commands for all operations
- Use Rich for all user output (Table, Panel, Progress)
- Collect and validate user input via Typer parameters/arguments
- Implement CLI flags for filtering (`--status`, `--category`, `--priority`)
- Implement search functionality
- Show Rich-formatted error messages
- Handle async command execution (using Typer's async support or `anyio.run`)
- Implement progress spinners for database operations

**Allowed Scope**:
- `src/cli.py` (Typer commands, Rich UI, input handling)
- Input validation for format only (business validation belongs to services)

**Forbidden**:
- SQL queries or database connections
- Business logic implementation (belongs in services)
- Schema changes (belongs in database agent)
- Direct repository calls (MUST use service layer)
- Direct database access

**Submission**: All output MUST be submitted to Spec Guardian for review

### 4.6 Documentation Agent

**Role**: Maintain project documentation

**Responsibilities**:
- Create and maintain `README.md` with setup instructions
- Update `CLAUDE.md` with development guidelines
- Document Neon DB configuration
- Document async architecture and patterns
- Maintain `specs/history` updates
- Generate quickstart guides

**Allowed Scope**:
- `README.md`
- `CLAUDE.md`
- `specs/` directory documentation
- Configuration documentation

**Forbidden**:
- Code changes (any `.py` files)
- Specification changes
- Architecture modifications

**Submission**: All output MUST be submitted to Spec Guardian for review

## Execution Flow (MANDATORY)

Work MUST proceed in the following order. No step may be skipped.

### Step 1: Spec Guardian Agent - Initial Validation

**Actions**:
- Validates constitution compliance
- Reviews feature specification
- Reviews implementation plan
- Confirms scope boundaries are clear
- Approves start of development

**Gate**: Development cannot proceed until Guardian approves

### Step 2: Database Agent - Schema and Connection Layer

**Actions**:
- Implements async Neon DB connection pool in `src/database.py`
- Creates relational schema (tasks, categories tables) with foreign keys
- Implements `init_db()` with `CREATE TABLE IF NOT EXISTS`
- Implements retry logic for Neon cold starts
- Configures connection pool parameters
- Submits output for review

**Deliverables**:
- `src/database.py` (async connection pool, schema initialization, retry logic)
- Schema documentation

### Step 3: Spec Guardian Agent - Database Review

**Actions**:
- Reviews database implementation
- Validates NO user interaction code
- Validates NO business logic or CRUD operations
- Validates async patterns followed
- Validates error handling for connections
- Approves or rejects with specific violations

**Gate**: Repository layer cannot proceed until approval

### Step 4: Repository Agent - CRUD SQL Layer

**Actions**:
- Implements all CRUD SQL functions in `src/repository.py`
- Uses parameterized queries (prepared statements)
- Implements full-text search (LIKE or tsvector)
- Implements filtering queries (status, category, priority)
- Ensures all functions are async
- Submits output for review

**Deliverables**:
- `src/repository.py` (async CRUD functions)

### Step 5: Spec Guardian Agent - Repository Review

**Actions**:
- Reviews repository implementation
- Validates NO user interaction
- Validates NO business logic
- Validates async patterns followed
- Validates parameterized queries
- Approves or rejects with specific violations

**Gate**: Service layer cannot proceed until approval

### Step 6: Service Logic Agent - Business Logic Implementation

**Actions**:
- Implements all service functions in `src/services.py`
- Defines Pydantic models in `src/models.py`
- Implements business orchestration and validation
- Ensures all functions are async
- Ensures NO direct database connections
- Ensures NO user interaction or Rich UI
- Submits output for review

**Deliverables**:
- `src/services.py` (async business orchestration)
- `src/models.py` (Pydantic schemas)

### Step 7: Spec Guardian Agent - Service Layer Review

**Actions**:
- Reviews service implementation
- Validates NO database connections in service code
- Validates NO user interaction (`input()`/`print()`/Rich)
- Validates proper separation from repository layer
- Validates Pydantic models used throughout
- Validates async patterns followed
- Approves or rejects with specific violations

**Gate**: CLI/UX layer cannot proceed until approval

### Step 8: CLI / UX Agent - User Interface Implementation

**Actions**:
- Implements Typer CLI commands in `src/cli.py`
- Uses Rich library for all output (Table, Panel, Progress)
- Implements CLI flags for filtering (`--status`, `--category`, `--priority`)
- Implements search functionality
- Shows Rich-formatted errors
- Handles async command execution properly
- Ensures all user interaction is in this layer
- Ensures NO SQL or business logic
- Submits output for review

**Deliverables**:
- `src/cli.py` (Typer commands, Rich UI)

### Step 9: Spec Guardian Agent - Final Compliance Audit

**Actions**:
- Performs comprehensive compliance audit
- Validates all scope boundaries respected
- Validates NO interaction mixing
- Validates NO schema drift
- Validates all code in correct locations
- Validates async architecture followed throughout
- Validates 100% type annotations present
- Validates Rich UI used for all output
- Validates logging to `.app.log`
- Approves or rejects entire implementation

**Gate**: Documentation cannot proceed until approval

### Step 10: Documentation Agent - Project Documentation

**Actions**:
- Produces `README.md` with setup instructions
- Updates `CLAUDE.md` with implementation guidance
- Documents Neon DB configuration
- Documents async architecture patterns
- Creates quickstart guide
- Documents CLI commands and flags

**Deliverables**:
- `README.md`
- `CLAUDE.md` updates
- Configuration documentation

## Enforcement Rules

### No Skip Review

No agent may skip Guardian review. Every implementation phase MUST be reviewed and approved before proceeding to the next phase.

### No Scope Violations

No agent may operate outside its assigned scope. Violations MUST trigger immediate rejection and correction before any further work proceeds.

### No Cross-Agent Work

Agents cannot perform work assigned to other agents. Database Agent cannot write repository logic. Repository Agent cannot implement business rules. Service Agent cannot write CLI code.

### Correction Protocol

When Guardian rejects output:
1. Guardian MUST specify exact violations
2. Violating agent MUST correct violations
3. Guardian MUST re-review corrections
4. Process repeats until approval

### Orchestrator Authority

The Orchestrator coordinates all agents and enforces workflow order. The Orchestrator's decisions on workflow progression are final.

## Advanced Features (MUST-HAVE Capabilities)

### Rich UI Components

The application MUST use Rich for all user-facing output:

- **Task Listings**: Use `rich.table.Table` with columns for ID, Title, Category, Status, Priority, Created Date
- **Task Details**: Use `rich.panel.Panel` for showing full task information
- **Progress Indicators**: Use `rich.progress.Progress` with spinner for database operations
- **Error Messages**: Use `rich.console.Console` with colors (red for errors, yellow for warnings, green for success)

### Smart Filtering

CLI MUST support filtering flags:

- `--status [pending|completed|all]`: Filter by task status
- `--category [name]`: Filter tasks by category name
- `--priority [low|medium|high]`: Filter by priority level

Filters are composable (can combine multiple flags).

### Search

Implement full-text search across task descriptions:

- Use Postgres `LIKE` with `%term%` pattern matching
- Optional: Use `tsvector` for advanced full-text search
- Search matches against title and description fields
- CLI flag: `--search <term>`

### Validation

All user inputs MUST be validated via Pydantic before reaching the database:

- **Task Model**: Validate title (required, max length), description (optional), priority (enum), category (exists), status (enum)
- **Category Model**: Validate name (required, unique), description (optional)
- Show Rich-formatted validation errors

### Error Handling

Catch specific Postgres errors and show human-readable Rich messages:

- **Connection Errors**: Neon cold start, network issues → Show "Connection error, retrying..." with spinner
- **Integrity Errors**: Duplicate category, foreign key violations → Show specific validation error
- **Operational Errors**: Database downtime → Show user-friendly error with suggestion
- NEVER show raw tracebacks to CLI user (only to `.app.log`)

## Change Control

Any of the following requires updating this constitution first:
- Adding or removing sub-agents
- Changing execution order
- Modifying scope boundaries
- Changing storage model (e.g., switching from Neon DB)
- Changing async architecture (switching to sync)
- Changing UI framework (e.g., switching from Rich)
- Removing type safety requirements

Constitution changes require:
1. Explicit proposal with rationale
2. Version increment (semantic versioning)
3. Template consistency updates
4. Guardian approval of amended constitution

## Non-Goals

This constitution explicitly EXCLUDES:
- Autonomous agent decision-making (Orchestrator controls workflow)
- Feature expansion beyond specification (Guardian blocks scope creep)
- Runtime agent behavior modification
- Cross-agent direct communication
- Agent self-modification of scope
- Synchronous I/O patterns (async is mandatory)
- Plain text terminal output (Rich UI is mandatory)
- Untyped code (type annotations are mandatory)

## Governance

This constitution is the highest authority in the project. All specifications, plans, agent outputs, and code MUST comply with it.

**Hierarchy of Authority**:
1. This Constitution (absolute authority)
2. Spec Guardian Agent (enforcement authority)
3. Feature Specification (`sp.specify`)
4. Implementation Plan (`sp.plan`)
5. All other agents and outputs

**Compliance Verification**:
- All agent outputs MUST pass Guardian review
- Guardian MUST reference specific constitution sections in approvals/rejections
- Orchestrator MUST enforce workflow order
- No exceptions without constitutional amendment
- Async patterns MUST be verified at each gate
- Type annotations MUST be complete
- Rich UI MUST be used for all output

**Amendment Procedure**:
- Amendments require explicit documentation
- Version increments follow semantic versioning (MAJOR.MINOR.PATCH)
  - MAJOR: Backward incompatible changes (e.g., sync → async, layer restructuring)
  - MINOR: New principle or section added (e.g., new agent type)
  - PATCH: Clarifications, wording fixes, non-semantic refinements
- All dependent templates MUST be updated for consistency
- Migration plan required for breaking changes

**Version**: 2.0.0 | **Ratified**: 2025-12-17 | **Last Amended**: 2025-12-30
