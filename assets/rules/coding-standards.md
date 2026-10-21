# Coding Standards — Next.js · TypeScript

How code is written in this web client. The tools are named in `tech-stack`; security policy is in `security`; what tests must prove is in `testing`. Where a rule here touches those, it gives the mechanism in this stack and leaves the policy to them.

## 1. Architecture

Next.js renders and routes. TanStack Query caches. The backend decides everything else.

| Layer | Owns |
| --- | --- |
| **Next.js** | Routing and layouts, server rendering, initial data fetching, cookie forwarding, auth redirects |
| **TanStack Query** | Client caching, refetching, mutations, pagination, optimistic updates |
| **Backend** | Data, validation, business rules, authentication, authorization, issuing and rotating tokens |

Business rules are enforced by the backend. The client repeats a rule only where the user would otherwise wait for a round trip to learn it (a form field's limit, a disabled action), and that repetition is a convenience, never the enforcement.

### Data flow

Data reaches the backend by two paths. Both carry the same cookies, and the backend checks them on every request.

**Server path** — page loads and navigation. The browser requests a page → `proxy.ts` refreshes the session if needed → a Server Component fetches with `serverFetch` → the browser receives HTML with the data already loaded.

**Client path** — refetches, mutations, pagination. A TanStack Query hook calls `clientFetch` → it calls the backend directly, Next.js is not involved → the cache updates.

**Two kinds of page:**

- **Interactive pages.** The server prefetches into a `QueryClient`, passes it down with `dehydrate` and `HydrationBoundary`, and the client hooks start with the data loaded: no second request, no loading flash.
- **Server-only pages** (read-only, no interaction). Fetch and render in Server Components alone. No TanStack Query, so the bundle stays small.

**Rules:**

1. Default `staleTime` above zero (60 seconds unless an entity says otherwise), or hydrated data is refetched the moment the page loads.
2. One `QueryClient` per request on the server, one shared client in the browser. A shared server client would leak data between users.
3. Query keys are defined once per entity, in the feature's `api/keys.ts`, and used by server prefetch and client hooks alike. Different keys mean the hydrated data is ignored.
4. Mutations go from TanStack Query straight to the backend, then invalidate the affected keys. No Server Actions: with a separate backend they only add a hop and a second place to validate.
5. User-specific data is never cached by Next.js. Authenticated server fetches are always `cache: 'no-store'`; `proxy.ts` sets `Cache-Control: private, no-store` on authenticated routes.
6. No `useEffect` to fetch. Data comes from a query hook or a Server Component.

## 2. Project structure

```
src/
  app/                          routes only: layouts, pages, loading, error, not-found
    (auth)/                     sign-in, sign-up, reset — public
    (app)/                      the product — protected by proxy.ts
    api/mock/[...path]/         the mock API route handler; answers 404 outside mock mode
  features/
    <feature-area>/             one folder per feature area, named after docs/feature-areas.md
      api/
        keys.ts                 query keys
        queries.ts              useX() hooks
        mutations.ts            useXMutation() hooks
      components/               screens and parts that belong to this area only
      schemas/                  Zod schemas for this area's forms
      mocks/
        fixtures.ts             typed sample data
        handlers.ts             mock handlers: method + path → response
      store.ts                  Zustand store, only if the area has client-only state
      index.ts                  what other code may import from this area
  components/
    ui/                         shadcn components, owned and edited here
    shared/                     product-wide pieces: app shell, page header, empty state, data table
  lib/
    http/
      core.ts                   base URL (the mock API in mock mode), JSON, ApiError
      server-fetch.ts           serverFetch
      client-fetch.ts           clientFetch, refresh-and-retry
    query-client.ts             QueryClient factory and defaults
    env.ts                      validated environment
    utils.ts                    cn() and small pure helpers
  hooks/                        product-wide hooks (useMe, useMediaQuery)
  mocks/
    index.ts                    collects every feature's handlers; the mock API route and MSW read it
    msw.ts                      converts handlers to MSW for tests
    auth.ts                     mock sign-in, sign-out, refresh and me, until the auth area's own mocks replace them
  types/
    api.ts                      generated from OpenAPI — never edited
  styles/
    globals.css                 Tailwind theme: the design.md tokens as CSS variables
  proxy.ts                      session refresh and auth redirects
```

**Dependency direction:** `lib` ← `components` ← `hooks` ← `features` ← `app`. Nothing imports upward, and a feature never imports another feature's internals: cross-feature use goes through the other feature's `index.ts`, and a component needed by two features moves to `components/shared/`.

**Routes stay thin.** A page in `app/` composes a screen from a feature and prefetches its data. Screens, hooks and schemas live in `features/`. A screen that belongs to several areas lives in the area whose entity's lifecycle it advances, as the screen specs decide. Folders are always feature areas: a later phase's feature puts each of its screens in the area it belongs to, never in a folder named after the feature.

## 3. HTTP client

Native `fetch` everywhere, through two thin wrappers that share one core.

| Wrapper | Runs in | Does |
| --- | --- | --- |
| `core` | Both | Base URL (the real API, or this app's mock API in mock mode), JSON encoding and parsing, turns every failure into `ApiError` |
| `serverFetch` | Server Components, route handlers | Forwards the request's cookies, never cached, redirects to sign-in on 401 |
| `clientFetch` | Browser | Sends cookies (`credentials: 'include'`), adds the CSRF header to every non-GET, refreshes and retries once on 401 |

**`ApiError`** carries `status`, `code` (the backend's error code), `message` and, for validation failures, `fieldErrors` keyed by field path. Every failure is handled by status, the same way everywhere:

| Status | Browser | Server |
| --- | --- | --- |
| **401** | Refresh once, retry once; if the refresh fails, clear the cache and go to sign-in | Redirect to sign-in |
| **403** | Show the no-access state | Render the no-access page |
| **404** | Show the not-found state | `notFound()` |
| **422 / validation** | Map `fieldErrors` onto the form | — |
| **409, 429, and other client errors** | Show the backend's message in a toast | Render the error boundary |
| **5xx, network** | Toast with a retry | `error.tsx` boundary |

Types come from `types/api.ts`, generated from the backend's OpenAPI spec (`pnpm api:types`), and are regenerated whenever the backend's api-docs change. Nobody writes an API type by hand. **Unknown enum values are tolerated**: a `switch` on a backend enum always has a default branch that renders something sensible, so the backend can add a value without breaking a released client.

### Mock mode

Before a unit is integrated, its screens run on mock data that goes **through the real data layer**: the same hooks, the same `clientFetch` and `serverFetch`, the same types. Only where the requests go differs.

- `NEXT_PUBLIC_API_MODE=mock` points `core`'s base URL at **this app's mock API**: the catch-all route handler `app/api/mock/[...path]/route.ts`, which dispatches each request to the handlers collected in `src/mocks/index.ts`. It runs on the Next.js server, so it serves Server Components and the browser alike, and it can set real cookies. It answers 404 outside mock mode, and a test proves that. With the mode off (`direct`, the default), `core` calls `NEXT_PUBLIC_API_URL` and the cookies are the backend's parent-domain ones.
- **Passthrough.** A path with no handler is forwarded to `NEXT_PUBLIC_API_URL` with the request's cookies and headers, and the response is returned with one change: the `Domain` attribute is stripped from every `Set-Cookie`, so the backend's session cookies land on **this app's origin**. The mock API is therefore a same-origin proxy: it works on `localhost`, on a `*.vercel.app` preview and on staging alike, and the same session serves mocked and real units. Integration removes a unit's handlers and its traffic falls through; an environment with every unit integrated runs `direct`.
- **The unit that owns sign-in is integrated first.** Until it is, every environment runs on the mock auth and only its handlers answer; after it, sign-in passes through to the real backend and every handler sees the real session. Nothing else is integrated before it.
- Each feature's `mocks/handlers.ts` maps `METHOD /path` to a resolver taking `{ params, query, body, session }` and returning the response, using the typed fixtures in `mocks/fixtures.ts`. `session` is the access cookie's decoded payload (`sub`, the user id), whichever side issued it; it is never verified, which is acceptable only because the mock API does not exist in production. A user the fixtures do not know is treated as the default persona. Resolvers keep state in the server process for the session (a created record appears in the next list) and simulate latency.
- Fixtures are realistic: real-looking names, dates and amounts, every state the screen specs describe, edge cases (long text, empty lists, the maximum count). A mock that always succeeds hides the error states the design asked for, so handlers can be told to fail (`?mock=error`).
- **Mock auth** (`src/mocks/auth.ts`) handles sign-in, sign-out, refresh and `me` until the sign-in unit is integrated: it sets and clears the real cookie names with JWT-shaped tokens carrying the same claims the backend's tokens carry, from the route handler's response, so `proxy.ts`, `serverFetch` and `clientFetch` run unchanged. The sign-in unit's own `mocks/` replaces it when that unit is built, and that unit's integration removes it.
- The same handlers feed MSW in tests through `src/mocks/msw.ts`. Integration removes the handlers for the unit's endpoints; the fixtures stay for tests.

## 4. Data layer

- **One hook per query, one per mutation**, in the feature's `api/`. Components never call `clientFetch` directly.
- **Keys** are hierarchical and built by functions: `orderKeys.all`, `orderKeys.list(filters)`, `orderKeys.detail(id)`. Invalidate the narrowest key that covers the change.
- **Lists** use `useInfiniteQuery` for cursor pagination and `useQuery` with `placeholderData: keepPreviousData` for page numbers, matching whichever the api-docs give. Filters, sort and page live in the URL through `nuqs`, so a list is shareable and survives refresh.
- **Mutations** invalidate on settle, not on success alone. Optimistic updates are for actions where the user expects instant feedback (toggles, reordering) and always roll back from the snapshot on error.
- **No retries on 401, 403, 404 or 422.** Retry other failures twice with backoff, queries only; mutations never retry on their own.
- **Derived data is computed with `select`** or a memo, never copied into state.
- **Real-time** updates (SSE or WebSocket, as the api-docs specify) write into the cache with `setQueryData` or invalidate, so every screen reading that entity updates.

## 5. Authentication

The backend issues, rotates and checks tokens. Next.js and the browser only carry the cookies and trigger refreshes.

| Trigger | Who refreshes |
| --- | --- |
| Page load or navigation | `proxy.ts`, before the page renders |
| A browser API call returns 401 | `clientFetch` |

- **`proxy.ts`** runs before page requests (its matcher skips static files and `/api/`). It refreshes when the access cookie is missing or about to expire and a refresh cookie exists, reading the token's `exp` without verifying it — the backend verifies. It sets the new cookies on both the response and the forwarded request, so the page renders with the fresh token. It redirects a protected route with no session to sign-in, carrying a validated `returnTo`, and sends a signed-in user away from the auth pages. **It is a convenience, never authorization**: the backend decides access on every request.
- **`clientFetch`** refreshes single-flight: when several requests fail together, they wait on one shared refresh. It skips the logic for the refresh route itself. If the refresh fails, it clears the query cache and redirects to sign-in.
- **`serverFetch`** never refreshes, because Server Components cannot set cookies; `proxy.ts` already did.
- **Current user:** one `useMe` query, prefetched on the server. Its roles show or hide UI; the backend enforces them.
- **Sign-in and sign-out** clear the query cache, so the next user never sees the previous one's data. Sign-out calls the backend, then tells other tabs through a `BroadcastChannel` so they sign out too.

## 6. Routing and pages

- **Route groups** separate public and protected trees. The protected group's layout renders the app shell and prefetches `me`.
- **Every page has `loading.tsx` and `error.tsx`** at the level where they make sense, and a `not-found.tsx` at the root. Loading UI is a skeleton of the real layout, never a spinner alone.
- **Metadata** is set per page with `generateMetadata` where it depends on data; otherwise static `metadata`. Authenticated pages are `noindex`.
- **Navigation** uses `<Link>` and `router.push`; full-page loads only for sign-out. Prefetch is left on for in-app links and turned off for links to large or sensitive pages.
- **Dynamic segments** are validated at the top of the page (Zod on `params` and `searchParams`); an invalid one is `notFound()`, not a crash.
- **Multi-step flows** keep their step in the URL, so refresh and back behave.

## 7. Components

- **Server by default.** `'use client'` goes on the smallest component that needs state, effects, browser APIs or event handlers, never on a page. Client components receive data as props from a Server Component or from a hook; they do not fetch on mount.
- **shadcn components are ours.** Edit them in `components/ui/` to match `design.md`; never wrap one to restyle it. Add components with the shadcn CLI, then adjust.
- **Variants with `cva`**, states with `data-*` attributes (`data-state="open"`), styling through Tailwind. No inline styles except for values computed at runtime.
- **One component per file**, named in PascalCase, file in kebab-case (`order-summary.tsx`). A screen is `{screen-name}-screen.tsx`; its parts live beside it in a folder once there are more than two.
- **Props are typed with an interface**, named `{Component}Props`, and destructured. Boolean props read as predicates (`isOpen`, `canEdit`). Callbacks start with `on`.
- **Composition over configuration**: slots and children rather than a dozen boolean props. A component that takes more than seven props is usually two components.
- **Every list has an empty state**, every async component a loading and an error state, as the design system specifies. A screen is not done with only its happy state.
- **Accessibility is part of the component**: semantic elements, labels on every input, focus visible, keyboard reachable, `aria-*` only where semantics do not already say it. `eslint-plugin-jsx-a11y` is on and not disabled per line.

## 8. State

In this order; use the first that fits.

1. **Server state** — TanStack Query. Anything the backend owns.
2. **URL state** — `nuqs`. Filters, sort, pagination, active tab, open panel: anything the user would expect a link to reproduce.
3. **Form state** — React Hook Form. Never mirrored into a store.
4. **Local state** — `useState` in the component that owns it.
5. **Global UI state** — Zustand, one store per concern, for what survives navigation and nothing above fits: sidebar, unsaved draft, a wizard's progress. Stores hold no server data and no derived data.

No React Context for state that changes often; it re-renders every consumer. Context is for dependency injection (theme, the query client).

## 9. Forms

- **One Zod schema per form**, in the feature's `schemas/`, derived from the unit's business rules and the screen spec's field constraints — same names, same limits. The type is `z.infer`, never written twice.
- **React Hook Form with `zodResolver`**, validating on blur and then on change, submit disabled while submitting, never while merely invalid.
- **Server validation wins.** A 422 maps `fieldErrors` onto the fields with `setError`; an error with no field goes to the form's root error.
- **Submission goes through a mutation hook.** The form never calls `clientFetch`.
- **Unsaved changes** are guarded on navigation when the form is long enough that losing it hurts.
- Inputs are controlled through RHF's `Controller` only for components that need it (selects, date pickers); native inputs use `register`.

## 10. Effects

- `useEffect` is for synchronising with something outside React: a subscription, a browser API, a third-party widget. Not for fetching, not for deriving state, not for responding to a prop change that could be computed during render.
- Every effect with a subscription returns its cleanup. Every async effect guards against the component having unmounted.
- Event handlers do the work that an event causes. If logic runs "after X changes", ask whether it belongs in the handler that changed X.

## 11. Styling

- **Tokens, not values.** Colors, spacing, radii, shadows and type sizes come from the CSS variables `styles/globals.css` defines from `design.md`, exposed as Tailwind utilities. Arbitrary values (`w-[13px]`, `text-[#333]`) need a comment saying why the scale did not fit.
- **`cn()` merges classes**; conditional classes go through it, never through string concatenation.
- **Dark mode** is CSS variables switched by `next-themes`; components never check the theme in JavaScript to pick a color.
- **Responsive, mobile first**: base styles are the phone layout, breakpoints add from there. Touch targets at least 44px on touch layouts. No horizontal page scroll at any width.
- **Class order** is Prettier's (`prettier-plugin-tailwindcss`), so diffs stay readable.
- **Motion** follows `design.md`'s motion section: durations and easings from its tokens, `prefers-reduced-motion` respected everywhere, layout animations only through Motion's `layout` prop.

## 12. Performance

- **Images through `next/image`** with explicit sizes, from the backend's CDN domains listed in `next.config`. Above-the-fold images are `priority`; the rest lazy. Formats and sizes come from `design.md` → Imagery.
- **Fonts through `next/font`**, self-hosted, with `display: swap`, two families at most.
- **Heavy, rarely used components load with `next/dynamic`**: editors, charts, maps, anything over ~50 KB that is below the fold or behind an action.
- **Lists over ~100 rows are virtualised**; everything else is paginated as the api-docs allow.
- **`memo`, `useMemo` and `useCallback` are applied where a profile shows the cost**, not by habit. React Compiler, where enabled, makes most of them redundant.
- **Third-party scripts** load through `next/script` with the least eager strategy that works.
- **The bundle is checked** (`pnpm analyze`) when a dependency is added, and a route's first-load JS stays under the budget `design.md` or `CLAUDE.md` records (default 200 KB).

## 13. Errors and observability

- **Errors render, they do not vanish.** A failed query shows the error state with retry; a failed mutation shows a toast with the backend's message; an uncaught render error reaches the nearest `error.tsx`.
- **Sentry captures** uncaught errors, `error.tsx` boundaries and failed mutations with the request's method and path, never its body. Expected `ApiError`s (401, 403, 404, 422) are not reported.
- **No `console.log` in committed code.** Debug logging goes through a `logger` that is silent in production.
- **User-facing messages come from the backend's message keys** where the business rules define them, and from `design.md` → Content and voice otherwise.

## 14. Code style

- **TypeScript strict**, no `any`, no non-null assertions without a comment, no `as` casts to silence the compiler. Narrow with type guards; model variants as discriminated unions.
- **Imports** in groups, blank line between: Node and framework, third-party, `@/` aliases, relative, then styles. Type imports with `import type`. No default exports except where Next.js requires them (pages, layouts, `proxy.ts`).
- **Naming:** `camelCase` for values and functions, `PascalCase` for components and types, `SCREAMING_CASE` for true constants, `use` prefix for hooks, `is`/`has`/`can` for booleans. Files `kebab-case`. Folders in `features/` match the slugs in `docs/feature-areas.md`.
- **Functions are small and do one thing**; a file over ~300 lines is a sign to split by responsibility.
- **Comments explain why, not what.** A `TODO` carries the ticket or issue it refers to.
- **Dead code is deleted**, not commented out. Git keeps the history.
- **Formatting and linting pass before commit**, enforced by a pre-commit hook; CI repeats the check.

## 15. Tests in this stack

What tests must prove is in `testing`. In this stack:

- Vitest with jsdom, `@testing-library/react` and `user-event`; MSW started in `vitest.setup.ts` from the shared handlers, with `onUnhandledRequest: 'error'`.
- Test files sit beside the code (`order-summary.test.tsx`); E2E specs in `e2e/`.
- Tests render through the real providers via a `renderWithProviders` helper (a fresh `QueryClient` with retries off, the theme provider, `nuqs`'s test adapter).
- Async Server Components are not rendered by Vitest: their data logic is extracted into functions that are unit-tested, and the rendered page is covered by Playwright.
