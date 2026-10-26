# web-app-blueprint

**Design and build a product's web app with Claude Code, then connect it to the real backend.**

Every screen follows one design system. Each part of the app is designed for your approval, built and tested on realistic mock data, then connected to the backend's real endpoints as they're ready. Six Claude Code skills and the rules behind them, for Next.js.

**Works with** the [Backend Blueprint](https://github.com/dev-emekaa/backend-blueprint), which it is the web half of: it reads the backend's product docs and API docs, and sends back screen specs and issues.

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
| `--analytics` | Also install the analytics rule, for products that track usage with PostHog |
| `--personal` | Install skills to `~/.claude/skills` instead of the project |
| `--force` | Overwrite existing files. Default is to skip them |
| `--dry-run` | Show what would change, write nothing |

There is one stack — Next.js, TypeScript, Tailwind CSS, shadcn/ui, TanStack Query — so there is no `--stack` option. Re-running is safe: existing files are skipped and listed.

### What it writes

```
.claude/
  skills/        6 skills
  rules/         6 rules (plus analytics.md with --analytics)
CLAUDE.md        documentation map · audiences · project configuration
docs/
  product-docs/  business-rules/  app-flows/  api-docs/  api-testing/   arrive from the backend
  screen-specs/  design/screens/  integration-handoff/  infra/           written here
```

`docs/feature-areas.md` isn't created — it arrives from the backend.

### After installing

1. Fill in the **audiences** and **project configuration** in `CLAUDE.md` — above all, the path to the backend repo. Anything you leave blank, the skill that needs it asks for once.
2. Run `/web-setup`. It builds the foundation and pulls the backend's docs.
3. Run `/design-system` once the PRD and feature areas are in.
4. Run `/web-build` for the first feature area.

Commit `.claude/` with your project: cloud sessions and scheduled runs only read project skills. `--personal` is for throwaway work.

### Updating

```bash
npx github:dev-emekaa/web-app-blueprint init --force
```

- Skills and rules are overwritten, including any edits you made — run it on a clean working tree and read the diff before committing.
- `CLAUDE.md` is never overwritten: it holds your project's values.
- To pin a version: `npx github:dev-emekaa/web-app-blueprint#v1.3.4 init`. If an update doesn't seem to land: `npm cache clean --force`.

---

## How the app gets built

Each step needs the one before it; a skill whose inputs don't exist yet stops and says so.

```
From the backend: feature areas · business rules · app flows · (later) API docs
 │
 ├─ /web-setup        the foundation, once; pulls the backend's docs
 ├─ /design-system    DESIGN.md, once
 └─ /web-build        per feature area, in order:
      design   every screen it needs, designed for your approval        → Designed
      build    screens on mock data, with tests                         → Built
               → /screen-specs, sent to the backend                     → Specced
      ↓ once every area is Specced, the backend designs its database, then
        builds the areas in order, writing each one's API docs
    /web-integrate    real endpoints in place of the mocks              → Integrated

/web-issue-fix — any time, for anything already built
```

**Later phases** follow the backend's: the unit becomes a feature, and `/web-build` reads the existing screens before designing new ones.

---

## The skills

### `/web-setup`
The foundation, once: the Next.js project, sign-in and the data layer, the mock API that lets screens run before the backend exists, the test setup, error tracking, the CI/CD pipelines on Vercel, the docs sync with the backend repo, and a guide to the steps only you can do.
**Writes** the foundation, `docs/infra/infra-setup.md`

### `/design-system`
How everything looks, moves and reads: researches the product's category, proposes a direction for your approval, then writes `DESIGN.md` — colours, type, every component and its states, layouts, motion, writing style and accessibility — and applies it to the code.
**Writes** `DESIGN.md`

### `/web-build`
Designs and builds one feature area's screens: lists every screen its flows need, designs them for your approval, then builds them on realistic mock data with images and tests, checks every flow against the designs, and writes the screen specs the backend builds from.
**Writes** the designs, screens, mocks, tests and screen specs

### `/web-integrate`
Replaces the mocks with the real backend once its API docs land: checks the screens against the specs, connects every endpoint, handles every documented error, logs anything the backend must fix, and walks every flow on staging.
**Writes** the integration, the backend-issues file

### `/web-issue-fix`
Changes what already exists: reproduces the issue, writes a failing test, fixes it, updates the docs it touched, and adopts changes the backend hands over.
**Writes** the fix, its tests, the docs it changes

### `/screen-specs`
What each screen sends to the backend and shows from it — the same skill the Backend Blueprint ships, run here on the built screens.
**Writes** `docs/screen-specs/`

---

## The rules

Loaded by Claude Code at the start of every session — except `ci-cd.md`, which loads only when deploy files are touched.

| Rule | Covers |
| --- | --- |
| `documentation-conventions.md` | Which docs come from the backend, which this repo owns, and each screen's progress from Draft to Integrated |
| `tech-stack.md` | The tools — Next.js, Tailwind, shadcn/ui, TanStack Query, forms, motion, tests, Sentry — and the services on Vercel |
| `coding-standards.md` | How the app is structured and written: data, sign-in, mock mode, routing, components, state, styling |
| `security.md` | Sessions, input, headers, secrets, payments, personal data — and what the backend must provide |
| `testing.md` | The test layers, environments and data, and a test for every bug |
| `ci-cd.md` | PR checks, staging and production deploys, rollback, releasing alongside the backend |
| `analytics.md` *(with `--analytics`)* | Event naming, no personal data, consent |

---

## Working with the backend

- **Sharing docs:** `pnpm docs:pull` brings the backend's docs here; `pnpm docs:push` sends screen specs and backend issues there. `/web-setup` installs both, reading the backend repo's path from `CLAUDE.md`.
- **Issues:** `/web-integrate` logs what the backend must fix in `backend-issues-web.md`; the backend's `/backend-issue-fix` fixes it and hands back a message that `/web-issue-fix` adopts.
- **The backend's side:** it must meet `security.md` → Backend contract — cookies, CORS, the CSRF header, and real-time connections that accept the session cookies. Backend Blueprint v1.2.0 or later does.

---

## The folder version

A downloadable alternative to the installer, built from the same files:

```bash
npm run package
```

It writes `dist/web-app-blueprint/` and `dist/web-app-blueprint-v{version}.zip`. Its guide is `folder/README.md`.

### Releasing

```bash
npm version patch        # bumps package.json, commits and tags (patch · minor · major)
git push --follow-tags   # pushes the commit and the tag
```

The zip appears under the release's **Assets** as `web-app-blueprint-v{version}.zip` — that is the file to share, not the Source code archives. A tag that doesn't match `package.json` fails the build.

---

## Requires

- Node 22 or later, and Claude Code
- For design research: Mobbin, connected through its MCP — `/design-system` and `/web-build` use it for references
- For the `/design` canvas: a recent Claude Code signed in to a claude.ai account — without it, `/web-build` designs as HTML prototypes instead

**License** proprietary — all rights reserved.
