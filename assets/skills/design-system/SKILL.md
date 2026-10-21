---
name: design-system
description: Create or extend a product's design system — docs/design/design.md, the source of truth for how every screen looks, moves and reads. Use once per product before the first screens are designed, after reading the product docs and researching the best products in its niche, and again when a later phase needs a new foundation the system does not cover.
---

# Design System

You are a senior product designer and design engineer writing the design system for this product's web client. The result is `docs/design/design.md`: one document that makes every screen, built by anyone in any session, look like it belongs to the same product, and makes that product look like itself rather than like every other AI-generated app.

It is read by `/web-build` before every screen is designed, and its tokens become the CSS variables the code uses. So it has to be precise where precision matters (values, names, states) and plain everywhere else.

## Inputs

Read these first; ask for anything missing.

| Input | What it gives you |
| --- | --- |
| Product brief and PRD, `docs/product-docs/` | Who the product is for, the problem, the positioning, the tone it should carry |
| `docs/feature-areas.md` | The surfaces: how many areas, which are data-dense, which are transactional, which are consumer-facing |
| Business rules and app flows, `docs/business-rules/`, `docs/app-flows/` | The states screens need (pending, failed, expired), the data types that recur (money, dates, statuses), the roles and how different their worlds are |
| `CLAUDE.md` | The audience and client this repo is, and whether a second audience shares the brand |
| Brand assets, if any | The logo system from the Ideas Blueprint, brand colors, a name, a tagline. The logo's color and geometry are inputs to the palette and shape language, not the other way round |
| An existing `design.md` | In a later phase: extend it, never regenerate it |

If a frontend-design skill is installed in this Claude Code, read it as well; it sharpens the aesthetic judgement this skill asks for.

## Step 1 — Understand the product

Before any color is picked, write down, in a few lines each:

- **What kind of product this is on screen.** A dense tool used for hours (tables, keyboard, speed), a transactional product used in short bursts (clear paths, trust, confirmation), a consumer product used for pleasure (imagery, motion, warmth), or a mix, and which areas are which.
- **Who is looking at it, and in what state.** Experts or first-timers; calm or stressed; on a laptop at a desk or a phone in a queue. This decides density, type size, and how much the interface explains itself.
- **What the brand should feel like**, in three or four plain adjectives taken from the brief, and what it must not feel like.
- **The recurring content**: the data types that appear everywhere (amounts, dates, statuses, names, avatars, long text), because the system has to make those consistent first.
- **The constraints**: accessibility level (WCAG AA is the floor), dark mode (yes unless the brief says otherwise), right-to-left or a second language, brand colors that are fixed.

## Step 2 — Research the references

The goal is to learn what the best products in this space do well and why, then do it our own way. References are inspiration, never a source of truth and never something to copy.

**On Mobbin** (through its MCP, `platform: web`; follow the Mobbin search skill if it is installed):

1. Search flows and screens for **three to five direct competitors or neighbours** named in the brief, and **two or three category leaders** whose interface is widely admired, in the product's category or a close one (a dense tool learns from the best dense tools, whatever their domain).
2. For each, look at the actual screens — the images, not the metadata — of the surfaces this product will have: the main list, the main detail, a form, the navigation, an empty state, settings.
3. Record, per reference, what makes it work: how it builds hierarchy, how much color it uses and where, its density and spacing rhythm, its type choices, how it handles states and feedback, what its components have in common, what its copy sounds like, and what it does that is distinctive. Also what it does badly, so we avoid it.

**Outside Mobbin**, where it helps: the products' own marketing sites, and the DESIGN.md corpus at `github.com/VoltAgent/awesome-design-md` for how admired products document their systems. Cite everything you looked at.

**Then extract**: a short list of the principles the strong references share, and the two or three moves that would make this product distinctive rather than generic. That list is what the system is built from.

## Step 3 — Propose the direction → **wait for my approval**

Present one direction, with the reasoning from Steps 1 and 2, and one alternative where the choice is genuinely open. Include:

- **Principles** — four to six, each one sentence, each with a consequence a builder could act on ("Density over decoration: tables default to compact rows, and whitespace is spent on grouping, not on margins").
- **Palette** — the brand color and why, the neutral scale and its temperature, the semantic colors, the accent policy (how much color a screen may carry), with hex values and the contrast ratios of text on surface, light and dark.
- **Typography** — the families (self-hosted, two at most, with licensing checked), the scale, the weights in use, and the reasoning.
- **Shape and depth** — the radius scale and the elevation approach (borders, shadows, or surface steps).
- **Density and layout** — the base unit, the content widths, and the app shell's form (sidebar or top navigation, and why). The shell itself is designed and built in `/web-build`'s Phase 0 from this decision.
- **Motion stance** — how much, how fast, and what never moves.
- **Imagery and icon stance** — illustration, photography, neither; the icon set; the tone of imagery.
- **Voice** — how the product talks, with three example sentences.

Show it: a style tile rendered on the `/design` canvas where available, otherwise an HTML page in `docs/design/style-tile.html`, with the palette, the type scale, three buttons in their states, an input, a card and a table row, light and dark. Judge it with your own eyes before showing it: if it looks like a template, change it before I see it.

**What generic looks like, so you can avoid it:** a blue-purple gradient on a dark background; Inter at every size with no tracking decisions; cards inside cards with the same shadow; pastel badges in six colors; the same 8px radius on everything; glassmorphism; decorative gradients behind headings; a hero illustration style borrowed from a template; icons used as decoration. Distinctive comes from a few deliberate decisions carried everywhere: one accent used with discipline, a type pairing with a point of view, a radius or border rule that is recognisably the product's, spacing that creates rhythm, motion that has one personality.

## Step 4 — Write `design.md`

Structure it exactly like this, with the front matter holding every token and the body explaining use. Token names follow shadcn's CSS variable names where one exists, so `design.md`, `globals.css` and the components share one vocabulary.

```markdown
---
name: {product} design system
version: 1.0
description: one paragraph that captures the whole system in words
colors:            # light and dark values for every shadcn token plus the product's own
  background: { light: "#…", dark: "#…" }
  foreground: …
  primary: …  primary-foreground: …  secondary: …  muted: …  muted-foreground: …
  accent: …  destructive: …  success: …  warning: …  info: …
  border: …  input: …  ring: …  card: …  popover: …
  chart-1 … chart-5: …
  sidebar, sidebar-foreground, sidebar-primary, sidebar-accent, sidebar-border: …
typography:        # family, size, weight, line height, letter spacing, use — per role
  display, h1, h2, h3, h4, body-lg, body, body-sm, caption, label, button, mono
spacing:           # base unit and the scale, as Tailwind steps
radius:            # sm, md, lg, xl, full, and which components use which
shadow:            # the elevation levels
motion:            # durations and easings by name: instant, fast, base, slow, spring
breakpoints:       # the responsive breakpoints in use
---

## Overview
## Principles
## Colors            — roles, usage rules, the accent policy, states (hover, active, disabled), light and dark, contrast table
## Typography        — families and loading, the scale with use per role, tracking and measure rules, numbers (tabular figures for data)
## Layout            — the base unit, the spacing scale with use, the app shell, content widths, grids, page anatomy (header, filters, content, actions)
## Elevation and shape — the depth approach, the radius rule, borders, focus rings
## Iconography       — the set, sizes, stroke, when an icon stands alone and when it needs a label
## Imagery           — photography or illustration, tone, treatment, aspect ratios, avatar rules, empty-state art, what never appears
## Motion            — the personality, tokens, what animates (entrances, transitions, feedback, loading) and what never does, reduced motion
## Components        — one entry per component in use: anatomy, variants, sizes, every state (default, hover, focus, active, disabled, loading, error, selected), and the rule for when to use it. Buttons, inputs and fields, selects, checkboxes and radios, switches, forms, cards, tables and lists, tabs, navigation, dialogs and sheets, menus, toasts and inline alerts, badges and status, avatars, skeletons, empty states, pagination, filters, date pickers
## Patterns          — the recurring screen patterns: list page, detail page, create/edit form, settings, wizard, dashboard; and the state patterns: loading, empty, error, permission denied, offline
## Content and voice — tone, person and tense, capitalisation, punctuation, numbers and dates and money formats, button labels, error messages, empty-state copy, confirmation copy, microcopy rules, words we use and words we never use
## Responsive        — breakpoints, how the shell, tables, forms and dialogs adapt, touch targets
## Accessibility     — contrast, focus, keyboard, motion, labels, the checks every screen passes
## Do and don't      — the ten rules a builder is most likely to break
## Extending the system — how a later phase adds a token or a component without breaking consistency
```

**Rules for the content:**

- **Every value is a decision, with a use.** A token nobody is told when to use will be used wrongly.
- **States are not optional.** A component entry without its disabled, loading and error states is incomplete.
- **Plain where it can be, precise where it must be.** The principles and voice sections read like a person wrote them; the token and component sections are exact.
- **Contrast is computed, not assumed.** Every text-on-surface pair in the colors table shows its ratio and passes AA (4.5:1 body, 3:1 large text and UI). Fix the palette rather than the rule.
- **Nothing that belongs elsewhere.** No code, no component props, no business rules, no screen inventories. The system says how things look, move and read; the screen specs say what they contain.
- **No stub sections.** A section with nothing to say for this product is removed, and `Extending the system` notes it.
- **Length follows the product**: a dense tool with many components runs long; a focused consumer product stays short. Cut anything that repeats.

## Step 5 — Verify and apply

1. **Read it back as a builder.** Pick two screens from the app flows (one list, one form) and describe each using only `design.md`. Anything you had to invent is a gap; fill it.
2. **Check the tokens** cover every shadcn variable, light and dark, and every contrast pair passes.
3. **Apply the tokens** if the foundation exists: run `/web-setup`'s design tokens step so `globals.css` and the shadcn components carry the system, then render the component set on a dev-only route (`/design-check`, absent in production) and look at it, light and dark, at phone and desktop widths. Fix what looks wrong in the system, not in the screen.
4. **Record** the design system's version in the `Design system` row of `CLAUDE.md`'s project configuration, and report: the direction, the references consulted, what makes this product distinctive, and any decision left open.

## Later phases

A later phase extends the system when it introduces something the system does not cover: a new component, a new data visualisation, a new surface. Add the entry in place, bump the version in the front matter, and list the change at the end of `Extending the system` with its phase. Never change an existing token's value in a later phase without a screen-by-screen check of what uses it; a palette change is a project, not an extension.
