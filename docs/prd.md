# PRD: Discipline

## Summary

Personal development web app for a single user to:

- Track habits.
- Set recurring and non-recurring tasks with different priorities.
- Analyze data from those two areas.

The app will use a backend database and synchronize the same data between PC and phone.

## Problem

I use multiple tools to solve these needs, but no app covers everything I need in the way I want.

## Users

Myself. I will use the app from my PC and phone, with synchronized data.

## Goals

- Make myself independent from multiple tools and personalize the app to my needs.
- Quickly plan and execute each day from either device.
- Maintain a trustworthy history of habits and tasks.
- Keep control of my data through a self-owned backend database.

## Non-Goals

- Any AI features.

## Out of Scope (For the Moment)

- Account / user login.
- Full offline support and conflict resolution.

## MVP: Habits and Daily Tasks

### Habits

- As a user, I want to create daily habits.
- As a user, I want to create weekly habits (for example, repeating on Monday, Wednesday, and Friday).
- As a user, I want to create monthly habits.
- As a user, I want to mark a habit as completed once per scheduled day.
- As a user, I want to undo a habit completion on the same day.
- As a user, I want to delete habits.
- As a user, I want to edit habits.
- As a user, I want to add an icon to a habit.

Habit rules:

- A habit becomes eligible for completion on its creation date; it is not backfilled for earlier dates.
- A habit can be completed at most once per scheduled day.
- The MVP supports daily, weekly, and monthly schedules. More custom recurrence rules can be added later.

### Tasks

- As a user, I want to create tasks in a backburner, which is the default place for newly created tasks.
- As a user, I want to schedule a task for a specific date and optional time.
- As a user, I want to complete a task from the backburner or any other view where it appears.
- As a user, I want to set one Must Accomplish task for the day.
- As a user, I want to set up to three Should Accomplish tasks for the day.
- As a user, I want to change my Must and Should selections every day.

Task rules:

- Tasks are separate from habits.
- A task has one of these states: uncompleted, completed, or deleted.
- A task remains uncompleted after its due date until I complete, reschedule, or delete it. Overdue tasks must be visibly distinguishable.
- Deleted tasks are excluded from normal views. The retention/restoration behavior is to be decided.
- Each day has exactly one Must task and zero to three Should tasks. Changing a selection only changes that day's plan; it does not alter the task itself.

## Later Milestone: Calendar Analytics

- As a user, I want to see how many habits I completed on past days in a calendar view.
- As a user, I want to see how many days in a row I have completed a specific habit.
- As a user, I want to see which days I have and have not completed a specific habit.

## Future Improvements

- More custom habit recurrence rules.
- Offline support: cache previously loaded data on the device, queue edits made without a connection, and synchronize them with the backend when connectivity returns. This will require a conflict-resolution policy if the same item is changed on PC and phone while offline.

## Future Integrations

- Google Calendar integration: display Google Calendar events and allow creating, editing, and deleting events from this app. This requires connecting the user's Google account through Google OAuth and defining how app tasks relate to Google Calendar events.

## Success Metrics

- I can create or plan today's Must and Should tasks in under two minutes.
- I can mark a scheduled habit complete or undo it from either PC or phone in under 10 seconds.
- Habit completion history remains accurate and visible for at least 90 days.
- A task created, scheduled, completed, or edited on one device is reflected on the other device after synchronization.
- I use the app consistently for at least four weeks without needing another habit or task app for its MVP capabilities.

## Open Questions

- Should deleted tasks be permanently removed, retained for a period, or restorable from an archive?
- What should happen to a scheduled task when a daily Must/Should plan changes after midnight?
