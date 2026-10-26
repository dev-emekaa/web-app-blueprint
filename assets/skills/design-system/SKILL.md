---
name: design-system
description: Create or extend a product's design system — DESIGN.md at the project root, the source of truth for how every screen looks, moves and reads, in the DESIGN.md format. Use once per product before the first screens are designed, after reading the product docs and researching the best products in its niche, and again when a later phase needs a foundation the system does not cover.
---

# Design System

You are a senior product designer and design engineer writing the design system for this product's web client. The result is `DESIGN.md` at the project root: one document that makes every screen, built by anyone in any session, look like it belongs to the same product, and makes that product look like itself rather than like every other AI-generated app.

It follows the DESIGN.md format (`github.com/google-labs-code/design.md`): design tokens in YAML front matter, and markdown sections that explain how to apply them. `/web-build` reads it before every screen is designed, design tools find it at the root, and its tokens become the CSS variables the code uses. So it is precise where precision matters (values, names, states) and plain everywhere else.

## Inputs

Read these first; ask for anything missing.

| Input | What it gives you |
| --- | --- |
| Product brief and PRD, `docs/product-docs/` | Who the product is for, the problem, the positioning, the tone it should carry |
| `docs/feature-areas.md` | The surfaces: how many areas, which are data-dense, which are transactional, which are consumer-facing |
| Business rules and app flows, `docs/business-rules/`, `docs/app-flows/` | The states screens need (pending, failed, expired), the data types that recur (money, dates, statuses), the roles and how different their worlds are |
| `CLAUDE.md` | The audience and client this repo is, and whether a second audience shares the brand |
| Brand assets, if any | The logo system from the Ideas Blueprint, brand colors, a name, a tagline. The logo's color and geometry are inputs to the palette and shape language, not the other way round |
| My screenshots in `docs/design/references/`, if any | Screens I picked from products I admire — the most direct statement of what I want. Optional; never committed |
| Another client's `DESIGN.md` for the same product, if one exists (at the sibling client repo path in `CLAUDE.md`) | The brand decisions — palette, type, voice, shape — are taken from it so the product looks like itself on every client; the web components, patterns and motion are this system's own |
| An existing `DESIGN.md` | In a later phase: extend it, never regenerate it |

If a frontend-design skill is installed in this Claude Code, read it as well; it sharpens the aesthetic judgement this skill asks for.

## Step 1 — Understand the product

Before any color is picked, write down, in a few lines each:

- **What kind of product this is on screen**, because it sets density, pace and tone. Most products are one of these, or a mix by feature area; say which applies where.
  - *A work tool used for long sessions* (dashboards, admin, data-heavy apps): users scan a lot and act fast, so the system leans compact. Denser tables and lists, smaller type, visible focus and keyboard shortcuts, restrained color.
  - *A transactional product used in short visits* (booking, payments, sign-ups): users need the confidence to finish. One clear primary action per screen, generous spacing, strong feedback and confirmation states.
  - *A consumer product used for enjoyment* (social, media, lifestyle): users come back for the experience, so imagery, motion and personality carry more weight.
- **Who is looking at it, and in what state.** Experts or first-timers; calm or stressed; on a laptop at a desk or a phone in a queue. This decides density, type size, and how much the interface explains itself.
- **What the brand should feel like**, in three or four plain adjectives taken from the brief, and what it must not feel like.
- **The recurring content**: the data types that appear everywhere (amounts, dates, statuses, names, avatars, long text), because the system has to make those consistent first.
- **The constraints**: accessibility level (WCAG AA is the floor), dark mode (yes unless the brief says otherwise), right-to-left or a second language, brand colors that are fixed.

## Step 2 — Research the references

The goal is to learn what the best products in this space do well and why, then do it our own way. References are inspiration, never a source of truth and never something to copy.

**Choose them:** **three to five direct competitors or neighbours** named in the brief, and **two or three category leaders** whose interface is widely admired, in the product's category or a close one (a dense tool learns from the best dense tools, whatever their domain). For each, look at the actual screens — the images, not the metadata — of the surfaces this product will have: the main list, the main detail, a form, the navigation, an empty state, settings.

**Where to see them** is set by the `Design references` row of `CLAUDE.md` (called `Design reference library` in projects set up before it; rename it when you meet it). If Mobbin isn't connected and the row doesn't say what to use instead, ask me once — web research, my screenshots, or both — and record the answer in the row; if I'm not there to answer, record and use web research plus any screenshots in the folder. A missing library never stops the work. Use what the row allows, in this order:

1. **My screenshots**, if `docs/design/references/` holds any. They are optional: screens I picked from products I admire, so they say most directly what I want. Look at every one. The folder is never committed — confirm `.gitignore` lists `docs/design/references/` and add it if not — because the images are other companies' and may show personal data; the docs record what each one taught and its file name, never the image.
2. **Mobbin**, when connected (through its MCP, `platform: web`; follow the Mobbin search skill if it is installed).
3. **Web research**, when Mobbin isn't connected or leaves a gap: the chosen products' own sites, product tours, help-center articles and changelogs with screenshots, opened in a browser so you see the screens themselves, not marketing illustrations.
4. **None of these available:** build the direction from the brief, the category's conventions and the corpus below, and say so in the report.

**Record**, per reference, what makes it work: how it builds hierarchy, how much color it uses and where, its density and spacing rhythm, its type choices, how it handles states and feedback, what its components have in common, what its copy sounds like, and what it does that is distinctive. Also what it does badly, so we avoid it. Where the direction draws on a product documented in the corpus at `github.com/VoltAgent/awesome-design-md`, that product's DESIGN.md shows its exact decisions; learn from it, never copy. Cite everything you looked at — its link, or the screenshot's file name.

**Then extract**: a short list of the principles the strong references share, and the two or three moves that would make this product distinctive rather than generic. That list is what the system is built from.

## Step 3 — Propose the direction → **wait for my approval**

Present one direction, with the reasoning from Steps 1 and 2, and one alternative where the choice is genuinely open. Include:

- **Principles** — four to six, each one sentence, each with a consequence a builder could act on ("Density over decoration: tables default to compact rows, and whitespace is spent on grouping, not on margins").
- **Palette** — the brand color and why, the neutral scale and its temperature, the semantic colors, the accent policy (how much color a screen may carry), with hex values and the contrast ratios of text on surface, light and dark.
- **Typography** — the families, the scale, the weights in use, and the reasoning. Free fonts only, two families at most: Google Fonts by default, or another open-source font (SIL OFL or similar). Never a font that must be bought, or one licensed for personal use only.
- **Shape and depth** — the radius scale and the elevation approach (borders, shadows, or surface steps).
- **Density and layout** — the base unit, the content widths, and the app shell's form (sidebar or top navigation, and why). The shell itself is designed and built in `/web-build`'s Phase 0 from this decision.
- **Motion stance** — how much, how fast, and what never moves.
- **Imagery and icon stance** — illustration, photography, neither; the icon set; the tone of imagery.
- **Voice** — how the product talks, with three example sentences.

Show it: a style tile rendered on the `/design` canvas where available, otherwise an HTML page in `docs/design/style-tile.html`, with the palette, the type scale, three buttons in their states, an input, a card and a table row, light and dark. Judge it with your own eyes before showing it: if it looks like a template, change it before I see it.

**What generic looks like, so you can avoid it:** a blue-purple gradient on a dark background; Inter at every size with no tracking decisions; cards inside cards with the same shadow; pastel badges in six colors; the same 8px radius on everything; glassmorphism; decorative gradients behind headings; a hero illustration style borrowed from a template; icons used as decoration. Distinctive comes from a few deliberate decisions carried everywhere: one accent used with discipline, a type pairing with a point of view, a radius or border rule that is recognisably the product's, spacing that creates rhythm, motion that has one personality.

## Step 4 — Write `DESIGN.md`

Tokens go in the front matter, using only the format's keys. Values the format has no key for — shadows, motion, breakpoints — are written as tables in their sections. Color tokens use shadcn's variable names, so `DESIGN.md`, `globals.css` and the components share one vocabulary; each color's light value has a `-dark` twin.

```yaml
---
version: alpha
name: "{Product} Design System"
description: One paragraph that captures the whole system in words.
colors:
  background: "#…"
  background-dark: "#…"
  foreground: "#…"
  foreground-dark: "#…"
  # …the same pair for primary, primary-foreground, secondary, secondary-foreground, muted, muted-foreground,
  # accent, accent-foreground, destructive, success, warning, info, border, input, ring, card, card-foreground,
  # popover, popover-foreground, chart-1 to chart-5, sidebar, sidebar-foreground, sidebar-primary,
  # sidebar-primary-foreground, sidebar-accent, sidebar-accent-foreground, sidebar-border, sidebar-ring,
  # and the product's own tokens
typography:
  display: { fontFamily: …, fontSize: …, fontWeight: …, lineHeight: …, letterSpacing: … }
  # …h1, h2, h3, h4, body-lg, body-md, body-sm, caption, label-md, button, mono, numeric (fontFeature: '"tnum"')
rounded: { none: 0px, sm: …, md: …, lg: …, xl: …, full: 9999px }
spacing: { base: 4px, … }            # the scale, matching Tailwind's steps
components:
  button-primary: { backgroundColor: "{colors.primary}", textColor: "{colors.primary-foreground}", typography: "{typography.button}", rounded: "{rounded.md}", padding: … }
  button-primary-hover: { backgroundColor: … }
  button-primary-dark: { backgroundColor: "{colors.primary-dark}", textColor: "{colors.primary-foreground-dark}" }
  # …one entry per component and state, with a -dark entry carrying its dark colors
---
```

Component entries use only the format's properties: `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`. Borders, shadows and focus rings are described in the component's section.

Then the sections, with exactly these `##` headings in this order:

```markdown
## Overview            — the system in a paragraph; the principles from Step 3; then Key Characteristics: the five to eight visual decisions that make it this product
## Colors              — grouped by role (brand and accent, surface, text, semantic, charts), each with its use; the accent policy; hover, active and disabled; light and dark; a contrast table
## Typography          — the families, why, and where each loads from; a hierarchy table (token, size, weight, line height, tracking, use); measure and tracking rules; tabular figures for numbers in data
## Layout              — the base unit and the spacing scale with use; the app shell; content widths; grids; page anatomy (header, filters, content, actions)
## Elevation & Depth   — how hierarchy is shown (shadows, borders or surface steps), as a table of level, treatment, shadow value and use
## Shapes              — the radius scale as a table of token and use; borders; focus rings
## Iconography         — the set, sizes, stroke, when an icon stands alone and when it needs a label
## Imagery             — photography or illustration, tone, treatment, aspect ratios, avatars, empty-state art, what never appears
## Motion              — the personality; a table of durations and easings by name; what animates and what never does; reduced motion
## Components          — one entry per component in use: anatomy, variants, sizes, every state (default, hover, focus, active, disabled, loading, error, selected), and when to use it. Buttons, inputs and fields, selects, checkboxes and radios, switches, forms, cards, tables and lists, tabs, navigation, dialogs and sheets, menus, toasts and inline alerts, badges and status, avatars, skeletons, empty states, pagination, filters, date pickers; then the signature components only this product has
## Patterns            — screen patterns (list, detail, create/edit form, settings, wizard, dashboard) and state patterns (loading, empty, error, permission denied, offline)
## Content and Voice   — tone, person and tense, capitalisation, punctuation; formats for numbers, dates and money; button labels; error, empty-state and confirmation copy; words we use and words we never use
## Responsive          — a table of breakpoints; how the shell, tables, forms and dialogs adapt; touch targets
## Accessibility       — contrast, focus, keyboard, motion, labels, and the checks every screen passes
## Do's and Don'ts     — the ten rules a builder is most likely to break
## Iteration Guide     — how to apply the system (reference components and tokens by name, one component at a time, the default body style, how scarce the accent is) and how a later phase extends it
```

**Rules for the content:**

- **Every value is a decision, with a use.** A token nobody is told when to use will be used wrongly.
- **Tokens are the source; prose explains them.** A value in prose always matches its token, and names it (`{colors.primary}`).
- **States are not optional.** A component entry without its disabled, loading and error states is incomplete.
- **Plain where it can be, precise where it must be.** The Overview and Content and Voice sections read like a person wrote them; the token and component sections are exact.
- **Contrast is computed, not assumed.** Every text-on-surface pair in the colors table shows its ratio and passes AA (4.5:1 body, 3:1 large text and UI). Fix the palette rather than the rule.
- **Nothing that belongs elsewhere.** No code, no component props, no business rules, no screen inventories. The system says how things look, move and read; the screen specs say what they contain.
- **No stub sections.** A section with nothing to say for this product is left out.
- **Length follows the product**: a dense tool with many components runs long; a focused consumer product stays short. Cut anything that repeats.

## Step 5 — Verify and apply

1. **Lint it:** `npx @google/design.md lint DESIGN.md`. No errors. The linter checks every component pair against 4.5:1, so a contrast warning is fixed in the palette unless the pair is large text or a non-text element that the contrast table shows at 3:1 or more. Orphaned-token warnings are expected for colors no component token can reference (borders, rings, charts); any other warning is resolved.
2. **Read it back as a builder.** Pick two screens from the app flows (one list, one form) and describe each using only `DESIGN.md`. Anything you had to invent is a gap; fill it.
3. **Check the tokens** cover every shadcn variable with its `-dark` twin, and every pair in the contrast table passes.
4. **Apply the tokens** if the foundation exists: run `/web-setup`'s design tokens step so `globals.css` and the shadcn components carry the system, then render the component set on a dev-only route (`/design-check`, created here if it does not exist, absent in production) and look at it, light and dark, at phone and desktop widths. Fix what looks wrong in the system, not in the screen.
5. **Record** in the `Design system` row of `CLAUDE.md`'s project configuration that `DESIGN.md` exists and the phase it was last extended in, and report: the direction, the references consulted, what makes this product distinctive, and any decision left open.

## Later phases

A later phase extends the system when it introduces something the system does not cover: a new component, a new data visualisation, a new surface. Add the tokens and the section text in place, lint again, and compare with the committed version (`git show HEAD:DESIGN.md > /tmp/DESIGN.prev.md`, then `npx @google/design.md diff /tmp/DESIGN.prev.md DESIGN.md`); the token changes it lists go in the report. Update the `Design system` row in `CLAUDE.md` with the phase. Never change the value of an existing design token (a color, size, radius or other value in the front matter) in a later phase. Every built screen already uses it, so changing it restyles screens nobody asked to change. If an existing value has to change — a new brand color, say — that is a redesign, not part of a phase: raise it with me, and if I approve it, plan it as its own piece of work that checks every screen using that value and updates every client that shares the brand.
