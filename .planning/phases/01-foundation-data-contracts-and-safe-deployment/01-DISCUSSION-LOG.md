# Phase 1: Foundation, Data Contracts, and Safe Deployment - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-08-17
**Phase:** 1-Foundation, Data Contracts, and Safe Deployment
**Areas discussed:** Public access and account scope, persistence boundary, calendar-day ownership, foundation proof workflow

---

## Public access and account scope

### Product access model

| Option | Description | Selected |
|--------|-------------|----------|
| Reverse-proxy credentials | Protect a single-user application before requests reach it | |
| External access service | Use a third-party identity-aware proxy | |
| Private VPN | Restrict the application to enrolled devices | |
| Application accounts | Make sign-in and accounts part of Discipline | ✓ |

**User's choice:** Change the product scope to require sign-in and add accounts.

### Account breadth

| Option | Description | Selected |
|--------|-------------|----------|
| Single-owner account | One predefined owner, no public registration | |
| Invited accounts | Multiple users controlled by the owner | |
| Full account system | Registration, sessions, multiple users, and isolated data | ✓ |
| External identity allowlist | Approved external identities only | |

**User's choice:** Full account system, expanded into Phase 1.

### Sign-in and registration

| Option | Description | Selected |
|--------|-------------|----------|
| Email and password | Local credentials plus verification and reset | |
| Email magic link | Passwordless email-dependent sign-in | |
| Google sign-in | Delegate identity verification to Google | ✓ |
| Email/password plus Google | Multiple sign-in methods | |

| Registration policy | Description | Selected |
|---------------------|-------------|----------|
| Any Google account | Create an account on first successful sign-in | ✓ |
| Admin approval | Hold new accounts pending approval | |
| Invitation only | Permit only pre-approved addresses | |
| Workspace domain | Restrict access to one Google domain | |

**User's choice:** Google-only sign-in with automatic public registration for any Google account.

### Session and deletion lifecycle

| Option | Description | Selected |
|--------|-------------|----------|
| 30-day session | Stay signed in for up to 30 days | ✓ |
| 7-day session | Reauthenticate weekly | |
| Until sign-out | No fixed expiry | |
| Browser session | End when the browser closes | |

| Deletion policy | Description | Selected |
|-----------------|-------------|----------|
| Recovery window | Deactivate, then delete after 30 days | |
| Immediate permanent deletion | Delete account and live data with no recovery | ✓ |
| Deactivate only | Retain data indefinitely | |
| Administrator-managed | Require a manual deletion request | |

**User's choice:** Thirty-day sessions and immediate, irreversible account deletion.

---

## Persistence boundary

| Option | Description | Selected |
|--------|-------------|----------|
| Minimal off-host backup | Nightly encrypted backup with documented recovery | |
| Persistence only | Durable PostgreSQL data without disaster recovery | ✓ |
| Same-server dumps | Local dumps that do not protect against host loss | |
| Full recovery automation | Off-host backups and recurring restore tests | |

**User's choice:** Change Phase 1 to promise persistence only. Remove recoverability, off-host backups, retention, and restore testing.
**Notes:** Backblaze B2 and Restic were considered, then rejected as unnecessary for the desired MVP promise.

---

## Calendar-day ownership

### Authoritative timezone

| Option | Description | Selected |
|--------|-------------|----------|
| Per-account timezone | One IANA timezone shared by all of an account's devices | ✓ |
| Current device timezone | Let each active device define today | |
| Application-wide timezone | Use one timezone for every account | |
| UTC calendar days | Use UTC day boundaries | |

### Timezone onboarding and changes

| Decision | Options considered | Selected |
|----------|--------------------|----------|
| Initial selection | Detect and confirm; detect automatically; manual selection; application default | Detect and confirm |
| Change timing | Immediate; next old-timezone midnight; selected effective date; immutable | Immediate |
| Historical records | Preserve dates; reinterpret/migrate dates | Preserve original dates |
| Change confirmation | Show old/new local dates; save immediately; typed phrase; only allow matching dates | Show dates and confirm |

**User's choice:** Browser-detected, user-confirmed per-account IANA timezone. Changes apply immediately to current and future activity, while historical dates remain unchanged.

---

## Foundation proof workflow

### Tracer workflow

| Option | Description | Selected |
|--------|-------------|----------|
| Account onboarding and settings | Sign-in, timezone onboarding, home, settings, sign-out, deletion | ✓ |
| Minimal task creation | Bring a thin task workflow forward | |
| Minimal habit creation | Bring a thin habit workflow forward | |
| Technical status page | Prove infrastructure without useful user data | |

### Stored identity and shell

| Decision | Options considered | Selected |
|----------|--------------------|----------|
| Stored account fields | Full Google identity fields; minimal identity; editable profile; extended profile | Google ID, email, display name, profile-image URL, timezone |
| Onboarding gate | Require confirmation; provisional access; automatic save; defer until date feature | Require timezone confirmation immediately |
| Signed-in landing | Minimal home plus settings; settings-only home; full placeholder navigation; technical dashboard | Minimal home plus settings |

**User's choice:** A responsive account onboarding and settings slice is the Phase 1 production tracer. Only timezone is editable.

---

## the agent's Discretion

- Authentication library and implementation details.
- Exact hosting provider, responsive styling, schema names, and deployment automation.
- Technical abuse-prevention controls that preserve open Google registration.

## Deferred Ideas

- Off-host backup and disaster recovery.
- Additional identity providers and local credentials.
- Habit, task, and daily-planning UI, which remain in later phases.
