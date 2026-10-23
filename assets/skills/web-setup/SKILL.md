---
name: web-setup
description: Set up a Next.js web client's foundation once, before the first screen is designed or built — the project scaffold, the HTTP and auth layer, the data layer, mock mode, the design tokens, the test harness, the CI/CD pipelines, Vercel, and the docs sync with the backend repo. Use at the start of a web client project, or to bring an existing one up to the foundation the build skills assume.
---

# Web Setup

Builds the foundation every screen stands on. It runs once per project, before the first `/web-build`. It needs no product docs and no design system, so it can run while those are being written — only the design tokens step waits for `DESIGN.md`, and it can be run again later for that step alone.

It builds no screens: not even sign-in, which is a feature area designed and built from its own docs. It adds only what every web client needs. A library only some features use (tables, charts, editors, GSAP, Lottie, 3D) is added by the first build that needs it.

## Read first

- **The rules in `.claude/rules/`:** `tech-stack`, `coding-standards`, `security`, `testing` and `ci-cd` (and `analytics` where installed). They define what this skill builds, and it does not restate them.
- **`CLAUDE.md`:** the audiences and clients, which one this repo is, and the project configuration. If the backend repo path or the staging API URL is not recorded, ask once.
- **The backend's `docs/api-docs/`** if any exist yet, for the auth endpoints' exact paths. Until they do, the auth paths are configuration (`AUTH_REFRESH_PATH` and friends in `lib/env.ts`), set to the backend's conventional paths and corrected when the auth area is integrated.

If the repository already has code, take stock first. Keep what meets the rules, add only what is missing, and list anything that conflicts with the rules for me to decide. Never rebuild working pieces unasked.

## Step 1 — Plan → **wait for my approval**

State what exists, what will be created, and anything the rules leave to this project. Check the current stable version of each tool in `tech-stack` rather than relying on memory, and say which versions the scaffold will use. Check the current names of the settings the rules name (pnpm's supply-chain settings above all) against each tool's current docs: they change between major versions.

## Step 2 — Scaffold the application

- **The project per `tech-stack`:** Next.js with the App Router and `src/`, TypeScript strict, Tailwind, ESLint and Prettier configured as `coding-standards` says, a pre-commit hook running lint-staged, pnpm with the supply-chain settings `security` requires, and a committed lockfile.
- **shadcn/ui initialised** with the base components every product needs: button, input, label, form, select, checkbox, radio, switch, textarea, dialog, sheet, dropdown menu, popover, tooltip, toast (sonner), tabs, card, badge, avatar, skeleton, separator, table, command. More are added per build.
- **The folder structure from `coding-standards`**, with a placeholder `features/.gitkeep` and the `components/shared/` pieces every product has: page header, empty state, error state, confirm dialog, and a loading skeleton for lists and forms. The app shell is only a placeholder here (a header and a content area): its form is decided by `/design-system` and it is designed and built in `/web-build`'s Phase 0.
- **Environment handling:** `lib/env.ts` declaring every variable with Zod — including `NEXT_PUBLIC_APP_ENV` (`preview`, `staging`, `production`), which the readiness gate in `coding-standards` → Mock mode reads — and `.env.example` naming each with a comment and no secret values.
- **Route groups** `(auth)` and `(app)`, each with a layout, `loading.tsx`, `error.tsx`, and a root `not-found.tsx`. The `(app)` layout renders the shell and prefetches `me`. Pages are placeholders that the builds replace.
- **Fonts through `next/font`** and the theme provider, wired for dark mode.
- **Where the product has a mobile client** (the audiences table in `CLAUDE.md`): serve `/.well-known/apple-app-site-association` (no extension, `application/json`) and `/.well-known/assetlinks.json` as public routes, excluded from `proxy.ts`'s matcher and never redirected, with that environment's mobile identifiers from the mobile repo's `CLAUDE.md`; and a public account-deletion-request page at the path the mobile repo records, since Google Play requires one on the web. Record both in the infra guide per environment.
- **Where the product has a desktop client**: a public download page at `/download`, excluded from `proxy.ts`'s matcher, with its copy from the desktop repo's `docs/release/download-page.md` and one installer link per platform and architecture read from the `downloads.json` the desktop client's production pipeline writes next to its update feed, so a desktop release never needs a web deploy. Record it in the infra guide.

## Step 3 — HTTP, auth and data layer

Exactly as `coding-standards` → sections 3 to 5 describe:

- `lib/http/core.ts` with `ApiError`, the status table, and the base URL that mock mode points at the mock API.
- `lib/http/server-fetch.ts` and `lib/http/client-fetch.ts`, with the CSRF header and single-flight refresh in the client wrapper.
- `proxy.ts` with its matcher, the refresh-before-render logic, the protected-route redirect with a validated `returnTo`, and the per-request CSP nonce and headers from `security`.
- `lib/query-client.ts` with the defaults (`staleTime`, retry rules) and the per-request server factory. The `HydrationBoundary` pattern is shown by the example feature in Step 4.
- `hooks/use-me.ts`, the sign-out helper with cache clearing and the `BroadcastChannel`, and the sign-in cache reset.
- `pnpm api:types`, generating `types/api.ts` from the backend's OpenAPI spec recorded in `CLAUDE.md`. The backend exposes one only once it is built, so the script is wired here and first run at the first integration; until then every feature's types are provisional, per `coding-standards` → HTTP client.
- `nuqs` and its adapter in the root layout.
- **Local dev against a deployed backend:** a `pnpm dev:staging` script that runs Next's HTTPS dev mode on `local.{domain}` with `NEXT_PUBLIC_API_URL` pointed at staging, and a note in the infra guide on the hosts entry. Without it the backend's parent-domain cookies never reach `localhost`, and sign-in appears broken.

## Step 4 — Mock mode

The semantics — the mode switch, passthrough, cookie re-homing, sessions, the sign-in-first rule — live in `coding-standards` → Mock mode and are not restated here. This step creates the pieces:

- `app/api/mock/[...path]/route.ts`, the mock API route, and the `NEXT_PUBLIC_API_MODE` switch in `core`.
- The handler helper: `defineHandlers({ 'GET /orders': ({ query, session }) => ... })`, with path parameters, latency simulation, in-memory state per session, and the `?mock=error` switch.
- `src/mocks/msw.ts`, converting the same handlers to MSW for Vitest.
- `src/mocks/auth.ts`, the mock auth for the conventional auth paths (`/auth/sign-in`, `/auth/sign-out`, `/auth/refresh`, `/auth/me` — corrected at integration).
- One example feature folder, `features/_example/`, showing provisional types, a query hook, a mutation hook, fixtures, handlers, a screen, and the server prefetch with `HydrationBoundary` on its page, deleted by the first real build.

## Step 5 — Design tokens *(when `DESIGN.md` exists; otherwise placeholders, and re-run this step later)*

- `styles/globals.css` defining `DESIGN.md`'s colors, radii, spacing and type scale as CSS variables under the shadcn names — each color in `:root`, its `-dark` twin under `.dark` — the shadows from its Elevation & Depth table, and the breakpoints from its Responsive table where they differ from Tailwind's, all exposed to Tailwind.
- The shadcn components adjusted to the design system's radii, borders, focus rings and type.
- The fonts `DESIGN.md` names, loaded with `next/font`.
- The durations and easings from its Motion table as CSS variables, and a `lib/motion.ts` exporting them for Motion.

## Step 6 — Test harness

- Vitest with jsdom, `vitest.setup.ts` starting MSW from the shared handlers with `onUnhandledRequest: 'error'`, and `renderWithProviders`.
- Playwright configured for the preview, staging and production base URLs from the environment, with the axe check helper, the smoke project (3–5 specs, tagged) and the full project.
- `e2e/README.md` listing the journeys and why each is there; the smoke suite starts with "the app loads and sign-in renders".
- One working test of each kind against the foundation: a unit test of a formatter, an integration test of the example screen, and the smoke spec.

## Step 7 — Observability and analytics

- Sentry per `security`: scrubbing, source maps upload in the build, environment per deployment, `error.tsx` boundaries reporting, expected `ApiError`s filtered.
- Where `analytics` is installed: the PostHog provider, the same-origin rewrite, identify and reset wired to sign-in and sign-out, and `docs/analytics-events.md` started.

## Step 8 — Pipelines and hosting

- **The three GitHub Actions workflows `ci-cd` defines:** PR checks (waiting on the Vercel preview before the smoke run), the staging deploy serialized in merge order with the `staging-current` tag, and the production deploy with its release-owner check, SHA-only `commit` input, smoke test, automatic rollback and `prod-*` tag.
- **Vercel configuration** for two projects on one repo, with automatic production deploys off and previews on. Check Vercel's current documentation for CLI deploys and project linking; it changes.
- **Dependabot or Renovate** configuration, and the security policies `security` → Supply chain requires.

## Step 9 — Docs sync with the backend

- `scripts/docs-sync.mjs` with `pnpm docs:pull` and `pnpm docs:push`, reading the backend repo path from `CLAUDE.md` and keeping, per synced file, the hash of the version last pulled:
  - **pull** copies from the backend every doc `documentation-conventions` lists as arriving from it, plus the backend's `docs/screen-specs/` and phase screen-spec files where they exist, and `docs/integration-handoff/` where it exists, replacing this repo's copies, and reports what changed. A file an earlier pull brought that no longer exists in the backend is deleted here and reported — the backend's promotion deletes a feature's phase folder, and the canonical files are then the specs. It refuses to overwrite or delete a file with uncommitted local changes. **This client's handoff files are the one exception to replacing:** they are three-way merged against the last-pulled version, so entries committed here and resolutions appended there both survive — both sides only append, so the merge never conflicts, and a real conflict stops with a message.
  - **push `--specs {file}`** copies one screen-spec file (a canonical area file, or a phase feature's file) to the same path in the backend's `docs/`; **push `--handoff`** copies `backend-issues-web.md` and `backend-issues-web-resolved.md` and nothing else. Both first check that the backend's copy is the version this repo last pulled, and stop with a message to pull first if it is not, so a stale copy never overwrites the backend's edits. Push copies nothing else, and refuses to copy a file with uncommitted changes, so what the backend receives is always a committed version.
- The `docs/` folders from the documentation map, with `.gitkeep` files where nothing exists yet. `docs/feature-areas.md` is never created here; it arrives from the backend.

## Step 10 — Infra guide

Write `docs/infra/infra-setup.md`: every step only I can do, numbered, so staging and production can be set up start to finish by following it.

- **On Vercel:** the two projects, their domains (`staging.{domain}`, `app.{domain}`), environment values per project (from `.env.example`, production values separate), automatic production deploys off, the deploy token for the workflows, and the Git integration limited to previews.
- **On GitHub:** branch protection with PR checks required, squash merges, the release owners (whose usernames go into the production workflow), the `staging` and `production` environments holding the Vercel token and project ids, secret scanning with push protection.
- **DNS:** the app subdomain per environment, on the same parent domain as the API so the cookies are shared.
- **Elsewhere:** the Sentry project per environment, the stock photo API keys (into `.env`, with the providers listed in `CLAUDE.md`) and the image generation account if `/web-build` will use them, and the analytics projects where installed.
- **With the backend:** confirm the backend contract from `security` with the backend developer, item by item, before the first integration.

Then **offer to walk me through it live.** Once staging exists, record the app URLs, the dev and test commands, and the sync commands in `CLAUDE.md`.

## Step 11 — Verify and report

1. **Verify from a clean clone:** install, `pnpm dev` in mock mode renders the example screen with mock data, including a server-rendered page; the sign-in redirect fires for a protected route with no cookies; the mock sign-in sets the cookies and `useMe` resolves; lint, type check, unit and integration pass; the production build succeeds, its first-load JS is under budget, and the smoke suite passes against `next start` of that build.
2. **Verify the mock API on both paths:** a Server Component and a client hook both resolve the same handler; a path with no handler is forwarded to the configured API URL and a `Set-Cookie` it returns comes back without its `Domain`. Boot with the mode off and confirm the mock route answers 404.
3. **Report:** what was created, the commands, what remains for me (the infra guide), and any open decisions.
4. **Deliver it as a PR.** Create a branch off `origin/main`, push to it, and give me the link to create the PR. Never create the PR yourself. The PR checks workflow added here runs on that PR.
