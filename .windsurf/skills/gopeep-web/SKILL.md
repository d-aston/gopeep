---
name: gopeep-web
description: >
  Analyze a website's HTML, CSS, and JavaScript — from source files, DevTools output, or
  pasted markup — and produce a structured reproduction prompt. Output is a complete
  gopeep-spec/ directory that another agent can use to recreate the frontend faithfully.
  Includes a human-editable 00-customization.md where colors, typography, UX copy, layout,
  and data structures can be adjusted before handing off to a recreating agent.
  Trigger: /gopeep-web, "analyze this website", "reverse engineer this UI", "document this frontend",
  "recreate this page", when given HTML/JS source to analyze.
---

Analyze the provided website source (HTML, CSS, JS files or pasted markup) and produce a `gopeep-spec/` directory. The primary output is `00-customization.md` — a human-editable file where design tokens, UX decisions, data structures, and behavior can be adjusted. The remaining phase files are read-only analysis that a recreating agent uses alongside the customization file.

## Invocation

```
/gopeep-web [source] [--output dir] [--depth quick|standard|full]
```

Source options (in priority order):
1. Path to directory containing HTML/CSS/JS files
2. Path to a single HTML file
3. URL — fetch live HTML, extract inline/linked CSS and JS
4. Pasted HTML/JS in the conversation — analyze directly

Default depth: `full`. Default output: `./gopeep-spec/`

## Input Acquisition

Before analysis, collect all available source material:

**From files/directory:**
- Read all `.html`, `.css`, `.js`, `.ts`, `.jsx`, `.tsx`, `.vue`, `.svelte` files
- Read build configs: `vite.config.*`, `webpack.config.*`, `next.config.*`, `nuxt.config.*`, etc.
- Read package manifests for framework/dependency detection

**From a URL:**
- Fetch page HTML
- Extract all `<link rel="stylesheet">` hrefs — fetch each
- Extract all `<script src="...">` hrefs — fetch each (skip CDN bundles >500KB)
- Extract inline `<style>` and `<script>` blocks
- Flag assets that returned 403/404 in confidence notes

**From pasted markup:**
- Work directly with provided content
- Note missing assets (external stylesheets, etc.) in confidence notes

---

## Execution Order

Run phases 1-5 first to gather all data. Then generate `00-customization.md` last, pulling editable values from across all phases into one place. Write each file before starting the next.

---

## Phase 1: Surface Reconnaissance → `01-reconnaissance.md`

Without deep analysis:
- Identify page type: landing page / app shell / dashboard / e-commerce / docs / blog / other
- Detect rendering model: static HTML / SSR / CSR SPA / hybrid
- Identify framework signals in HTML: `data-reactroot`, `__NEXT_DATA__`, `ng-version`, `data-v-*` Vue scoping, Svelte component comments, HTMX `hx-*` attributes
- Count: HTML files, CSS files, JS files, total assets
- Map page structure: header/nav/main/sidebar/footer presence
- Detect design system signals: Tailwind utility classes, Bootstrap grid, Material `mat-*`, Ant `ant-*`, Chakra, custom CSS vars
- Detect i18n signals
- Estimate complexity: `simple` / `moderate` / `complex`

---

## Phase 2: Stack & Dependency Analysis → `02-stack-and-dependencies.md`

From manifests and source signals, record:
- Framework + version: React/Vue/Angular/Svelte/vanilla/etc.
- Build tool: Vite/webpack/Parcel/Next.js/Nuxt/Remix/SvelteKit/etc.
- CSS approach: CSS Modules / Tailwind / styled-components / emotion / SCSS / BEM / vanilla
- State management: Redux/Zustand/Pinia/Jotai/signals/context/none
- Routing: React Router/Next router/Vue Router/hash/server-side/none
- Data fetching: fetch / Axios / React Query / SWR / GraphQL / tRPC
- UI component library: MUI / Ant / Chakra / Radix / shadcn/ui / Headless UI / custom
- Animation: Framer Motion / GSAP / Animate.css / CSS-only
- Form handling: React Hook Form / Formik / Vee-Validate / native
- Testing: Jest / Vitest / Playwright / Cypress / none

---

## Phase 3: Component & Layout Architecture → `03-architecture.md`

Map the full component/layout tree.

**Layout structure:**
- Identify layout regions (nav, sidebar, main, footer, modals, drawers)
- Map responsive breakpoints and layout shifts at each
- Identify grid system: CSS Grid / Flexbox / framework grid / custom

**Component inventory** — for each identifiable component:
- Name (inferred from class names, file names, function names)
- Props/inputs (HTML attributes, data-* attrs, JS function signatures)
- Internal state (from JS source or component analysis)
- Children/slots
- Events emitted or handled

**Data patterns:**
- API endpoints called (from fetch/axios/XHR calls)
- Data shapes (from response handling code or TypeScript types)
- Hardcoded fixture data
- localStorage/sessionStorage/cookie usage

**Routing:**
- All routes defined
- Route params and query string usage
- Protected routes / auth guards

Include Mermaid diagram of component hierarchy.

Mark editable items inline:
> **✏️ Editable** — see `00-customization.md` › [section name]

---

## Phase 4: Visual & Style System → `04-style-system.md`

Extract the complete design token system with exact values.

**Colors** — extract every distinct color value in use:
- Categorize as: primary / secondary / accent / neutral / semantic (success/warning/error/info) / surface / text
- Record exact hex/rgb/hsl values
- Note where each is used (background, border, text, etc.)

**Typography:**
- Font families (list all, note which is body vs heading vs mono)
- Size scale (all values in use, in px or rem)
- Weight scale (all weights in use)
- Line height values
- Letter spacing values if notable

**Spacing:**
- All padding/margin/gap values in use
- Whether a consistent scale is detectable (4px, 8px, 16px etc.)

**Shape:**
- All border-radius values
- All box-shadow/drop-shadow values

**Motion:**
- All transition durations in use
- Easing functions
- Animation keyframe names and behavior

**CSS conventions:**
- Naming: BEM / utility / CSS Modules / atomic / custom
- CSS custom properties: list all `--var-name` definitions and values
- Breakpoint values and names
- Dark mode approach (class toggle / `prefers-color-scheme` / none)

Mark editable items inline:
> **✏️ Editable** — see `00-customization.md` › [section name]

---

## Phase 5: Reproduction Blueprint → `05-reproduction-blueprint.md`

Synthesize phases 1-4 into an ordered build plan.

**Ordered tasks:**
1. Project scaffolding (framework, build tool, package setup)
2. Design token setup — use values from `00-customization.md › Colors`, `Typography`, `Spacing`
3. Layout shell (top-level regions)
4. Shared/primitive components (buttons, inputs, typography, icons)
5. Complex components (leaf-to-composite order)
6. Routing setup
7. Data fetching layer — use shapes from `00-customization.md › Data Structures`
8. Page-level assembly
9. Responsive behavior
10. Animations and micro-interactions
11. Accessibility pass

**For each component:**
- Exact props interface (TypeScript preferred)
- Styling approach referencing customization token names
- Behavior description
- Example usage

**Critical path:** Minimum steps for a visually representative skeleton.

**Decision points:** Choices the recreating agent must replicate.

**Verification steps:** What to check after each task group.

**Important:** When values in `00-customization.md` differ from what was analyzed, the customization file wins. Always read `00-customization.md` before starting any recreation task.

---

## Phase 6 (Final): Human Customization File → `00-customization.md`

**Generate this file last.** Pull editable values from all prior phases. This is the file a human edits. Write it to be readable without any technical background — avoid jargon where possible, explain what each setting does in plain English.

### Format

```markdown
<!--
  GOPEEP CUSTOMIZATION FILE
  ═══════════════════════════════════════════════════════════════
  Edit this file to customize the recreation before handing it
  to an agent. Change the values — not the labels or structure.

  Rules:
  - This file OVERRIDES the analyzed spec files
  - Change only the VALUE columns / content blocks below
  - Leave labels, section headers, and comments alone
  - If you're unsure about a value, leave it as-is (the original will be used)
  ═══════════════════════════════════════════════════════════════
-->

---
gopeep_version: "1.0"
source: <url or path>
analyzed_at: <ISO-8601>
---

# Customization

## How to use this file

Gopeep analyzed the source and filled in the values below. Change anything you want adapted, leave the rest as-is. When you hand the `gopeep-spec/` folder to an agent to recreate the project, it will read this file first and apply your changes.

---

## 🎨 Colors

> Change the **Value** column. Use any valid CSS color format (hex, rgb, hsl, oklch).
> The **Role** column explains where each color appears.

| Token | Value | Role | Used in |
|-------|-------|------|---------|
| `--color-primary` | `#______` | Main brand color | Buttons, links, active states |
| `--color-primary-hover` | `#______` | Primary on hover | Button hover, link hover |
| `--color-secondary` | `#______` | Secondary actions | Secondary buttons, badges |
| `--color-accent` | `#______` | Highlight / CTA emphasis | Banners, highlights |
| `--color-surface` | `#______` | Page/card background | Page bg, card bg |
| `--color-surface-raised` | `#______` | Elevated surface | Dropdowns, modals, popovers |
| `--color-border` | `#______` | Borders and dividers | Input borders, dividers, outlines |
| `--color-text` | `#______` | Body text | Paragraphs, labels |
| `--color-text-muted` | `#______` | Secondary/supporting text | Captions, placeholders |
| `--color-text-inverse` | `#______` | Text on dark backgrounds | Text on primary buttons |
| `--color-success` | `#______` | Positive states | Success alerts, valid inputs |
| `--color-warning` | `#______` | Caution states | Warning alerts |
| `--color-error` | `#______` | Error states | Error messages, invalid inputs |
| `--color-info` | `#______` | Informational | Info alerts, tooltips |

<!-- Add/remove rows as needed. Remove a row to keep the original analyzed value. -->

---

## 🔤 Typography

> Change font names to any Google Fonts or system font name.
> Change size values to adjust scale (keep the same unit — px or rem).

**Fonts**

| Role | Font Family | Fallback |
|------|-------------|----------|
| Body text | `______` | `system-ui, sans-serif` |
| Headings | `______` | `system-ui, sans-serif` |
| Monospace / code | `______` | `ui-monospace, monospace` |

**Size scale** (in order, smallest to largest)

| Name | Value | Used for |
|------|-------|---------|
| `--text-xs` | `______` | Captions, labels, badges |
| `--text-sm` | `______` | Supporting text, small UI |
| `--text-base` | `______` | Body copy |
| `--text-lg` | `______` | Lead text, card titles |
| `--text-xl` | `______` | Section headings |
| `--text-2xl` | `______` | Page headings |
| `--text-3xl` | `______` | Hero headings |

---

## 📐 Layout & Spacing

> Adjust the base unit to scale all spacing proportionally,
> or change individual values.

| Token | Value | Notes |
|-------|-------|-------|
| Base spacing unit | `______` | All spacing is a multiple of this |
| Max content width | `______` | Container max-width |
| Sidebar width | `______` | (if applicable) |
| Header height | `______` | (if applicable) |
| Border radius (small) | `______` | Inputs, badges |
| Border radius (medium) | `______` | Cards, buttons |
| Border radius (large) | `______` | Modals, large panels |

**Breakpoints**

| Name | Breakpoint | Behavior at this width |
|------|------------|----------------------|
| Mobile | `< ______` | ______ |
| Tablet | `______` | ______ |
| Desktop | `> ______` | ______ |

---

## 🧩 Components

> Describe how you want each component to behave or look.
> Be as specific or as vague as you like — the agent will fill in gaps.

### Buttons

```
Primary button style:   ______  (e.g. filled, rounded, with shadow)
Secondary button style: ______  (e.g. outlined, no shadow)
Button size (default):  ______  (e.g. 40px height, 16px padding)
Icon placement:         ______  (e.g. left of label, right of label)
Loading state:          ______  (e.g. spinner replaces text, disabled while loading)
```

### Forms & Inputs

```
Input style:            ______  (e.g. outlined, filled, underline)
Label position:         ______  (e.g. above input, floating placeholder)
Error message style:    ______  (e.g. red text below input, tooltip)
Required field marker:  ______  (e.g. asterisk, "(required)" text, none)
```

### Navigation

```
Nav style:              ______  (e.g. top bar, left sidebar, bottom tabs)
Active item indicator:  ______  (e.g. underline, filled background, left border)
Mobile nav:             ______  (e.g. hamburger menu, bottom bar, drawer)
```

### Cards

```
Card style:             ______  (e.g. flat with border, elevated with shadow, ghost)
Card padding:           ______
Card hover behavior:    ______  (e.g. lift shadow, border highlight, none)
```

### Modals & Overlays

```
Overlay backdrop:       ______  (e.g. dark blur, solid dark, light blur)
Modal enter animation:  ______  (e.g. fade + scale, slide up, none)
Close on backdrop click: ______  (yes / no)
```

---

## 📊 Data Structures

> Edit field names, types, or add/remove fields.
> The agent will use these shapes when generating mock data and API integration code.

<!-- Gopeep fills these in from analyzed API calls and data shapes -->
<!-- Format: TypeScript interface — edit field names and types freely -->

### [Primary entity — e.g. User, Product, Post]

```typescript
// EDIT THIS — change field names, types, add/remove fields
interface ______ {
  id: ______;
  // ... (gopeep fills from analysis)
}
```

<!-- Add more interfaces as needed, one per major data entity -->

---

## 🗺️ Routes & Navigation

> Add, remove, or rename routes.
> Mark routes as `public` or `protected`.

| Route | Page/View | Access | Notes |
|-------|-----------|--------|-------|
| `/` | ______ | public | |
| `______` | ______ | ______ | |

<!-- Add rows as needed -->

---

## ✍️ UX Copy

> Edit any text that should be different in the recreation.
> Leave blank to keep the original analyzed copy.

**Page titles**

| Page | Title |
|------|-------|
| Home | `______` |
| *(add pages as needed)* | |

**Key labels & CTAs**

| Element | Text |
|---------|------|
| Primary CTA button | `______` |
| Sign in label | `______` |
| Empty state message | `______` |
| Generic error message | `______` |
| Loading message | `______` |

---

## ⚙️ Behavior & UX Decisions

> Plain English — describe what you want. The agent interprets these.

```
Animations:         ______  (e.g. "subtle and fast", "none", "expressive")
Transitions:        ______  (e.g. "200ms ease", "snappy, no delay")
Dark mode:          ______  (yes / no / system-default)
Accessibility:      ______  (e.g. "WCAG AA minimum", "keyboard nav required", "standard")
Error handling:     ______  (e.g. "toast notifications", "inline errors only", "both")
Loading states:     ______  (e.g. "skeleton screens", "spinner", "none")
Scroll behavior:    ______  (e.g. "smooth scroll", "instant", "page-level only")
```

---

## 🔧 Technical Overrides

> Use this section to change framework, tooling, or architectural choices.
> Leave blank to keep what was analyzed.

```
Framework:          ______  (leave blank to keep analyzed: ______)
CSS approach:       ______  (leave blank to keep analyzed: ______)
State management:   ______  (leave blank to keep analyzed: ______)
Build tool:         ______  (leave blank to keep analyzed: ______)
TypeScript:         ______  (yes / no — leave blank to keep analyzed: ______)
Testing:            ______  (leave blank to keep analyzed: ______)
```
```

**When generating this file:**
- Fill in all `______` placeholders with extracted values from phases 1-5
- For colors: use exact extracted hex values
- For typography: use exact extracted font names and sizes
- For data structures: use TypeScript interfaces matching analyzed shapes
- For UX copy: use exact text found in the source
- Write `[not detected]` for any value that couldn't be determined — the human must fill it in
- Keep all comments and instructions in the file intact — they guide the human editor

---

## Output File Structure

```
gopeep-spec/
├── 00-customization.md          ← PRIMARY HUMAN-EDIT FILE
├── 01-reconnaissance.md
├── 02-stack-and-dependencies.md
├── 03-architecture.md
├── 04-style-system.md
├── 05-reproduction-blueprint.md
└── diagrams/
    ├── component-hierarchy.mmd
    └── layout-regions.mmd
```

## Phase File Format

```markdown
---
gopeep_version: "1.0"
phase: <N>
source: <url or path or "pasted">
analyzed_at: <ISO-8601>
complexity: <simple|moderate|complex>
editable: false
---

# <Phase Title>

> **Note for recreating agents:** Read `00-customization.md` first.
> Values there override anything in this file.

## Summary

## Details

## Confidence Notes
<!-- Sampled vs exhaustive. Missing assets. Dynamic content not visible in static HTML. -->

## Reproduction Instructions
```

## Editable Inline Markers

When a phase file contains a value that appears in `00-customization.md`, annotate it:

```markdown
Primary color: `#2563eb`
> **✏️ Editable** — override in `00-customization.md` › Colors › `--color-primary`
```

## Caveats

Document these in confidence notes:
- **Dynamic content:** Static snapshot misses empty/loading/error/authenticated states
- **Minified JS:** Best-effort analysis; flag when source maps unavailable
- **Server-side data:** API response shapes inferred from JS processing code
- **Fonts/icons:** Note external sources (Google Fonts, Lucide, Font Awesome, etc.)
- **Images:** Describe usage patterns and dimensions; don't reproduce actual images

## Success Criteria

A human can open `00-customization.md`, change colors, swap fonts, rename routes, and edit data shapes — then hand the entire `gopeep-spec/` folder to an agent, which recreates the project using the customized values throughout.
