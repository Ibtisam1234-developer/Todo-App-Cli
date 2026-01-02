# Feature Specification: Task Organization & Usability

**Feature Branch**: `001-task-organization`
**Created**: 2025-12-30
**Status**: Draft
**Input**: Target: Intermediate Todo CLI (Organization & Usability)
Features:
1. Schema Enhancement: Update 'tasks' table in Neon to include priority, category, and timestamps
2. Priorities & Tags: Allow users to assign priority and category during add and update
3. Search & Filter: Command to search tasks, list with filtering flags
4. Sorting: Add sorting options to list command
5. UI: Use Rich library with color-coded priority display

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Enhanced Task Creation (Priority: P1)

As a user managing tasks, I want to add tasks with priority levels, categories, and due dates so that I can better organize and track what needs attention.

**Why this priority**: Core enhancement enabling all other organization features; without structured task data, filtering and sorting cannot work effectively.

**Independent Test**: Can create a task with title "Complete project report", priority "high", category "work", and due date "2025-01-15". The task persists with all attributes and can be viewed in a color-coded table.

**Acceptance Scenarios**:

1. **Given** user runs add command with all fields, **When** title, description, priority, category, and due date are provided, **Then** task is created with all attributes and displayed
2. **Given** user provides only title, **When** other fields are omitted, **Then** task is created with default priority (medium), no category, and no due date
3. **Given** user provides invalid priority value, **When** add command is executed, **Then** clear error message shows valid options (low, medium, high) and task is not created
4. **Given** user provides past due date, **When** add command is executed, **Then** task is created (users may need to record overdue tasks)

---

### User Story 2 - Task Filtering & Search (Priority: P1)

As a user managing many tasks, I want to filter and search tasks so that I can quickly find relevant items without scanning the entire list.

**Why this priority**: Directly addresses usability pain point of managing growing task lists; high user value for organization.

**Independent Test**: Can create 10 tasks with varying priorities and categories, then filter by "work" category and "high" priority to see only matching results. Can also search for keyword "project" to find all related tasks.

**Acceptance Scenarios**:

1. **Given** multiple tasks exist with different categories, **When** user runs list with `--category work`, **Then** only work category tasks are displayed
2. **Given** multiple tasks exist with different statuses, **When** user runs list with `--status pending`, **Then** only incomplete tasks are displayed
3. **Given** multiple tasks exist with different priorities, **When** user runs list with `--priority high`, **Then** only high priority tasks are displayed
4. **Given** multiple tasks exist, **When** user runs search with keyword "meeting", **Then** all tasks containing "meeting" in title or description are displayed
5. **Given** user combines multiple filters, **When** list runs with `--category work --priority high --status pending`, **Then** only tasks matching all criteria are displayed

---

### User Story 3 - Task Sorting (Priority: P2)

As a user reviewing my task list, I want to sort tasks by different criteria so that I can prioritize my work in the way that makes sense for my current needs.

**Why this priority**: Improves usability but depends on Story 1 (structured data); secondary to filtering/search which are more commonly needed.

**Independent Test**: Can create tasks with different priorities and dates, then sort by priority to see high-priority items first, or sort by date to see upcoming deadlines.

**Acceptance Scenarios**:

1. **Given** tasks exist with various priorities, **When** user runs list with `--sort priority`, **Then** tasks display in order: high → medium → low
2. **Given** tasks exist with various due dates, **When** user runs list with `--sort date`, **Then** tasks display chronologically (earliest due date first)
3. **Given** tasks exist with various titles, **When** user runs list with `--sort alpha`, **Then** tasks display alphabetically by title
4. **Given** user combines sort with filters, **When** list runs with `--category work --sort priority`, **Then** filtered work tasks display sorted by priority

---

### User Story 4 - Rich UI with Color Coding (Priority: P2)

As a user viewing my task list, I want visual indicators of task importance so that I can quickly identify critical items.

**Why this priority**: Improves UX but does not add new functionality; nice-to-have enhancement that builds on other features.

**Independent Test**: Can display a list of tasks and see high-priority tasks in red, medium in yellow, and low in green. Table formatting is clean and readable with columns for all attributes.

**Acceptance Scenarios**:

1. **Given** tasks exist with different priorities, **When** user runs list command, **Then** tasks display in a table with color-coded priority columns
2. **Given** task has high priority, **When** displayed in list, **Then** priority cell shows in red
3. **Given** task has low priority, **When** displayed in list, **Then** priority cell shows in green
4. **Given** task has medium priority, **When** displayed in list, **Then** priority cell shows in yellow
5. **Given** no tasks match filters, **When** list command runs, **Then** appropriate message displays (not empty table)

---

### Edge Cases

- What happens when user provides invalid priority (e.g., "urgent" instead of "high")? → Show clear error with valid options
- What happens when filtering returns no results? → Display message "No tasks found matching criteria"
- What happens when due date is in past? → Allow creation (users may need to track overdue items)
- What happens when category has special characters or spaces? → Store as-is, handle in search/filter
- What happens when search term is empty? → Show all tasks or error requiring non-empty term
- What happens when multiple sort flags are provided? → Use the last one specified or show error

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST store tasks with priority field supporting values: low, medium, high
- **FR-002**: System MUST store tasks with category field as free-form text
- **FR-003**: System MUST store tasks with created_at timestamp (auto-generated on creation)
- **FR-004**: System MUST store tasks with due_date timestamp (optional, user-provided)
- **FR-005**: Users MUST be able to specify priority when creating or updating tasks (default: medium)
- **FR-006**: Users MUST be able to specify category when creating or updating tasks (optional)
- **FR-007**: Users MUST be able to specify due date when creating or updating tasks (optional)
- **FR-008**: System MUST provide list command with optional `--priority` flag (values: low, medium, high)
- **FR-009**: System MUST provide list command with optional `--category` flag (filter by category name)
- **FR-010**: System MUST provide list command with optional `--status` flag (values: pending, completed, all)
- **FR-011**: System MUST provide search command accepting keyword argument
- **FR-012**: Search MUST match keyword against both task title and description fields
- **FR-013**: System MUST provide list command with optional `--sort` flag (values: date, priority, alpha)
- **FR-014**: When sorting by priority, order MUST be: high → medium → low
- **FR-015**: When sorting by date, tasks without due dates MUST appear at the end (or beginning - to be decided)
- **FR-016**: System MUST display task lists using Rich library table format
- **FR-017**: High priority tasks MUST display with red color indication
- **FR-018**: Medium priority tasks MUST display with yellow color indication
- **FR-019**: Low priority tasks MUST display with green color indication
- **FR-020**: Filter flags MUST be composable (can combine multiple filters)
- **FR-021**: When filtering returns no results, system MUST display user-friendly message

### Key Entities

- **Task**: Represents a to-do item with attributes: title (required), description (optional), priority (low/medium/high, default: medium), category (text, optional), status (pending/completed, default: pending), created_at (timestamp, auto), due_date (timestamp, optional)
- **Category**: A text label grouping related tasks (free-form, not enforced schema entity in this phase - stored as text field on tasks)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can create a task with all attributes (priority, category, due date) in under 30 seconds
- **SC-002**: Filtered task lists display within 1 second regardless of total task volume
- **SC-003**: Search operation returns results for 50+ task database in under 1 second
- **SC-004**: 100% of priority values are color-coded correctly in task listings
- **SC-005**: Users can locate specific tasks using search or filters in under 5 seconds
- **SC-006**: Task listing shows all relevant attributes (title, priority, category, status, dates) in a single view

## Assumptions

- Priority values are limited to three levels (low, medium, high) as specified in input
- Category is a free-form text field; category management (create/edit/delete categories) is not in scope
- Due dates are optional; tasks can exist without due dates
- Date format for input will be standard ISO or human-readable format (implementation decision)
- Sort by date refers to due_date, not created_at (assumed from user priority context)
- "status" field already exists (pending/completed); this feature adds filtering capability, not the field itself
