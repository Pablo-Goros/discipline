# Requirements: Discipline

**Defined:** 2026-08-17
**Core Value:** Make it fast and trustworthy to plan and act on today's habits and priority tasks from either device.

## v1 Requirements

### Accounts & Identity

- [ ] **AUTH-01**: Any user with a Google account can sign in and automatically create a Discipline account.
- [ ] **AUTH-02**: Each account's habits, tasks, daily plans, settings, and lifecycle data are isolated from every other account.
- [ ] **AUTH-03**: User sessions last up to 30 days, and explicit sign-out revokes the active session.
- [ ] **AUTH-04**: A first-time user must confirm a browser-detected IANA timezone before entering the app and can later change it after confirming the old and new local dates.
- [ ] **AUTH-05**: A user can immediately and permanently delete their account and all user-owned live data after strong confirmation.

### Foundation & Synchronization

- [ ] **SYNC-01**: User can use the responsive web app from both a PC and phone against the same operator-controlled backend database while signed into the same account.
- [ ] **SYNC-02**: A habit or task created, edited, scheduled, completed, or deleted on one device is reflected after synchronization when viewed from the other device.
- [ ] **SYNC-03**: User can see a clear save, retry, or refresh outcome when a data change cannot be synchronized.
- [ ] **SYNC-04**: User data persists in PostgreSQL across application restarts and application-container replacement.

### Habits

- [ ] **HAB-01**: User can create a habit with a name, optional icon, and daily, selected-weekday, or monthly schedule.
- [ ] **HAB-02**: User can edit or delete a habit.
- [ ] **HAB-03**: User can see only habits eligible for the current local calendar day, beginning no earlier than each habit's creation date.
- [ ] **HAB-04**: User can mark an eligible habit complete once for its scheduled local day from either device.
- [ ] **HAB-05**: User can undo a habit completion on the same local day.
- [ ] **HAB-06**: User can retain and view accurate dated habit completion history for at least 90 days.

### Tasks

- [ ] **TASK-01**: User can create a task in the backburner, the default location for new tasks.
- [ ] **TASK-02**: User can edit, schedule for a specific date with an optional time, reschedule, complete, or delete a task.
- [ ] **TASK-03**: User can complete a task from the backburner or any other view where it appears.
- [ ] **TASK-04**: User can distinguish an overdue uncompleted task from other uncompleted tasks until it is completed, rescheduled, or deleted.
- [ ] **TASK-05**: Deleted tasks are excluded from normal task views while preserving the history needed by dated daily plans.

### Daily Planning

- [ ] **PLAN-01**: User can select exactly one uncompleted task as the Must Accomplish task for a chosen local date.
- [ ] **PLAN-02**: User can select zero to three uncompleted tasks as Should Accomplish tasks for a chosen local date.
- [ ] **PLAN-03**: User can replace or remove a date's Must and Should selections without changing the underlying tasks or another date's plan.
- [ ] **PLAN-04**: User can use a Today view that combines today's eligible habits with the current date's Must, Should, scheduled, and overdue tasks.
- [ ] **PLAN-05**: User can create or revise today's Must and Should plan in under two minutes.

### Quality & Reliability

- [ ] **QUAL-01**: User can complete or undo a scheduled habit from PC or phone in under ten seconds under normal connectivity.
- [ ] **QUAL-02**: Habit completion and daily-plan limits remain correct when updates are retried or sent concurrently from two devices.
- [ ] **QUAL-03**: User can complete core habit and task workflows with touch-friendly mobile controls and accessible feedback.

## v2 Requirements

### Habit Analytics

- **ANLY-01**: User can view how many habits were completed on each past day in a calendar.
- **ANLY-02**: User can see a schedule-aware completion streak for a specific habit.
- **ANLY-03**: User can see which scheduled days for a specific habit were completed and missed.

### Recurrence, Offline & Integrations

- **RECR-01**: User can define custom habit recurrence rules beyond daily, selected-weekday, and monthly schedules.
- **OFFL-01**: User can work from cached data while offline and queue changes for later synchronization with a defined conflict-resolution policy.
- **GCAL-01**: User can connect Google Calendar and manage Google Calendar events from Discipline.

## Out of Scope

| Feature | Reason |
|---------|--------|
| Local passwords, password recovery, and non-Google identity providers | V1 uses Google OAuth exclusively. |
| AI features | Explicitly excluded by the PRD. |
| Full offline support and conflict resolution | Deferred until the online-first MVP workflow is proven. |
| Native mobile applications | A responsive web app is sufficient for the PC/phone MVP. |
| Google Calendar integration | Requires OAuth and task/event policy decisions; deferred. |
| Collaboration, sharing, or team features | Accounts are isolated personal workspaces; sharing and team workflows remain out of scope. |
| Off-host backups and disaster recovery | V1 guarantees durable live PostgreSQL persistence, not recovery from host loss or database corruption. |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 1 | Pending |
| AUTH-02 | Phase 1 | Pending |
| AUTH-03 | Phase 1 | Pending |
| AUTH-04 | Phase 1 | Pending |
| AUTH-05 | Phase 1 | Pending |
| SYNC-01 | Phase 1 | Pending |
| SYNC-02 | Phase 5 | Pending |
| SYNC-03 | Phase 5 | Pending |
| SYNC-04 | Phase 1 | Pending |
| HAB-01 | Phase 2 | Pending |
| HAB-02 | Phase 2 | Pending |
| HAB-03 | Phase 2 | Pending |
| HAB-04 | Phase 2 | Pending |
| HAB-05 | Phase 2 | Pending |
| HAB-06 | Phase 2 | Pending |
| TASK-01 | Phase 3 | Pending |
| TASK-02 | Phase 3 | Pending |
| TASK-03 | Phase 3 | Pending |
| TASK-04 | Phase 3 | Pending |
| TASK-05 | Phase 3 | Pending |
| PLAN-01 | Phase 4 | Pending |
| PLAN-02 | Phase 4 | Pending |
| PLAN-03 | Phase 4 | Pending |
| PLAN-04 | Phase 4 | Pending |
| PLAN-05 | Phase 4 | Pending |
| QUAL-01 | Phase 5 | Pending |
| QUAL-02 | Phase 5 | Pending |
| QUAL-03 | Phase 5 | Pending |

**Coverage:**
- v1 requirements: 28 total
- Mapped to phases: 28
- Unmapped: 0 ✓

---
*Requirements defined: 2026-08-17*
*Last updated: 2026-08-17 after initial roadmap creation*
