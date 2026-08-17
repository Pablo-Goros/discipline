# Technology Stack

**Project:** Discipline
**Researched:** 2026-08-17
**Confidence:** MEDIUM — recommendations are based on current official documentation and releases; exact patch versions must be refreshed immediately before implementation.

## Recommended Stack

Discipline should be a single TypeScript Next.js application backed by a separately running, self-owned PostgreSQL database.  Use a responsive web UI rather than a native mobile app: the product is one person using a PC and phone, so a deployable web application is the shortest path to the same synchronized experience on both devices.  Keep the first release a modular monolith—there is no scale or team boundary that warrants a separate API service, event bus, cache, or mobile codebase.

### Core Framework

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Node.js | 24.19 LTS | Development and application runtime | Current LTS line at research time; use its supported, predictable runtime rather than Node Current. Pin this in `.nvmrc`/container image. |
| Next.js | 16.2 LTS, at least `16.2.11` | Full-stack responsive web app | App Router gives route layouts and server rendering; Route Handlers provide a small same-origin JSON mutation/query API without maintaining a separate backend. Stay on the maintained 16.2 security-patch line, not the 16.3 preview. |
| React + React DOM | 19.2 | UI runtime | Version aligned with Next.js 16.2. Use Server Components for read-heavy pages and Client Components only for interactive forms, completion controls, and local UI state. |
| TypeScript | 5.9 | Application types | Strict TypeScript makes schedule, task-state, and daily-plan invariants visible at compile time. Enable `strict`, `noUncheckedIndexedAccess`, and `exactOptionalPropertyTypes`. |
| Tailwind CSS | 4.3 | Responsive styling and design tokens | Fast, CSS-first configuration and excellent mobile-first composition. It avoids building and maintaining a bespoke CSS architecture for a personal app. |
| shadcn/ui + Radix primitives | current compatible releases | Accessible controls and dialogs | Copy the small set of generated, locally owned components needed for the MVP (button, input, select, dialog, popover, calendar). This keeps control of UI code and avoids a heavy, opaque component suite. |

### Database

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| PostgreSQL | 18 series; start with patched `18.4` or newer | Self-owned system of record | A relational database naturally models habits, tasks, schedule occurrences, and one-per-day daily plans. Transactions, foreign keys, `CHECK`s, and composite `UNIQUE` constraints make the PRD's integrity rules durable across simultaneous PC/phone requests. Use the official `postgres:18` image and apply security patches promptly. |
| Prisma ORM + Prisma Client | 7.x | Typed data access and checked-in schema migrations | Prisma 7 is current GA. Its schema keeps application types and database mapping together, while Prisma Migrate provides reviewable migration history. Use Prisma for routine queries, with reviewed SQL migrations for constraints or indexes that the schema cannot express cleanly. |
| Prisma Migrate | 7.x | Development and production database migration workflow | Create migrations with `prisma migrate dev`; commit `prisma/schema.prisma` and the migration directory; run `prisma migrate deploy` as the deployment migration step. Never use `migrate reset` against the self-owned production data. |

Database rules must not be enforced only in the browser or ORM. At minimum, the initial migration should create: `UNIQUE (habit_id, scheduled_date)` for completions; a `scheduled_date` SQL `DATE` (not a device-local timestamp); constrained task status values; and a transactionally enforced one-Must/up-to-three-Should daily-plan model. Store instants such as completion/update time as `timestamptz` in UTC, but resolve habit eligibility and plan membership against the user's configured IANA time zone.

### Infrastructure

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Docker Compose | Compose Specification / current Docker Engine | Repeatable self-hosting | Run only application and PostgreSQL services in one declarative deployment. A named PostgreSQL volume keeps data independent of application container replacement. |
| Caddy | 2.x current stable | HTTPS reverse proxy and automatic certificates | A compact, self-hostable reverse proxy with automatic TLS; terminate public HTTPS there and expose PostgreSQL only to the Compose network. |
| Restic | 0.18+ | Encrypted off-host backups | A self-hosted database is only owned if it is recoverable. Run a scheduled `pg_dump` (or physical backup) and back up the output to a separately controlled destination using Restic encryption. Test restoration before relying on it. |
| GitHub Actions (or host-local CI) | current hosted runner | Checks and image build | Run typecheck, unit tests, Playwright smoke tests, and migration validation on every change. It is CI only; it is not the data host. |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `zod` | 4.x | Validate Route Handler inputs and share schemas with forms | Validate every mutation payload at the server boundary; do not trust client form validation. |
| `react-hook-form` | 7.x | Performant task/habit forms | Use for create/edit and daily-plan forms, with a Zod resolver. Keep simple completion buttons as direct actions. |
| `@tanstack/react-query` | 5.x | Client query cache, mutation state, and refetching | Use on interactive client views. Invalidate focused keys after successful mutations and refetch on window focus so the second device sees changes naturally. Do not duplicate data in another global state store. |
| `date-fns` | 4.x | Date display and pure calendar calculations | Use only after explicitly converting between the user's time zone and UTC. Represent schedule dates as `YYYY-MM-DD`/SQL `DATE`; never infer a day from a browser-local `Date` alone. |
| `lucide-react` | current compatible release | Habit and task icons | Provides the requested icon selection without shipping a custom icon system. Persist the chosen icon's stable string name, not SVG markup. |
| `vitest` + Testing Library | 4.x + current | Fast unit/component tests | Test recurrence expansion, eligibility, task-state transitions, and plan-limit logic. |
| Playwright | current compatible release | End-to-end browser checks | Cover phone-sized and desktop workflows plus the two-device synchronization acceptance case. |

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Full-stack framework | Next.js modular monolith | React + Vite plus a separate FastAPI/Express backend | A separate API, deployment, validation boundary, and authentication surface adds operational work without delivering MVP value. |
| Database | Self-hosted PostgreSQL | SQLite | SQLite is excellent for one local device, but does not itself provide the required PC/phone synchronized backend. |
| Data access | Prisma 7 + SQL migrations | Drizzle ORM | Drizzle is viable, but Prisma's migration workflow and generated client are a clearer fit for this greenfield app. Do not mix ORMs. |
| API style | Next.js Route Handlers | tRPC / GraphQL | The app needs a small, same-origin internal API. Route Handlers plus Zod are simpler to debug, deploy, and later expose to a PWA if needed. |
| Synchronization | Request/response + React Query invalidation/refetch | WebSockets, CRDTs, or event sourcing | The PRD requires synchronization after normal requests, not live collaboration or offline conflict resolution. Add polling/websocket notifications only if real usage shows a need. |
| UI | Tailwind + selected shadcn components | Material UI or a paid component suite | A large design system would add weight and theming decisions to a highly focused personal app. |
| Authentication | No auth in MVP | Better Auth | Better Auth is self-hostable and supports PostgreSQL/Prisma, but it introduces user, session, and account schema expressly excluded by the PRD. Reconsider it only when the project adds a real access-control requirement. |
| Hosting | Docker Compose on an owned VPS/home server | Vercel/Supabase managed services | These are convenient but do not satisfy the stated self-owned backend-database constraint as directly. |

## Explicit Non-Choices for the MVP

- Do not build native iOS/Android clients, a PWA offline queue, CRDT syncing, service-worker conflict resolution, or a second API service.
- Do not add Redis, Kafka, Elasticsearch, a message queue, a vector database, analytics SaaS, or AI services. None supports the core daily habit/task loop.
- Do not implement user accounts, social sign-in, role-based access, or Better Auth yet. Protect the single-user deployment at the network/reverse-proxy layer and revisit an application-level owner gate before making it publicly reachable.
- Do not use `postgres:latest`, unpinned npm ranges without a lockfile, or a database exposed on the public Internet. Pin the tested container/image digest and commit the package lockfile.

## Installation

Use `pnpm` for deterministic, space-efficient dependency management. The commands below establish compatible major versions; generate and commit `pnpm-lock.yaml` so the deployment uses the tested exact patches.

```bash
# Scaffold with the currently patched Next.js 16.2 line and TypeScript
pnpm create next-app@latest discipline --ts --tailwind --app --src-dir --use-pnpm
cd discipline
pnpm add next@16.2.11 react@19.2 react-dom@19.2 @prisma/client@7 zod@4 \
  react-hook-form@7 @hookform/resolvers@5 @tanstack/react-query@5 \
  date-fns@4 lucide-react

# Database, validation, and browser-test tooling
pnpm add -D prisma@7 vitest@4 @testing-library/react @testing-library/jest-dom \
  playwright eslint prettier
pnpm prisma init
```

At implementation time, use `pnpm outdated` and the official release notes to update the sample patch versions before the first lockfile is committed. Use a `compose.yaml` with `app`, `db`, and `caddy` services; set all database/app secrets from an uncommitted environment file or secret store.

## Sources

- [Next.js blog and release guidance](https://nextjs.org/blog) — MEDIUM: Next.js 16.2 is stable; 16.3 was preview at research time; the July 2026 advisory identifies `16.2.11` as the patched active-LTS line.
- [Prisma 7 getting started](https://docs.prisma.io/docs/getting-started) and [Prisma Migrate commands](https://docs.prisma.io/docs/cli/migrate) — MEDIUM: Prisma 7 is GA and documents separate development and production migration commands.
- [PostgreSQL 18.4 release notes](https://www.postgresql.org/docs/18/release-18-4.html) and [supported-version information](https://www.postgresql.org/docs/18/reference-server.html) — MEDIUM: PostgreSQL 18 is current and maintained; apply the then-current patch release.
- [TanStack Query mutation invalidation guide](https://tanstack.com/query/v5/docs/framework/react/guides/invalidations-from-mutations) — MEDIUM: mutation success is the appropriate point to invalidate/refetch related query keys.
- [Tailwind CSS 4.3 announcement](https://tailwindcss.com/blog) — MEDIUM: current Tailwind v4 release line and CSS-first configuration.
- [Better Auth database documentation](https://better-auth.com/docs/concepts/database) — MEDIUM: validates it is a self-hostable future option, not a reason to add login before it is required.
- [Node.js downloads](https://nodejs.org/en/download/current) — MEDIUM: Node 24 is the current LTS series at research time.

