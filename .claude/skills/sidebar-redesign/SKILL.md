---
name: sidebar-redesign
description: Redesign the Vue 3 client's navigation from a top nav bar into a modern SaaS-style vertical left sidebar, with consistent spacing and a polished professional look. Use this skill when asked to add a sidebar layout, convert the top nav to a left nav, or give the app a "modern SaaS" redesign.
---

# Sidebar Redesign

Converts the Factory Inventory Management client from its current top-nav layout into a left vertical sidebar layout, in the style of a modern SaaS product (e.g. Linear, Stripe Dashboard, Vercel). This is a layout/chrome change only — page content inside each view is untouched.

## Mandatory: delegate to vue-expert

Per the project's root `CLAUDE.md`: **any creation or significant modification of a `.vue` file must be delegated to the `vue-expert` subagent.** This skill's job is to plan the redesign precisely; the actual edits to `App.vue` / `FilterBar.vue` must be executed by `vue-expert`, not done directly. Hand `vue-expert` the concrete plan below (target structure, class names, exact values to change) rather than a vague "make it a sidebar" instruction.

## Current State (as of last audit)

All nav/layout markup and its styling live in **`client/src/App.vue`** — the `<style>` block there is global (unscoped), so every view (`Dashboard`, `Inventory`, `Orders`, `Demand`, `Spending`, `Reports`) inherits `.page-header`, `.stats-grid`, `.card`, `.badge`, `.table` etc. from it. Redesigning the sidebar does **not** require touching the view files themselves.

- `App.vue` template: `<header class="top-nav">` (logo + `<nav class="nav-tabs">` of `router-link`s + `LanguageSwitcher` + `ProfileMenu`) → `<FilterBar />` → `<main class="main-content"><router-view /></main>`.
- `.app` is currently `display: flex; flex-direction: column`.
- `.top-nav` is `position: sticky; top: 0` at a fixed **70px** height (`.nav-container { height: 70px }`).
- `FilterBar.vue` is itself `position: sticky; top: 70px` — hardcoded to sit directly under the 70px top nav. **This offset breaks the moment the top nav is removed** and must be updated in the same change.
- `.main-content { max-width: 1600px; margin: 0 auto; padding: 1.5rem 2rem }` assumes full viewport width available — must be revised once a sidebar occupies fixed width on the left.
- Routes (`client/src/main.js`): `/` Dashboard, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports`. Note `Backlog.vue` exists but is unrouted — do not add it to nav unless separately asked.
- Existing design tokens (reuse these, don't invent new ones):
  - Ink `#0f172a`, slate `#64748b`, muted border `#e2e8f0`, hover surface `#f1f5f9`/`#f8fafc`
  - Accent (active state) `#2563eb` text on `#eff6ff` background
  - Radii: `6px` (buttons/inputs), `10px` (cards)
  - Shadow: `0 1px 3px 0 rgba(0,0,0,0.05)` (resting), `0 4px 12px rgba(0,0,0,0.06)` (hover)
  - Spacing scale: multiples of `0.25rem`/`0.5rem`, section padding `1.25rem`–`2rem`

## Target Structure

```
.app                     → flex-direction: row (was column)
├── aside.sidebar        → fixed-width column, full viewport height, sticky/fixed left
│   ├── .sidebar-brand   → logo + company name + subtitle (was .logo)
│   ├── nav.sidebar-nav  → vertical stack of router-links (was .nav-tabs, horizontal)
│   └── .sidebar-footer  → LanguageSwitcher + ProfileMenu, pinned to bottom via margin-top:auto
└── .content-column      → flex: 1, min-width: 0 (new wrapper)
    ├── FilterBar        → no longer needs top-nav offset; sticky top: 0 within content column
    └── main.main-content → router-view, own scroll if sidebar is fixed height
```

**Sidebar spec:**
- Width: `240px`–`260px` fixed (pick `256px`), full height (`100vh`), non-scrolling with content.
- Background: white surface (`#ffffff`) with a single right-hand border `1px solid #e2e8f0` (matches existing `.top-nav` border-bottom treatment, just rotated).
- Brand block at top: company name + subtitle, same typography as current `.logo`/`.subtitle`, padded consistently with nav items below it.
- Nav links: one per line, full-width tap targets, icon + label (reuse simple inline SVGs — this app has no icon library and should not add one; keep 16–20px monochrome SVGs, `currentColor` fill, consistent with the reset-filters button icon already in `FilterBar.vue`). No emojis (project-wide rule).
- Active state: reuse the existing pattern — accent text `#2563eb` on `#eff6ff` background — but as a full-width rounded rect behind the row instead of the current bottom-border underline (the underline pattern doesn't read well vertically).
- Footer block: `LanguageSwitcher` + `ProfileMenu`, separated from nav by a top border, pushed to the bottom of the sidebar with `margin-top: auto` on a flex column.

**Content column:**
- `FilterBar.vue`: change `.filters-bar { top: 70px }` → `top: 0` (it's now the first sticky element in its own column, not stacked under a global header).
- `.main-content`: drop the `max-width: 1600px; margin: 0 auto` centering (no longer needed once width is naturally bounded by the sidebar) — keep it if a max content width is still desired for very wide screens, but re-derive the value relative to viewport minus sidebar width, not the old 1600px assumption.

## Implementation Steps

1. Read `client/src/App.vue` and `client/src/components/FilterBar.vue` fresh (don't assume the audit above is still accurate — verify).
2. Hand `vue-expert` a single task covering both files together (the sticky-offset coupling between them means they must change atomically):
   - Restructure `App.vue`'s template: `<header class="top-nav">` → `<aside class="sidebar">`, `.nav-tabs` → vertical `.sidebar-nav`, add `.content-column` wrapper around `<FilterBar />` + `<main>`.
   - Rewrite the corresponding global `<style>` rules (`.app`, `.top-nav` → `.sidebar`, `.nav-container` → `.sidebar-brand`/`.sidebar-nav`/`.sidebar-footer`, `.nav-tabs a` → vertical nav item styles, `.main-content`).
   - Update `FilterBar.vue`'s `.filters-bar { top: 70px }` → `top: 0`.
   - Preserve all existing behavior: `useI18n` (`t()` calls), `useAuth`, active-route highlighting via `$route.path`, `ProfileMenu`/`TasksModal`/`ProfileDetailsModal` wiring, `LanguageSwitcher`.
3. After edits, start/confirm the dev server (`npm run dev`, port 3000) and use Playwright MCP tools to verify:
   - Sidebar renders full-height on `/`, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports`.
   - Each nav link navigates and shows the active state correctly.
   - FilterBar still filters data correctly (no visual overlap/z-index issues with the sidebar).
   - ProfileMenu and LanguageSwitcher still open/work from the sidebar footer.
   - Check a narrow viewport (e.g. 768px) isn't badly broken — a full collapse/hamburger pattern is a reasonable stretch goal but not required unless asked.
4. Run the `code-reviewer` subagent on the diff before considering the task done (project convention for significant code changes).

## Non-goals

- Don't touch `server/` or any API contracts.
- Don't change view-level content (`views/*.vue` internals) — only the app chrome.
- Don't add a component/icon library — keep it dependency-free, matching the rest of the app.
- Don't route the unrouted `Backlog.vue` as part of this redesign unless explicitly asked.
