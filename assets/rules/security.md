# Security

Security policy for this web client. The backend enforces every rule about who may do what; this client's job is to never leak, never trust what it did not verify, and never expose what the browser should not hold. The mechanism in this stack is in `coding-standards`; this file is the policy.

## Sessions and tokens

- **Tokens live only in cookies the backend sets:** `HttpOnly`, `Secure`, `SameSite=Lax`, on the parent domain so the app and API subdomains share them. JavaScript never reads, stores or forwards a token. Nothing auth-related goes in `localStorage`, `sessionStorage`, Zustand or the URL. In mock mode the mock API re-homes the same cookies on the app's own origin, with the same attributes; production never runs mock mode, and a test proves the mock route answers 404 with the mode off.
- **`proxy.ts` is not authorization.** It redirects for convenience and refreshes for continuity. Every request to the backend carries the cookies and the backend decides. A page that renders is not proof the user may see its data; the data it shows came through the backend's checks.
- **CSRF:** every non-GET request from the browser carries the custom header the backend requires (`X-Requested-With: fetch` unless the backend's api-docs name another). GET requests never change state, so a link or an image tag can never perform an action.
- **Refresh is single-flight and bounded:** one refresh at a time, one retry per request, never for the refresh route itself. A failed refresh ends the session on this client: cache cleared, redirect to sign-in, other tabs told.
- **Sign-out is server-side first.** The backend revokes; the client then clears its cache and tells other tabs. Closing a tab signs nobody out.
- **Role checks in the UI only hide controls.** A hidden button is a courtesy; the backend rejects the call either way. Never treat a role read from `useMe` as permission to show cached data from another role's session: the cache is cleared on every sign-in and sign-out.

## Input and output

- **Everything from the URL is input.** `params` and `searchParams` are parsed with a schema before use; an unexpected shape is a 404, not an exception and not a guess.
- **`returnTo` and every redirect target are validated:** relative paths only, starting with a single `/`, no scheme, no `//`. Anything else goes to the default route.
- **No `dangerouslySetInnerHTML`** except for content the backend has already sanitised and the screen spec marks as rich text, rendered through a sanitiser (DOMPurify with an explicit allowlist) as a second line. Markdown renders through a renderer with HTML disabled.
- **User-supplied URLs** (profile links, images) are rendered only with `http:` or `https:` schemes, `rel="noopener noreferrer"` on external links, and never as `javascript:` targets.
- **Uploads go straight to storage with a presigned URL the backend issues**, never through this app. The client checks type and size before asking for the URL as a courtesy; the backend's background verification is the check.
- **Forms validate with the same limits as the business rules**, and the backend's 422 is always honoured over the client's own schema.

## Headers and the browser

Set in `proxy.ts` (per-request, with a CSP nonce) and `next.config` (static headers):

- **Content-Security-Policy** with a per-request nonce for scripts, `default-src 'self'`, explicit allowlists for the API origin, the media CDN, Sentry, and analytics when installed; `frame-ancestors 'none'`; `object-src 'none'`; `base-uri 'self'`. Started in report-only on staging, enforced before production launch. A third party that needs `unsafe-inline` or `unsafe-eval` is a decision, raised in the build plan, not a line added quietly.
- **Strict-Transport-Security** with `includeSubDomains` and preload, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` denying camera, microphone, geolocation and payment unless a feature uses them, `X-Content-Type-Options: nosniff`.
- **Authenticated responses are `Cache-Control: private, no-store`.** No authenticated page is statically generated or stored in a shared cache, on Vercel or anywhere in between.
- **Nothing user-specific in the URL:** no tokens, no emails, no identifiers beyond the resource's own id. URLs end up in logs, history and referrers.

## Environment and secrets

- **`NEXT_PUBLIC_` is public.** Only values safe to ship to every browser carry the prefix: the API URL, the Sentry DSN, the analytics key. Every variable is declared in `lib/env.ts`, so an undeclared one fails the build and a secret cannot be exposed by a typo.
- **This client holds no secrets.** It has no API keys to the backend's providers, no storage credentials, no signing keys. Server-side code that needs a non-public value (a route handler, an edge function) imports `server-only` so the bundler fails if it ever reaches the client.
- **Secrets never enter the repository**: not in `.env` files that are committed, not in test fixtures, not in CI logs. Environment values live in Vercel per environment and in GitHub secrets for the workflows, with preview, staging and production separate.
- **Source maps** are uploaded to Sentry and not served publicly.

## Data in the browser

- **Server data lives only in the TanStack Query cache**, in memory. Persisted cache is off for authenticated data. A Zustand store that persists to storage holds UI preferences only.
- **Sensitive fields are masked by default** (card last four, partial identifiers) and revealed by an explicit action, as the design system specifies. Autocomplete attributes are set so password managers work and so browsers do not autofill into the wrong field (`autocomplete="one-time-code"`, `"new-password"`).
- **Clipboard, downloads and printing** of sensitive data happen only on an explicit user action.
- **Authenticated file downloads** (exports, receipts, documents) use a short-lived signed URL the backend issues per request, or stream through `clientFetch`; never a long-lived public URL.
- **Copy for auth failures is generic**: a sign-in error does not say whether the email exists.

## Third parties

- **Every third-party script is allowlisted in the CSP**, loaded through `next/script`, and listed in `tech-stack` or the build plan with what it is for. Adding one is a decision.
- **Analytics is proxied through this origin and carries no personal data:** no emails, names or free text in events or properties; the user is identified by the backend's id only after sign-in and reset on sign-out. Consent is asked where the product's users require it, and nothing is sent before it. The `analytics` rule has the detail.
- **Sentry scrubs before sending:** cookies, authorization headers, tokens, request and response bodies, form values, and every field the privacy section names. Session replay, if enabled, masks all text and inputs by default.
- **Embedded content** (maps, videos, payment fields) runs in iframes with explicit `allow` and `sandbox` attributes, from origins the CSP names.

## Payments

- **Card data never touches this app.** Payments use the provider's hosted checkout or hosted fields, in the provider's iframe. The client receives a token or session id and the backend does everything else.
- **Amounts shown come from the backend**; the client never computes a price the user will pay. It formats.
- **Payment confirmation comes from the backend's state**, polled or pushed after the provider redirects back, never from the redirect URL's parameters.

## Supply chain

- **The lockfile is committed and CI installs with `--frozen-lockfile`.**
- **New dependencies wait:** pnpm's `minimumReleaseAge` is set to seven days in `pnpm-workspace.yaml` (its default is one day), so a package version published an hour ago cannot be installed, which blocks most hijacked releases.
- **Install scripts are off by default** (pnpm's `allowBuilds` map in `pnpm-workspace.yaml`, with `strictDepBuilds` on) and enabled per package with a reason.
- **`pnpm audit` runs in CI**; a high or critical finding blocks the merge unless the PR records why it does not apply.
- **GitHub Actions are pinned to a commit SHA**, not a tag.
- **Dependabot or Renovate** opens update PRs that go through the same checks as any other.

## Errors and logging

- **Users see a message, never a stack trace, a path or an internal id.** `error.tsx` renders the designed error state; the detail goes to Sentry.
- **The browser console carries nothing in production**: no request bodies, no user data, no tokens.
- **Error states fail closed**: a screen that cannot verify what it may show shows nothing from the cache for that entity.

## Personal data

- **Collect only what the screen spec asks for**, and show only what the user's role needs for the action in front of them.
- **No personal data in URLs, analytics, logs, Sentry, or error messages.**
- **Deletion and export** flows render what the backend provides; the client keeps no copy that would survive the backend deleting the account.

## Backend contract

What this client assumes the backend provides. Items 1 to 4 and 7 are lines in the Backend Blueprint's security rule (v1.1.0 and later; item 7 from v1.2.0); 5 is what its api-docs document per endpoint; 6 is its build rule. A backend that differs is a conflict to raise before the first integration, never a workaround on this side.

1. **Cookies:** access and refresh tokens as `HttpOnly`, `Secure`, `SameSite=Lax` cookies, `Path=/`, on the parent domain (`.example.com`), so `app.example.com` and `api.example.com` share them. The sign-in and refresh responses set them; the sign-out response clears them.
2. **CORS:** the exact app origin per environment, with credentials. Never `*`, never the request's own origin echoed back.
3. **CSRF:** the backend requires the custom header on every state-changing request and rejects a non-GET without it; this client sends it on every non-GET.
4. **Refresh grace window:** a just-rotated refresh token is accepted for a short window (about ten seconds), so `proxy.ts`, `clientFetch` and several open tabs refreshing together do not trip reuse detection and sign the user out.
5. **Errors as documented:** 401 means no valid session (refresh or sign in), 403 means a valid session without permission (never refresh), and validation failures and every client-actionable error carry the shape and stable code the api-docs document for that endpoint.
6. **Additive changes only** once this client is in production, or a coordinated deploy for a breaking one, as the backend's build rules say.
7. **Real-time connections accept the session cookies:** an `EventSource`, `WebSocket` or Socket.IO connection opened with credentials from the app's origin is authenticated by the same cookies as HTTP, with the handshake's `Origin` checked against the CORS allowlist, so this client never fetches a ticket or handles a token for a channel.
