# Meta Ads Autopilot

A Claude setup for running Meta ads across many client ad accounts. You approve, Claude builds, verifies and keeps the books.

Built and used by [RedZen Cloud Solutions](https://redzen.cloud) to manage 30+ client ad accounts.

## What it does
- **Builds campaigns** from a per-client doc (audience with IDs, rules), using one fixed naming and structure. Nothing runs without your "Go".
- **Verifies every launch**: it reads back the live campaign, ad set and ads, makes sure nothing is stuck in drafts, then runs a delivery check 2–3h later.
- **Daily check (scheduled, read-only)**: flags anything not delivering, ending, overspending, stuck in review or left in drafts.
- **Reconciles spend** into a Google Sheet master (one row per campaign, month blocks, totals, stats).
- **Logs everything**: one backlog file per day, and the playbook learns every new rule and blocker.

## What's in here
| Path | What |
|---|---|
| `CLAUDE.md` | Operating instructions: paste into your Claude Project instructions or use as-is in Claude Code |
| `playbook.md` | Build rules, naming, default build, launch verification, known API blockers and fixes |
| `workflow.md` | Daily schedules, alerts, backlog format |
| `reconciliation.md` | Meta → Google Sheet reconciliation procedure |
| `ads-sheet.md` | Sheet layout, style system, Sheets API write rules |
| `skills/meta-ads-reconciliation/` | Installable Claude skill for the reconciliation |
| `templates/` | Blank client doc, client index, backlog day |
| `SETUP.md` | How to set it up from scratch |
| `CHANGELOG.md` | What changed, so you can pull updates |

## Get started
See [SETUP.md](SETUP.md).

## Staying up to date
Watch this repo or check `CHANGELOG.md`. New rules and API blockers get added as they're found in live use. Pull them into your own copy; your client docs stay untouched because they live only in your project.

## License
MIT
