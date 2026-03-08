## ADDED Requirements

### Requirement: All resource IDs validated against authenticated user

Before any route uses a submitted resource ID (contextId, projectId, parentProjectId, taskId), the backend MUST verify that the referenced resource belongs to the authenticated user. Requests referencing resources owned by another user MUST be rejected with 403.

This applies to:

- `contextIds` submitted on task create/update (`syncTaskContexts`)
- `projectId` submitted on task create/update
- `contextId` submitted on project create/update
- `parentProjectId` submitted on project create/update

#### Scenario: Valid owned context ID accepted

- **WHEN** a user submits a contextId that belongs to them
- **THEN** the request proceeds normally

#### Scenario: Foreign context ID rejected

- **WHEN** a user submits a contextId that exists but belongs to another user
- **THEN** the route returns 403 Forbidden
- **AND** the resource is not modified

#### Scenario: Non-existent context ID rejected

- **WHEN** a user submits a contextId that does not exist
- **THEN** the route returns 400 Bad Request or 404 Not Found

#### Scenario: Foreign project ID rejected on task create

- **WHEN** a user submits a projectId that belongs to another user
- **THEN** the route returns 403 Forbidden

#### Scenario: Foreign parentProjectId rejected on nested project create

- **WHEN** a user submits a parentProjectId that belongs to another user
- **THEN** the route returns 403 Forbidden

### Requirement: Ownership check is a shared utility

Ownership validation SHALL be implemented as a reusable helper (e.g., `assertOwnership(sql, table, id, userId)`) rather than inline per-route logic, to ensure consistency and prevent omission.

#### Scenario: Ownership helper used consistently

- **WHEN** any route performs an ownership check
- **THEN** it uses the shared `assertOwnership` utility
- **AND** no route performs ad-hoc ownership SQL inline

### Requirement: Ownership validation is tested

Every ownership check MUST have a corresponding integration test that verifies a 403 is returned when a foreign resource ID is submitted.

#### Scenario: Integration test for each ownership-guarded field

- **WHEN** integration tests are run
- **THEN** there are tests covering foreign contextId, foreign projectId, and foreign parentProjectId on the relevant routes
