# Discipline

## What This Is

Discipline is a personal development web application where people use Google accounts to manage their own isolated habits and tasks from a PC or phone. It runs against an operator-controlled backend database so each account's activity and planning data synchronize between devices.

## Core Value

Make it fast and trustworthy to plan and act on today's habits and priority tasks from either device.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Track scheduled habits and their completion history.
- [ ] Capture, schedule, and complete tasks with a daily Must/Should plan.
- [ ] Let any Google account register, maintain a 30-day session, and keep its data isolated from every other account.
- [ ] Synchronize each account's personal data between PC and phone through an operator-controlled backend.

### Out of Scope

- Local passwords, password recovery, and identity providers other than Google — Google OAuth is the only v1 sign-in method.
- AI features — explicitly excluded from the product.
- Full offline support and conflict resolution — deferred until after the MVP.
- Google Calendar integration — a future integration requiring OAuth and task/event relationship decisions.
- Off-host backups and disaster recovery — v1 guarantees durable PostgreSQL persistence, not recoverability after host loss or database corruption.

## Context

The product began as a replacement for one owner's habit and task tools and now supports open registration through Google. Every account receives isolated daily, weekly, and monthly habits plus a backburner-based task workflow, scheduled tasks, and daily priorities. It must preserve trustworthy habit history for at least 90 days and make changes on one device visible on another device signed into the same account.

Open product decisions remain: deleted-task retention/restoration and how a changed daily plan should treat scheduled tasks after midnight.

## Constraints

- **Ownership**: Use an operator-controlled PostgreSQL backend rather than a managed application database.
- **Audience**: Public multi-account application — any Google account may register, and all user-owned data must be isolated by account.
- **Identity**: Google OAuth only in v1; sessions last up to 30 days and explicit sign-out revokes access.
- **Devices**: PC and phone web access must operate on synchronized data.
- **Performance**: Planning today's Must and Should tasks should take under two minutes; habit completion or undo should take under ten seconds.
- **Data integrity**: Habit completion is eligible only on or after creation, and at most once for each scheduled day.
- **Persistence**: PostgreSQL data survives application restarts and container replacement; off-host backup and disaster recovery are not v1 guarantees.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Build a web app with an operator-controlled backend database | Supports synchronized PC/phone access without outsourcing the system of record | — Pending |
| Begin with a vertical MVP | Deliver usable habit and daily-task workflows quickly | — Pending |
| Use Google OAuth with open registration and isolated account data | Makes the public deployment a multi-account product without local credential handling | — Pending |
| Guarantee persistence but not disaster recovery in v1 | Keeps the first phase focused on durable live storage rather than backup operations | — Pending |
| Exclude AI from v1 | AI does not support the MVP core value | — Pending |

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
