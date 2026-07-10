# Handoff: Compliance Dashboard

## Overview
An internal CI/compliance tracking tool. Compliance "initiatives" (e.g. a required Xcode version, a security scan) are defined, move through a lifecycle (Draft → Submitted → Review → Development → Test → Ready), and are enforced against a portfolio of engineering "projects" (repos). Each project's compliance status per initiative is tracked, with manual-override support. Includes a per-project detail page (repo overview, build/PR/release stats, docs, compliance status per initiative, build history).

## About the Design Files
The files in this bundle (`CI Portal.html`, `ci-styles.css`) are **design references built in HTML** — high-fidelity prototypes of the intended look, states, and interactions. They are not production code to import as-is. The task is to **recreate this design in the target codebase's existing environment** (React, Vue, native, etc., whatever the real Compliance Dashboard app uses) using its established component patterns, routing, and data layer — or, if no environment exists yet, choose the most appropriate framework and implement there. Do not literally copy the vanilla-JS `nav()` / `render*()` functions into a real app; treat them only as a spec for what each screen shows and how it changes state.

## Fidelity
**High-fidelity.** Colors, typography, spacing, and states below are final — implement pixel-for-pixel, not just "in the spirit of." The #1 failure mode when this design gets reimplemented is color drift (see Design Tokens) — always reference the exact hex/rgba values below, never approximate or invent new ones.

## Design Tokens
All values are CSS custom properties in `ci-styles.css`, switched by `[data-theme]` and `[data-accent]` attributes on `<html>`/`<body>`. **Copy these tokens verbatim into the target codebase's theme layer** (CSS variables, Tailwind config, styled-components theme, etc.) rather than re-deriving colors from the screenshots.

### Typography
- Sans (UI): `IBM Plex Sans` (weights 300/400/500/600/700) — all body text, buttons, labels
- Mono: `IBM Plex Mono` (weights 300–600) — code, dates, IDs, stats
- Serif: `Instrument Serif` — imported but currently unused in-page; safe to drop if not needed
- Base font size: `14px` (`html { font-size: 14px }`) — all other sizes are px, not rem
- Headings: `font-weight: 700; letter-spacing: -0.03em`

### Color — Dark theme (default, `[data-theme="dark"]`)
| Token | Hex | Usage |
|---|---|---|
| `--bg` | `#14110a` | page background |
| `--surface` | `#1b1710` | nav bar, docs panel |
| `--card` | `#1f1b13` | card / input backgrounds |
| `--card-2` | `#26201a` | raised card, hover bg, table header |
| `--border` | `#2e2719` | card/input borders |
| `--border-s` | `#241e14` | subtle row dividers |
| `--text` | `#ece3cd` | primary text |
| `--muted` | `#b9ad8b` | secondary text |
| `--dim` | `#76694e` | tertiary/label text |

### Color — Light theme (`[data-theme="light"]`)
| Token | Hex |
|---|---|
| `--bg` | `#faf6ef` |
| `--surface` / `--card` | `#ffffff` |
| `--card-2` | `#f4efe5` |
| `--border` | `#e2d8c5` |
| `--border-s` | `#ede7d8` |
| `--text` | `#1e1a12` |
| `--muted` | `#6b5e44` |
| `--dim` | `#9a8a6e` |

### Accent (3 selectable palettes, `[data-accent]` on `<html>`/`<body>`)
- **Indigo/terracotta (default)**: `--accent:#d97a4a` `--accent-2:#c19a4b` `--accent-fg:#e8966a` `--accent-dim:rgba(217,122,74,.12)`
- **Teal**: `--accent:#c19a4b` `--accent-2:#d4b16a` `--accent-fg:#d4b06a` `--accent-dim:rgba(193,154,75,.12)`
- **Violet**: `--accent:#7ba35e` `--accent-2:#b8c75c` `--accent-fg:#96bf76` `--accent-dim:rgba(123,163,94,.12)`

### Status colors (theme-independent, used for badges/dots/bars)
- Green (compliant/ready/success): `--green:#7ba35e` `--green-dim:rgba(123,163,94,.16)`
- Amber (in-progress/warn/review): `--amber:#c19a4b` `--amber-dim:rgba(193,154,75,.16)`
- Red (non-compliant/block/fail): `--red:#d97a4a` `--red-dim:rgba(217,122,74,.14)`
- Blue (submitted/dev, hardcoded not tokenized): `#82b0e8` (submitted), `#6baed8` (development)
- Violet (test phase, hardcoded): `#b88ee0`, bg `rgba(150,90,210,.14)`
- N/A / muted state: uses `--card-2` bg + `--dim` text

**Rule for the implementer:** never hardcode a new hex value for anything that maps to one of the tokens above — always reference the variable/token. If a color is needed that isn't listed, derive it in the same hue family (warm near-black neutrals + terracotta/amber/olive-green accents) rather than introducing a colder gray or a saturated primary blue/red.

### Radii & Shadows
- `--r: 6px` (small controls), `--r-lg: 10px` (cards), `--r-sm: 4px` (inputs/badges)
- `--shadow: 0 1px 3px rgba(0,0,0,.4)` — active tab
- `--shadow-md: 0 4px 20px rgba(0,0,0,.55)` — floating panels

### Spacing
No formal scale — spacing is authored ad hoc in px (commonly 4/6/8/10/12/14/16/18/20/24/28px). Card padding is `20px` (`14px 16px` for `.card-sm`). Page container: `max-width: 1380px`, `padding: 0 28px`.

## Screens / Views

### 1. Home Dashboard (`page-dashboard`)
- Header: icon badge (46×46, rounded 12px, accent-dim bg) + "Compliance Dashboard" title + subtitle + "Lifecycle Docs" / "+ New Initiative" buttons top-right.
- 4-up stat card grid (`.grid-4`): Active Requirements, Compliance Rate (green), PRs Blocked (red), My Initiatives (amber). Each card has a 2px top border in its accent color, a small uppercase label, and a 38px bold value.
- Two tabs: "Active Requirements" (card grid of `init-card`s — title, status badge, category tag, big % compliant top-right, 3-segment compliance bar, compliant/in-progress/non-compliant counts, PRs Blocked/Warn Date/Block Date footer) and "My Initiatives" (table).

### 2. All Initiatives (`page-all-initiatives`)
- Analytics 4-up grid (same visual pattern as home stats).
- Portfolio Compliance Health: single wide 4-segment bar (green/amber/red/dim) + legend.
- Status filter pill row (All/Draft/Submitted/Review/Development/Test/Ready/Paused/Deprecated/Rejected) with counts.
- Full initiative table: Initiative, Lifecycle (mini stepper of pill badges joined by →), Projects (count), Compliance (% + mini bar), Key Dates (warn amber / block red, stacked), Status badge, Owner (avatar+name), View button.

### 3. Initiative Detail (`page-detail`)
- Breadcrumb "← All Initiatives / {title}", H1, status badge + category tag + owner.
- Lifecycle Progress card: horizontal stepper of circular dots (done=green check, current=accent ring+glow, todo=gray) connected by lines, with branch-offs for Rejected (off "Review") and Paused/Deprecated (off "Ready").
- Stats bar (`.stats-bar`): equal-width cells separated by hairline dividers — Compliant/Non-Compliant/In Progress/N/A counts + PRs Blocked + a wide "Compliance" cell with its own 3-segment bar.
- Contextual banners depending on status: submit banner (accent-tinted), awaiting-review banner (blue-tinted), paused banner (amber-tinted), deprecated banner (neutral), locked banner, CI-team action panel (violet-tinted, only visible in "CI Team" role).
- 4-up info cards: Current Status, Scope, Warn Date, Block Date.
- 2-col: Description + links, Technical Requirements (monospace `<pre>`).
- Affected Projects table (only once Ready/Paused): search + status filter pills, table with #, Project (**click-through to Project Detail**, shows a "MANUAL" tag if overridden), BizUnit, Status badge, PRs Blocked, Last Checked, Override button (opens inline override row: new-status select + evidence text input + Save/Cancel).
- 2-col activity timelines: Project Activity (auto vs manual, colored dot + connecting line) and Initiative Activity (status-colored dot).

### 4. Create / Edit Initiative (`page-create`, `page-edit`)
- Lifecycle banner showing the 6-step flow with current step highlighted.
- Left column: form sections in bordered/rounded groups (`.fsec`) — Basic Information, CI Enforcement Timeline (with info notice), Project Scope (checkbox rows expanding into pill-based sub-selectors for LOB/Platform/Repo, each pill showing a project count).
- Right sidebar (sticky): "What happens after saving?" explainer card.
- Locked/frozen states show an amber "FROZEN" tag next to the section header and disable inputs (opacity 0.45, `pointer-events:none`).

### 5. CI Review (`page-ci-review`)
- Violet-tinted icon header. Role-gated notice (only actionable when "CI Team" role is active).
- 5-up stat cards (Submitted/In Review/Development/In Test/Ready), each a plain number in its status color.
- Action Queue table: Initiative, Author, Submitted date, Status badge, CI Actions (status-advance select + Update button, violet `#8a5cc8`).
- Recent CI Activity timeline (same dot+line pattern as initiative detail).

### 6. Lifecycle Docs (`page-docs`)
- 6-slide horizontal carousel (`.slides-viewport` / `.slides-track`, CSS transform, dot pagination + prev/next). Explains the initiative lifecycle, roles, and project-level compliance concepts using cards, colored icon chips, and a mini lifecycle diagram.

### 7. Project Detail (`page-project`) — newest screen
- Breadcrumb "← All Initiatives / {bizUnit}/{project}".
- Header: `{bizUnit} / {projectName}` as H1, monospace repo SSH URL below with a git-branch icon, "☆ Favorite" button top-right.
- Stats bar (reuses `.stats-bar`): Builds (7d), Open PRs, Releases (3mo), Last Build (relative time).
- Tab row (`.tabs`/`.tab`, reused from generic tab component): **Overview / Compliance / History**.
  - **Overview**: 2-col Git Repository + Charts link cards (icon + one-line description, click-through placeholders), Documentation card with a Branch → Docs-link table, Static Website card with an info notice when none is configured.
  - **Compliance**: table of every initiative that scopes this project — Initiative, Category tag, Status badge (+ "MANUAL" tag if overridden), PRs Blocked, Last Checked, View button back to that Initiative Detail page.
  - **History**: build/PR history table — small circular status icon (green check / red X / blue spinner), pipeline context (e.g. `release/2026.06: #21`), build type (Pull Request / Branch Build / Custom Workflow), relative date.
- Entry point: clicking any project name in an Initiative Detail's "Affected Projects" table navigates here.

## Interactions & Behavior
- Client-side "nav" pattern: only one `.page` has `.active` (display:block) at a time; everything else is `display:none`. In a real app this maps to routes.
- Tabs (`.tab.active`) and pills (`.pill.active` / `.proj-pill.active`) are local UI state, not navigation.
- Role toggle (User / CI Team) in the nav bar changes which actions/panels render on Detail and CI Review — model this as a permissions/role check, not a real auth system.
- Theme (dark/light) and accent (indigo/teal/violet) are persisted to `localStorage` (`ci-theme`, `ci-accent`) and applied via `data-theme`/`data-accent` attributes — recreate as a user preference, not hardcoded.
- Manual override flow: inline table row expands (`.override-row.open`) with a status select + required evidence field; submitting updates the row status, tags it "MANUAL", and logs a Project Activity entry.
- Status transitions (Submit, Retract, Pause, Resume, Deprecate, CI Advance) each append an entry to that initiative's activity log with a timestamp, actor, and description — implement as an audit trail, not just a status field.
- Hover states: cards lift 1px + border turns accent on `init-card`; table rows get a subtle `--card-2` background; buttons brighten (`filter: brightness(1.1)`) or shift to `--card-2`.

## State Management
Needed state/entities (see `INITIATIVES` array in the HTML for exact shape):
- **Initiative**: id, title, shortTitle, category, desc, techReqs, docsUrl, jiraUrl, status (lifecycle enum), warnDate, blockDate, scopeLabel, projects (count), owner/ownerInitials, `projectData[]` (per-project compliance rows), `projectStats` (aggregate counts), `projectActivities[]`, `activities[]` (initiative-level audit log).
- **Project** (new — currently keyed by `"name | bizUnit"` string, e.g. `"ios-trends | dlci"`): repoUrl, builds7d, openPRs, releases3mo, lastBuild, branches[] (name + whether docs exist), history[] (status/context/type/date). Compliance-per-initiative is derived by scanning all initiatives' `projectData` for a matching project name — in a real app this should instead be a proper foreign key / join, not a string match.
- Global UI state: currentRole (user/ci), selectedId/selectedProject, active filters per table, current tab, current docs slide, theme/accent.

## Assets
No external images — all icons are inline `stroke="currentColor"` SVGs (lucide-style, 12–26px, stroke-width 2). No custom illustrations. Google Fonts: Instrument Serif, IBM Plex Sans, IBM Plex Mono.

## Files
- `CI Portal.html` — full app markup + all page templates + JS (data, render functions, interactions)
- `ci-styles.css` — all design tokens and shared component styles (buttons, cards, badges, tables, tabs, forms, etc.)
