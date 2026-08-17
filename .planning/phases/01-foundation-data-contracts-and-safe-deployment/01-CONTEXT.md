# Phase 1: Foundation, Data Contracts, and Safe Deployment - Context

**Gathered:** 2026-08-17
**Status:** Ready for scope reconciliation, then planning

<domain>
## Phase Boundary

Phase 1 establishes Discipline as a publicly reachable, responsive multi-account web application. It delivers Google sign-in, automatic account registration, isolated account data, required timezone onboarding, a signed-in home and account settings, durable PostgreSQL persistence, session management, sign-out, and permanent account deletion. Habit, task, and daily-planning workflows remain in their later roadmap phases.

This discussion intentionally changes the earlier single-user/no-auth and backup/recovery assumptions. Project-level scope, requirements, roadmap criteria, and stack research must be reconciled with these decisions before Phase 1 is planned.

</domain>

<decisions>
## Implementation Decisions

### Public accounts and access
- **D-01:** Discipline is publicly reachable over HTTPS and requires application-level sign-in; an external reverse-proxy password or identity gate is not the product access model.
- **D-02:** V1 is a full multi-account product. Every account's habits, tasks, daily plans, settings, and lifecycle data must be isolated from every other account. — **Reversibility:** costly — Undoing multi-account ownership would require schema and query changes across every user-owned domain record.
- **D-03:** Google OAuth is the only sign-in method in Phase 1; there are no local passwords, password-reset emails, magic links, or additional identity providers.
- **D-04:** Any Google account may register. The first successful Google sign-in creates the corresponding Discipline account automatically.
- **D-05:** A session lasts up to 30 days; explicit sign-out revokes it earlier.
- **D-06:** Account deletion immediately and permanently removes the live account and all user-owned data after strong confirmation, with no recovery window. — **Reversibility:** one-way — With backups explicitly out of scope, deleted account data cannot be recovered.

### Persistence boundary
- **D-07:** PostgreSQL data must survive application restarts and application-container replacement through a durable database volume.
- **D-08:** Phase 1 promises persistence only, not disaster recovery. Do not add off-host backups, Backblaze B2, Restic, backup retention, recurring restore tests, or a recoverability claim.

### Calendar-day ownership
- **D-09:** Each account stores one authoritative IANA timezone. All devices use that account timezone to determine calendar dates such as "today."
- **D-10:** During first-sign-in onboarding, detect the browser's IANA timezone and require the user to confirm it before the main application becomes available.
- **D-11:** A later timezone change takes effect immediately for current and future activity. Existing dated habit completions, task dates, and daily plans keep their original calendar dates. — **Reversibility:** costly — Changing this rule after dated activity exists may require reinterpretation or migration of historical records.
- **D-12:** Before saving a timezone change, show the old and new local calendar date and require confirmation.

### Phase 1 proof workflow
- **D-13:** The production-quality tracer is account onboarding and settings: Google sign-in, automatic account creation, timezone confirmation, responsive signed-in home, account settings, timezone change, sign-out, and permanent account deletion.
- **D-14:** Store the Google identity identifier, email, display name, profile-image URL, and confirmed IANA timezone. Only the timezone is user-editable in Phase 1.
- **D-15:** After onboarding, show a minimal responsive home containing recognizable account identity, confirmed timezone, settings access, and sign-out. Do not add unfinished Today, Habits, or Tasks placeholder navigation.

### the agent's Discretion
- Authentication library and adapter configuration, provided the implementation honors Google-only sign-in and the account/session decisions above.
- Exact hosting provider, domain, responsive visual styling, database schema names, and deployment automation.
- Technical abuse-prevention mechanisms required for safe public registration, provided they do not introduce invitation or approval gates.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.** Where an older artifact conflicts with the numbered decisions above, this context records the newer user-approved scope and wins until the project artifacts are reconciled.

### Product scope and phase contract
- `.planning/PROJECT.md` — Original product definition and constraints; its single-user/no-auth and recovery statements are superseded by D-01 through D-08.
- `.planning/REQUIREMENTS.md` — Existing requirement inventory and phase traceability; must be revised to add account requirements and change `SYNC-04` from recoverability to persistence only.
- `.planning/ROADMAP.md` — Existing five-phase delivery sequence; Phase 1 must be expanded with the account scope and revised persistence success criterion.
- `docs/prd.md` — Original PRD and product behavior; its account-login exclusion is superseded by D-01 through D-06.

### Technical baseline
- `.planning/research/STACK.md` — Baseline Next.js/PostgreSQL/Prisma deployment stack; its explicit no-auth choice is superseded, and its previously deferred authentication option must be re-evaluated.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None. The repository currently contains planning documents and a minimal README; there is no application scaffold or reusable source code.

### Established Patterns
- Greenfield implementation must establish the project structure, database schema, authentication boundary, responsive shell, tests, and deployment conventions.
- The selected baseline is a Next.js modular monolith with PostgreSQL and Prisma, subject to refreshed authentication research.

### Integration Points
- Google OAuth application credentials and callback URL.
- PostgreSQL-backed account, session, and user-settings data.
- Caddy HTTPS reverse proxy and public application origin.
- Durable PostgreSQL volume across application-container replacement.

</code_context>

<specifics>
## Specific Ideas

- Account creation should feel automatic on first Google sign-in, followed by exactly one required timezone-confirmation step.
- The Phase 1 home should be a real, minimal product surface rather than a technical status dashboard or unfinished domain-feature shell.
- Account deletion must clearly communicate that it is immediate and irreversible.

</specifics>

<deferred>
## Deferred Ideas

- Off-host backups and disaster recovery are explicitly deferred; Phase 1 guarantees persistence only.
- Additional identity providers and local credential recovery are outside Phase 1.
- Habit, task, and daily-planning interfaces remain assigned to their existing later phases.

</deferred>

---

*Phase: 1-Foundation, Data Contracts, and Safe Deployment*
*Context gathered: 2026-08-17*
