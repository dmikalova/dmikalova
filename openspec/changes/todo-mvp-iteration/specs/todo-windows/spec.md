## ADDED Requirements

### Requirement: Define windows on a context

A context SHALL support one or more windows. A window defines a time range during which todos in that context are considered active/surfaceable. The system MUST persist windows and associate them with their parent context.

#### Scenario: Create a window on a context

- **WHEN** a user defines a window with a start time and end time on a context
- **THEN** the backend persists the window linked to that context

#### Scenario: Fetch context includes its windows

- **WHEN** the system fetches a context
- **THEN** the response includes the list of windows defined on that context

### Requirement: Window active state evaluation

The backend SHALL evaluate whether a window is currently active by comparing the current UTC time against the window's time range. A window is active if `now()` falls within its defined range.

#### Scenario: Window is active during its time range

- **WHEN** the current time is within a window's start and end time
- **THEN** the window is evaluated as active

#### Scenario: Window is inactive outside its time range

- **WHEN** the current time is before or after a window's time range
- **THEN** the window is evaluated as inactive

#### Scenario: Multiple windows on one context — any active counts

- **WHEN** a context has multiple windows and at least one is active
- **THEN** the context is considered active for the purpose of the Next view

### Requirement: Windows use UTC time

All window time comparisons MUST use UTC. The system SHALL store and evaluate window times in UTC.

#### Scenario: Window stored in UTC

- **WHEN** a window is created with a given time range
- **THEN** the backend stores the times in UTC

#### Scenario: Evaluation uses server UTC clock

- **WHEN** the backend evaluates window active state
- **THEN** it compares against the server's current UTC time
