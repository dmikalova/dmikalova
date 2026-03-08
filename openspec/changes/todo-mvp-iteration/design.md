## Context

The todos app has a data model that includes todos, contexts, and windows. A todo can be assigned one or more contexts. A context can have windows — time-bounded periods during which todos in that context should surface. The "Next" view is intended to show the most actionable todos based on which contexts have active windows right now.

Current bugs: the frontend context field on the todo detail form is not correctly saving context assignments back to the backend. Windows have been modeled but not exercised end-to-end. The Next view queries all todos rather than filtering by context/window. The sidebar uses mixed-case labels.

Testing is sparse — there are no systematic backend unit tests for context assignment or window evaluation, and no frontend component tests for the todo form or Next view.

## Goals / Non-Goals

**Goals:**

- Fix context assignment: todo ↔ context relationship correctly saved and loaded
- Verify windows: context window time evaluation works and surfaces/hides todos correctly
- Fix Next view: query uses active context windows to filter the todo list
- Achieve meaningful test coverage: backend unit + integration tests for todos/contexts/windows; frontend component tests for todo form, sidebar, Next view
- Lowercase all sidebar/nav labels and document this as a project tenet

**Non-Goals:**

- Redesigning the data model (contexts, windows, todos relationships stay as-is)
- Adding new features beyond what is needed to reach MVP
- Full e2e browser automation tests (out of scope for this iteration)

## Decisions

### Context assignment uses a join table (many-to-many)

**Decision:** Todos and contexts have a many-to-many relationship stored in a join table (`todo_contexts`). The frontend submits an array of context IDs when saving a todo.

**Rationale:** A todo can be relevant in multiple contexts (e.g., "Work" and "Errands"). A join table is idiomatic and avoids denormalization.

**Alternative considered:** Storing a single context_id on the todo. Rejected — too limiting for real GTD usage.

---

### Window evaluation is server-side

**Decision:** The backend evaluates whether a window is currently active (based on `start_time`/`end_time` or day-of-week + time range) and the Next query filters on this server-side.

**Rationale:** Keeps evaluation logic in one place, prevents stale client-side time comparisons, and makes the Next query a single DB call.

**Alternative considered:** Client-side window filtering. Rejected — inconsistent across clients, harder to test.

---

### Next view query: active windows → contexts → todos

**Decision:** The Next query pipeline:

1. Find all windows where `now()` falls within the window's active range
2. Collect the contexts those windows belong to
3. Return todos assigned to any of those contexts (ordered by priority/due date)

**Rationale:** Directly maps the GTD "Next Actions" mental model — you act based on your current context.

---

### Lowercase labels enforced via constants/i18n

**Decision:** All sidebar label strings are defined in a single constants or i18n file rather than inline in templates. The tenet is enforced by convention (and a linter rule if feasible).

**Rationale:** Single source of truth prevents drift; easy to audit.

## Risks / Trade-offs

- [Risk] Window time logic depends on server clock → Mitigation: use UTC consistently, document this assumption
- [Risk] Fixing context save may require a migration if data is corrupted → Mitigation: write a one-time cleanup migration before deploying
- [Risk] Test coverage gap may uncover additional bugs → Mitigation: file them as follow-up tasks; don't block MVP on them

## Migration Plan

1. Audit and fix the `todo_contexts` join table insert/update logic on the backend
2. Fix the frontend context field to submit IDs correctly
3. Write and run backend tests to confirm context round-trip works
4. Implement and test window evaluation function
5. Update Next view query to use window-filtered contexts
6. Update all sidebar label strings to lowercase
7. Deploy — no destructive schema changes expected

## Open Questions

- Are windows defined as recurring (e.g., "weekday mornings") or one-off time ranges? → Clarify before implementing window evaluation
- Should Next show todos with _no_ context assignment as a fallback, or only context-matched todos?
