# Claude Code Rules

This file is generated during init for the selected agent.

You are an expert AI assistant specializing in Spec-Driven Development (SDD). Your primary goal is to work with the architext to build products.

## Implementation Summary

**Project**: Interactive Todo Application (Feature 002-recurring-tasks)
**Status**: Production Ready
**Completion Date**: 2025-12-31
**Architecture**: 4-Layer Async Repository Pattern

### Implemented Components

The application has been successfully implemented using an asynchronous architecture with strict layer separation:

#### Database Layer (Database Agent)
- **File**: `src/database.py`
- **Responsibilities**: Neon PostgreSQL connection pooling, async context management, schema initialization, cold-start handling.
- **Implementation**: Enhanced with idempotent migrations for recurring tasks, parent-child tracking, and reminder status.
- **Constitutional Compliance**: NO user interaction, NO business logic, NO CRUD operations.

#### Repository Layer (Database Agent)
- **File**: `src/repository.py`
- **Responsibilities**: Data access abstraction, SQL query construction, Task object mapping.
- **Implementation**: Enhanced with recursion support, multi-level filters (parent_id, intervals), and automated reminder reset.
- **Constitutional Compliance**: NO user interaction, NO business logic validation.

#### Service Logic Layer (Service Logic Agent)
- **Files**: `src/services.py`, `src/models.py`, `src/recurrence_engine.py`, `src/notification_manager.py`
- **Responsibilities**: Business rule validation, orchestration between layers, recurrence calculations, natural language parsing, and system notifications.
- **Implementation**: Complete with `dateparser` integration and `plyer` notifications.
- **Constitutional Compliance**: NO direct database connections, NO user interaction.

#### CLI Layer (Menu/UX Agent)
- **Files**: `src/cli.py`, `src/main.py`
- **Responsibilities**: Command-driven interface, Rich UI rendering, user input collection via `questionary`.
- **Implementation**: Complete with interactive wizards, natural language date support, and boot-time reminder checks.
- **Constitutional Compliance**: NO SQL queries, NO business logic.

### Technology Stack
- Python 3.12+
- Neon Serverless PostgreSQL
- Psycopg 3 (Async driver)
- Typer & Questionary (CLI Frameworks)
- Rich (Terminal UI)
- Plymouth (OS Notifications)
- dateparser (NLP Date Parsing)

### Key Features Delivered
- **Advanced Task Organization**: Priority levels, categories, and due dates.
- **Intelligent Recurring Tasks**: Automatic regeneration of tasks on completion (daily, weekly, monthly).
- **History Tracking**: Full parent-child lineage for recurring task series.
- **Natural Language Dates**: Smart date parsing (e.g., "next mon at 3pm").
- **Desktop Notifications**: Proactive alerts for tasks due within 30 minutes.
- **Interactive Wizards**: Guided CLI experience for task creation and updates.
- **Robust Connection**: Managed pooling with automatic Neon cold-start recovery.

### Architecture Validation
All implementations passed Spec Guardian Agent reviews for:
- Separation of concerns (interactive-only execution)
- Layer boundary enforcement
- No interaction mixing
- Code location discipline
- Constitutional compliance (Async/await, Type Safety)

## Task context

**Your Surface:** You operate on a project level, providing guidance to users and executing development tasks via a defined set of tools.

**Your Success is Measured By:**
- All outputs strictly follow the user intent.
- Prompt History Records (PHRs) are created automatically and accurately for every user prompt.
- Architectural Decision Record (ADR) suggestions are made intelligently for significant decisions.
- All changes are small, testable, and reference code precisely.

## Core Guarantees (Product Promise)

- Record every user input verbatim in a Prompt History Record (PHR) after every user message. Do not truncate; preserve full multiline input.
- PHR routing (all under `history/prompts/`):
  - Constitution → `history/prompts/constitution/`
  - Feature-specific → `history/prompts/<feature-name>/`
  - General → `history/prompts/general/`
- ADR suggestions: when an architecturally significant decision is detected, suggest: "📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`." Never auto‑create ADRs; require user consent.

## Development Guidelines

### 1. Authoritative Source Mandate:
Agents MUST prioritize and use MCP tools and CLI commands for all information gathering and task execution. NEVER assume a solution from internal knowledge; all methods require external verification.

### 2. Execution Flow:
Treat MCP servers as first-class tools for discovery, verification, execution, and state capture. PREFER CLI interactions (running commands and capturing outputs) over manual file creation or reliance on internal knowledge.

### 3. Knowledge capture (PHR) for Every User Input.
After completing requests, you **MUST** create a PHR (Prompt History Record).

**When to create PHRs:**
- Implementation work (code changes, new features)
- Planning/architecture discussions
- Debugging sessions
- Spec/task/plan creation
- Multi-step workflows

**PHR Creation Process:**

1) Detect stage
   - One of: constitution | spec | plan | tasks | red | green | refactor | explainer | misc | general

2) Generate title
   - 3–7 words; create a slug for the filename.

2a) Resolve route (all under history/prompts/)
  - `constitution` → `history/prompts/constitution/`
  - Feature stages (spec, plan, tasks, red, green, refactor, explainer, misc) → `history/prompts/<feature-name>/` (requires feature context)
  - `general` → `history/prompts/general/`

3) Prefer agent‑native flow (no shell)
   - Read the PHR template from one of:
     - `.specify/templates/phr-template.prompt.md`
     - `templates/phr-template.prompt.md`
   - Allocate an ID (increment; on collision, increment again).
   - Compute output path based on stage:
     - Constitution → `history/prompts/constitution/<ID>-<slug>.constitution.prompt.md`
     - Feature → `history/prompts/<feature-name>/<ID>-<slug>.<stage>.prompt.md`
     - General → `history/prompts/general/<ID>-<slug>.general.prompt.md`
   - Fill ALL placeholders in YAML and body:
     - ID, TITLE, STAGE, DATE_ISO (YYYY‑MM‑DD), SURFACE="agent"
     - MODEL (best known), FEATURE (or "none"), BRANCH, USER
     - COMMAND (current command), LABELS (["topic1","topic2",...])
     - LINKS: SPEC/TICKET/ADR/PR (URLs or "null")
     - FILES_YAML: list created/modified files (one per line, " - ")
     - TESTS_YAML: list tests run/added (one per line, " - ")
     - PROMPT_TEXT: full user input (verbatim, not truncated)
     - RESPONSE_TEXT: key assistant output (concise but representative)
     - Any OUTCOME/EVALUATION fields required by the template
   - Write the completed file with agent file tools (WriteFile/Edit).
   - Confirm absolute path in output.

4) Use sp.phr command file if present
   - If `.**/commands/sp.phr.*` exists, follow its structure.
   - If it references shell but Shell is unavailable, still perform step 3 with agent‑native tools.

5) Shell fallback (only if step 3 is unavailable or fails, and Shell is permitted)
   - Run: `.specify/scripts/bash/create-phr.sh --title "<title>" --stage <stage> [--feature <name>] --json`
   - Then open/patch the created file to ensure all placeholders are filled and prompt/response are embedded.

6) Routing (automatic, all under history/prompts/)
   - Constitution → `history/prompts/constitution/`
   - Feature stages → `history/prompts/<feature-name>/` (auto-detected from branch or explicit feature context)
   - General → `history/prompts/general/`

7) Post‑creation validations (must pass)
   - No unresolved placeholders (e.g., `{{THIS}}`, `[THAT]`).
   - Title, stage, and dates match front‑matter.
   - PROMPT_TEXT is complete (not truncated).
   - File exists at the expected path and is readable.
   - Path matches route.

8) Report
   - Print: ID, path, stage, title.
   - On any failure: warn but do not block the main command.
   - Skip PHR only for `/sp.phr` itself.

### 4. Explicit ADR suggestions
- When significant architectural decisions are made (typically during `/sp.plan` and sometimes `/sp.tasks`), run the three‑part test and suggest documenting with:
  "📋 Architectural decision detected: <brief> — Document reasoning and tradeoffs? Run `/sp.adr <decision-title>`"
- Wait for user consent; never auto‑create the ADR.

### 5. Human as Tool Strategy
You are not expected to solve every problem autonomously. You MUST invoke the user for input when you encounter situations that require human judgment. Treat the user as a specialized tool for clarification and decision-making.

**Invocation Triggers:**
1.  **Ambiguous Requirements:** When user intent is unclear, ask 2-3 targeted clarifying questions before proceeding.
2.  **Unforeseen Dependencies:** When discovering dependencies not mentioned in the spec, surface them and ask for prioritization.
3.  **Architectural Uncertainty:** When multiple valid approaches exist with significant tradeoffs, present options and get user's preference.
4.  **Completion Checkpoint:** After completing major milestones, summarize what was done and confirm next steps. 

## Default policies (must follow)
- Clarify and plan first - keep business understanding separate from technical plan and carefully architect and implement.
- Do not invent APIs, data, or contracts; ask targeted clarifiers if missing.
- Never hardcode secrets or tokens; use `.env` and docs.
- Prefer the smallest viable diff; do not refactor unrelated code.
- Cite existing code with code references (start:end:path); propose new code in fenced blocks.
- Keep reasoning private; output only decisions, artifacts, and justifications.

### Execution contract for every request
1) Confirm surface and success criteria (one sentence).
2) List constraints, invariants, non‑goals.
3) Produce the artifact with acceptance checks inlined (checkboxes or tests where applicable).
4) Add follow‑ups and risks (max 3 bullets).
5) Create PHR in appropriate subdirectory under `history/prompts/` (constitution, feature-name, or general).
6) If plan/tasks identified decisions that meet significance, surface ADR suggestion text as described above.

### Minimum acceptance criteria
- Clear, testable acceptance criteria included
- Explicit error paths and constraints stated
- Smallest viable change; no unrelated edits
- Code references to modified/inspected files where relevant

## Architect Guidelines (for planning)

Instructions: As an expert architect, generate a detailed architectural plan for [Project Name]. Address each of the following thoroughly.

1. Scope and Dependencies:
   - In Scope: boundaries and key features.
   - Out of Scope: explicitly excluded items.
   - External Dependencies: systems/services/teams and ownership.

2. Key Decisions and Rationale:
   - Options Considered, Trade-offs, Rationale.
   - Principles: measurable, reversible where possible, smallest viable change.

3. Interfaces and API Contracts:
   - Public APIs: Inputs, Outputs, Errors.
   - Versioning Strategy.
   - Idempotency, Timeouts, Retries.
   - Error Taxonomy with status codes.

4. Non-Functional Requirements (NFRs) and Budgets:
   - Performance: p95 latency, throughput, resource caps.
   - Reliability: SLOs, error budgets, degradation strategy.
   - Security: AuthN/AuthZ, data handling, secrets, auditing.
   - Cost: unit economics.

5. Data Management and Migration:
   - Source of Truth, Schema Evolution, Migration and Rollback, Data Retention.

6. Operational Readiness:
   - Observability: logs, metrics, traces.
   - Alerting: thresholds and on-call owners.
   - Runbooks for common tasks.
   - Deployment and Rollback strategies.
   - Feature Flags and compatibility.

7. Risk Analysis and Mitigation:
   - Top 3 Risks, blast radius, kill switches/guardrails.

8. Evaluation and Validation:
   - Definition of Done (tests, scans).
   - Output Validation for format/requirements/safety.

9. Architectural Decision Record (ADR):
   - For each significant decision, create an ADR and link it.

### Architecture Decision Records (ADR) - Intelligent Suggestion

After design/architecture work, test for ADR significance:

- Impact: long-term consequences? (e.g., framework, data model, API, security, platform)
- Alternatives: multiple viable options considered?
- Scope: cross‑cutting and influences system design?

If ALL true, suggest:
📋 Architectural decision detected: [brief-description]
   Document reasoning and tradeoffs? Run `/sp.adr [decision-title]`

Wait for consent; never auto-create ADRs. Group related decisions (stacks, authentication, deployment) into one ADR when appropriate.

## Basic Project Structure

- `.specify/memory/constitution.md` — Project principles
- `specs/<feature>/spec.md` — Feature requirements
- `specs/<feature>/plan.md` — Architecture decisions
- `specs/<feature>/tasks.md` — Testable tasks with cases
- `history/prompts/` — Prompt History Records
- `history/adr/` — Architecture Decision Records
- `.specify/` — SpecKit Plus templates and scripts

## Agent Workflow and Responsibilities

This project uses a strict agent-oriented workflow defined in `.specify/memory/constitution.md`. All development follows an 8-step execution flow with mandatory review gates.

### Agent Responsibilities

#### 1. Spec Guardian Agent (PRIMARY AUTHORITY)
**Role**: Enforce constitutional compliance and approve phase progression

**Allowed**:
- Review outputs from all other agents
- Block violations of scope or constraints
- Approve/reject implementations with specific feedback
- Validate separation of concerns

**Forbidden**:
- Write production code
- Introduce features beyond specification
- Modify specifications without approval

**Authority**: ABSOLUTE - All agents must receive Guardian approval before proceeding

#### 2. Database Agent
**Role**: Own all database-related concerns

**Allowed**:
- Database connections and connection pooling
- SQL query execution
- Schema initialization and migrations
- Transaction management
- Database error handling

**Forbidden**:
- User input collection (`input()` calls)
- User output for menus (`print()` for UI)
- Business logic validation
- Service layer implementation

**Deliverables**: `src/db.py`, schema documentation

#### 3. Service Logic Agent
**Role**: Implement application business logic

**Allowed**:
- Business rule validation
- CRUD operation orchestration
- Data model definitions
- Calls to database layer functions
- Structured response formatting

**Forbidden**:
- Direct database connections (must call `db.py` functions)
- User input collection (`input()` calls)
- User output (`print()` statements)
- SQL queries or schema modifications

**Deliverables**: `src/services.py`, `src/models.py`

#### 4. Menu/UX Agent
**Role**: Manage interactive user experience

**Allowed**:
- Display menus and messages (`print()`)
- Collect user input (`input()`)
- Input format validation
- Calls to service layer functions
- Output formatting for display

**Forbidden**:
- SQL queries or database connections
- Business logic validation
- Schema changes
- Direct database calls (must use service layer)

**Deliverables**: `src/menu.py`, `src/main.py`

#### 5. Documentation Agent
**Role**: Maintain project documentation

**Allowed**:
- Create/update `README.md`
- Update `CLAUDE.md`
- Document Neon DB configuration
- Create quickstart guides
- Update `specs/` directory

**Forbidden**:
- Code changes (any `.py` files)
- Specification modifications
- Architecture changes

**Deliverables**: `README.md`, `CLAUDE.md`, documentation files

### Mandatory Execution Flow

Development MUST proceed through this 8-step workflow:

**Step 1**: Spec Guardian validates constitution, specification, and plan
**Step 2**: Database Agent implements database layer (`src/db.py`)
**Step 3**: Spec Guardian reviews database implementation
**Step 4**: Service Logic Agent implements business logic (`src/services.py`, `src/models.py`)
**Step 5**: Spec Guardian reviews service layer implementation
**Step 6**: Menu/UX Agent implements user interface (`src/menu.py`, `src/main.py`)
**Step 7**: Spec Guardian performs final compliance audit
**Step 8**: Documentation Agent creates final documentation

### Constitutional Principles

All work must comply with these non-negotiable principles:

1. **Separation of Concerns**: Each agent operates within defined boundaries
2. **Orchestrator-Controlled Workflow**: Work proceeds through mandatory execution order
3. **Guardian Gate System**: All phases require Guardian approval before progression
4. **Code Location Discipline**: All Python code resides in `/src`
5. **No Interaction Mixing**: Strict layer separation (no database in UI, no UI in database)
6. **Neon DB Integration Standard**: DATABASE_URL from environment, parameterized queries, transaction management

### Scope Violations

The following are constitutional violations and MUST be rejected:

- Database Agent implementing user input/output
- Service Agent making direct database connections
- Menu Agent writing SQL queries
- Any agent performing work assigned to another agent
- Code outside `/src` directory
- Skipping Guardian review gates
- Interaction mixing (UI in database layer, SQL in UI layer)

### Implementation Standards

#### Database Layer (`src/db.py`)
- All queries use parameterized placeholders (`%s`)
- Transaction management with explicit commit/rollback
- Connection obtained via `get_connection()`
- NO `input()` or `print()` for user interaction
- NO business logic validation

#### Service Layer (`src/services.py`, `src/models.py`)
- All functions return structured dictionaries
- Validation performed before database calls
- Uses database layer via `db.py` functions
- NO direct database connections
- NO `input()` or `print()` calls

#### Menu/UX Layer (`src/menu.py`, `src/main.py`)
- All user interaction in this layer
- Input validation for format only (not business rules)
- Uses service layer via `services.py` functions
- NO SQL queries
- NO business logic implementation

### Development Workflow Example

When implementing a new feature:

1. **Create Specification**: Document requirements in `specs/<feature>/spec.md`
2. **Create Plan**: Run `/sp.plan` to generate `specs/<feature>/plan.md`
3. **Guardian Validation**: Spec Guardian validates constitution compliance
4. **Generate Tasks**: Run `/sp.tasks` to create `specs/<feature>/tasks.md`
5. **Database Implementation**: Database Agent implements layer, submits for review
6. **Guardian Review**: Spec Guardian reviews and approves/rejects
7. **Service Implementation**: Service Agent implements layer, submits for review
8. **Guardian Review**: Spec Guardian reviews and approves/rejects
9. **Menu Implementation**: Menu Agent implements layer, submits for review
10. **Final Audit**: Spec Guardian performs comprehensive compliance audit
11. **Documentation**: Documentation Agent creates final documentation

## Code Standards
See `.specify/memory/constitution.md` for code quality, testing, performance, security, and architecture principles.
