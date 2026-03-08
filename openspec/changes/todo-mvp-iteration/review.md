## Summary

The plan is well-scoped and grounded in the actual codebase. The model simplification (context on project, not task) is the right call. Key findings: the backend currently has context on tasks (not projects) and the migration needs to be explicit; the Next route uses server-side context detection which conflicts with the agreed client-side model and needs to be removed; the project task_count query is unverified; and the testing spec is comprehensive but the frontend test tooling approach needs to be confirmed. No security concerns are blocking. All action items are in-scope for this change.

## Security

- [x] **Input validation is already in place** — Zod schemas on all routes, UUID validation on IDs. No new attack surface from context/project changes. **Address: no action needed.**
- [x] **Auth middleware covers all routes** — `src/middleware.ts` gates the API. Context/project changes don't bypass this. **Address: no action needed.**
- [ ] **Context ID injection on task create/update** — `syncTaskContexts` trusts the submitted context UUIDs without verifying they belong to the authenticated user. If multi-user support is ever added, this becomes a privilege escalation vector. **Defer: single-user app at MVP; note as a future concern.**

## Patterns

- [x] **`syncTaskContexts` pattern exists and works** — the delete-then-reinsert pattern for join table sync is already in `tasks.ts`. The same pattern should be used for project context assignment (single FK, not a join table — just a direct UPDATE). **Address: confirm projects use direct `context_id` column, not a join table.**
- [ ] **`getContextWithWindows` helper pattern** — the context route already has a helper for fetching a context with its windows. The same helper pattern should be used consistently in any new context-related queries. **Address: follow existing helper convention.**
- [x] **Frontend store follows fetch-then-notify pattern** — all new store methods (history pagination, next pipeline) should follow the existing `async method → api() → this._field = result → notify()` pattern. **Address: document in tasks.**
- [ ] **`context_id` is currently on tasks in the schema** — the store's `Task` interface has `context_id?: string` and `context_ids?: string[]`. The migration must remove these fields from the DB schema and the TypeScript interfaces, and update all callers. **Address: explicit migration step required.**

## Alternatives

- [ ] **Infinite scroll implementation** — the browser's `IntersectionObserver` API is the idiomatic way to implement scroll-to-load in Lit components without scroll event listeners. **Address: specify IntersectionObserver in tasks.**
- [ ] **Project context inheritance traversal** — at MVP, projects are unlikely to be more than 2–3 levels deep. A simple recursive lookup in the store (not a DB recursive CTE) is sufficient and easier to test. **Address: confirm client-side traversal approach in tasks.**

## Simplifications

- [x] **Remove `/api/contexts/current` endpoint** — this server-side endpoint was designed for the old server-side window evaluation model. With client-side evaluation, it is now redundant. **Address: remove endpoint and its integration tests.**
- [x] **Remove `contextId`/`projectId` query params from `GET /api/next`** — the Next pipeline is now fully client-side. The backend `/api/next` scoring endpoint may still be useful but its context detection logic should be removed. **Address: audit and simplify or remove the `/api/next` backend route.**
- [ ] **`context_id` on tasks schema** — tasks currently store `context_id` (single) AND a `task_contexts` join table. The agreed model removes context from tasks entirely. Both the column and the join table need to be dropped. **Address: migration drops `tasks.context_id` column AND `task_contexts` table.**

## Missing Considerations

- [x] **Migration for existing task_contexts data** — if users have existing context assignments on tasks, dropping the join table destroys that data. A migration step should map existing task context → task's project context before dropping. **Address: add to migration plan.**
- [ ] **`task_count` on projects is not currently filtered** — the existing `GET /api/projects` query includes a `task_count` subquery. It needs to be verified and fixed to `WHERE completed_at IS NULL`. **Address: audit and fix the projects list query.**
- [x] **Frontend test tooling not specified** — the codebase uses Deno + Lit web components. There is no existing frontend test setup. The testing spec assumes component tests exist but the tooling (e.g., `@web/test-runner`, `happy-dom`, or Deno's built-in test runner with a DOM shim) needs to be decided before tasks are written. **Address: decide frontend test tooling as the first task.**
- [ ] **History view currently loads via `fetchHistory()` which has no pagination** — the store's `fetchHistory()` method fetches all history entries. This needs to be replaced with the paginated approach (initial 100, then infinite scroll). **Address: update `fetchHistory` and add `fetchMoreHistory` to store.**
- [ ] **Error state for infinite scroll** — if a history page fetch fails mid-scroll, the UI should show an error and allow retry. **Address: add error handling to history fetch-more logic.**

## Valuable Additions

- [ ] **Loading indicator at bottom of history list** — while a next page is fetching, a spinner or "loading..." text at the bottom improves UX. **Address: include in history view tasks.**
- [ ] **Empty state for Next view** — if no contexts have active windows right now, the Next view should show a helpful empty state ("no active context right now") rather than a blank list. **Defer: nice-to-have for MVP+1.**
- [ ] **Context color used in project list** — since projects now show their context, displaying the context color as a small swatch next to the project name would reinforce the association visually. **Defer: polish pass after MVP.**

## Action Items

Items addressed in this change:

- Remove `/api/contexts/current` endpoint and its tests
- Audit and simplify `/api/next` — remove server-side context detection logic
- Migration: drop `tasks.context_id` column AND `task_contexts` table (after mapping data to project context)
- Fix `task_count` in projects list query to filter `WHERE completed_at IS NULL`
- Decide and set up frontend test tooling as first task
- Update store `fetchHistory` to paginated approach; add `fetchMoreHistory`
- Add error handling + loading indicator to history infinite scroll
- Use `IntersectionObserver` for infinite scroll trigger
- Confirm client-side recursive traversal for project context inheritance

## Deferred Items

- Multi-user context ID ownership validation (single-user app at MVP)
- Empty state for Next view when no active context
- Context color swatch in project list

## Updates Required

- **Design — Migration Plan**: Add step to map existing task context assignments to project context before dropping `task_contexts` table and `tasks.context_id` column
- **Design — Decisions**: Add note that `/api/contexts/current` is removed; `/api/next` server-side context detection is removed in favor of client-side pipeline
- **Specs — testing**: Add requirement for frontend test tooling setup as a prerequisite task
