# Domain Pitfalls

**Domain:** Single-user, cross-device habit and daily-task web app
**Researched:** 2026-08-17
**Confidence:** MEDIUM — critical technical claims are cross-checked against current PostgreSQL, MDN, W3C, and OWASP guidance; product-policy choices need user validation.

## Critical Pitfalls

### Pitfall 1: Modelling a scheduled day as a UTC timestamp
**Confidence:** HIGH

**What goes wrong:** A habit due on a local calendar day appears eligible on the wrong day, or a completion is stored against yesterday/tomorrow after a timezone change. Weekly and monthly recurrence calculations also drift around daylight-saving transitions or short months.

**Why it happens:** A “day” is a civil-date concept, while an instant is global. JavaScript `Date` offsets depend on the represented time and can change with daylight saving; offset-only representations cannot safely derive future local dates across an offset change. Monthly recurrence also needs an explicit policy for dates that do not exist in every month (for example, the 31st).

**Consequences:** The core promise—truthful history—is broken. Users can lose a completion, complete it twice, or see impossible streak/analytics data.

**Prevention:**

- Store habit `created_on`, recurrence evaluation date, completion `scheduled_for`, task due date, and daily-plan date as `DATE`/`YYYY-MM-DD`, never as a midnight UTC instant.
- Store the user’s named IANA timezone separately for display and local “today” computation; use it at the server boundary as the authoritative clock.
- Define and test the monthly policy before implementation: recommended MVP policy is “run on the same day when it exists; otherwise skip that month,” rather than silently moving it to the last day.
- Make recurrence a pure, tested function of schedule + local date + creation date. Include leap day, 28/29/30/31, week-boundary, timezone, and DST cases even if the owner currently does not observe DST.

**Detection:** Device dates disagree near midnight; a new habit has older completions; test fixtures use ISO instants for date-only fields; an end-of-month habit behaves differently on PC and phone.

**Phase mapping:** Foundation/data contract, then Habit scheduling. Do not build the habit UI before the date semantics and test matrix are locked.

### Pitfall 2: Enforcing invariant rules only in the client
**Confidence:** HIGH

**What goes wrong:** Two tabs or two devices can create duplicate completions, more than three Should tasks, or an invalid Must/Should relationship—even when each individual screen appears correct.

**Why it happens:** Client state is stale and requests race. “Check then insert” logic is not an invariant; it can run concurrently. A database must be the last line of defence.

**Consequences:** Duplicate habit records distort 90-day history and future analytics; daily plans become ambiguous; manual repair becomes necessary.

**Prevention:**

- Use a completion/event table with a database uniqueness constraint on `(habit_id, scheduled_for)` and convert a duplicate-insert response into an idempotent “already complete” result.
- Use database constraints and transactions for state shape: non-null primary keys, constrained task state enum, `should` position limited to 1–3, and one row per plan date/slot. Use a unique partial index where an invariant applies only to active rows.
- Make completion and undo command endpoints idempotent, returning the canonical server record/version. Never trust a client-supplied “eligible” flag.
- Add concurrency integration tests: simultaneous completion requests, duplicate retry after network timeout, Must replacement, fourth Should attempt, and delete/complete race.

**Detection:** Duplicate rows found by audit query; intermittent 409/500 errors around double taps; UI counts differ after refresh; tests exercise only one browser session.

**Phase mapping:** Foundation/data integrity. Re-run the concurrent-write suite during habits and daily-plan phases.

### Pitfall 3: Ambiguous daily-plan and deletion policies
**Confidence:** MEDIUM

**What goes wrong:** The app cannot answer what a Must/Should slot means after midnight, when a task is deleted, or when a planned task is completed/rescheduled. Different screens then apply different interpretations.

**Why it happens:** The PRD intentionally leaves deleted-task retention/restoration and post-midnight plan changes open. These are product semantics, not implementation details.

**Consequences:** Yesterday’s planning history mutates unexpectedly, deleted items reappear, or valid data is permanently lost. Analytics later has no coherent source of truth.

**Prevention:**

- Decide and document the lifecycle before schema migration. Recommended MVP: soft-delete tasks with `deleted_at`, hide them from normal queries, and provide a small Archive/Restore view; purge is deferred.
- Treat a daily plan as a dated snapshot of task references. A task edit changes the task; it does not rewrite historical plans. Define whether a deleted task stays visible as “deleted” in past plan history.
- At local midnight, start a new plan date; do not auto-carry scheduled tasks into Must/Should. If a plan is edited after midnight, edit the plan selected by its explicit date only.
- Capture these decisions as acceptance examples and database/API contract tests before task views are built.

**Detection:** Requirement discussions use “today” without a date; list queries have ad-hoc `deleted` filters; restore is impossible without database intervention; the same task is shown in contradictory statuses across views.

**Phase mapping:** Task data model and daily-priorities phase. Block analytics until its historical semantics are settled.

### Pitfall 4: Treating “no login” as “no security work”
**Confidence:** HIGH

**What goes wrong:** A self-hosted backend exposed to a phone is reachable by anyone who can reach its URL, letting them read, alter, or delete personal records. Direct database credentials, tokens, or overly broad object serialization can also leak through deployment/configuration.

**Why it happens:** Single-user scope removes account-management features but does not establish an access boundary. Public API endpoints still accept untrusted requests and IDs.

**Consequences:** Disclosure or destruction of personal planning data and a difficult recovery if backups are absent.

**Prevention:**

- Choose an explicit MVP access model before deployment: private-network/VPN access is preferred; if internet-exposed, require a gateway-level authentication layer or a strong single-user secret/session over TLS. Do not leave the API publicly writable.
- Keep database private to the app network; use least-privilege credentials, environment secrets, CORS allowlists, request schema validation, origin/CSRF protection if cookies are used, and dependency/security updates.
- Return explicit response DTOs, not raw ORM/database rows; test that every mutation endpoint rejects unauthenticated or invalid requests.
- Automate encrypted, restorable database backups and do a recovery drill before relying on the app.

**Detection:** API works in an incognito browser without a boundary; the database port is publicly reachable; `.env`/secrets are committed; API responses contain internal fields; no successful restore has been performed.

**Phase mapping:** Foundation/deployment. This is mandatory even though account UI is explicitly out of scope.

## Moderate Pitfalls

### Pitfall 1: Optimistic UI that silently loses or overwrites state
**Confidence:** MEDIUM

**What goes wrong:** A completion looks saved but was rejected; an undo is overwritten by a delayed completion response; another device does not visibly refresh. The user stops trusting the app.

**Prevention:** Make mutations commands with server-confirmed state/version; disable or serialize a control while its request is in flight; abort or ignore stale fetches; reconcile every response with the canonical server record. Show unobtrusive “saving/synced/failed—retry” state and retain enough local UI state to retry safely. On navigation/focus, refresh the current day rather than assuming a long-lived cache is current.

**Detection:** Reports of “it changed back”; arbitrary `setTimeout` refreshes; response handlers update state without checking request/version relevance; no artificial-latency tests.

**Phase mapping:** Habit interaction and cross-device synchronization/reliability phase.

### Pitfall 2: Confusing scheduled, overdue, completed, and deleted task views
**Confidence:** MEDIUM

**What goes wrong:** A task disappears after its due date, appears in every list, or is accidentally presented as completed because it is no longer scheduled.

**Prevention:** Use one task lifecycle state (`uncompleted`, `completed`, `deleted`) plus independent nullable scheduling fields. Derive `overdue` at query/display time only when uncompleted, not deleted, with a due date before the user’s local today. Specify view inclusion rules and test every state/date combination.

**Detection:** Separate “overdue” database state appears in schema; filtering is duplicated across UI components; completion status is inferred from the absence of a due date.

**Phase mapping:** Task model and task-list views.

### Pitfall 3: Mobile completion controls that cause irreversible errors
**Confidence:** HIGH

**What goes wrong:** Small or closely packed tap targets cause accidental completion/deletion, especially in a fast daily-use flow; an undo affordance is hidden or expires before a user notices.

**Prevention:** Make the primary completion target at least 44×44 CSS px where practical (and no less than WCAG 2.2’s 24×24 minimum or its spacing alternative); distinguish destructive actions, provide immediate undo, and test with a real phone in portrait, with keyboard open, and on a slow network. Keep “today” actions one-handed and visible without hover.

**Detection:** Icon-only controls are under 24 px or adjacent; desktop screenshots are the only QA evidence; completing and undoing takes more than the PRD’s ten seconds.

**Phase mapping:** Habit and task UI; include phone acceptance testing before declaring either workflow complete.

### Pitfall 4: Deferring backup and recovery until data matters
**Confidence:** HIGH

**What goes wrong:** A bad migration, hosting incident, accidental delete, or deployment error destroys the only history the app was built to preserve.

**Prevention:** Start automated database backups from the first persistent deployment; define retention, keep backups outside the primary host, encrypt them, and document a restore procedure. Test restore into a disposable database and check that 90+ days of completions, tasks, and plans return intact.

**Detection:** “We can export later”; backups exist only on the production machine; no restore timestamp or runbook; schema migration has no rollback/forward-fix plan.

**Phase mapping:** Foundation/deployment and every schema-migration phase.

## Minor Pitfalls

### Pitfall 1: Turning routine management into an overbuilt productivity system
**Confidence:** MEDIUM

**What goes wrong:** Tags, projects, recurrence languages, notifications, streak gamification, collaboration, and calendar integration delay the two core loops: complete habits and plan today.

**Prevention:** Enforce the PRD’s explicit MVP limits: three recurrence types, one Must, at most three Shoulds, no accounts, AI, offline conflict resolution, or Google Calendar. Track requests as deferred backlog items and measure the under-two-minute planning goal before adding surface area.

**Detection:** A feature has no direct connection to a stated success metric; a new data model generalizes for hypothetical users; the daily screen needs multiple navigation steps to act.

**Phase mapping:** Roadmap scope control; review at every phase transition.

### Pitfall 2: Building analytics from UI state instead of durable events
**Confidence:** MEDIUM

**What goes wrong:** Later calendar counts and streaks cannot be reconstructed after edits/deletions, or they disagree with visible history.

**Prevention:** Preserve completion records as the source of truth, attach their scheduled local date, and create analytics as read models/queries from that data. Define deletion behavior for habits (archive/history preserved is recommended) before allowing hard deletion.

**Detection:** Completion is stored as a boolean on a habit; deleting a habit cascades completion history; analytics requires parsing logs or client cache.

**Phase mapping:** Habit data model now; analytics milestone later.

## Phase-Specific Warnings

| Phase topic | Likely pitfall | Mitigation |
|-------------|----------------|------------|
| Foundation: data contract and deployment | UTC timestamps for civil days; no API access boundary or recovery plan | Lock date-only/timezone model, database constraints, private access model, backups, and restore test before UI work. |
| Habits: schedules and completion | Duplicate/misdated completions; month-end recurrence ambiguity | Pure recurrence engine, named timezone, database uniqueness on `(habit_id, scheduled_for)`, idempotent complete/undo endpoints, edge-case tests. |
| Tasks: lifecycle and scheduling | Overdue confused with state; deletion behavior undefined | Separate lifecycle from due date; decide soft-delete/archive and explicit inclusion rules. |
| Daily Must/Should planning | Slot limits enforced only in UI; midnight edits mutate wrong day | Dated plan snapshots with database-backed slot constraints and explicit target date in all edits. |
| Mobile and sync reliability | Optimistic UI hides failed/stale writes; accidental taps | Server-confirmed reconciliation, visible sync state, stale-response protection, 44 px primary targets, real-device tests. |
| Analytics (later) | Historical data removed or reinterpreted | Retain completion events and plan snapshots; derive analytics from durable records rather than current task/habit rows. |

## Sources

- [PostgreSQL constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) — HIGH confidence; database-level uniqueness and integrity constraints.
- [PostgreSQL partial indexes](https://www.postgresql.org/docs/current/indexes-partial.html) — HIGH confidence; conditional uniqueness enforcement.
- [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html) — HIGH confidence; concurrent transaction and retry considerations.
- [MDN: Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) and [Temporal.ZonedDateTime](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal/ZonedDateTime) — HIGH confidence; offset transitions and named-zone behaviour.
- [MDN: date input](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/date) — HIGH confidence; date-only input values use `yyyy-mm-dd` without a time.
- [MDN: using Fetch / AbortController](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — HIGH confidence; cancelling in-flight requests.
- [OWASP API1: Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/) and [OWASP API3: Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/) — HIGH confidence; API access and response-shape risks.
- [W3C WCAG 2.2 target size](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum) — HIGH confidence; minimum target sizing and spacing guidance.

