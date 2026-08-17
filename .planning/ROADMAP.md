# Roadmap: Discipline

## Overview

Discipline reaches its MVP through five delivery boundaries: establish a safe, recoverable shared foundation; prove the daily habit loop; add the independent task lifecycle; layer dated Must/Should planning into Today; then validate the dependable PC-and-phone experience. Each phase strengthens the user's ability to plan and act on the same trusted personal data from either device.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation, Data Contracts, and Safe Deployment** - Establish a responsive, self-owned, recoverable data foundation.
- [ ] **Phase 2: Habit Tracking** - Deliver scheduled habits, completion/undo, and durable dated history.
- [ ] **Phase 3: Task Lifecycle and Backburner** - Deliver capture, scheduling, completion, deletion, and overdue task workflows.
- [ ] **Phase 4: Dated Daily Planning and Today** - Deliver constrained Must/Should plans and a unified daily view.
- [ ] **Phase 5: Cross-Device Reliability and Release Validation** - Make the full daily workflow trustworthy, accessible, and reliable on PC and phone.

## Phase Details

### Phase 1: Foundation, Data Contracts, and Safe Deployment
**Goal**: User data has a responsive, secure, self-owned, and recoverable foundation for daily use from PC and phone.
**Mode:** mvp
**Depends on**: Nothing (first phase)
**Requirements**: SYNC-01, SYNC-04
**Success Criteria** (what must be TRUE):
  1. User can open the responsive web app on a PC or phone while both use the same self-owned backend database.
  2. User's saved habit and task data remains available after an application restart and can be recovered from the backend database.
**Plans**: TBD
**UI hint**: yes

### Phase 2: Habit Tracking
**Goal**: User can manage scheduled habits and trust the current-day completion record.
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: HAB-01, HAB-02, HAB-03, HAB-04, HAB-05, HAB-06
**Success Criteria** (what must be TRUE):
  1. User can create, edit, or delete a habit with an optional icon and a daily, selected-weekday, or monthly schedule.
  2. User can view only habits eligible for the current local date, never before a habit's creation date.
  3. User can mark an eligible habit complete once for its scheduled day from either device and undo that completion on the same local day.
  4. User can view accurate dated completion history that is retained for at least 90 days.
**Plans**: TBD
**UI hint**: yes

### Phase 3: Task Lifecycle and Backburner
**Goal**: User can capture, organize, and complete discrete tasks without losing important historical context.
**Mode:** mvp
**Depends on**: Phase 2
**Requirements**: TASK-01, TASK-02, TASK-03, TASK-04, TASK-05
**Success Criteria** (what must be TRUE):
  1. User can capture a new task in the backburner, then edit, schedule or reschedule it for a date with an optional time, complete it, or delete it.
  2. User can complete a task from the backburner or another task view where it appears.
  3. User can distinguish an overdue, uncompleted task from other uncompleted tasks until it is completed, rescheduled, or deleted.
  4. User can delete a task so it no longer appears in normal task views while dated-plan history can be preserved.
**Plans**: TBD
**UI hint**: yes

### Phase 4: Dated Daily Planning and Today
**Goal**: User can quickly commit to a focused daily plan and act on all of today's work in one view.
**Mode:** mvp
**Depends on**: Phase 3
**Requirements**: PLAN-01, PLAN-02, PLAN-03, PLAN-04, PLAN-05
**Success Criteria** (what must be TRUE):
  1. User can select exactly one uncompleted task as Must Accomplish for a chosen local date.
  2. User can select zero to three uncompleted tasks as Should Accomplish for a chosen local date, and cannot exceed that limit.
  3. User can replace or remove a date's Must and Should selections without changing the underlying tasks or a different date's plan.
  4. User can use a Today view that combines today's eligible habits with the date's Must, Should, scheduled, and overdue tasks.
  5. User can create or revise today's Must and Should plan in under two minutes.
**Plans**: TBD
**UI hint**: yes

### Phase 5: Cross-Device Reliability and Release Validation
**Goal**: User can rely on the complete habit and daily-task workflow from either device under normal connectivity.
**Mode:** mvp
**Depends on**: Phase 4
**Requirements**: SYNC-02, SYNC-03, QUAL-01, QUAL-02, QUAL-03
**Success Criteria** (what must be TRUE):
  1. User sees habit and task creations, edits, scheduling changes, completions, and deletions made on one device after synchronization on the other device.
  2. User receives a clear saved, retry, or refresh outcome whenever a data change cannot synchronize.
  3. User can complete or undo a scheduled habit from PC or phone in under ten seconds under normal connectivity.
  4. User's habit completions and daily-plan limits stay correct when updates are retried or sent concurrently from two devices.
  5. User can complete core habit and task workflows with touch-friendly mobile controls and accessible feedback.
**Plans**: TBD
**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation, Data Contracts, and Safe Deployment | 0/TBD | Not started | - |
| 2. Habit Tracking | 0/TBD | Not started | - |
| 3. Task Lifecycle and Backburner | 0/TBD | Not started | - |
| 4. Dated Daily Planning and Today | 0/TBD | Not started | - |
| 5. Cross-Device Reliability and Release Validation | 0/TBD | Not started | - |
