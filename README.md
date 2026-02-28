# Jordan's Moneyballs

A tournament management app for 8-16 team doubles pickleball tournaments with configurable round robin, pools, and single or double elimination brackets.

## Links

- **Public Site:** [jdenish.github.io/moneyballs](https://jdenish.github.io/moneyballs/) — View tournament history and results
- **Organizer:** [jdenish.github.io/moneyballs/Jordan_Moneyball.html](https://jdenish.github.io/moneyballs/Jordan_Moneyball.html) — Password-protected tournament management

## Features

- **8-16 team support** with automatic payout adjustments
- **Configurable RR games (0-7)** - skip RR (0), seed-balanced (2), pools (3 for 12/16), or random with repeat-avoidance
- **Configurable pool system** - pool sizes 3-6, snake draft assignment, intra-pool round robin
- **Single or double elimination** bracket with dynamic generation
- **Configurable finals** - Best of 3 or single game
- **Smart court assignment** - pool-based for RR, lowest-available for bracket, manual override
- **3rd place match** in single elimination
- **No-repeat constraint** - pool teammates won't face each other in bracket R1
- **Drag & drop seeding** - reorder teams on Setup page
- **Live spectator link** via npoint.io - one stable URL, publish to update, spectators reload
- **Live score tracking** with numbered court display
- **Share link** for spectators (compressed URL, read-only view)
- **Save/load** tournament progress (backwards compatible)
- **Results tab** with gold/silver/bronze highlighting
- **Public tournament history** - browse past and upcoming tournaments at the public URL

## Tournament Format

| Teams | Default Winners | Payouts (1st / 2nd / 3rd) |
|-------|-----------------|---------------------------|
| 8 | 8 (all) | $200 / $100 / $40 |
| 9 | 8 | $240 / $100 / $40 |
| 10 | 8 | $260 / $120 / $40 |
| 11 | 8 | $300 / $140 / $40 |
| 12 | 12 (all) | $360 / $160 / $60 |
| 13 | 8 | $400 / $180 / $60 |
| 14 | 8 | $440 / $200 / $60 |
| 15 | 8 | $480 / $200 / $80 |
| 16 | 8 | $500 / $220 / $100 |

- **Entry:** $45/player (covers entry + court fees)
- **Seeding games:** To 11, win by 2
- **Bracket games:** To 15, win by 2
- **Finals:** Best of 3 or single game (configurable)

## Quick Start

1. Open the [organizer site](https://jdenish.github.io/moneyballs/Jordan_Moneyball.html)
2. Enter the organizer password
3. Select team count (8-16, auto-detects when you paste)
4. Paste team list (format: `Player 1 / Player 2`)
5. Click "Import Teams"
6. Configure: RR games (0-7 or Pools), bracket type (single/double), finals format, court count
7. (Optional) Paste an [npoint.io](https://www.npoint.io) ID for live spectator link
8. Click "Randomize Matchups" and start entering scores
9. Click **Publish** to push state to live link, or use **Share Link** for one-off snapshots

## Documentation

See [DESIGN_DOC.md](DESIGN_DOC.md) for full tournament rules, schedule, and technical details.
