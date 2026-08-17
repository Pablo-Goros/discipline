# Project Research Summary

**Project:** Discipline
**Domain:** Single-user, cross-device habit and daily-task web application
**Researched:** 2026-08-17
**Confidence:** MEDIUM

## Executive Summary

Discipline is a focused personal daily-practice application rather than a general productivity suite: one owner needs to capture tasks, plan a small daily commitment, complete scheduled habits, and trust the shared history from either a phone or PC. The recommended MVP is a responsive, self-hosted web app with one authoritative PostgreSQL database. Habits remain schedule-driven completion records; tasks remain discrete lifecycle records. They meet only in a fast Today view and a dated Must/Should plan.

Build it as a TypeScript Next.js modular monolith with server-side domain rules, PostgreSQL constraints, and a deliberately small same-origin API. Start with date/timezone semantics, access protection, backups, and data integrity before feature UI. Then deliver habits, tasks/planning, and reliability as vertical slices. The principal risks are corrupt calendar-day history, client-only invariants, insecure single-user deployment, and ambiguous deletion/planning semantics; mitigate them through an IANA timezone plus SQL `DATE` model, transactional constraints, gateway/private-network protection, tested restore, and explicit policy decisions before task schema work.

## Key Findings

### Recommended Stack

Use a single responsive TypeScript application: Next.js App Router and Route Handlers with React 19, Tailwind CSS, selected locally owned shadcn/Radix controls, and a PostgreSQL-backed Prisma data layer. This keeps one codebase and one deployable service for PC and phone while preserving database ownership. Zod validates every server mutation; React Query invalidates/refetches after writes rather than introducing real-time infrastructure.

**Core technologies:**

- **Node.js 24 LTS + Next.js 16.2 + React 19.2:** full-stack responsive application with server rendering and a small same-origin API.
- **TypeScript 5.9:** strict domain modelling; enable strict settings that expose invalid schedule and plan assumptions early.
- **PostgreSQL 18 + Prisma 7/Migrate:** authoritative relational storage, schema migrations, transactions, foreign keys, and durable constraints.
- **Tailwind 4 + shadcn/Radix:** mobile-first styling and accessible, locally controlled interaction primitives.
- **Zod, React Hook Form, React Query, date-fns:** validated form/mutation boundaries and cache refresh; date helpers only after explicit timezone conversion.
- **Docker Compose, Caddy, Restic:** self-host deployment, TLS, private database networking, encrypted off-host backups, and restore testing.
- **Vitest/Testing Library + Playwright:** recurrence/domain tests plus phone, desktop, and two-session end-to-end coverage.

Pin patched versions and lockfiles before implementation. The database must enforce `UNIQUE(habit_id, scheduled_date)`, valid task states, and daily-plan limits; browser validation is only a usability aid.

### Expected Features

The MVP should optimize the daily loop: open Today, complete an eligible habit or act on a task, and confidently see the result on both devices. Preserve dated history now so later calendar and streak views are derived from facts rather than reconstructed from UI state.

**Must have (table stakes):**

- Responsive synchronized PC/phone access with clear saving, retry, and refresh behavior.
- Habit creation/editing/archiving, optional icon, and daily/selected-weekday/monthly schedules.
- Eligible Today habit checklist with one-tap completion, same-day undo, and durable dated completion history.
- Backburner task capture; schedule, reschedule, complete, edit, and delete tasks; clearly review overdue work.
- A fast Today planning surface with exactly one Must and up to three Shoulds for an explicit date.

**Should have (competitive):**

- One deliberate daily-practice hub that unifies habits and concise tasks without merging their models.
- Backburner-to-day workflow and constrained Must/Should commitment model.
- Trust-first, analytics-ready completion history and self-owned deployment.

**Defer (v2+):**

- Habit calendar coverage and schedule-aware streaks.
- Browser/mobile reminders, custom recurrence, offline queue/conflict resolution, and Google Calendar integration.
- Accounts/sharing, AI planning, project-management features, gamification, and broad recurrence rules.

### Architecture Approach

Run one responsive web client against a stateless Next.js application/API service and one PostgreSQL source of truth. Keep client state ephemeral and replaceable; put local-day derivation, validation, transactions, and query composition on the server. No separate mobile app, sync service, WebSockets, event bus, or secondary read store is justified for this MVP.

**Major components:**

1. **Responsive web client** — renders Today, habits, task, plan, and history views; supports touch-friendly commands and reflects fresh server state.
2. **API/application service** — protects the deployment boundary, validates requests, resolves the configured timezone, applies domain rules, and returns explicit DTOs.
3. **Habit domain module** — owns schedule descriptors, pure eligibility calculation, and idempotent completion/undo commands.
4. **Task and planning domain module** — owns task lifecycle, scheduling, derived overdue state, and dated Must/Should associations.
5. **Today read-model module** — composes one coherent dashboard payload from habits, completions, tasks, overdue work, and daily-plan links.
6. **PostgreSQL and operations boundary** — persist facts and constraints; provide migrations, TLS/private access, secrets, backups, restore, and health checks.

Calendar semantics are non-negotiable: configure one IANA timezone; store scheduled/calendar keys as SQL `DATE` / `YYYY-MM-DD`; store actual instants as UTC `timestamptz`. A task's lifecycle state is independent of its scheduled date, and a daily plan is a dated relationship—not a priority field on the task.

### Critical Pitfalls

1. **Using UTC instants for civil schedule dates** — model scheduled dates, completion dates, and plan dates as `DATE`; compute “today” server-side in the configured IANA timezone; define and test monthly skip behavior for nonexistent dates.
2. **Enforcing integrity only in the client** — put uniqueness, plan cardinality, lifecycle validity, and writes inside database-backed transactions; run concurrent-request and retry tests.
3. **Leaving task deletion and after-midnight plan semantics undefined** — decide soft-delete/archive and dated-plan snapshot rules before migrations; ensure edits never rewrite historical plans implicitly.
4. **Treating no application login as no security work** — select a private-network/VPN or reverse-proxy access boundary, keep PostgreSQL private, validate requests, protect secrets, and test recovery.
5. **Untrustworthy mobile/sync interactions** — serialize in-flight commands, reconcile from canonical server responses, show retry/sync state, refetch on focus, preserve immediate undo, and test 44px touch targets on real phones.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Foundation, Data Contracts, and Safe Deployment

**Rationale:** Every daily action depends on a correct local-date model, an authoritative database, and a protected/recoverable deployment; retrofitting any of these risks corrupting history.

**Delivers:** Next.js application shell; PostgreSQL/Prisma migrations; strict domain conventions; configured IANA timezone; date/time separation; Docker Compose/Caddy private access boundary; secrets; backup/restore runbook and proof; baseline CI.

**Addresses:** Responsive shared persistence and synchronization prerequisite.

**Avoids:** UTC civil-day bugs, public writable APIs, lost records, and client-only integrity assumptions.

### Phase 2: Habit Tracking Vertical Slice

**Rationale:** Habits validate the strictest product invariant and the core cross-device mutation loop before task complexity is added.

**Delivers:** Habit CRUD/archive, supported recurrence descriptors, pure eligibility service, Today habit list, idempotent complete/undo commands, dated completion history, and cross-session tests.

**Addresses:** Habit schedules, Today eligibility, fast complete/undo, and 90+ day durable history.

**Avoids:** Duplicate or pre-creation completions, recurrence drift, ambiguous month-end behavior, and analytics-unready data.

### Phase 3: Task Lifecycle and Backburner Workflow

**Rationale:** Tasks have a separate lifecycle and query model; establish it independently before layering priorities on top.

**Delivers:** Backburner capture, task edit/schedule/reschedule/complete/delete flows, chosen soft-delete/archive behavior, and uncompleted scheduled versus derived-overdue views.

**Addresses:** Discrete task management, scheduled tasks, and explicit overdue treatment.

**Avoids:** Persisting `overdue` as a lifecycle state, disappearing late tasks, and irreversible/undefined deletion behavior.

### Phase 4: Dated Daily Planning and Unified Today

**Rationale:** Must/Should is a date-scoped relationship over stable task records, so it depends on phase 3’s task policy and persistence.

**Delivers:** One-Must/zero-to-three-Should transactional plan model, selection/replacement API, combined Today read model, and under-two-minute planning workflow.

**Addresses:** The differentiated daily-practice hub, constrained daily planning, and backburner-to-day workflow.

**Avoids:** Encoding plans in task fields, fourth-Should races, task edits mutating historical plans, and implicit post-midnight carryover.

### Phase 5: Cross-Device UX, Reliability, and Release Validation

**Rationale:** The product promise is trustworthy daily action from either device; it needs end-to-end validation after both loops exist.

**Delivers:** Mobile-first interaction refinements, touch-target/undo behavior, mutation error/retry and focus-refresh strategy, two-session synchronization tests, accessibility checks, migration/recovery drill, and deployment readiness.

**Addresses:** Reliable synchronized PC/phone experience and the under-ten-second completion goal.

**Avoids:** Silent optimistic-write failures, stale response overwrites, accidental mobile completion, and untested backups.

### Phase 6: Calendar History and Streak Analytics (Next Milestone)

**Rationale:** Analytics must consume stable completion facts and proven recurrence semantics, never determine the MVP data model.

**Delivers:** Calendar coverage, per-habit completion views, and explicitly defined schedule-aware streak calculations.

**Addresses:** Deferred habit analytics.

**Avoids:** Analytics derived from client cache or mutable habit/task state.

### Phase Ordering Rationale

- Foundation precedes UI so database constraints, civil-date semantics, secure access, and recoverability are structural rather than corrective work.
- Habits precede tasks because their recurrence/completion invariant is the clearest proof that cross-device writes are correct.
- Tasks establish stable records before dated plans reference them; the unified Today view is therefore a composition layer, not a tangled primary data model.
- Reliability validation follows both vertical loops so tests cover real workflows, two browser sessions, slow networks, and a restore of the data users value.

### Research Flags

Phases likely needing deeper research during planning:

- **Phase 1:** Select and document the deployment access boundary, backup destination, and timezone-change policy for the real host.
- **Phase 2:** Lock monthly recurrence semantics and build the DST/leap-day/end-of-month test matrix.
- **Phase 3:** Resolve user-facing soft-delete/archive/restore policy before final schema and APIs.
- **Phase 4:** Define precise after-midnight and deleted/completed/re-scheduled task visibility rules for historical daily plans.
- **Phase 6:** Research and specify schedule-aware streak rules before exposing analytics.

Phases with standard patterns (skip research-phase):

- **Phase 5:** Responsive layout, React Query invalidation/refetch, explicit mutation states, accessibility target sizing, Vitest, and Playwright are established patterns; apply the documented contracts.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | MEDIUM | Current official release documentation supports the choices, but exact patch versions must be refreshed and pinned at implementation time. |
| Features | MEDIUM | Project PRD is authoritative and current Todoist/market patterns support the daily workflow; prioritization remains a product choice. |
| Architecture | MEDIUM | Modular-monolith boundaries and PostgreSQL/MDN guidance are well supported; operational access details depend on the selected host. |
| Pitfalls | HIGH | Core date, database-integrity, security, accessibility, and recovery risks are grounded in PostgreSQL, MDN, OWASP, and W3C primary guidance. |

**Overall confidence:** MEDIUM

### Gaps to Address

- **Task deletion/restoration:** choose soft-delete/archive/restore behavior and past-plan visibility before task migrations; do not silently hard-delete history.
- **After-midnight planning:** specify whether editing an explicit prior date is allowed and exactly how scheduled tasks relate to a new local day.
- **Monthly recurrence:** confirm “skip months without the selected day” as the owner’s desired behavior and encode examples/tests.
- **Deployment access:** choose private network/VPN or reverse-proxy authentication before any internet exposure; no account UI does not authorize an open API.
- **Timezone changes:** define whether they are disallowed after setup or require a deliberate migration of future schedule interpretation.

## Sources

### Primary (HIGH confidence)

- [PostgreSQL constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [unique indexes](https://www.postgresql.org/docs/current/indexes-unique.html), and [transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html) — data integrity and concurrent-write controls.
- [MDN Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date), [Temporal.ZonedDateTime](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal/ZonedDateTime), and [date input](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/date) — civil-date and timezone handling.
- [OWASP API Security](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/) and [W3C WCAG target size](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum) — access and mobile interaction safeguards.

### Secondary (MEDIUM confidence)

- [Next.js release guidance](https://nextjs.org/blog), [Prisma Migrate](https://docs.prisma.io/docs/cli/migrate), and [TanStack Query invalidation guidance](https://tanstack.com/query/v5/docs/framework/react/guides/invalidations-from-mutations) — implementation stack and synchronization practices.
- [Todoist Today view guidance](https://www.todoist.com/help/articles/plan-your-day-with-the-today-view-UVUXaiSs) and the [Discipline PRD](../../docs/prd.md) — feature expectations and authoritative project scope.

---
*Research completed: 2026-08-17*
*Ready for roadmap: yes*
