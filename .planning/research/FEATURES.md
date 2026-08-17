# Feature Landscape

**Domain:** Personal single-user habit and daily-task web app
**Researched:** 2026-08-17
**Confidence:** MEDIUM — based on current official Todoist documentation, current market comparisons, and the project PRD; the MVP scope is primarily a product decision rather than a generic market prescription.

## Product Positioning

Discipline should be a quick personal daily-practice tool, not a general productivity suite. Its MVP wins when the owner can open one responsive web app on a phone or PC, see what matters today, mark a habit or task complete, and trust that the record is preserved everywhere. Habits and tasks must remain separate concepts: habits are scheduled practices with a completion record; tasks are discrete work items that move from a backburner into a date and, optionally, a daily plan.

Current task-management patterns reinforce the need for a focused Today view, a small set of highest-priority work, completion history, scheduling, rescheduling, and explicit overdue review. Current habit products commonly add reminders, streaks, calendar history, and statistics; the first release should retain the underlying history needed for those features without making analytics a dependency for daily use.

## Table Stakes

Features users need for the stated MVP. Omission would make the product unable to replace the owner's existing habit and task tools.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Responsive PC and phone access with synchronized data | The core use case explicitly alternates devices; a change must appear on the other device after synchronization. | High | Backend persistence and refresh/error states are prerequisites for every mutating feature. Full offline conflict resolution is not required. |
| Habit creation and management | A tracker must let the owner name, edit, delete, and recognize a habit. | Medium | Support an optional icon. Changes apply prospectively; retain the historical completion record. |
| Daily, selected-weekday, and monthly habit schedules | These are the PRD's supported recurrence patterns and cover the daily routine use case. | Medium | Do not substitute generic cron/RRULE rules in MVP. A habit is not eligible before its creation date. |
| Today's eligible-habit list | The daily action surface must show only habits that can be completed today. | Medium | It should be a fast, mobile-friendly checklist rather than an analytics dashboard. |
| One-tap habit completion and same-day undo | Low-friction completion is fundamental to a tracker and directly serves the under-10-second success metric. | Medium | Enforce at most one completion per scheduled day; undo reverses that day's completion only. |
| Durable habit completion history | Users need a trustworthy record, and the project promises at least 90 days of accurate visible history. | Medium | Store individual dated events, not only a mutable streak counter. Calendar analytics can consume this later. |
| Backburner task capture | An unscheduled default inbox prevents new tasks from being lost while keeping the daily plan intentional. | Low | New tasks enter the backburner unless explicitly scheduled. |
| Task lifecycle: schedule, complete, edit, delete, reschedule | Scheduling and moving unfinished work are standard personal task-manager behavior. | Medium | Support specific date and optional time; task state is uncompleted, completed, or deleted. |
| Clear overdue-task treatment | A task should remain actionable after its date passes, but must be visually distinct so it can be rescheduled or completed deliberately. | Low | Never silently discard or auto-complete overdue tasks. |
| Daily Must/Should plan | The owner needs one Must and up to three Should tasks, adjusted per date without changing the underlying tasks. | Medium | This is the app's concrete priority model, preferable to exposing a broad four-level priority system. |
| Fast Today/planning surface | A daily list must combine today's scheduled tasks, overdue tasks, and the Must/Should selection flow well enough to plan in under two minutes. | Medium | The interface must make it obvious which item is a habit versus a task. |

## Differentiators

Features that make Discipline more useful than a generic habit tracker plus a generic to-do list.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| One deliberate daily-practice hub | Shows eligible habits alongside a concise task plan while preserving distinct data models and actions. | Medium | The primary differentiator: a unified daily experience without pretending habits are recurring tasks. |
| Constrained daily planning: exactly one Must, zero to three Shoulds | Turns an unbounded task list into a realistic daily commitment and reduces choice overload. | Medium | Enforce the cap in UI and backend; plans are date-specific snapshots/references, not task fields. |
| Backburner-to-day workflow | Lets the owner capture freely, schedule selectively, then promote only a few tasks into today's commitments. | Medium | The hierarchy is intentionally simpler than projects, labels, boards, and complex filters. |
| Trust-first habit rules and undo | Prospective eligibility plus an idempotent once-per-day completion record protects the meaning of history while still allowing correction. | Medium | Make undo prominent and immediate; do not retroactively backfill when creating a habit. |
| Self-owned, single-user data model | Avoids account, subscription, and collaboration overhead while giving the owner control over synchronized personal history. | Medium | A deployment/ownership differentiator; it should not force an authentication system into the MVP. |
| Analytics-ready event history | Individual completion dates make later calendar coverage, streaks, and missed-day views possible without a schema rewrite. | Low now / Medium later | Capture the data in MVP; expose the calendar analytics in the later milestone. |

## Later-Milestone Features

These features have clear user value but should follow a stable daily workflow.

| Feature | Value Proposition | Complexity | Why Later |
|---------|-------------------|------------|-----------|
| Habit calendar coverage | Shows how many habits were completed on each past day. | Medium | Depends on reliable event history and is not needed to complete today's habits. |
| Per-habit streak and completion calendar | Makes consistency and missed dates visible for a chosen habit. | Medium | Requires precise definitions for schedule-aware streak calculation; ship after schedule semantics are proven. |
| Browser/mobile reminders | Helps prompt behavior at the right time. | Medium | Notification permissions, delivery reliability, and scheduling policy add operational complexity without being needed for the first daily workflow. |
| Custom recurrence rules | Supports more personalized patterns. | High | Must not distort or invalidate the simple daily/weekly/monthly eligibility rules. |
| Offline queue and conflict resolution | Lets both devices mutate data while disconnected. | High | Needs an explicit conflict policy and is expressly out of MVP scope. |
| Google Calendar integration | Brings external events into planning. | High | Requires OAuth, data-sync decisions, and a task/event relationship model. |

## Anti-Features

Features to explicitly avoid in this milestone so the personal daily loop stays fast, understandable, and trustworthy.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Accounts, login, sharing, or team collaboration | The application has one owner; identity, permissions, invitations, and shared-task semantics add no MVP value. | Use a single-user deployment with a self-owned database. |
| AI coaching, automatic prioritization, or generated plans | Explicitly out of scope and would obscure the owner's deliberate Must/Should choice. | Keep planning manual and visible. |
| Treating habits as recurring tasks | This loses schedule-aware completion history and makes prospective eligibility/undo semantics ambiguous. | Maintain separate habit and task models that share the Today surface only. |
| General project management: teams, assignees, projects, labels, subtasks, boards, custom filters | These create setup and navigation burden without advancing the personal daily-use goal. | Use the backburner, scheduled date, overdue state, and Must/Should plan as the MVP's organizing model. |
| Gamification, social accountability, leaderboards, or rewards | They address a different motivation model and can make a private consistency tool feel noisy or punitive. | Provide accurate history first; validate whether streak visualizations are useful later. |
| Full offline editing in MVP | Cross-device concurrent edits need conflict-resolution rules to avoid corrupting trusted history. | Require connectivity for mutations and communicate sync failures clearly; add offline support later. |
| Calendar/OAuth integrations in MVP | They make the app's data boundary, deployment, and task/event semantics much more complex. | First prove the standalone daily workflow. |
| Broad recurrence language or arbitrary custom schedules | Increases edge cases around eligibility, missed days, month boundaries, and historical edits. | Support only daily, selected weekdays, and monthly schedules initially. |

## Feature Dependencies

```text
Self-owned backend + sync
  -> habit/task persistence and cross-device refresh
  -> trustworthy history and all daily mutations

Habit lifecycle + supported schedule rules
  -> date-specific eligibility
  -> Today's habit list
  -> one-per-scheduled-day completion and undo
  -> future calendar coverage and streak analytics

Task lifecycle + backburner
  -> date/time scheduling and rescheduling
  -> overdue identification
  -> Today's task list
  -> date-specific Must/Should plan

Today's habit list + Today's task list
  -> unified daily-practice hub
```

## MVP Recommendation

Prioritize:

1. **Shared persistence and responsive daily shell** — establish the self-owned backend, mobile/PC access, and reliable synchronization before layering product features.
2. **Habit vertical slice** — create/edit/delete habits; daily, weekly, and monthly eligibility; fast completion/undo; immutable dated history.
3. **Task vertical slice** — backburner capture, schedule/reschedule, completion/deletion, and an explicit overdue view.
4. **Daily planning vertical slice** — Today view with exactly one Must and zero to three Shoulds, without mutating the task itself.

Defer:

- **Calendar analytics and streaks:** capture accurate dated events now, then add schedule-aware visualizations in the next milestone.
- **Reminders, custom recurrence, offline conflict resolution, and Google Calendar:** each creates its own policy and reliability surface; none is essential to the MVP success metrics.
- **Deleted-task restoration:** decide the retention policy before the task slice is considered final; the UI should not imply restore is available until that rule is chosen.

## Sources

- [Todoist — Plan your day with the Today view](https://www.todoist.com/help/articles/plan-your-day-with-the-today-view-UVUXaiSs) — official documentation, updated July 2026; MEDIUM confidence through verified Brave discovery.
- [Todoist — Get started with Todoist](https://www.todoist.com/help/articles/get-started-with-todoist-OgNNJR) — official documentation, updated July 2026; MEDIUM confidence through verified Brave discovery.
- [Todoist — Complete a task with a recurring date](https://www.todoist.com/help/articles/complete-a-task-with-a-recurring-date-dmI6SVqdP) — official documentation, updated June 2026; MEDIUM confidence through verified Brave discovery.
- [ClickUp — Habit tracking apps: streaks and analytics compared](https://clickup.com/learn/topic/productivity/tools/features/habit-tracking/) — current comparative market source; MEDIUM confidence through verified Brave discovery.
- [Discipline PRD](../../docs/prd.md) — primary project scope and explicit product decisions.
