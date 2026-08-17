# Discipline

## What This Is

Discipline is a personal development web application for one user to manage habits and tasks from a PC or phone. It provides a self-owned backend database so activity and planning data synchronize between devices, replacing multiple separate tools with a workflow tailored to the user's daily practice.

## Core Value

Make it fast and trustworthy to plan and act on today's habits and priority tasks from either device.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Track scheduled habits and their completion history.
- [ ] Capture, schedule, and complete tasks with a daily Must/Should plan.
- [ ] Synchronize the same personal data between PC and phone through a self-owned backend.

### Out of Scope

- Account and user login — the MVP is explicitly for one user.
- AI features — explicitly excluded from the product.
- Full offline support and conflict resolution — deferred until after the MVP.
- Google Calendar integration — a future integration requiring OAuth and task/event relationship decisions.

## Context

The product replaces multiple existing habit and task tools for its single owner. The MVP covers daily, weekly, and monthly habits plus a backburner-based task workflow, scheduled tasks, and daily priorities. It must preserve trustworthy habit history for at least 90 days and make changes on one device visible on the other after synchronization.

Open product decisions remain: deleted-task retention/restoration and how a changed daily plan should treat scheduled tasks after midnight.

## Constraints

- **Ownership**: Use a self-owned backend database — the user wants control of personal data.
- **Audience**: Single user only — authentication and multi-user features are not MVP requirements.
- **Devices**: PC and phone web access must operate on synchronized data.
- **Performance**: Planning today's Must and Should tasks should take under two minutes; habit completion or undo should take under ten seconds.
- **Data integrity**: Habit completion is eligible only on or after creation, and at most once for each scheduled day.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build a web app with a self-owned backend database | Supports PC/phone access while retaining data ownership | — Pending |
| Begin with a vertical MVP | Deliver usable habit and daily-task workflows quickly | — Pending |
| Exclude authentication and AI from v1 | The application is for its single owner; neither supports the MVP core value | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `$gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `$gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-08-17 after initialization*
