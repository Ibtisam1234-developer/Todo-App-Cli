---
id: 001
title: Create Project Constitution
stage: constitution
date: 2025-12-17
surface: agent
model: claude-sonnet-4-5
feature: none
branch: master
user: Ibtisam1234-developer
command: /sp.constitution
labels: ["constitution", "sub-agent-system", "orchestration", "neon-db", "spec-driven"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - .specify/memory/constitution.md
tests:
 - N/A (documentation only)
---

## Prompt

# SP Constitution – Interactive Todo Application (Neon DB)

## 1. Objective
Build an interactive, terminal-based Todo application that accepts user input via prompts and persists tasks using Neon Serverless PostgreSQL, developed using Claude Code and Spec-Kit Plus with controlled sub-agent orchestration.

## 2. Scope
The system is a local, interactive CLI application.
All business logic, database access, and user interaction are handled through structured Python modules.
All Python code resides strictly in `/src`.

## 3. Development Methodology
This project MUST follow **spec-driven development** using **Spec-Kit Plus** and **Claude Code**.

Work is executed through **specialized sub-agents** operating under strict authority boundaries and an enforced execution order.

No agent may violate this constitution.

---

## 4. Sub-Agent System (Authoritative)

### 4.1 Spec Guardian Agent (PRIMARY AUTHORITY)

**Role**
- Enforce compliance with:
  - This constitution
  - `sp.specify`
  - `sp.plan`

**Responsibilities**
- Review outputs from all other agents
- Block any violation of scope or constraints
- Approve progression between phases

**Restrictions**
- Cannot write production code
- Cannot introduce features

---

### 4.2 Database Agent (Neon DB)

**Role**
Own all database-related concerns.

**Responsibilities**
- Define SQL schema
- Implement Neon Serverless PostgreSQL connection
- Ensure safe query execution
- Handle database errors

**Allowed Scope**
- `db.py`
- SQL schema documentation

**Forbidden**
- User input handling
- Business logic
- Menu or UI code

---

### 4.3 Service Logic Agent

**Role**
Implement application business logic.

**Responsibilities**
- CRUD operations for tasks
- Completion state toggling
- Validation of task existence

**Allowed Scope**
- `services.py`
- `models.py`

**Forbidden**
- Direct database connections
- User input (`input()`)
- Output rendering (`print()`)

---

### 4.4 Menu / UX Agent

**Role**
Manage interactive user experience.

**Responsibilities**
- Display menus
- Collect user input
- Validate selections
- Render output messages

**Allowed Scope**
- `menu.py`
- `main.py`

**Forbidden**
- SQL
- Business logic
- Schema changes

---

### 4.5 Documentation Agent

**Role**
Maintain project documentation.

**Responsibilities**
- README.md
- CLAUDE.md
- specs/history updates

**Forbidden**
- Code changes
- Spec changes

---

## 5. Execution Flow (MANDATORY)

Work MUST proceed in the following order:

1. **Spec Guardian Agent**
   - Validates constitution, specification, and plan
   - Approves start of development

2. **Database Agent**
   - Creates schema
   - Implements DB connection layer
   - Submits output for review

3. **Spec Guardian Agent**
   - Reviews database implementation
   - Approves or rejects

4. **Service Logic Agent**
   - Implements business logic
   - Submits output for review

5. **Spec Guardian Agent**
   - Reviews service layer
   - Approves or rejects

6. **Menu / UX Agent**
   - Implements interactive user input flow
   - Submits output for review

7. **Spec Guardian Agent**
   - Performs final compliance audit

8. **Documentation Agent**
   - Produces README and CLAUDE instructions

---

## 6. Enforcement Rules
- No agent may skip review
- No agent may operate outside its assigned scope
- Violations require correction before proceeding

---

## 7. Change Control
Any of the following requires updating this constitution first:
- Adding or removing sub-agents
- Changing execution order
- Modifying storage or interaction model

---

## 8. Non-Goals
- Autonomous agent decision-making
- Feature expansion beyond specification
- Runtime agent behavior

---

## 9. Authority
This constitution is the highest authority in the project.
All specifications, plans, and agent outputs must comply with it.

## Response snapshot

Constitution successfully created at `.specify/memory/constitution.md` with version 1.0.0.

The constitution establishes:

**Core Principles (6 NON-NEGOTIABLE)**:
1. Separation of Concerns - Each agent has defined boundaries
2. Orchestrator-Controlled Workflow - Mandatory execution order
3. Guardian Gate System - Review gates between all phases
4. Code Location Discipline - All Python in `/src`
5. No Interaction Mixing - Strict layer separation
6. Neon DB Integration Standard - Serverless PostgreSQL requirements

**Sub-Agent System (5 agents)**:
- Spec Guardian Agent (PRIMARY AUTHORITY) - Enforces compliance
- Database Agent - Owns `src/db.py` and schema
- Service Logic Agent - Owns `src/services.py` and `src/models.py`
- Menu/UX Agent - Owns `src/menu.py` and `src/main.py`
- Documentation Agent - Owns docs only

**Execution Flow (8 mandatory steps)**:
Guardian validation → Database implementation → Guardian review → Service implementation → Guardian review → Menu implementation → Guardian audit → Documentation

**Enforcement Rules**:
- No skip review
- No scope violations
- No cross-agent work
- Correction protocol for rejections
- Orchestrator authority is final

## Outcome

- ✅ Impact: Established authoritative constitution defining sub-agent orchestration model for todo app development
- 🧪 Tests: N/A (documentation artifact)
- 📁 Files: Created `.specify/memory/constitution.md` (v1.0.0, 384 lines)
- 🔁 Next prompts: `/sp.specify` to create feature specification, followed by `/sp.plan` for implementation plan
- 🧠 Reflection: Constitution provides clear authority hierarchy, scope boundaries, and enforcement mechanisms for controlled agent coordination

## Evaluation notes (flywheel)

- Failure modes observed: None (initial creation)
- Graders run and results (PASS/FAIL): Manual validation PASS (no placeholders, dates ISO format, declarative principles)
- Prompt variant (if applicable): N/A
- Next experiment (smallest change to try): Monitor Guardian agent enforcement in practice during implementation phases
