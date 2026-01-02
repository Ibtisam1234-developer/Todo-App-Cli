# Feature Specification: Advanced Todo CLI - Recurring Tasks & Smart Features

**Feature Branch**: `002-recurring-tasks`
**Created**: 2025-12-31
**Status**: Draft
**Input**: User description: "Target: Advanced Todo CLI (Strict CLI Intelligence)
Features:
1. Recurring Logic:
   - Add a 'recurrence' field (daily, weekly, monthly, none).
   - Logic: When a task with a recurrence interval is marked 'completed', the app must automatically calculate the next due date and create a new pending task in Neon.
2. Smart Input & Date Picking:
   - Use 'dateparser' to allow natural language inputs (e.g., "next mon at 3pm").
   - Use 'questionary' to create an interactive CLI wizard for selecting dates/times if the user doesn't provide them via flags.
3. Desktop Notifications:
   - Integrate 'plyer' for cross-platform desktop notifications.
   - The CLI should check for tasks due within the next 30 minutes every time it is launched and trigger a system notification.
4. Advanced State Management:
   - Track 'parent_task_id' for recurring tasks to maintain a history of completed instances in the database."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Recurring Task Management (Priority: P1)

As a user, I want to create tasks that repeat on a regular schedule (daily, weekly, or monthly) so that I don't have to manually recreate the same task every time I complete it.

**Why this priority**: This is the core feature that provides the most value. Users frequently have recurring tasks (daily stand-ups, weekly reports, monthly reviews) and automating their creation saves significant time and reduces the risk of forgetting them.

**Independent Test**: Can be fully tested by creating a task with a recurrence interval, marking it complete, and verifying a new task is automatically created with the correct next due date. This delivers immediate value to users with repetitive tasks.

**Acceptance Scenarios**:

1. **Given** a user creates a task with recurrence set to "daily" and due date of "2025-01-01", **When** the user marks that task as completed, **Then** the system automatically creates a new pending task with the same title, description, priority, and category, with a due date of "2025-01-02"
2. **Given** a user creates a task with recurrence set to "weekly" and due date of "2025-01-06 (Monday)", **When** the user marks that task as completed, **Then** the system creates a new pending task with due date of "2025-01-13 (Monday)"
3. **Given** a user creates a task with recurrence set to "monthly" and due date of "2025-01-15", **When** the user marks that task as completed, **Then** the system creates a new pending task with due date of "2025-02-15"
4. **Given** a user creates a task with recurrence set to "none", **When** the user marks that task as completed, **Then** no new task is created
5. **Given** a user creates a recurring task without a due date, **When** the user marks that task as completed, **Then** the system creates a new pending task with the same attributes but no due date (recurrence interval cannot be applied meaningfully)
6. **Given** a user creates a recurring task and marks it complete, **When** the system creates the next instance, **Then** the original task retains its completed status and timestamps for history tracking

---

### User Story 2 - Smart Date Input (Priority: P2)

As a user, I want to enter dates and times using natural language (like "next Monday at 3pm" or "tomorrow morning") so that I can quickly create tasks without memorizing date formats or calculating dates manually.

**Why this priority**: Natural language date input significantly improves user experience and reduces friction when creating tasks. Users think in relative terms ("tomorrow", "next week") rather than absolute dates. This is important for adoption but not critical for core functionality.

**Independent Test**: Can be fully tested by creating tasks using various natural language date expressions and verifying the system correctly interprets them as the intended dates. Delivers immediate value by speeding up task creation.

**Acceptance Scenarios**:

1. **Given** a user creates a task with due date "tomorrow", **When** the system parses the input, **Then** the task's due date is set to the next calendar day at midnight
2. **Given** a user creates a task with due date "next Monday at 3pm", **When** the system parses the input, **Then** the task's due date is set to the next Monday at 15:00:00
3. **Given** a user creates a task with due date "in 2 weeks", **When** the system parses the input, **Then** the task's due date is set to exactly 14 days from the current date
4. **Given** a user creates a task with due date "Dec 31", **When** the system parses the input, **Then** the task's due date is set to December 31st of the current year
5. **Given** a user enters an ambiguous date expression that the system cannot parse, **When** the system attempts to interpret it, **Then** the system displays a helpful error message suggesting valid formats or offers an interactive date picker
6. **Given** a user creates a task without specifying a due date, **When** the system processes the task, **Then** the task is created without a due date (optional field)

---

### User Story 3 - Desktop Notifications (Priority: P3)

As a user, I want to receive system notifications when I have tasks due within the next 30 minutes so that I don't miss important deadlines.

**Why this priority**: Desktop notifications enhance the application's value by providing proactive reminders, improving task completion rates. This is a nice-to-have feature that adds value but doesn't block core functionality.

**Independent Test**: Can be fully tested by launching the application when tasks are due within 30 minutes and verifying a system notification appears with relevant task details. Delivers value by improving task awareness.

**Acceptance Scenarios**:

1. **Given** a user has a task due in 15 minutes, **When** the user launches the application, **Then** the system displays a desktop notification showing the task title, due time, and priority
2. **Given** a user has 3 tasks due within the next 30 minutes, **When** the user launches the application, **Then** the system displays a single notification summarizing all 3 upcoming tasks or individual notifications for each (grouped appropriately)
3. **Given** a user has no tasks due within the next 30 minutes, **When** the user launches the application, **Then** no notification is displayed
4. **Given** a user has a task due in 31 minutes, **When** the user launches the application, **Then** no notification is displayed (outside the 30-minute window)
5. **Given** a user dismisses a notification, **When** they launch the application again within the same 30-minute window, **Then** the notification may or may not reappear (implementation choice)
6. **Given** a user is on an operating system that does not support desktop notifications, **When** the application launches, **Then** the system gracefully handles the limitation and continues to function normally

---

### User Story 4 - Recurring Task History (Priority: P4)

As a user, I want to track the history of completed instances of a recurring task so that I can see what has been accomplished and maintain an audit trail.

**Why this priority**: This feature supports the core recurring task functionality by maintaining historical context. It's valuable for tracking purposes and accountability but is not critical for the core value of creating recurring tasks.

**Independent Test**: Can be tested by creating a recurring task, completing it multiple times, and verifying that all instances are linked and can be queried together. Provides value for users who need to track recurring task completion patterns.

**Acceptance Scenarios**:

1. **Given** a user creates a recurring task, **When** the first instance is created, **Then** it has a parent_task_id of null (original task)
2. **Given** a user marks a recurring task as completed, **When** the system creates the next instance, **Then** the new instance's parent_task_id references the completed task
3. **Given** a user has completed 5 instances of a recurring task, **When** they view task history, **Then** they can see the chain of completed instances linked through parent_task_id
4. **Given** a user deletes an instance of a recurring task, **When** future instances are created, **Then** they continue to reference the most recent parent
5. **Given** a user wants to view all instances of a recurring task series, **When** they query by parent_task_id or original task ID, **Then** all related instances are returned

---

### Edge Cases

- **Recurrence with no due date**: What happens when a recurring task has no due date? The system creates the next instance but cannot apply the recurrence interval to calculate a due date.
- **Time zone handling**: What happens when users create tasks with natural language dates in different time zones? The system uses the user's local time zone for parsing and storage.
- **Notification flood**: How does the system handle notifying about 20+ tasks due in 30 minutes? The system groups notifications or limits the number displayed to avoid overwhelming the user.
- **Ambiguous date expressions**: What happens when users enter ambiguous dates like "Friday" and today is Friday? The system interprets as the current Friday if the time hasn't passed, or next Friday if it has.
- **Completed recurring task with parent deletion**: What happens when a user deletes the parent task of a completed recurring instance? The completed instance retains its data but the chain is broken.
- **Leap year handling**: What happens when a monthly recurring task is due on February 29th in a leap year? The system handles non-leap years by defaulting to February 28th or March 1st.
- **End of month recurrence**: What happens when a monthly recurring task is due on January 31st? In February, it defaults to the last day of the month.

## Requirements *(mandatory)*

### Functional Requirements

#### Recurring Task Management

- **FR-001**: System MUST allow users to set a recurrence interval when creating a task with options: daily, weekly, monthly, or none
- **FR-002**: System MUST store the recurrence interval as a task attribute
- **FR-003**: System MUST automatically create a new pending task when a task with a recurrence interval (not "none") is marked as completed
- **FR-004**: System MUST copy the following attributes from the completed task to the new instance: title, description, priority, category, and recurrence interval
- **FR-005**: System MUST calculate the next due date by adding the recurrence interval to the completed task's due date (daily: +1 day, weekly: +7 days, monthly: +1 month)
- **FR-006**: System MUST set the new task's due date based on the calculated next due date if the original task had a due date
- **FR-007**: System MUST NOT create a new instance when a task with recurrence set to "none" is completed
- **FR-008**: System MUST retain the completed task's status and timestamps for historical tracking
- **FR-009**: System MUST set the new task's completion status to "pending"
- **FR-010**: System MUST set the new task's created_at timestamp to the current time

#### Smart Date Input

- **FR-011**: System MUST accept natural language date expressions for due dates (e.g., "tomorrow", "next Monday", "in 2 weeks", "at 3pm")
- **FR-012**: System MUST parse natural language expressions into specific date/times
- **FR-013**: System MUST display clear error messages when date expressions cannot be parsed
- **FR-014**: System MUST allow users to skip providing a due date (optional field)
- **FR-015**: System MUST provide an interactive date/time selection wizard when natural language parsing fails or is not used
- **FR-016**: System MUST interpret relative dates based on the current date and time at the moment of parsing

#### Desktop Notifications

- **FR-017**: System MUST check for tasks due within the next 30 minutes when the application launches
- **FR-018**: System MUST display a system notification when tasks are due within 30 minutes
- **FR-019**: System MUST include task title and due time in the notification
- **FR-020**: System MUST group multiple due tasks into a single notification or display multiple notifications as appropriate
- **FR-021**: System MUST NOT display notifications when no tasks are due within the 30-minute window
- **FR-022**: System MUST gracefully handle operating systems that do not support desktop notifications without crashing
- **FR-023**: System MUST filter out completed tasks from the notification check

#### Recurring Task History

- **FR-024**: System MUST maintain a parent_task_id attribute for tracking task relationships
- **FR-025**: System MUST set the original recurring task's parent_task_id to null
- **FR-026**: System MUST set the new instance's parent_task_id to reference the completed task's ID when creating the next instance
- **FR-027**: System MUST allow users to query tasks by parent_task_id to view task history
- **FR-028**: System MUST maintain the chain of parent_task_id references across all instances of a recurring task series

### Key Entities

- **Task**: Represents a todo item with attributes including id, title, description, priority, category, due date, completion status, recurrence interval, parent_task_id, and timestamps (created_at, updated_at)
- **Recurrence Interval**: Defines how often a task repeats (daily, weekly, monthly, none)
- **Task Instance**: Represents a single occurrence of a recurring task, linked to its parent through parent_task_id

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can create a recurring task and have it automatically regenerate within 2 seconds of marking it complete
- **SC-002**: 95% of natural language date expressions are correctly interpreted without user intervention
- **SC-003**: Users receive notifications for all tasks due within 30 minutes when launching the application
- **SC-004**: 90% of users with recurring tasks report that the feature saves them at least 30 minutes per week
- **SC-005**: Task creation time decreases by 40% when using natural language date input compared to manual date entry
- **SC-006**: The system handles 100+ recurring task completions without performance degradation
- **SC-007**: All instances of a recurring task can be traced back to the original task within 3 database queries
- **SC-008**: Notification latency is under 1 second from application launch when tasks are due

## Assumptions

- Users have a system date/time that is reasonably accurate
- Users are creating tasks in their local time zone
- The application has permission to display desktop notifications on the user's system
- The Neon database is available and accessible during task creation and updates
- Natural language parsing is used for English language expressions only
- Recurring tasks with monthly intervals default to the same day of the month; if that day doesn't exist (e.g., February 30th), the system defaults to the last day of the month
