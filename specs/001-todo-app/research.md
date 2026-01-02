# Research: Interactive Todo Application

**Feature**: 001-todo-app
**Date**: 2025-12-17
**Purpose**: Technology decisions and architectural research for agent-oriented todo application

## Research Areas

### 1. Neon Serverless PostgreSQL Integration

**Decision**: Use psycopg2-binary for Neon connection

**Rationale**:
- Official PostgreSQL adapter with broad industry adoption
- Fully compatible with Neon Serverless PostgreSQL
- Supports connection pooling natively
- Binary package includes all dependencies (no compilation required)
- Stable and mature (v2.9+ recommended)

**Alternatives Considered**:
- **psycopg3**: Newer async-first design but less mature ecosystem, potential compatibility issues
- **SQLAlchemy**: Full ORM framework adds unnecessary complexity for simple CRUD operations, violates simplicity principle
- **asyncpg**: Async-only, adds complexity not needed for single-user terminal app

**Implementation Notes**:
- Use `psycopg2.connect()` with DATABASE_URL
- Connection string format: `postgresql://user:password@host/database?sslmode=require`
- Enable SSL mode for Neon security requirements

---

### 2. Environment Configuration Management

**Decision**: python-dotenv for DATABASE_URL management

**Rationale**:
- Industry standard for Python environment variable management
- Simple `.env` file support (no code changes for different environments)
- Zero runtime dependencies (development-time only)
- Works cross-platform (Windows, Linux, macOS)
- Prevents hardcoding sensitive credentials

**Alternatives Considered**:
- **Manual os.environ**: No `.env` file support, requires environment setup before running
- **configparser**: INI-file format less common for env vars, overkill for single variable
- **PyYAML**: Adds parsing complexity, not standard for env vars

**Implementation Notes**:
- Place `.env` file in project root
- Load with `load_dotenv()` at application start
- Add `.env` to `.gitignore` to prevent credential commits

---

### 3. Terminal Menu Interaction Pattern

**Decision**: Simple numbered menu with input() loops

**Rationale**:
- Built into Python stdlib (no external dependencies)
- Cross-platform compatible (Windows, Linux, macOS)
- Simple implementation matches app complexity
- Clear user experience (type number, press Enter)
- No terminal control sequences required

**Alternatives Considered**:
- **curses library**: Platform compatibility issues on Windows, requires terminal control knowledge
- **prompt_toolkit**: Additional dependency, overkill for simple menu, learning curve
- **rich library**: Beautiful output but adds dependency, not required for MVP

**Implementation Notes**:
- Display numbered menu (1-6)
- Use `input()` for selection
- Validate input is numeric and in range
- Loop until user selects exit option

---

### 4. Error Handling Strategy

**Decision**: Try-except blocks at layer boundaries with descriptive messages

**Rationale**:
- Clear error propagation through layers
- User-friendly error messages at UI layer
- Technical error logging at database/service layers
- Simple to implement and maintain
- Follows Python best practices

**Alternatives Considered**:
- **Custom exception hierarchy**: Over-engineered for application scope, adds complexity
- **Result types**: Non-Pythonic pattern, better suited for functional languages
- **Error codes**: Less clear than exception messages, harder to debug

**Implementation Notes**:
- Database layer: Catch psycopg2 exceptions, raise with context
- Service layer: Catch database errors, add business context
- Menu layer: Catch all errors, display user-friendly messages
- Log technical details for debugging

---

### 5. Connection Pooling Strategy

**Decision**: Single persistent connection per application session

**Rationale**:
- Single-user local application (no concurrency)
- Simple connection lifecycle (open on start, close on exit)
- Neon Serverless PostgreSQL handles server-side pooling
- Reduces complexity vs connection pool library
- Sufficient for expected load (< 10 ops/minute)

**Alternatives Considered**:
- **psycopg2.pool.SimpleConnectionPool**: Unnecessary overhead for single user
- **Connection per operation**: Higher latency, connection overhead
- **SQLAlchemy connection pool**: Too heavy for simple use case

**Implementation Notes**:
- Open connection in `main()` at startup
- Pass connection to service layer functions
- Close connection in finally block at exit
- Handle connection errors gracefully

---

## Technology Stack Summary

| Component | Technology | Version | Purpose | Justification |
|-----------|-----------|---------|---------|---------------|
| Language | Python | 3.11+ | Application runtime | Modern Python features (type hints, match), wide adoption |
| Database | Neon PostgreSQL | Serverless | Task persistence | Serverless simplicity, PostgreSQL compatibility |
| DB Driver | psycopg2-binary | Latest | Database connection | Official adapter, stable, Neon compatible |
| Config | python-dotenv | Latest | Environment vars | Industry standard, simple `.env` support |
| Terminal | Built-in input() | Stdlib | User interaction | Cross-platform, no dependencies |

---

## Architecture Decisions

### Layer Separation Enforcement

**Decision**: File-level agent ownership with strict interface contracts

**Rationale**:
- Constitution mandates separation of concerns
- Each Python file owned by single agent
- Clear contracts prevent layer mixing
- Spec Guardian can review file boundaries

**Implementation**:
- `src/db.py`: Database Agent only
- `src/models.py`, `src/services.py`: Service Logic Agent only
- `src/menu.py`, `src/main.py`: Menu/UX Agent only

### Data Flow Pattern

**Decision**: Unidirectional data flow (Menu → Service → Database)

**Rationale**:
- Prevents circular dependencies
- Clear ownership of each operation
- Easy to trace bugs through layers
- Matches agent workflow (UX → Logic → Storage)

**Implementation**:
- Menu layer calls service functions
- Service layer calls database functions
- Database layer returns raw data
- Service layer transforms to models
- Menu layer displays models

### Error Propagation Pattern

**Decision**: Bubble up with context, handle at UI layer

**Rationale**:
- Technical errors logged at origin
- Business errors added at service layer
- User-friendly messages at menu layer
- Full error context preserved for debugging

**Implementation**:
- Database: Raise with SQL context
- Service: Add business rule context
- Menu: Display user-friendly message

---

## Dependencies Analysis

### Required Dependencies

```txt
psycopg2-binary>=2.9.0  # Neon PostgreSQL driver
python-dotenv>=1.0.0     # Environment configuration
```

### Development Dependencies (Optional)

```txt
pytest>=7.0.0           # Testing framework (if tests requested)
black>=23.0.0           # Code formatting
mypy>=1.0.0            # Type checking
```

### Dependency Justification

- **psycopg2-binary**: Required for Neon connection
- **python-dotenv**: Required for secure credential management
- **Testing/quality tools**: Optional per specification (no testing requirement)

---

## Research Conclusions

All technology decisions finalized and justified. No unresolved NEEDS CLARIFICATION items.

**Key Findings**:
1. Python 3.11+ with psycopg2-binary provides complete Neon integration
2. Terminal menu pattern sufficient for user interaction requirements
3. Single persistent connection adequate for single-user scope
4. Layer separation enforceable through file ownership
5. No external dependencies beyond database driver and config loader

**Constitution Compliance**: ✅ All decisions respect agent boundaries and mandatory workflow

**Next Phase**: Proceed to detailed design (data model and contracts)
