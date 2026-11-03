# Setup

## 1. Requirements
- Claude (claude.ai with Projects, or Claude Code)
- Connectors: **Meta Ads**, **Google Sheets**, **Google Drive**
- Ad accounts you manage in Meta Business Manager, with each client's Instagram account **connected to its ad account** (the API only sees connected IG accounts)

## 2. Create the project
1. Create a Claude Project (e.g. "Meta Ads").
2. Add these files as project docs under `claude/`: `instructions.md` (= `CLAUDE.md`), `playbook.md`, `workflow.md`, `reconciliation.md`, `ads-sheet.md`.
3. Project instructions: `Read claude/instructions.md first and follow it.`
4. Install the skill in `skills/meta-ads-reconciliation/` (upload it as a skill in Claude settings).

Claude Code: clone the repo. `CLAUDE.md` is picked up automatically.

## 3. Add your clients
1. Copy `templates/clients-index.md` → `claude/clients/index.md`. Fill one row per ad account (client, account name, ID, currency, doc, sheet tab).
2. For each client, copy `templates/client.md` → `claude/clients/<client>.md`. Mirror the audience from their last live ad set **with IDs** (interest groups, geo, ages, exclusions). The API can't search interests or apply saved audiences, so this doc *is* the audience.

## 4. Set up the sheet
1. Create a Google Sheet master in **My Drive** (not a Drive "Computers" folder, because desktop sync deletes cloud-added files there).
2. Lay it out per `ads-sheet.md` (one tab per client, ADS STATS, ADS Clients, TEMPLATE).
3. Create a backups folder. Put both IDs in `ads-sheet.md`, `reconciliation.md` and the skill (replace `<SHEET_ID>` / `<BACKUP_FOLDER_ID>`).

## 5. Schedule the daily check
Ask Claude: "Create the scheduled tasks in workflow.md." It creates the daily check (read-only on Meta) with push notifications.

## 6. First build
Tell Claude the client and the need ("Client X, IG DMs, $150, 10 days"). It reads the client doc, states the full plan once, and ends with "Go?". You say Go, and it builds, verifies live and schedules the delivery check.

## Updating
Pull new versions of `playbook.md`, `workflow.md`, `reconciliation.md`, `CLAUDE.md` and the skill. Never overwrite your `clients/` or `backlog/`.
