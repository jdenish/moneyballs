# Jordan's Moneyballs - Tournament Design Document

## Overview

A 16-team doubles pickleball tournament with a seeding round robin followed by double elimination bracket.

---

## Event Details

| Detail | Info |
|--------|------|
| **Venue** | Bounce Philly (indoor) |
| **Skill Level** | 4.5+ DUPR |
| **Registration** | Sign up on CourtReserve |
| **Payment** | Court fees via CourtReserve, $30 entry via Venmo/cash day-of |
| **Warmup** | 6:00 PM |
| **Tournament Start** | 6:30 PM |

### Game Rules
- Win by 2 (no cap)
- Seeding games to 11
- Bracket games to 15

---

## Tournament Format: 8/8 Split Bracket

### Phase 1: Seeding Games (2 games per team)
- **Format:** Games to 11
- **Duration:** ~20 minutes per game
- **Purpose:** Determine bracket seeding based on record + point differential

After seeding:
- **Top 8 teams** → Winners Bracket
- **Bottom 8 teams** → Losers Bracket

### Phase 2: Double Elimination Bracket
- **Format:** Games to 15
- **Duration:** ~30 minutes per game
- **Winners Final & Grand Final:** Best of 3 (~50 minutes)

---

## Pod System (Balanced Scheduling)

Teams are seeded 1-16 and divided into 4 pots:

| Pod A (1-4) | Pod B (5-8) | Pod C (9-12) | Pod D (13-16) |
|-------------|-------------|--------------|---------------|
| Seed 1      | Seed 5      | Seed 9       | Seed 13       |
| Seed 2      | Seed 6      | Seed 10      | Seed 14       |
| Seed 3      | Seed 7      | Seed 11      | Seed 15       |
| Seed 4      | Seed 8      | Seed 12      | Seed 16       |

### Matchup Rules (Balanced Strength of Schedule)

| Game   | Matchups      | Result                           |
|--------|---------------|----------------------------------|
| Game 1 | A vs B, C vs D | Each pot plays adjacent pot     |
| Game 2 | A vs C, B vs D | Cross-matchup for balance       |

**Why this works:**
- A & D: Play 2 mid-tier opponents (B+C or C+B)
- B & C: Play 1 hard + 1 easy (A+D or D+A)
- No pot plays itself
- Strength of schedule is roughly equal for all teams

### Randomization
Within these rules, matchups are randomized (shuffle within pots) for variety and drama.

---

## Standings Tiebreakers

1. **Wins** (most wins first)
2. **Head-to-Head** (if teams played each other)
3. **Point Differential** (PF - PA)

---

## Schedule (8 courts, starting 6:30 PM)

| Time     | Round              | Duration | Matches |
|----------|--------------------|----------|---------|
| 6:30 PM  | Seeding Game 1     | 20 min   | 8       |
| 6:50 PM  | Seeding Game 2     | 20 min   | 8       |
| 7:10 PM  | Break (standings)  | 5 min    | —       |
| 7:15 PM  | W-QF + L-R1        | 30 min   | 8       |
| 7:45 PM  | W-SF + L-R2        | 30 min   | 6       |
| 8:15 PM  | W-Final (Bo3) + L-R3 | 50 min | 3       |
| 8:45 PM  | L-R4               | 30 min   | 2       |
| 9:15 PM  | L-Semi             | 30 min   | 1       |
| 9:45 PM  | L-Final            | 30 min   | 1       |
| 10:15 PM | Grand Final (Bo3)  | 50 min   | 1       |
| **11:05 PM** | **Done**       |          |         |

**Total Duration:** ~4 hours 35 minutes

---

## Payout Structure

| Entry Fee | Per Player | Per Team |
|-----------|------------|----------|
| Entry     | $30        | $60      |

| Place | Payout | Per Player |
|-------|--------|------------|
| 1st   | $520   | $260       |
| 2nd   | $220   | $110       |
| 3rd   | $100   | $50        |
| Organizer | $120 | —        |
| **Total** | **$960** | |

**Math:** 16 teams × 2 players × $30 = $960

---

## Double Elimination Bracket Structure

### Winners Bracket
```
W-QF:
  W1 vs W8 → WSF1
  W4 vs W5 → WSF1
  W2 vs W7 → WSF2
  W3 vs W6 → WSF2

W-SF:
  WSF1 winner vs WSF2 winner → W-Final (Bo3)

W-Final:
  Winner → Grand Final
  Loser → L-Final
```

### Losers Bracket
```
L-R1 (bottom 8 from seeding):
  L1 vs L8, L4 vs L5, L2 vs L7, L3 vs L6

L-R2 (L-R1 winners vs W-QF losers):
  4 matches

L-R3:
  2 matches (L-R2 winners face each other)

L-R4 (L-R3 winners vs W-SF losers):
  2 matches

L-Semi:
  1 match

L-Final (L-Semi winner vs W-Final loser):
  1 match → Winner to Grand Final
```

### Grand Final (Bo3)
- Winners Bracket champion vs Losers Bracket champion
- No bracket reset (single Bo3)

---

## App Features

### Live Site
**https://jdenish.github.io/moneyballs/Jordan_Moneyball.html**

### Deployment
Files are deployed to GitHub Pages via the GitHub API. After making changes locally, Claude can push updates to the repo without touching work git config.

**GitHub Repo:** https://github.com/jdenish/moneyballs

### Files
- `Jordan_Moneyball.html` - CDN version (needs internet on first load)
- `Jordan_Moneyball_offline.html` - Fully offline, no internet needed

### Tabs/Phases
1. **Import** - Paste team list in "Player 1 / Player 2" format
2. **Setup** - Edit teams, add DUPR ratings, randomize matchups
3. **Round Robin** - Enter scores for seeding games
4. **Standings** - Live rankings with W-L, PF, PA, PD
5. **Bracket** - Double elimination with score entry

### Key Features
- **DUPR Tracking** - Optional player ratings (placeholder "4.5"), shows combined team DUPR
- **On Court Tracking** - Mark matches as "on court" (orange highlight)
- **Next Up Queue** - Shows ready matches not yet on court
- **Manual Override** - Force winner without entering scores (checkbox reveals "W" buttons)
- **Save/Load Progress** - Export/import JSON file
- **Download Results** - Generate .txt file with final standings
- **🎲 Test Scores** - Generate fake scores based on seeding for testing (higher seeds favored ~3% per seed difference)
- **Drag & Drop Seeding** - Reorder teams on Setup page by dragging (☰ handle)
- **🔗 Share Link** - Copy URL to clipboard for spectators to view live bracket
- **Spectator Mode** - Read-only view when opening shared link (no editing controls)

### How Share Link Works
The **🔗 Share Link** button encodes the entire tournament state (teams, scores, bracket) into the URL as a base64 string after `?s=`.

**Tournament day workflow:**
1. Enter scores as games finish
2. Click **🔗 Share Link** → URL copied to clipboard
3. Paste new link in group chat / text message
4. Spectators open link to see current bracket (read-only)
5. Repeat after each round of updates

**Note:** Each click generates a NEW URL because the state changes. Spectators need the latest link to see latest scores (no auto-refresh).

### Color Coding
- **Gray border** - Waiting for teams
- **Yellow border** - Ready to play
- **Orange background** - On court
- **Green border** - Complete
- **Yellow ring** - Best of 3 match

---

## Alternative Formats Considered

### 4/12 (Pools → Double Elim)
- 4 pools of 4 teams (round robin)
- Pool winners → Winners Bracket
- Everyone else (12 teams) → Losers Bracket
- **Duration:** ~5 hours 15 minutes (longer than 8/8)
- **Rejected:** Takes longer, more complex

### True 8.5 Average Difficulty
- Seed 1-16, pair so every team's opponents average to 8.5
- **Rejected:** Creates duplicate matchup issues, no randomness, harder to explain

---

## Upcoming Tournament Team List

Copy/paste this into the Import tab:

```
Oscar Serra / Sanil Jagtiani
Jack Hared / Preston Gordon
Ted Holway / Alex Szczepkowski
Matvey Radionov / Matt Meadows
Lukas Choi / William Hayes
Jordan Denish / Alex Tong
Conor Landrigan / Geoff Watson
Cody Sadreameli / Dan Wach
Jeff Comer / Kenoa Tio
Matthew Matro / Zach Bowe
Bruno Casino Remondo / Drew Von Bargen
Austin Gow / Peter Weaver
Alex Boory / Ronald Marchese
Austin Keefer / Mason McCabe
Shashank Kamdar / Zachary Lessner
Matthew Chen / Johny Mario
```

---

## Sample Team List Format

Supported formats:
- `Player 1 / Player 2`
- `* Player 1 / Player 2`
- `1. Player 1 / Player 2`
- `- Player 1 / Player 2`

---

## Technical Implementation Notes

### File Versions
| File | Description | Internet Required |
|------|-------------|-------------------|
| `Jordan_Moneyball.html` | React + Tailwind CSS via CDN | Yes (first load, then cached) |
| `Jordan_Moneyball_offline.html` | Vanilla JS, all CSS inline | No |

### React Version Architecture
- Components (`SeedingMatch`, `BracketMatch`, `TeamInput`, `CourtStatus`) are defined **outside** of `App` to prevent recreation on each render
- Components wrapped with `React.memo()` to prevent unnecessary re-renders
- All handler functions use `useCallback` for stable references
- Data passed via props instead of closure capture
- This prevents input focus loss when typing scores

### Offline Version Architecture
- Uses vanilla JavaScript with manual DOM rendering
- `render()` function rebuilds entire DOM on state change
- Focus preservation implemented via:
  - `data-input-id` attributes on all inputs
  - Save active element ID + cursor position before render
  - Restore focus after render completes

### Known Issues Fixed
1. **TextEdit HTML corruption** - Mac TextEdit saves as "Cocoa HTML Writer" with escaped code. Files must be created programmatically or via plain text editor.
2. **Input focus loss** - React components defined inside `App` get recreated each render, losing focus. Fixed by moving components outside and using `memo()`.
3. **Test scores button not working** - `generateFakeSeedingScores` and `generateFakeBracketScores` were defined before their dependencies (`seedingMatches` and `bracketTeams`). React `useCallback` captures values at creation time, so they were undefined. Fixed by moving these functions after their dependencies are defined.

### UI Improvements
- **Bracket shows both partners** - Each team displays both player names stacked vertically with smaller text (`text-xs`) instead of truncated single line
- **Wider bracket boxes** - Increased from `w-36`/`w-40` to `w-44` (176px) to fit both names
- **Bo3 game scores** - Winners Final and Grand Final show individual game scores in separate boxes (e.g., `[11] [9] [11]` vs `[9] [11] [5]`) instead of dashed format
- **Champion display** - Shows both player names on separate lines with trophy emoji
- **Drag handles** - ☰ icon next to seed numbers indicates draggable teams
- **Share button feedback** - Shows "✓ Copied!" confirmation when link copied

### Mobile Friendliness
- Setup/Import/Standings pages work well on mobile (single column layout)
- Bracket page requires horizontal scrolling on phones (boxes are 176px wide)
- Touch inputs work for all controls
- Spectator mode is fully functional on mobile

---

## Master Tournament Site (Future)

### Overview
A public website to host all Jordan's Moneyballs tournaments with:
- Historical archive of all past tournaments
- Individual tournament pages with full brackets
- Player statistics and all-time leaderboard

### Proposed Architecture

```
jordans-moneyballs.github.io/
├── index.html              # Main tournament app (current)
├── history.html            # List of all past tournaments
├── tournament.html         # Individual tournament detail view
├── leaderboard.html        # All-time player stats
├── tournaments/
│   ├── 2025-02-07.json     # Tournament data files
│   ├── 2025-02-14.json
│   └── ...
└── DESIGN_DOC.md
```

### Tournament JSON Schema
```json
{
  "id": "2025-02-07",
  "name": "Jordan's Moneyballs #1",
  "date": "2025-02-07",
  "venue": "Bounce Philly",
  "teams": [...],
  "seedingScores": {...},
  "bracketScores": {...},
  "standings": {
    "1st": { "team": "Oscar Serra / Sanil Jagtiani", "payout": 520 },
    "2nd": { "team": "...", "payout": 220 },
    "3rd": { "team": "...", "payout": 100 }
  }
}
```

### Features

#### History Page (`history.html`)
- List of all tournaments with date, winner, runner-up
- Click any tournament to view full details
- Filter by date range, search by player name

#### Tournament Detail Page (`tournament.html#2025-02-07`)
- Full bracket visualization (same as current app, read-only)
- Final standings with payouts
- All teams and their seeding scores
- Shareable URL for each tournament

#### Leaderboard Page (`leaderboard.html`)
- All-time earnings leaderboard
- Tournament wins, 2nd places, 3rd places
- Win rate statistics
- Player search

Example:
```
| Rank | Player        | Wins | 2nd | 3rd | Earnings | Tournaments |
|------|---------------|------|-----|-----|----------|-------------|
| 1    | Oscar Serra   | 3    | 1   | 0   | $1,670   | 4           |
| 2    | Jack Hared    | 2    | 2   | 1   | $1,380   | 5           |
```

### Submit Tournament Flow
1. Complete Grand Final in main app
2. Click "Submit Tournament" button
3. Enter tournament name (defaults to "Jordan's Moneyballs #N")
4. Confirm submission
5. Downloads JSON file (e.g., `2025-02-07.json`)
6. Drag file into GitHub `/tournaments/` folder
7. Site auto-updates within minutes

### Why GitHub Pages
| Feature | Benefit |
|---------|---------|
| **Free hosting** | No costs ever |
| **Version history** | Every change tracked, can undo mistakes |
| **Easy editing** | Edit JSON files directly in GitHub web UI |
| **Custom domain** | Can use `moneyballs.jordandenish.com` |
| **Auto-deploy** | Push changes, site updates automatically |
| **Reliable** | GitHub has 99.9% uptime |

### Backwards Compatibility
Current tournament app already exports compatible JSON via **💾 Save** button. All tournaments run before the master site is built can be imported later.

**Action for each tournament:**
1. After Grand Final, click **💾 Save**
2. Rename file to `YYYY-MM-DD.json` (e.g., `2025-02-07.json`)
3. Keep file safe until master site is built

---

## Implementation Phases

### Phase 1: Current App ✅
- [x] Tournament manager app
- [x] Import teams, DUPR tracking
- [x] Pod system with balanced scheduling
- [x] Round robin scoring
- [x] Double elimination bracket
- [x] Save/Load progress (JSON)
- [x] Download results (TXT)
- [x] Test scores generator
- [x] Focus preservation fix
- [x] Drag & drop seeding reorder
- [x] Share link for spectators
- [x] Spectator mode (read-only view)
- [x] Bo3 individual game score display

### Phase 2: Master Site Foundation
- [x] Set up GitHub repository (https://github.com/jdenish/moneyballs)
- [x] Deploy current app to GitHub Pages (https://jdenish.github.io/moneyballs/)
- [ ] Create history page (list all tournaments)
- [ ] Create tournament detail page (read-only bracket view)
- [ ] Add "Submit Tournament" button to main app

### Phase 3: Player Statistics
- [ ] Build leaderboard page
- [ ] Calculate all-time earnings per player
- [ ] Track wins/2nd/3rd placements
- [ ] Player search functionality

### Phase 4: Polish & Enhancements
- [ ] Custom domain setup
- [ ] Print-friendly bracket view
- [ ] Score validation (must reach 11 or 15)
- [ ] Mobile-responsive improvements
- [ ] Social sharing (share tournament results to Twitter/Instagram)

---

## Future Enhancement Ideas

- Print-friendly bracket view
- Score validation (must reach 11 or 15)
- Timer/clock display
- Undo last score entry
- Court assignment display
- Real-time sync (Firebase) for instant spectator updates
- Email/SMS notifications for match assignments
- Integration with DUPR API for auto-fetching ratings
- Mobile-optimized bracket list view
