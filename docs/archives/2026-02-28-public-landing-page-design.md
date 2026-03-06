# Public Landing Page & Scorekeeper Removal

**Date:** 2026-02-28
**Status:** Approved

## Goal

Create a stable, clean public URL (`jdenish.github.io/moneyballs`) for Jordan's bio that lets anyone view tournament history and click through full results. Remove scorekeeper mode (Jordan is the sole score-entry person).

## Design

### New File: `index.html` (Public Landing Page)

**URL:** `jdenish.github.io/moneyballs`

**Home View (default):**
- Title: "Jordan's Moneyballs"
- Subtitle: "Bounce Philly"
- Tournament card grid (dark theme, same visual style as organizer home page)
  - Date, status badge (completed/upcoming/in-progress), medal winners for completed
  - Team count for upcoming tournaments
- "Organizer Login" link in footer/corner -> `Jordan_Moneyball.html`

**Tournament Detail View (click a card):**
- Full read-only tournament view with tabs: Round Robin, Standings, Bracket, Results
- Bracket tab is the DEFAULT when clicking into a completed tournament
- Same rendering components as organizer app, stripped of all edit controls:
  - No score inputs (display scores as text)
  - No court management buttons
  - No override panels
  - No save/load/share/publish toolbar
  - No paid status display
  - No drag-and-drop
  - No test scores button
- "Back to All Tournaments" button at top
- URL hash routing: `#t=2026-02-06` for shareable links to specific tournaments

**Data Source:**
- Same `TOURNAMENT_STATES` and `TOURNAMENTS` arrays embedded in HTML
- When organizer app is updated and pushed, public page gets same data

**Tech Stack:**
- Same as organizer: React 18, Tailwind CSS, Babel standalone, Pako (all via CDN)
- Single HTML file, no build step

### Changes to `Jordan_Moneyball.html` (Scorekeeper Removal)

Remove:
- Scorekeeper mode (`?live=...&score=1` URL parameter handling)
- Scorer link generation from toolbar
- Score lock/unlock button and `sl` key from publish data
- "Scorekeepers" column from "How It Works" section on home page
- Scorekeeper banner ("Scorekeeper Mode") and submit button
- `scorekeeperMode` state variable and all conditional logic referencing it

Keep unchanged:
- Password-protected organizer workflow
- Score entry (organizer only)
- Save/Load/Quick Save
- Share Link (snapshot URLs)
- npoint.io live viewer mode (`?live=ID` without `&score=1`)
- Publish button (pushes to npoint for viewers)
- All tournament logic (RR, pools, brackets, etc.)

### Changes to `README.md`

Update with both URLs:
- Public site: `https://jdenish.github.io/moneyballs/`
- Organizer: `https://jdenish.github.io/moneyballs/Jordan_Moneyball.html`

## Quality Process

- **TDD:** Write tests before implementation for key behaviors
- **3 Code Review passes:**
  1. After `index.html` public page complete
  2. After scorekeeper removal from `Jordan_Moneyball.html`
  3. Final full review of all changes

## What Does NOT Change

- Deployment process (GitHub API push)
- Organizer workflow
- Tournament data format
- npoint.io integration (viewer mode only)
- Embedded tournament data structure
- File naming (`Jordan_Moneyball.html` stays as-is)
