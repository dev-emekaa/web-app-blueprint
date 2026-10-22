# Testing

How this web client is tested. The tools are named in `tech-stack`; how tests are set up in this stack is in `coding-standards`; which pipeline runs what is in `ci-cd`. What each build, integration or fix must cover is set by those skills.

## Principles

- **Test the way users use the app:** find elements by role, label or text, interact, assert on what is visible. Never on component internals, state or implementation details.
- **Mock only at the network boundary.** MSW, with the shared handlers. Never mock our own modules, hooks or child components.
- **Most confidence comes from integration tests.** A small E2E suite guards the critical journeys. Production only ever receives a commit already verified on staging.

## Layers

| Layer | Proves | Share | Runs against | When |
| --- | --- | --- | --- | --- |
| **Static** | Types, lint, formatting, bad test patterns, accessibility lint | Always on | The source | Before commit, and in every pipeline |
| **Unit** | Pure logic: utils, formatters, Zod schemas, query-key builders, store reducers, the data functions extracted from Server Components | ~15–20% | Nothing | Locally, on every PR and merge |
| **Integration** | A whole screen or feature rendered with its real children, providers, router and forms; the network mocked with MSW | ~60–70% | jsdom | Locally, on every PR and merge |
| **E2E** | 5–20 critical journeys through a real browser against a built, deployed app | ~10–15% | A deployment | Smoke on every PR preview (mock mode), full suite on staging against the real backend, smoke on production |
| **Manual QA** | The integrated flows a person should walk once | — | Staging | Before every production deploy, from a 10–15 minute checklist |

Coverage is read to find untested logic, never enforced as a percentage.

**Deliberately not used:** snapshot tests of markup; unit tests of presentational components with no logic; Cypress or Playwright component tests (integration tests already cover this); mocking our own modules.

## Environments and data

| | Unit | Integration | E2E | Manual QA |
| --- | --- | --- | --- | --- |
| **Backend** | None | MSW, from the shared handlers | The PR preview: mock mode, passing through to staging · Staging: the real backend, on its seed | Staging, on its seed |
| **Session** | None | A mocked `me`, through `renderWithProviders` | The mock auth until the sign-in unit is integrated; real sign-in with the seeded test users after | Seeded test users |
| **Data** | Built per test | Fixtures from `features/*/mocks/`, overridden per test with `server.use` | The backend's seed | The backend's seed |
| **Browser** | — | jsdom | Chromium; WebKit and Firefox on the staging suite | A real browser |

- **Automated tests never touch production.** Production smoke tests are read-only, with a dedicated low-privilege account.
- **E2E tests depend on the backend's seed.** They use the records the seed marks as disposable and clean up what they create; they never reseed staging on their own, because manual QA and the backend's own verification share it. A reseed is asked for, through the backend's reseed dev-tool endpoint, when the seed has drifted.
- **Integration tests start from the shared mock handlers** so a screen is tested against the same shapes it was built on; a test that needs a different response overrides the one handler it needs.

## Unit tests

- **No network, DOM, timers or randomness** unless faked. Schemas, formatters and key builders run for real.
- **Group by the function under test, one behaviour per test**, named for the behaviour: `rejects an amount above the limit`.
- **Assert outcomes**, never the internal sequence of calls.

## Integration tests

- **Render the screen through `renderWithProviders`**, with a fresh `QueryClient` per test, retries off, and MSW serving the fixtures.
- **Drive it as a user would**: `user-event`, not `fireEvent`; `findBy*` for what appears after a request; `getByRole` before `getByTestId`, and a test id only where no accessible query exists.
- **Cover the screen's states**, not only the happy one: loading, empty, error, the permission-denied state, and every conditional state the screen spec lists.
- **Forms:** submit valid data and assert the request MSW received; submit invalid data and assert the field errors; return a 422 from MSW and assert it maps onto the fields.
- **Each test sets up its own handlers and leaves no shared state.** Tests pass in any order.

## E2E tests

- **Critical journeys only:** sign-in, the primary flow of each feature area, checkout or payment where there is one, and anything that would make the product unusable if broken. The list is kept in `e2e/README.md` with the reason each journey is there.
- **Against a built app, never against `next dev`:** the PR preview for the smoke suite, staging for the full suite, and a local `next build && next start` only when verifying the foundation.
- **Accessibility:** every E2E page visit runs an axe check with no serious or critical violations.
- **Stable by design:** role-based selectors, `expect` with auto-waiting, no fixed sleeps. A flaky test is fixed, not retried until it passes.

## Smoke tests

- **Three to five read-only flows** in a real browser: the app loads, sign-in works, the main screens render their data. Until the sign-in unit is integrated the sign-in is the mock auth's; after, it is the smoke account against the real backend, through the mock API's passthrough on the preview and directly on staging and production.
- **Run after every deploy**, and a failure on production triggers the rollback in `ci-cd`.

## When something breaks

- **A bug found anywhere gets a test that reproduces it**, at the lowest layer that can (usually integration). The test fails before the fix, passes after, and ships in the same PR.
- **A bug that manual QA catches twice becomes an automated test.**
- **A failing test is never skipped, deleted or loosened** to turn a pipeline green unless the PR states why.
