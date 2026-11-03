# ADS SHEET (v4 layout)

**Master:** Google Sheet "ADS SHEET — Master" (`<SHEET_ID>`) in My Drive (folder `<MASTER_FOLDER_ID>`).
**Backups:** subfolder ADS Backups (`<BACKUP_FOLDER_ID>`). Daily Google copy titled `ADS SHEET backup YYYY-MM-DD`, made by the morning run.
**NEVER** put the master or backups inside a Drive "Computers" folder. The desktop sync bins cloud-added files.
**Automation:** see `claude/workflow.md`. Reconciliation rules: skill `meta-ads-reconciliation`.

## Tab order (most recent active → oldest dormant)
ADS Clients, ADS STATS, then client tabs by latest campaign end date (newest first, ongoing on top), active-but-empty new clients after the active ones, dormant after, then legacy tabs, TEMPLATE (80), Data Sources last. Re-sort after every reconciliation.

## Clean-sheet rules
- No empty rows: every numbered row has data; no empty month blocks; no blank grid past the REMAIN row.
- Rows numbered 1..n within each month. New rows go before the bottom TOTAL; then formulas are rewritten explicitly (see skill).
- Empty new client tabs keep exactly 1 "MONTH YEAR" divider + 1 row until first reconciliation.
- ADS STATS MIN/MAX date formulas return blank (not 30/12/1899) when a month has no dates.

## Pinned totals
- Client tabs: row 3 = TOTAL (mirrors bottom TOTAL), row 4 = headers, data from row 5. Frozen rows 1–4, cols A–B.
- ADS STATS: row 2 = TOTAL (ALL CURRENCIES), frozen rows 1–3. Bottom grand TOTAL row and SPENT USD/AED rows move when rows are inserted; find them by reading col B.
- ADS Clients: rows 2–3 = TOTAL USD / TOTAL AED (or your currencies), row 4 headers; bottom totals mirror them.

## Style system
- Font Inter 10, gridlines off. Header row #1F2937 white bold caps. Month rows #F3F4F6 bold. Totals #111827 white bold. Rows: thin #E5E7EB bottom border.
- Placement pill (col B, CF): META #DBEAFE/#1E40AF · INSTAGRAM #FCE7F3/#9D174D · TIKTOK #E5E7EB/#111827 · GOOGLE #FEF3C7/#92400E · YELP #FEE2E2/#991B1B.
- Status pill (col P): DONE #DCFCE7/#166534 · IN PROGRESS #FFEDD5/#9A3412 · PAUSED #FEE2E2/#991B1B. 12 CF rules per tab.
- Remain > 0: red on #FEF2F2. Money $#,##0.00 (AED tabs "AED "#,##0.00). Dates dd/mm/yyyy.

## Client tab layout
- A # · B PLACEMENT · C AD · D TYPE · E reach · F impressions · G results · H CPM · I CPA · J start · K end · L budget · M spent · N remain · O ACCOUNT (when a tab merges two accounts) · P STATUS · Q CAMPAIGN ID (TEXT format) · R MONTH key 'YYYY-MM · S NOTES · T LEGACY TYPE (hidden).
- Month divider: label in A (merged A:B), J/L/M = SUMIFS on R key with A>0.
- Bottom TOTAL = SUM / AVERAGEIF / SUMIF(A>0); REMAIN row under it.
- Rule B: budget = spent once DONE; not ended = ad-set lifetime budget; daily no end = spent + note.

## Sheets API write rules
- USER_ENTERED parses "2026-10" / "2026 OCTOBER" as dates → prefix `'`.
- Merges across the frozen A:B|C boundary are rejected → merge A:B only, PASTE_FORMAT J:S.
- Field masks: `userEnteredFormat.backgroundColor` (dotted, no parentheses).
- Single-row ranges collapse (E6:E6 → E6) → rewrite TOTAL formulas explicitly after inserts.
- Trashed files return "Permission denied" from the Sheets/Drive connectors.

## ADS STATS
Grouped by client (header: currency in A, name, J/L/M = SUM of its month rows), one row per month via whole-column SUMIFS / MINIFS / MAXIFS on the tab's R key. Never type numbers here.

## ADS Clients
E/F/J/K reference each tab's bottom totals; START/END via whole-column MINIFS/MAXIFS; END CURRENT green / TERMINATED red; TOTAL per currency pinned top and bottom.

## Adding a client
Duplicate TEMPLATE (80), set E2, add ADS STATS block, add ADS Clients row, place tab by the order rule.

## Last reconciliation
Record date, grand total spent per currency, and the check that ADS STATS = ADS Clients.
