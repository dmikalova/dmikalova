## Context

The todos app has a data model that includes tasks, projects, contexts, and windows. The revised model (agreed in design review):

- **Contexts** are named, reusable labels (e.g., "morning", "evening", "free time") shared across projects
- **Windows** are recurring schedules on a context (e.g., Mon–Fri 9–12, every evening 19–22) — not one-off time ranges
- **Projects** are assigned to a context. Nested projects inherit the nearest ancestor's context if none is explicitly set
- **Tasks** have no direct context — they inherit implicitly through their project

This means updating a context's windows automatically affects all projects (and their tasks) that use it, with no per-task changes needed.

Current bugs: context assignment is not working correctly on the project form. Windows have been modeled but not exercised end-to-end. The Next view queries all tasks rather than filtering by context/window. The sidebar uses mixed-case labels. Project task counts include done tasks instead of only open tasks. All tasks (including completed) are loaded on startup with no pagination or cap.

Testing is sparse — no systematic backend unit tests for context/window logic, and no frontend component tests for the project form, sidebar, or Next view.

## Goals / Non-Goals

**Goals:**

- Redesign context assignment: contexts live on projects (not tasks), with inheritance for nested projects
- Remove context from tasks entirely
- Fix project task count: show only open (incomplete) tasks
- Bound task loading: open tasks load fully; done tasks load as capped history (1000, ordered by completion date descending)
- Verify and fix windows: recurring schedule evaluation (days-of-week + time range) works correctly
- Fix Next view: pipeline is active windows → contexts → projects → tasks
- Achieve 100% test coverage across all `src/` files; add new unit tests (`next_pipeline_test.ts`, `project_task_count_test.ts`, `task_loading_test.ts`, `store_test.ts`); update existing integration tests; add frontend component tests for sidebar, project form context selector, and history infinite scroll
- Lowercase all sidebar/nav labels and document this as a project tenet

**Non-Goals:**

- One-off (non-recurring) windows — all windows are recurring schedules
- Full e2e browser automation tests (out of scope for this iteration)
- Allowing tasks to override their project's context

## Decisions

### Context is set on projects, not tasks

**Decision:** Tasks have no `context_id`. Context is set on a project (many projects can share one named context). Tasks in a project implicitly belong to that context.

**Rationale:** A task's context is determined by what you're working on (the project), not the task itself. Shared named contexts mean updating a window schedule propagates everywhere automatically.

**Alternative considered:** Task-level context assignment (many-to-many). Rejected — over-engineered for how tasks are actually used; the user could not identify a real case where a task needs multiple or different contexts from its project.

---

### Nested projects inherit context from nearest ancestor

**Decision:** If a project has no context set, it inherits the context from its parent project, walking up the tree until a context is found. If no ancestor has a context, the project/tasks are context-less and do not appear in Next.

**Rationale:** Mirrors real-world usage — sub-projects naturally belong to the same context as their parent. Avoids requiring repeated context assignment on every nested project.

---

### Windows are recurring schedules

**Decision:** A window is defined as `{ days: ["mon","tue","wed","thu","fri"], start: "09:00", end: "12:00" }` in local time. A window is active if today's day-of-week matches and current local time is within start–end.

**Rationale:** "Every weekday morning" is a stable, recurring intention — not a one-off appointment. Recurring schedules require no maintenance once set.

---

### Window evaluation is client-side

**Decision:** The client evaluates which windows are currently active using the device's local clock. With Supabase realtime, context/window data is always fresh on the client.

**Rationale:** Supabase realtime keeps the client in sync; server-side evaluation adds latency and complexity with no benefit. Local time is also correct for the user's actual schedule.

---

### Next view pipeline: active windows → contexts → projects → tasks

**Decision:** The Next view computes:

1. Evaluate all windows client-side → collect active contexts
2. Find all projects assigned to (or inheriting) an active context
3. Return incomplete tasks belonging to those projects, ordered by due date ascending (undated last)

**Rationale:** Directly maps the GTD "Next Actions" model — surface tasks you can act on right now given your current context.

---

### Lowercase labels enforced via constants/i18n

**Decision:** All sidebar label strings are defined in a single constants or i18n file. Labels MUST NOT be hardcoded inline in templates.

**Rationale:** Single source of truth prevents drift; easy to audit and enforce as a project tenet.

---

### Task loading: open tasks fully loaded, done tasks capped at 1000

**Decision:** On startup, the client loads all open (incomplete) tasks. Done tasks are loaded as an initial batch of 100 records (most recently completed first). The history view supports infinite scroll — scrolling to the end fetches the next batch of completed tasks on demand.

**Rationale:** Open tasks must all be available for Next, inbox, and project views. 100 done tasks covers recent history without a heavy initial load. Infinite scroll in history gives access to the full archive without upfront cost.

**Alternative considered:** Pagination for all tasks. Rejected for MVP — open tasks need to be fully available client-side for the Next view pipeline to work without extra round-trips.

---

### Project task count shows only open tasks

**Decision:** The task count displayed on a project shows only incomplete tasks. Done tasks are excluded.

**Rationale:** The count communicates remaining work, not total historical tasks. Including done tasks inflates the number in a misleading way.

## Risks / Trade-offs

- [Risk] Removing context from tasks is a schema change → Mitigation: migration to drop `context_id` from tasks table (or nullify and stop using it)
- [Risk] Context inheritance requires a tree traversal → Mitigation: projects are unlikely to be deeply nested; a simple recursive lookup is sufficient at MVP
- [Risk] Client-side window evaluation uses local device clock → Mitigation: this is intentional (user's local schedule), document it clearly
- [Risk] Capping initial done tasks at 100 means older completed tasks aren't immediately available → Mitigation: infinite scroll in history view loads them on demand
- [Risk] Test coverage gap may uncover additional bugs → Mitigation: file as follow-up tasks; don't block MVP

## Migration Plan

1. Remove `context_id` from tasks (migration)
2. Ensure projects have `context_id` (nullable) and `parent_project_id` for nesting
3. Fix project form context selector — save and load context correctly
4. Fix project task count query to filter `WHERE completed = false`
5. Update task loading to load all open tasks + cap done tasks at 1000
6. Implement recurring window model and client-side active evaluation
7. Implement context inheritance traversal for nested projects
8. Update Next view to use the new pipeline
9. Update all sidebar label strings to lowercase
10. Deploy

## Open Questions

- ~~Should tasks with no project (inbox tasks) ever appear in Next?~~ **Resolved:** No. Tasks with no project belong to the inbox and never appear in Next.
