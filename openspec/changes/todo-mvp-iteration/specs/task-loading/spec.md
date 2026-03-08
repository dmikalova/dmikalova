## ADDED Requirements

### Requirement: Open tasks load fully on startup

The client SHALL load all incomplete (open) tasks on startup. There is no cap on open task loading.

#### Scenario: All open tasks are available client-side

- **WHEN** the app initializes
- **THEN** all tasks where `completed = false` are loaded into the client
- **AND** they are available for the Next view, inbox, and project views without additional fetches

### Requirement: Done tasks load as capped history

The client SHALL load done (completed) tasks as a history query capped at 1000 records, ordered by completion date descending (most recently completed first).

#### Scenario: Done tasks load with a cap of 1000

- **WHEN** the app initializes
- **THEN** the client fetches at most 1000 completed tasks ordered by completion date descending

#### Scenario: Done tasks beyond 1000 are not loaded

- **WHEN** a user has more than 1000 completed tasks
- **THEN** only the 1000 most recently completed tasks are loaded
- **AND** older completed tasks are not shown (pagination is a future concern)

#### Scenario: Done tasks appear in history view

- **WHEN** the user opens the history view
- **THEN** it shows the locally-loaded completed tasks (up to 1000)
