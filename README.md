# web-app-blueprint

**Design-to-integration web client workflow for Claude Code — the docs come first, the design system governs every screen, and the backend's api-docs drive integration.**

Six Claude Code skills and the rules they depend on, installed into a Next.js project as one pipeline: set up the foundation, write the design system, design and build each feature area's screens on mock data, spec them for the backend, integrate the real endpoints, and fix what breaks. It is the frontend half of a pair: the [Backend Blueprint](https://github.com/dev-emekaa/backend-blueprint) writes the product docs this one reads, and reads the screen specs this one writes.

```bash
npx github:dev-emekaa/web-app-blueprint init
```

---

## Install

Run it in your project root. Nothing to install first — `npx` fetches this repo and runs it.

```bash
npx github:dev-emekaa/web-app-blueprint init [options]
```

| Option | Effect |
| --- | --- |
| `--analytics` | Also install the analytics rule. For products that track user behaviour with PostHog |
| `--personal` | Install skills to `~/.claude/skills` instead of the project |
| `--force` | Overwrite existing files. Default is to skip them |
| `--dry-run` | Print what would change, write nothing |

There is one stack — Next.js, TypeScript, Tailwind CSS, shadcn/ui, TanStack Query — so no `--stack` flag. Add `--dry-run` to any command to preview it first. Re-running is safe: existing files are skipped and listed, never clobbered.

### What it writes

```
.claude/
  skills/                          6 skills
  rules/
    documentation-conventions.md   which docs arrive from the backend, which this repo owns, readiness
    tech-stack.md                  the tools and services
    coding-standards.md            architecture, data layer, auth flow, mock mode, how code is written
    security.md                    security policy, and the contract the backend must meet
    testing.md                     test layers, what each proves, environments and data
    ci-cd.md                       pipelines, Vercel, tags, rollback — loads only when deploy files are touched
    analytics.md                   --analytics only
CLAUDE.md                          documentation map · audiences · project configuration
docs/
  product-docs/  business-rules/  app-flows/  api-docs/  api-testing/     arrive from the backend
  screen-specs/  design/screens/  integration-handoff/  infra/            written here
```

`docs/feature-areas.md` is deliberately **not** created: it arrives from the backend with `pnpm docs:pull`, which `/web-setup` installs. `docs/screen-specs/` is created empty; `/screen-specs` fills it from the built screens.

### After installing

1. Fill in the **audiences** and **project configuration** blocks in `CLAUDE.md`. The backend repo path matters most; the sync scripts read it. Anything you leave blank, the skill that needs it asks for once.
2. Run `/web-setup`. It builds the foundation and pulls the backend's docs.
3. Run `/design-system` once the PRD and feature areas are in.
4. Run `/web-build` for the first feature area.

### Skills are project-scoped by default

They install to `.claude/skills/` and are meant to be committed: cloud sessions, scheduled runs and Cowork read only project skills, and rules that travel without the skills that execute them are conventions with no tooling. `--personal` exists for throwaway work.

### Updating

In a project that already has it, the installer skips existing files by design, so an update needs `--force`:

```bash
npx github:dev-emekaa/web-app-blueprint init --force
```

`--force` overwrites, including local edits to a skill or a rule. Run it on a clean working tree and read the diff before committing. `CLAUDE.md` is never rewritten.

Pin a tag for a fixed version: `npx github:dev-emekaa/web-app-blueprint#v1.0.0 init`. If an update doesn't seem to have landed, `npm cache clean --force`.

---

## The pipeline

Each step is gated on the one before it. A skill whose inputs don't exist yet stops and says so.

```
Backend Blueprint writes: feature-areas · business-rules · app-flows · (later) api-docs
 │
 ├─ /web-setup        → the foundation, once; pulls the backend's docs
 ├─ /design-system    → DESIGN.md, once
 └─ /web-build        → per feature area, in order:
      Phase 0: information architecture and the app shell, once
      Design:  screen inventory → references → canvas designs → your approval   [Designed]
      Build:   screens on mock data through the real data layer → tests          [Built]
               → /screen-specs → pushed to the backend                           [Specced]
           ↓   once every phase-1 area is Specced, the backend designs its schema,
               then builds the areas in order, writing api-docs per area
      /web-integrate  → per area as its api-docs land: real endpoints,
                        orphan audit, backend-issues handoff                      [Integrated]

/web-issue-fix — any time, for anything that already exists
```

**Later phases** follow the backend's: `/feature-requirements` and `/phase-review` there produce the FRD, the feature files and the frontend actions; here the unit becomes a **feature**, tracked in `docs/phase-{n}/ui-implementation-order.md`, and `/web-build` reads the existing screens before designing new ones.

---

## The skills

### `/web-setup`
**The foundation, once.** Next.js scaffold with the folder structure and shadcn base set; the HTTP core with `serverFetch`, `clientFetch` (cookies, CSRF header, refresh-and-retry) and `proxy.ts` (cookie refresh, redirects, security headers); TanStack Query with server prefetch and hydration; generated API types; the mock API that lets screens run on realistic data before the backend exists, with passthrough to the real backend for everything already integrated; design tokens from `DESIGN.md`; Vitest, MSW and Playwright; Sentry; the three CI/CD workflows and Vercel; the docs sync with the backend repo; an infra guide for the steps only you can do.
**Reads** the rules, `CLAUDE.md` · **Writes** the foundation, `docs/infra/infra-setup.md`

### `/design-system`
**The source of truth for how everything looks, moves and reads.** Reads the product docs to understand what kind of product this is and who looks at it; researches competitors and category leaders on Mobbin and reverse-engineers what makes them work; proposes a direction with a style tile for your approval; then writes `DESIGN.md` in the DESIGN.md format, linted with Google's checker — tokens that map onto shadcn's variables, every component with every state, patterns for list, detail, form and empty states, motion, imagery, content and voice, accessibility — and applies the tokens to the code. Built to avoid the generic.
**Reads** PRD, brief, feature areas, rules, flows, brand assets · **Writes** `DESIGN.md`

### `/web-build`
**Design and build one unit's screens.** Derives every screen every flow needs, with its states, and gets the inventory approved; pulls references per screen type; designs the batch on the `/design` canvas (or as HTML prototypes where the canvas is unavailable) and iterates with you until approved; then builds the screens with realistic mock data through the real hooks, sources images (stock, looked at before use; generated; or a prompt in `images.md`), writes the tests, walks every flow against the approved designs, and runs `/screen-specs` so the backend can build. Two stages with a checkpoint at approval, or a whole phase's build stage in one session with subagents.
**Reads** flows, rules, DESIGN.md, existing screens · **Writes** designs, screens, mocks, tests, screen specs

### `/web-integrate`
**Replace mocks with the real backend.** Audits every screen against the specs and the api-docs first (build gaps, design gaps, missing endpoints); aligns the hooks to the api-docs and removes the handlers; wires every documented error and background behaviour; audits for orphan endpoints; logs anything the backend must fix in a handoff file the backend reads; updates the tests to the real shapes; walks every flow against staging and names the backend commit verified. One unit per session, or a whole phase in order.
**Reads** api-docs, specs, api-testing, flows, rules · **Writes** the integration, the handoff file

### `/web-issue-fix`
**Change what exists.** Reproduces the issue, classifies it (client bug, backend bug to log, doc error to report, change request, backend handoff to adopt), writes the failing test, fixes surgically, updates every doc the fix touched, verifies the flows it affected, and delivers a PR. Adopts a backend handoff breaking-items-first.
**Reads** the issue, the code, the docs · **Writes** the fix, tests, doc updates

### `/screen-specs`
**What each screen sends to the backend and shows from it.** The same skill the Backend Blueprint ships, unchanged, run here in its From designs mode on the built screens so the backend has the specs it builds against.
**Reads** the built screens, flows, rules · **Writes** `docs/screen-specs/`

---

## The rules

These install into `.claude/rules/`, loaded at the start of every session — except `ci-cd.md`, which loads only when a workflow, Vercel or Next config file is touched.

- **`documentation-conventions.md`** — which docs arrive from the backend and are read-only here, which this repo owns, how screen specs change hands, the UI tracker and its six readiness values (Draft, Ready, Designed, Built, Specced, Integrated).
- **`tech-stack.md`** — the tools: Next.js, TypeScript, Tailwind, shadcn/ui, TanStack Query, Zustand, nuqs, React Hook Form and Zod, Motion, EventSource and Socket.IO for real-time, Vitest and Playwright, Sentry, optional PostHog; GSAP, Lottie and 3D only when a feature needs them. And the services per environment on Vercel.
- **`coding-standards.md`** — the architecture (Next.js renders, TanStack caches, the backend decides), the folder structure and dependency direction, the HTTP client and error handling, mock mode, the data layer, the auth flow with `proxy.ts`, routing, components, state, forms, effects, styling, performance, errors, code style, tests in this stack.
- **`security.md`** — sessions and tokens, input and output, headers and CSP, environment and secrets, data in the browser, third parties, payments, supply chain, errors, personal data, and the **backend contract**: cookies on the parent domain, CORS with credentials, the CSRF header, the refresh grace window, status semantics.
- **`testing.md`** — static, unit, integration (the bulk, with MSW), E2E (critical journeys against a deployment), smoke, manual QA; environments and data; the regression rule.
- **`ci-cd.md`** — PR checks with a smoke run on the Vercel preview, the automatic staging deploy with `staging-current`, the manual production deploy of the verified commit with rollback and `prod-*` tags, releasing alongside the backend.
- **`analytics.md`** *(with `--analytics`)* — same-origin ingest, explicit events named `object_action`, no personal data, identify and reset, consent.

---

## Working with the Backend Blueprint

The two repos share the docs through `pnpm docs:pull` and `pnpm docs:push`, which `/web-setup` installs and which read the backend repo path from `CLAUDE.md`. Pull brings the backend's docs here, including its copy of the screen specs once it owns them; `push --specs` sends one screen-spec file there and `push --handoff` sends this client's backend-issues file (`backend-issues-web.md` — each client writes only its own). The backend's `/backend-issue-fix` reads its open entries itself, appends each resolution, and writes a client handoff message that `/web-issue-fix` adopts.

The backend must meet the contract in `security.md` → Backend contract: its `security.md` (v1.1.0 and later) states the cookie, CORS, CSRF and refresh-grace lines from its side, and (v1.2.0) that real-time connections accept the session cookies; its api-docs document the error shapes.

---

## The folder version

The downloadable, manual alternative to the installer — the skills, rules and template laid out to copy into a project by hand, with its own README. Build it from this repo:

```bash
npm run package
```

It writes `dist/web-app-blueprint/` and `dist/web-app-blueprint-v{version}.zip`, from the same assets the installer uses, so the two never drift apart. Its guide lives in `folder/README.md`.

### Releasing

Every release is a version tag. Pushing one builds the zip and publishes it on the repo's **Releases** page:

```bash
npm version minor        # bumps package.json, commits, and tags v1.3.0 (patch · minor · major)
git push --follow-tags   # pushes the commit and the tag
```

The zip appears under the release's **Assets** as `web-app-blueprint-v1.3.0.zip` — that is the file to share. The **Source code** archives GitHub adds to every release are the repo itself, not the folder version. A tag that doesn't match the `package.json` version fails the build, so the zip's name always matches its contents.

---

**Requires** Node 18 or later, and Claude Code. The `/design` canvas needs a recent Claude Code signed in to a claude.ai account; `/web-build` falls back to HTML prototypes without it. **License** proprietary — all rights reserved.
