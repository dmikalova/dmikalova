## ADDED Requirements

### Requirement: Next view shows todos from active contexts

The Next view SHALL display todos that are assigned to contexts with at least one currently active window. Todos not assigned to any active context MUST NOT appear in the Next view.

#### Scenario: Todos in active context appear in Next

- **WHEN** a todo is assigned to a context that has an active window
- **THEN** that todo appears in the Next view

#### Scenario: Todos in inactive context do not appear in Next

- **WHEN** a todo is assigned only to contexts with no active windows
- **THEN** that todo does not appear in the Next view

#### Scenario: Todos with no context do not appear in Next

- **WHEN** a todo has no context assignment
- **THEN** that todo does not appear in the Next view

### Requirement: Next view query is server-side

The Next view data SHALL be returned by a dedicated backend query that evaluates active windows and filters todos server-side. The frontend MUST NOT perform window evaluation logic.

#### Scenario: Next view fetches filtered list from backend

- **WHEN** the frontend loads the Next view
- **THEN** it calls a backend endpoint that returns only context-and-window-filtered todos

### Requirement: Next view ordering

Todos in the Next view SHALL be ordered by due date ascending (earliest first), with undated todos appearing last.

#### Scenario: Todos with due dates appear before undated todos

- **WHEN** the Next view contains both dated and undated todos
- **THEN** dated todos appear first, sorted by due date ascending
- **AND** undated todos appear after all dated todos
