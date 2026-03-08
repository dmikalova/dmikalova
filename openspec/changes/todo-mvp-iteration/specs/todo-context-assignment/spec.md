## ADDED Requirements

### Requirement: Assign contexts to a todo

A todo SHALL support assignment of one or more contexts. The system MUST persist context assignments when a todo is saved and return them when the todo is fetched.

#### Scenario: Save a todo with a context

- **WHEN** a user saves a todo with one or more contexts selected
- **THEN** the backend persists the context assignments in the join table
- **AND** subsequent fetch of the todo includes the assigned contexts

#### Scenario: Remove all contexts from a todo

- **WHEN** a user saves a todo with no contexts selected
- **THEN** the backend removes all context assignments for that todo
- **AND** subsequent fetch of the todo returns an empty contexts list

#### Scenario: Update contexts on an existing todo

- **WHEN** a user changes the contexts on an already-saved todo and saves
- **THEN** the backend replaces the previous context assignments with the new set

### Requirement: Context field in todo detail form

The todo detail form SHALL display a context selector that shows all available contexts. The selector MUST reflect the todo's current context assignments on load.

#### Scenario: Context field pre-populated on edit

- **WHEN** the user opens an existing todo that has contexts assigned
- **THEN** the context selector shows those contexts as selected

#### Scenario: Context field empty for new todo

- **WHEN** the user opens the form to create a new todo
- **THEN** the context selector shows no contexts selected

#### Scenario: Context selector submits correct IDs

- **WHEN** the user selects contexts and saves the todo
- **THEN** the form submits an array of context IDs to the backend
