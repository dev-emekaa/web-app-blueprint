## Documentation map

| Type | Location |
| --- | --- |
| Feature-areas registry *(from the backend)* | `docs/feature-areas.md` |
| PRD and product brief *(from the backend)* | `docs/product-docs/` |
| Business rules *(from the backend)* | `docs/business-rules/` |
| App flows *(from the backend)* | `docs/app-flows/` |
| API docs *(from the backend)* | `docs/api-docs/` |
| API testing *(from the backend)* | `docs/api-testing/` |
| Phase folders *(from the backend, plus this repo's UI tracker and the phase's screen specs)* | `docs/phase-{n}/` |
| UI tracker, first phase | `docs/ui-implementation-order.md` |
| Design system | `DESIGN.md` (project root) |
| Style tile *(only without the canvas)* | `docs/design/style-tile.html` |
| Route map | `docs/design/routes.md` |
| Approved designs | `docs/design/screens/` |
| Image prompts | `docs/design/images.md` |
| Screen specs | `docs/screen-specs/` |
| Backend issues handoff | `docs/integration-handoff/` |
| Analytics events *(with the analytics rule)* | `docs/analytics-events.md` |
| Infra setup | `docs/infra/` |

## Audiences and clients

<!-- Copy the table from the backend's CLAUDE.md, then mark which audience and client this repo is.
     An audience is a group of roles using its own separate product; a client is a separately designed app. -->

| Audience | Primary | Clients |
| --- | --- | --- |

| This repo is | Value |
| --- | --- |
| Audience | |
| Client | web |

## Project configuration

<!-- The web skills read these. Anything missing, they ask for once; /web-setup fills in the commands.
     Hosting is Vercel. The backend repo path is used by the docs sync scripts. -->

| Key | Value |
| --- | --- |
| Project name | |
| Stack / framework | {{STACK}} |
| Backend repo path | |
| Backend stack | |
| Backend OpenAPI spec | URL, once the backend is built — blank until then |
| Local API URL | |
| Staging API URL | |
| Production API URL | |
| Local app URL | `https://local.{domain}:3000` when running against a deployed backend |
| Staging app URL | |
| Production app URL | |
| Dev server command | |
| Test commands | |
| API types command | |
| Docs sync commands | |
| Design system | not written yet — `/design-system` updates this |
| Design canvas | /design (Claude Design in Claude Code) |
| Design reference library | Mobbin, through its MCP |
| Image generation | Gemini, through its MCP |
| Stock photo providers | Unsplash / Pexels — which are configured; their keys live in `.env` |
| Analytics | off |
| Production status | not yet launched |
| Production deploy trigger | Deploy Production workflow, run manually on the verified commit |
| First-load JS budget | 200 KB |
