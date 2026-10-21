# Analytics

Rules for product analytics in this web client, installed only when the product tracks user behaviour. The tool is PostHog, named in `tech-stack`; what may never be sent is in `security` → Third parties.

## Setup

- **Same-origin ingest.** Events go to a path on this app (`/ingest`), rewritten in `next.config` to PostHog, so ad blockers do not drop them and the CSP needs no extra origin.
- **One client, initialised once**, in a provider at the root of the protected layout, with autocapture off. Every event is explicit.
- **Identify after sign-in, reset on sign-out.** The distinct id is the backend's user id; the person properties are the role, plan and account age, never a name or email.
- **Environments are separate projects.** Local and preview send nothing; staging and production each have their own key.

## Events

- **Named `object_action`** in snake case, past tense for things that happened: `order_created`, `invite_sent`, `filter_applied`. No screen names in event names; the screen is a property.
- **Properties are enums, numbers and ids**, never free text and never personal data. An amount is a number; a status is the backend's enum value.
- **One event per user intent**, fired from the handler that completed it (after the mutation succeeds), not on render and not on click.
- **Page views** are tracked once per route change by the provider, with the route pattern (`/orders/[id]`), not the concrete URL.
- **Every event is listed in `docs/analytics-events.md`** with its properties and the question it answers. An event nobody can say the purpose of is removed.

## Feature flags

Where the product uses them: flags are read through the PostHog client with a default, evaluated once per page load, and never gate a security decision. A flag that has been fully rolled out for a release is removed from the code.

## Consent

Where the product's users require it, nothing is sent before consent, the choice is stored in a cookie this app sets, and the provider respects it on initialisation. Consent UI follows the design system's content rules: plain words, equal buttons.
