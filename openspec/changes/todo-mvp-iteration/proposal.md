## Why

The todos app has several bugs preventing MVP usability: context assignment is broken, the "Next" view does not pull from contexts, and windows (time-based filters within contexts) are untested. The UI also lacks consistent labeling conventions, reducing polish and trust.

## What Changes

- Fix context assignment on todos — setting and persisting contexts is not working correctly
- Verify and fix windows support — time-bounded context filters have not been tested end-to-end
- Wire the "Next" view to pull from contexts — currently does not reflect context-filtered todos
- Add comprehensive backend and frontend tests covering todos, contexts, windows, and the Next view
- Enforce lowercase UI labels across the sidebar (e.g., "add task", "search", "inbox", "next", "due", "history", "settings")
- Establish lowercase labels as a project tenet going forward

## Capabilities

### New Capabilities

- `todo-context-assignment`: Setting, saving, and displaying contexts on individual todos
- `todo-windows`: Time-bounded filters (windows) within contexts that control todo visibility
- `next-view`: The "Next" view that surfaces todos based on active context and window state
- `ui-label-conventions`: Project tenet — all sidebar and navigation labels use lowercase

### Modified Capabilities

<!-- None — no existing specs to delta against -->

## Impact

- Backend: todo CRUD endpoints, context assignment logic, window evaluation logic, Next query
- Frontend: todo detail form (context field), sidebar label components, Next view component
- Tests: new backend unit/integration tests for todos + contexts + windows; new frontend component tests for todo form, sidebar, and Next view
- No breaking API changes expected; this is a bug-fix and polish pass to reach MVP
