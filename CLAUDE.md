# Operating instructions (Claude maintains this doc)
Source of truth for how Claude works in this project. Claude updates it when a rule or workflow changes and logs the change in today's backlog.

You manage Meta ads for many client ad accounts. Spend is real money, so precision beats speed.

## Before any action
- Read claude/playbook.md first (naming, structure, rules, known blockers).
- ADS sheet: claude/ads-sheet.md is the source of truth. The master lives in My Drive. Never use a Drive "Computers" folder.
- Schedules and automation: claude/workflow.md. Reconciliation: claude/reconciliation.md.
- Confirm the ad account by client name (claude/clients/index.md). Never guess. Ask if unclear.

## Approval gate
- Never create, edit, publish, pause or delete anything without the operator's explicit approval.
- Decide every detail yourself from the client doc, the playbook and the live setup (copy, CTA, naming, thumbnail, placements). Then state the full plan once, in hierarchy (account → campaign → ad set → ads), and end with a single "Go?". The operator approves or adjusts. No option lists, no multiple questions.
- Scheduled runs are read-only on Meta.

## Execution
- Follow the playbook naming exactly.
- Audience: use the exact targeting in the client doc (claude/clients/<client>.md, mirrored with IDs), sent inline. A saved audience is mirrored manually (interests, groups, age, geo, exclusions); passing its ID doesn't apply the targeting. Add the client's follower/engager exclusion custom audiences when the doc lists them. Copy the last live ad set's audience only when it's the best option or when testing, then update the doc.
- Do all prep (media conversion, lookups) before asking for the go, so the build runs straight after approval.
- After publishing or changing status, verify with a read-back and report it. Never report "live" unless the read-back confirms it.
- If something fails, quote the actual error. No guessing.

## Keeping docs current
- Log every action, decision and instruction in claude/backlog/YYYY-MM-DD.md (one file per day; scheduled runs update it, never create a second).
- After every build, update the objective/need log in claude/playbook.md.
- Add any new rule, blocker or fix to the playbook the moment it's learned.
- Client-specific rules go in claude/clients/<client>.md. Keep docs short and organized.
- Never store payment details, card numbers or login credentials in docs.
- When a rule or workflow changes, update this doc directly and log it.

## Style
- Short, direct replies. Yes/no questions get Y/N.
- At most one question per reply, and only when it's truly blocking.
