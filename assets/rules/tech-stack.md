# Tech Stack — Next.js · TypeScript

The technologies this web client uses. Use the current stable release of each; the lockfile pins versions, not this file. Anything not listed is added only after approval: the build plan names what the feature needs and recommends the most secure, efficient and standard option for this stack.

Next.js is the frontend only. The backend (NestJS or FastAPI, built with the Backend Blueprint) is the single authority for data, business rules and authentication. Next.js holds no business logic, is not the API, and runs no auth of its own.

## Application

| Need | Choice |
| --- | --- |
| Language · runtime | TypeScript in strict mode · Node.js, active LTS |
| Framework | Next.js, App Router, `src/` directory |
| Styling | Tailwind CSS |
| UI components | shadcn/ui — copied into the repo and owned there, built on Radix primitives · `lucide-react` icons · `sonner` toasts · `next-themes` for dark mode |
| Server data | TanStack Query: caching, refetching, mutations, pagination, optimistic updates |
| HTTP | Native `fetch` through the project's `serverFetch` and `clientFetch` wrappers · `XMLHttpRequest` only for uploads that need a progress bar |
| API types | `openapi-typescript`, generated from the backend's OpenAPI spec into `src/types/api.ts` |
| Global UI state | Zustand — only for state the server does not own (sidebar, drafts, wizard steps) |
| URL state | `nuqs` — filters, sort, page and tab in the URL, typed |
| Forms · validation | React Hook Form · Zod, through `@hookform/resolvers` |
| Environment variables | `@t3-oss/env-nextjs` with Zod: validated at build, `NEXT_PUBLIC_` exposure explicit |
| Session | HttpOnly cookies issued by the backend; `proxy.ts` refreshes them and redirects. Tokens are never readable by JavaScript |
| Mocking | MSW for tests · this app's mock API route for the build-before-integration phase (`coding-standards` → Mock mode) |
| Real-time | SSE through the browser's `EventSource` with credentials · two-way through the protocol the backend uses: the browser's `WebSocket` for a FastAPI backend, the Socket.IO client for a NestJS one. All carry the session cookies, so no token is handled. Only where the api-docs specify a channel |
| Dates | `date-fns` |
| Animation | Motion (`motion/react`) for UI animation: transitions, micro-interactions, layout and gestures |
| Animation, on demand | GSAP for scroll-driven and timeline choreography · `@lottiefiles/dotlottie-react` for vector animations · React Three Fiber for 3D. Installed only when a feature needs them, never by default |
| Error tracking | `@sentry/nextjs` |
| Product analytics *(optional)* | `posthog-js`, through a same-origin rewrite. The `analytics` rule applies when it is installed |
| Lint · format · types | ESLint (`eslint-config-next`, `eslint-plugin-jsx-a11y`, `eslint-plugin-testing-library`) · Prettier with `prettier-plugin-tailwindcss` · `tsc` |
| Package manager | pnpm, with a committed lockfile |
| Unit and integration tests | Vitest · React Testing Library · `@testing-library/user-event` · `@testing-library/jest-dom` · MSW |
| E2E and smoke tests | Playwright, with `@axe-core/playwright` for accessibility checks |
| Bundle analysis | `@next/bundle-analyzer`, run by hand |

**Added when a feature needs them**, through the build plan: `@tanstack/react-table` for data tables, Recharts (shadcn charts) for charts, a rich-text editor, a map library, `react-dropzone` for uploads, `next-intl` for a second language.

## Services

| Need | Local | Preview | Staging | Production |
| --- | --- | --- | --- | --- |
| Hosting | `next dev` | Vercel preview deployment, per pull request | Vercel, staging project and domain | Vercel, production domain |
| Backend API | Mock mode, passing un-mocked units through to staging | Mock mode, passing through to staging | Staging backend; mock mode with passthrough while any unit is un-integrated | Production backend, direct |
| Error tracking | — | Sentry, `preview` environment | Sentry, `staging` environment | Sentry, `production` environment |
| Analytics *(optional)* | — | — | PostHog, staging project | PostHog, production project |
| CI/CD | — | GitHub Actions | GitHub Actions | GitHub Actions |
| Domains | `localhost` in mock mode · `https://local.{domain}:3000` (a hosts entry, HTTPS dev) in direct mode against a deployed backend | `*.vercel.app` | `staging.{domain}` with the API on a sibling subdomain | `app.{domain}` with the API on a sibling subdomain |

- **Custom domains are required from staging on.** The backend's session cookies are set on the parent domain and shared by the app and API subdomains; hosting platforms' default domains cannot share cookies. A Vercel preview therefore runs in mock mode, whose passthrough re-homes the backend's cookies on the preview's own origin. Locally, mock mode works on `localhost` the same way; direct mode against a deployed backend needs the dev server on `local.{domain}` (mapped to `127.0.0.1` in the hosts file, with Next's HTTPS dev mode), which is how a unit's integration is finally verified: the way production will run.
- **Environment variables are baked in at build.** `NEXT_PUBLIC_API_URL` and the rest differ per environment, so each environment is its own build.
- **Sentry** receives source maps from the build, scrubs what `security` says, and alerts on new issues per environment.
- **Media** is uploaded straight to the backend's bucket with presigned URLs the backend issues, and served from its CDN; the client never holds storage credentials.
