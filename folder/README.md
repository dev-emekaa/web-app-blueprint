# Web App Blueprint

Design and build a product's web app with Claude Code, then connect it to the real backend. Every screen follows one design system; each part of the app is designed for your approval, built and tested on realistic mock data, then connected to the backend's real endpoints. Six skills and the rules behind them, for Next.js · TypeScript · Tailwind CSS · shadcn/ui · TanStack Query, hosted on Vercel.

It works with the **Backend Blueprint**: it reads the backend's product docs and API docs, and sends back screen specs and issues.

---

## What's inside

```
web-app-blueprint/
  skills/                6 skills, one folder each
  rules/                 the rules every project uses, plus one optional
  templates/
    CLAUDE.md            the blocks to add to your project's CLAUDE.md
```

---

## Install

Everything goes into your project's `.claude/` folder, which Claude Code reads. In a terminal, from your project's root — set `BP` to wherever you unzipped this folder:

```bash
BP=~/Downloads/web-app-blueprint
mkdir -p .claude/rules docs
cp -R "$BP/skills" .claude/
cp "$BP"/rules/*.md .claude/rules/
```

Then:

1. **Remove the optional rule if you don't need it:** `analytics.md` — keep it only if the product tracks user behaviour with PostHog.
2. **Add the CLAUDE.md blocks.** Paste `templates/CLAUDE.md` into your project's `CLAUDE.md` (create the file if there is none). Fill in the audiences and the project configuration now — the backend repo path above all — or leave them; each skill asks for anything missing, once.
3. **Check it worked.** Start Claude Code and run `/context` — the rules appear under Memory files (`ci-cd.md` only once a deploy file is touched). Type `/` and the six skills are listed.

> `.claude` is a hidden folder. To see it in Finder, press **Cmd + Shift + .**

**Commit `.claude/` with your project.** Cloud sessions and scheduled runs only read project skills, and the rules should travel with the code.

---

## Which skill when

**Starting a new web client**

1. `/web-setup` — the foundation, once: scaffold, HTTP and auth layer, data layer, mock mode, tests, pipelines, Vercel, and the docs sync that pulls the backend's docs into `docs/`.
2. `/design-system` — the design system, once, after the PRD and feature areas exist. Researches the niche, proposes a direction for your approval, writes `DESIGN.md` at the project root.
3. `/web-build` — one feature area per session, in the order the backend's feature areas give, or the build stage of a whole phase in one session once every area is designed. First its screens are designed on the canvas and approved; then they are built on mock data, tested, and specced for the backend.
4. The backend builds the area from your specs and writes its api-docs.
5. `/web-integrate` — one feature area per session, or the whole phase: replaces the mocks with the real endpoints and verifies against staging.

**Adding a new phase**

1. The backend's `/feature-requirements` and `/phase-review` produce the phase's FRD, feature files and frontend actions; `pnpm docs:pull` brings them here.
2. `/web-build` — one feature per session; it reads the existing screens before designing new ones.
3. `/web-integrate` — one feature per session, or the whole phase.

**Fixing an issue, or adopting a backend change**

`/web-issue-fix` — reproduces, classifies, writes the failing test, fixes surgically, updates the docs, verifies, and delivers a PR.

---

## The skills

**`/web-setup`** — Builds the foundation every screen stands on: the project per the tech stack, `serverFetch`, `clientFetch` and `proxy.ts`, TanStack Query with server hydration, generated API types, the mock API that serves screens before the backend exists, design tokens, Vitest and Playwright, Sentry, the three CI/CD workflows, Vercel, the docs sync, and an infra guide for the steps only you can do.

**`/design-system`** — Writes `DESIGN.md`, the source of truth for how every screen looks, moves and reads: principles, colors and typography with contrast checked, spacing, shape and depth, icons and imagery, motion, every component with every state, screen patterns, content and voice, responsive and accessibility rules. Researches competitors and category leaders first — on Mobbin when it's connected, otherwise on the web, plus any screenshots you add — and proposes a direction for your approval.

**`/web-build`** — Designs and builds one feature area's screens: every screen every flow needs, with its states, inventoried for your approval; designed on the `/design` canvas and iterated with you; then built with realistic mock data through the real data layer, with images sourced and checked, tests written, every flow walked against the approved designs, and the screen specs written for the backend.

**`/web-integrate`** — Replaces the mocks with the backend's real endpoints from the api-docs: an audit first, then the hooks aligned and handlers removed, every documented error and background behaviour wired, an orphan-endpoint check, a handoff file for anything the backend must fix, tests on the real shapes, and a walk of every flow against staging.

**`/web-issue-fix`** — Resolves anything in what already exists: bugs, design drift, accessibility or performance problems, change requests, and the changes a backend handoff requires. Failing test first, surgical fix, docs updated, flows verified.

**`/screen-specs`** — What each screen sends to the backend and shows from it, written from the built screens. The same skill the Backend Blueprint ships.

---

## The rules

Rules load into every Claude Code session and apply to all the code it writes.

| Rule | What it covers |
| --- | --- |
| `documentation-conventions.md` | Which docs arrive from the backend and which this repo owns, how screen specs change hands, the UI tracker, and the six readiness values from `Draft` to `Integrated`. |
| `tech-stack.md` | The tools the client uses and the services each environment runs on. Anything not listed is proposed in a build plan and approved first. |
| `coding-standards.md` | The architecture, folder structure, HTTP client, mock mode, data layer, auth flow, routing, components, state, forms, styling, performance and code style. |
| `security.md` | Sessions and tokens, input and output, headers, secrets, data in the browser, third parties, payments, supply chain, privacy — and the contract the backend must meet. |
| `testing.md` | The test layers and what each proves, environments and data, and the rule that every bug gets a test that reproduces it. |
| `ci-cd.md` | PR checks, the automatic staging deploy, and the manual production deploy of the verified commit with rollback. Loads only when deploy files are touched. |
| `analytics.md` *(optional)* | Explicit, named events with no personal data, identify and reset, consent. |

---

**Requires** Claude Code, and a Next.js project (the setup skill creates it). Mobbin is optional: without it, the design skills research on the web and from any screenshots you put in `docs/design/references/`. The `/design` canvas needs a recent Claude Code signed in to a claude.ai account; the build skill falls back to HTML prototypes without it. Updates replace these files — any edits you made to a skill or rule are overwritten, so keep your own changes in `CLAUDE.md` or new rule files.

© dev-emeka. All rights reserved. Licensed for use in your own projects; not for redistribution.
