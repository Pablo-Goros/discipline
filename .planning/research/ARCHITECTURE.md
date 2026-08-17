# Architecture Patterns

**Domain:** Personal single-user habit and daily-task web application
**Researched:** 2026-08-17
**Confidence:** MEDIUM — the design is derived from the PRD and cross-checked against current PostgreSQL and MDN primary documentation. Framework selection remains intentionally separate from architecture.

## Recommended Architecture

Build one responsive web client, one stateless application/API service, and one self-owned PostgreSQL database. Both the PC and phone load the same client and use the same API; the database is the only durable source of truth. Do not create separate mobile and desktop applications, a local-first replica, or a synchronization service for the MVP.

```text
PC browser  ─┐                         ┌─ PostgreSQL (authoritative data)
             ├─ HTTPS ─> Web/API app ──┤
Phone browser┘            |            └─ schema migrations + backups
                          │
                          └─ Server-side domain rules and transaction boundaries
```

Responsive layout is the device adaptation mechanism. MDN describes responsive design as adapting layout across device sizes; it is the appropriate way to serve this app from one codebase. Mobile-specific work belongs in interaction design (large touch targets, concise forms, low typing), not in a separate architecture.

### Component Boundaries

| Component | Responsibility | Communicates With |
|---|---|---|
| Responsive web client | Render Today, Habits, Backburner, scheduled-task and history views; collect actions; display fresh server state and errors. Keep only ephemeral UI state and a cache that can be discarded. | API service over HTTPS |
| API / application service | Authenticate the deployment boundary, validate requests, derive the user's local day, enforce domain rules, execute transactions, and return view-ready data. Stateless between requests. | Client and PostgreSQL |
| Habit domain module | Maintain habit definitions and recurrence descriptors; calculate whether a habit is due on a requested local date; create/undo completions. | API handlers and database |
| Task and planning domain module | Maintain task lifecycle, scheduling, overdue classification, and daily Must/Should selections without mutating the underlying task when a plan changes. | API handlers and database |
| Query/read-model module | Assemble the Today response: scheduled eligible habits, their completion state, Must/Should plan, scheduled tasks, and overdue tasks. It is a server-side query composition layer, not a second database. | API handlers and database |
| PostgreSQL database | Enforce durable facts, uniqueness, referential integrity, timestamps, and indexes; hold migrations and backup-recoverable data. | Application service only |
| Operations boundary | TLS/reverse proxy, private deployment access, environment secrets, database backups, migration execution, and health checks. | Client, application service, database |

The absence of *application accounts* must not mean an internet-open write API. Before PC/phone deployment, choose a possession-based access boundary such as private-network/VPN access or reverse-proxy access control. It is operational protection rather than multi-user product functionality and should be treated as a launch requirement.

### Data Model Boundaries

Use normalised tables for facts and store local calendar semantics explicitly.

| Aggregate | Minimum persisted shape | Key rules |
|---|---|---|
| App settings | `timezone`, optional display preferences | One configured IANA timezone defines “today” and scheduled dates for this one-user app. Changing it later needs an explicit migration policy. |
| Habit | `id`, name, icon, created_on, active/deleted state, schedule type and schedule parameters | A recurrence describes eligibility; it is not a row pre-generated for every future day. |
| Habit completion | `habit_id`, `scheduled_date`, `completed_at` | `UNIQUE(habit_id, scheduled_date)`, both key columns `NOT NULL`; server rejects dates before `created_on` and dates the recurrence does not schedule. |
| Task | `id`, title/body, status (`uncompleted`, `completed`, `deleted`), scheduled_date, optional scheduled_time, timestamps | Task state is separate from habit completion. A task stays uncompleted through its due date; `overdue` is computed, not a persisted state. |
| Daily plan | one row per `plan_date` with current Must task reference, plus ordered Should task references | Planning is a date-scoped relationship to tasks. Replacing a plan does not change task schedule or task status. Enforce one Must and at most three distinct Should references transactionally. |
| Audit/soft-delete decision | Either a `deleted_at` marker or a dedicated archive record | The PRD has not selected retention/restoration behavior. Do not irreversibly delete data until that policy is decided. |

Use `DATE` for a planned calendar day and reserve a timezone-aware instant for actual events (`completed_at`, `created_at`, `updated_at`). A scheduled optional time should be represented separately from the scheduled date so all-day tasks do not acquire a misleading midnight instant.

PostgreSQL confirms that a multicolumn unique constraint both enforces the completion invariant and automatically creates its supporting unique B-tree index. That constraint is required even if the client disables a completion button: separate devices can submit the same completion concurrently.

## Data Flow

### 1. Read the Today view

```text
Browser requests /today?date=YYYY-MM-DD
  -> API establishes the configured local date/timezone
  -> read-model queries active habits due that day + completion rows
  -> read-model queries uncompleted scheduled/overdue tasks + daily plan links
  -> API returns one coherent dashboard payload
  -> client renders device-appropriate layout
```

The server, not browser clock alone, decides the default date and calendar boundary. Allowing the client to request a past/future display date is useful for history, but write rules must still be server validated.

### 2. Complete or undo a habit

```text
Client action
  -> POST /habits/{id}/completions for local scheduled date
  -> API validates habit is active, created on/before date, and due on date
  -> one database transaction inserts completion
  -> UNIQUE(habit_id, scheduled_date) prevents a duplicate race
  -> API returns normalized habit/today state; client replaces cached value

Undo
  -> DELETE the same completion fact after server validation
  -> return normalized state
```

Treat a duplicate completion as an idempotent success or a domain-specific conflict with a fresh state payload; choose and document one API contract. Do not implement “toggle” as the core endpoint, because duplicate network retries could accidentally undo a valid completion.

### 3. Create, schedule, complete, or delete a task

```text
Client mutation -> API validation -> task transaction -> updated task/today response
                                                └-> query cache invalidation/refetch
```

Task completion updates the task record; it does not remove the task from prior daily-plan history. Whether a deleted task is preserved/restorable is a product decision that the service should isolate behind its delete operation.

### 4. Replace a daily plan

```text
Client submits { planDate, mustTaskId, shouldTaskIds[] }
  -> API validates one Must, 0..3 unique Should values, and eligible task states
  -> transaction upserts daily-plan header and replaces plan-item rows
  -> API returns the resulting plan and affected Today data
```

The transaction boundary is essential: a partially applied Must/Should selection would violate the product’s planning rule. The API must explicitly define its policy when the client updates a previous day's plan after midnight; this is an unresolved PRD decision, not a UI detail.

### Synchronization Direction

Synchronization is ordinary online read/write synchronization, not bidirectional database replication:

```text
Device A write -> API transaction -> PostgreSQL commits -> API response updates A
Device B opens/focuses/refreshes -> API read -> PostgreSQL -> fresh state on B
```

After every mutation, refetch or replace all affected view data (usually Today plus the source list). On browser focus and a short visible-page interval, refresh the current date. This meets the MVP's “after synchronization” requirement without WebSockets, queues, offline edits, or conflict resolution. Add an `updated_at`/monotonic `version` field now; it costs little and enables conditional updates later. HTTP ETags and `If-Match` are an upgrade path for optimistic locking, where stale concurrent writes can return `412 Precondition Failed` instead of silently overwriting a newer edit.

## Patterns to Follow

### Pattern 1: Server-owned domain invariants

**What:** Enforce schedule eligibility, creation-date limits, task state transitions, plan cardinality, and duplicate completion prevention in application code plus database constraints.

**When:** Every write endpoint, even though only one person uses the app. Two devices and retrying requests still create concurrency.

**Example:**

```typescript
await db.transaction(async (tx) => {
  const habit = await tx.habits.getForUpdate(habitId);
  assertEligible(habit, scheduledDate, appTimezone);
  await tx.habitCompletions.insert({ habitId, scheduledDate, completedAt: now });
  // Database additionally enforces UNIQUE (habit_id, scheduled_date).
});
```

### Pattern 2: Recurrence as definition plus derived occurrences

**What:** Store the daily/weekly/monthly recurrence definition on a habit and calculate whether it is due for a requested date.

**When:** MVP schedules and history views.

**Why:** It avoids generating and later repairing unbounded future occurrence records while still making completed history immutable and queryable for the required 90 days.

### Pattern 3: Command responses contain authoritative view state

**What:** A mutation responds with the changed entity and/or the rebuilt Today slice needed by the client.

**When:** Habit completion/undo, task changes, and daily-plan replacement.

**Why:** Both devices converge through server data, while the client stays simple and no local synchronization queue is introduced.

### Pattern 4: Narrow transactional commands

**What:** One API command corresponds to one meaningful domain action: `completeHabit`, `undoHabitCompletion`, `completeTask`, `replaceDailyPlan`.

**When:** All writes.

**Why:** Explicit commands make validation and auditability clearer than generic table-shaped CRUD and keep open policy decisions localized.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Two independently writable device stores

**What:** Persisting a full local task/habit database on each device, then attempting to reconcile it.

**Why bad:** It introduces exactly the offline queue and conflict-resolution problem the MVP excludes.

**Instead:** Require a connection for writes, commit centrally, and refresh from the API.

### Anti-Pattern 2: Client-only habit validity checks

**What:** Disabling UI controls but accepting any completion date submitted to the API.

**Why bad:** Browser state can be stale or altered, and duplicate completions across devices break trustworthy history.

**Instead:** Validate in the service and enforce the unique database constraint.

### Anti-Pattern 3: Encoding a daily plan in task fields

**What:** Adding `priority = must|should` and overwriting it each day.

**Why bad:** It destroys the fact that a plan belongs to a specific day and makes plan changes alter the task itself.

**Instead:** Store dated plan-to-task associations.

### Anti-Pattern 4: Premature real-time or event infrastructure

**What:** Adding WebSockets, a message broker, or event sourcing before the first vertical slice works.

**Why bad:** It increases failure modes without serving a stated MVP requirement.

**Instead:** Post-mutation refresh and focus-based polling; evaluate push only after actual synchronization latency proves insufficient.

## Scalability Considerations

| Concern | At 100 users | At 10K users | At 1M users |
|---|---|---|---|
| Current scope | One user; simple single instance and PostgreSQL are more than sufficient. | Would require real user isolation, authentication, tenancy, and capacity planning. | A different product architecture and operations model. |
| Today queries | Index task state/date and completion `(habit_id, scheduled_date)`; calculate recurrences in service. | Add read profiling, pagination/history aggregates, and cache policies. | Materialized/read models and horizontal service scaling. |
| Concurrent edits | Central transaction and refresh are adequate. | Conditional update versions and clear conflict UX. | Formal conflict policy and audit stream. |
| History/analytics | Keep raw completion facts; 90 days is trivial. | Add aggregate queries/indexes for calendars and streaks. | Pre-aggregated analytics storage. |

## Build-Order Implications

1. **Foundation and deployable vertical shell** — establish the responsive app shell, API service, PostgreSQL migrations, configured timezone, backup/restore procedure, and non-account deployment access boundary. Prove a phone and PC reach the same database before feature work.
2. **Habit vertical slice** — implement habit definition/schedules, server-side eligibility, completion/undo transaction, the unique completion constraint, and a Today habit view. This validates the most stringent integrity rule and the cross-device synchronization loop.
3. **Task workflow and daily planning** — implement backburner tasks, scheduling, completion/deletion policy, overdue queries, and separate daily Must/Should plan tables plus transactional replacement. Resolve the two PRD open questions before finalising this phase’s API/database behavior.
4. **Cross-device usability and resilience** — tune mobile/desktop interaction speed, mutation refetch/focus refresh, error handling, migration/backup recovery, and integration tests that exercise two browser sessions. Add conditional versioning if real concurrent edits show lost-update risk.
5. **Calendar analytics (later milestone)** — derive calendar completion and streak views from the immutable completion facts. It depends on stable habit recurrence and trustworthy historical data, so it must follow rather than shape the MVP schema.

## Sources

- [PostgreSQL current documentation: constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) — MEDIUM confidence via verified primary source.
- [PostgreSQL current documentation: unique indexes](https://www.postgresql.org/docs/current/indexes-unique.html) — MEDIUM confidence via verified primary source.
- [MDN: HTTP conditional requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Conditional_requests) — MEDIUM confidence via verified primary source.
- [MDN: responsive web design](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Responsive_Design) — MEDIUM confidence via verified primary source.
- [MDN: mobile accessibility](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/Mobile) — MEDIUM confidence via verified primary source.
