# Jordan's Moneyballs

A tournament management app for 12-16 team doubles pickleball tournaments.

## Live Site

**https://jdenish.github.io/moneyballs/Jordan_Moneyball.html**

## Features

- **12-16 team support** with automatic payout adjustments
- **Pool system** for 12/16 teams (snake draft seeding, 3 RR games within pool)
- **Random matchups** for 13/14/15 teams (3 RR games)
- **Configurable bracket split** - choose how many teams go to Winners vs Losers
- **No-repeat constraint** - teams from same pool won't face each other in bracket R1
- **Drag & drop seeding** - reorder teams on Setup page
- **Double elimination bracket** with Bo3 finals
- **Live score tracking** with "on court" status
- **Share link** for spectators (compressed URL, read-only view)
- **Save/load** tournament progress
- **Results tab** with gold/silver/bronze highlighting

## Tournament Format

| Teams | RR Format | Default Winners | Payouts |
|-------|-----------|-----------------|---------|
| 12 | 3 pools of 4 | 12 (all) | $420 / $120 / $60 |
| 13 | 3 random games | 8 | $440 / $160 / $60 |
| 14 | 3 random games | 8 | $480 / $160 / $80 |
| 15 | 3 random games | 8 | $500 / $200 / $80 |
| 16 | 4 pools of 4 | 8 | $520 / $220 / $100 |

- **Entry:** $30/player ($60/team)
- **Seeding games:** To 11, win by 2
- **Bracket games:** To 15, win by 2
- **Finals:** Best of 3 (games to 11)

## Quick Start

1. Open the [live site](https://jdenish.github.io/moneyballs/Jordan_Moneyball.html)
2. Paste team list (format: `Player 1 / Player 2`)
3. Click "Import Teams"
4. Configure winners bracket size if needed
5. Click "Randomize Matchups"
6. Enter scores as games finish
7. Share link with spectators after each update

## Documentation

See [DESIGN_DOC.md](DESIGN_DOC.md) for full tournament rules, schedule, and technical details.
