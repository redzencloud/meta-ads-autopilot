# Workflow & schedules
How the Meta Ads operation runs day to day. Short and current. Owner: the operator. Executor: Claude.

## Docs map
| Doc | Purpose |
|---|---|
| `claude/instructions.md` | Operating rules (Claude-maintained) |
| `claude/playbook.md` | Rules, naming, default build, launch verification, blockers, need log |
| `claude/ads-sheet.md` | ADS SHEET location, layout, write rules |
| `claude/reconciliation.md` | Meta → sheet reconciliation: exact tab names, row-number rule, steps (same as skill `meta-ads-reconciliation`) |
| `claude/clients/<client>.md` + `index.md` | Per-client audience, rules; index maps client ↔ ad account ↔ sheet tab |
| `claude/backlog/YYYY-MM-DD.md` | Daily log: every action, decision, instruction. One per day |

## Assets
- ADS master: Google Sheet `<SHEET_ID>` in My Drive.
- Backups: folder `<BACKUP_FOLDER_ID>`. Keep last 14 dailies.
- Never use a Drive "Computers" folder (sync bins cloud files).

## Schedules (local time)
| Task | When | Does | Push |
|---|---|---|---|
| ADS daily check | ~08:40 daily | Missed-run check → backup → reconcile sheet → alerts → drafts → ended-but-active list → backlog | Always |
| Per-client check (optional) | daily | Client-specific check-in → client doc + backlog line | Always |
| Weekly summary (optional) | Mon morning | Weekly summary per client → client doc + backlog line | Push + email |

Record each scheduled task's ID here once created.
All scheduled runs are **read-only on Meta** (drafts included).

## Alerts (pushed by ADS daily check, in this order)
- MISSED RUN: yesterday's run or backup missing (then ENDED window widens to 49h).
- NOT DELIVERING: ACTIVE ad set > 6h past start with 0 impressions (+ delivery status, ads_get_errors).
- ENDED in the last 25h (+ whether a follow-up campaign is live).
- ENDING within 24h.
- DRAFTS: unpublished drafts in any active account (TEST builds listed as info).
- IN REVIEW > 12h after start.
- Overspend > lifetime budget × 1.02.
- Zero delivery in 24h for ad sets that delivered before.
- Cost per result (24h) > 2× lifetime, with ≥ $10 spent.
- 7-day frequency > 3.
- Rejected ads on live campaigns (copyright WITH_ISSUES = note).
- Account not active (grace period, unsettled) / spend stopped.
- NEW ACCOUNT: an ad account not in clients/index.md.
- Ended-but-active list → operator approves pausing in chat.

## After every build (manual, from playbook)
Launch verification read-back + a send_later delivery check ~2–3h after start. The daily check is the safety net, not the first check.

## Approval-gated actions (operator in chat)
- Turn-off list: operator replies Y → Claude pauses exactly those, reads back status, logs it.
- Any build/edit: approval gate in instructions.md + playbook.

## Backlog format
```
# Backlog — YYYY-MM-DD (Day)
## Instructions from <Operator>
## Actions (local time)   ← manual + skills
## Scheduled runs          ← "### RUN ADS daily check", per-client check lines
## Decisions
## Open
```
Runs re-read before writing and merge; never remove other lines.

## Manual asks
Any manual action in chat (build, edit, reconciliation, doc change) gets a line in today's backlog with time, what, IDs, result or quoted error.
