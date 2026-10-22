---
paths:
  - ".github/**"
  - "vercel.json"
  - "next.config.*"
  - "playwright.config.*"
  - ".env.example"
---

# CI/CD

How code moves from a pull request to production. The checks and runners are named in `tech-stack`; what each test layer proves is in `testing`.

## Principles

- **Every merge to `main` deploys to staging automatically.** Production deploys only by a manual trigger, after sign-off.
- **Production ships the exact commit verified on staging**, never "latest `main`".
- **Each environment is its own build**, because `NEXT_PUBLIC_*` values are baked in.
- **The frontend never deploys ahead of a backend change it depends on.** The backend deploys first and stays compatible; this client follows.

## Environments

| | Preview | Staging | Production |
| --- | --- | --- | --- |
| **Deployed by** | Vercel, on every PR | Pipeline 2, on every merge to `main` | Pipeline 3, manually |
| **Domain** | `*.vercel.app` | `staging.{domain}` | `app.{domain}` |
| **Env values** | Vercel preview environment | Vercel staging project | Vercel production project, separate values |
| **Error tracking** | Sentry `preview` | Sentry `staging` | Sentry `production` |

Which backend and mode each environment runs against is `tech-stack` → Services' table; it is not repeated here.

Staging and production are **two Vercel projects** on the same repo, so each has its own environment values, domain and deployment history, and a rollback on one never touches the other.

## Repo rules

- `main` is protected: no direct pushes, Pipeline 1 required to merge, one approving review where the team has more than one developer.
- Squash merge, so every commit on `main` is one reviewed PR.
- Vercel's automatic production deploys are **off**; its PR previews stay on. Only these pipelines deploy to staging and production.
- Staging deploys run one at a time, in merge order, so `staging-current` never moves backwards.
- Dependabot or Renovate PRs go through Pipeline 1 like any other.

## Pipelines

| Pipeline | Trigger | What it does | Deploys to |
| --- | --- | --- | --- |
| **1. PR checks** | Pull request opened or updated against `main` | Static checks, unit and integration tests, build, E2E smoke on the preview | Nothing (Vercel makes the preview) |
| **2. Staging deploy** | Push to `main` — every merge | Re-runs the checks, builds for staging, deploys, moves `staging-current`, runs the full E2E suite | Staging |
| **3. Production deploy** | Manual — release owners only | Builds and deploys the verified commit, smoke tests, rolls back on failure, tags the release | Production |

### 1. PR checks

**Required to merge.** Target: under ten minutes.

1. Install from the lockfile, cached.
2. In parallel: type check, lint and format check, unit and integration tests, dependency audit.
3. Build the app, to catch build-time errors.
4. Wait for the Vercel preview, then run the Playwright smoke suite against it, in mock mode.

### 2. Staging deploy

Runs automatically, one at a time in merge order.

1. Re-run Pipeline 1's checks on the merged commit.
2. Build with the staging environment and deploy it with the Vercel CLI to the staging project.
3. Move the `staging-current` tag to this commit, only if the deploy succeeded.
4. Run the full Playwright suite against staging, sharded in parallel.
5. Post the result to the team channel, if there is one: commit, pass or fail, and the changes since the last production release.

**After the workflow:** manual QA runs the staging checklist. A red E2E run or a failed checklist means no production deploy until fixed. When the release relies on backend changes, tell the backend which backend commit was verified; that is the backend's production sign-off.

### 3. Production deploy

**Trigger:** GitHub → Actions → Deploy Production → Run workflow. Only release owners can run it; running it is the approval. Its one optional input, `commit`, defaults to `staging-current`. A commit SHA releases exactly that commit instead; branch names are rejected, so `main` can never be deployed by mistake.

1. Check the actor is a release owner; stop if not. Show on the run page what is shipping: the commit, its message, the changes since the last release.
2. Check out the commit. Tests are not re-run: this commit already passed Pipelines 1 and 2.
3. Build with the production environment and deploy with the Vercel CLI to the production project.
4. Run the Playwright smoke suite against production, read-only, with the smoke account.
5. If it fails, roll back to the previous deployment (Vercel keeps every deployment) and stop.
6. If it passes, add a permanent release tag: `prod-<date>-<time>`, e.g. `prod-2026-10-21-1825`.

## Releasing alongside the backend

- A frontend release that needs new endpoints waits for the backend's production deploy, then ships. Order: backend, then frontend, always.
- A breaking backend change is coordinated: the backend tolerates both shapes across the window, or both deploy together, as the backend's build rules say. Open tabs still run the old client until refreshed, so the window is never zero.
- The production release notes name the backend commit the release was verified against.

## Tags

| Tag | Moves? | Set by | Purpose |
| --- | --- | --- | --- |
| `staging-current` | Yes, on every successful staging deploy | Pipeline 2 | The commit live on staging |
| `prod-<date>-<time>` | Never | Pipeline 3 | A permanent record of each production release |

There is no staging branch; the tag is the record. Comparing two `prod-*` tags shows exactly what changed between releases.

## Rollback

Vercel keeps every deployment, so rolling back is promoting the previous one: seconds, from the Vercel dashboard or the CLI, and the smoke test failure in Pipeline 3 does it automatically. A rollback is followed by a fix-forward PR; no release is re-pointed by hand twice.

## Infrastructure

Hosting, domains, DNS, Vercel projects, GitHub secrets and branch protection are the steps only a person can do; `/web-setup` writes them into `docs/infra/infra-setup.md` so each environment can be set up start to finish from that doc.
