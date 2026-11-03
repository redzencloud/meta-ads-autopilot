# Meta Ads → ADS SHEET reconciliation (v4 sheet)

Source of truth for layout: `claude/ads-sheet.md`. Read it first, plus `claude/clients/index.md` (tab ↔ ad account map).
Master: Google Sheet `<SHEET_ID>` in My Drive. Never put it inside a Computers/backup-synced folder (the sync client bins cloud-added files).
Meta tools: always send `client_model` and the conversation id the Meta tool returns. Read-only here. Never pause/edit/delete anything in Meta from this skill.

## Exact tab names (use these strings; a wrong name fails with "Unable to parse range")
List every tab here exactly as named in the sheet: active client tabs, summary (`ADS Clients` · `ADS STATS`), dormant/legacy, `TEMPLATE (80)`, `Data Sources`. If a tab is renamed, update this list.

## Row-number rule
- Sheet row = index in the get_values `values` array + the first row of the requested range (e.g. range `A1:S40` → values[0] is row 1; range `A5:S40` → values[0] is row 5). Rows that come back empty still count.
- Before every write, re-read the target row and confirm col C (campaign name) and col Q (campaign id) match what you're about to write. If they don't, stop and re-read.
- Never write into the TOTAL or REMAIN rows (col B = "TOTAL" / "REMAIN").

## 0. Before writing
1. Backup if none today: Drive `copy_file` master → folder `<BACKUP_FOLDER_ID>` (ADS Backups), title `ADS SHEET backup YYYY-MM-DD`.
2. For every tab you will touch: `get_values` `'Tab'!A1:R(T+2)` and locate rows by reading, never by memory: TOTAL row T = the row after row 4 whose col B = "TOTAL"; dividers = rows 5..T-1 whose col A is not a number; data rows = numeric col A. (Off-by-one row guesses break tabs — always read.)

## 1. Pull from Meta (per account in the tab map)
- Campaign level, `date_preset=maximum`, filter `start_time GREATER_THAN <first day of oldest open month>`: id, name, effective_status, objective, start_time, stop_time, reach, impressions, results, amount_spent.
- Ad-set level (same filter): campaign_id, end_time, lifetime_budget, daily_budget — only needed for campaigns not ended.
- A tab can merge two accounts (e.g. Men/Women) → col O = which account.
- Include every campaign with spend > 0 or status ACTIVE/PAUSED and not ended.

## 2. Row values (deterministic rules)
- A seq (1..n inside the month) · B `META` · C campaign name exactly as Meta · Q campaign ID (text) · R month key `YYYY-MM` of start_time (account timezone) · S notes.
- D type: reuse the label this client already uses for that objective on earlier rows (e.g. engagement = `Whatsapp MSG`, traffic with profile-visit result = `PROFILE VISIT`). If none: OUTCOME_ENGAGEMENT→Engagement, OUTCOME_TRAFFIC→Traffic (profile_visit_view result → PROFILE VISIT), OUTCOME_AWARENESS→Awareness, OUTCOME_SALES→Sales, OUTCOME_APP_PROMOTION→App promotion, LINK_CLICKS boost→PROFILE VISIT. Never relabel WhatsApp vs IG DM (playbook).
- E reach · F impressions · G results (blank if "Not available") · J start date · K end date (blank if no stop_time) · M spent.
- Status P: stop_time passed → `DONE`; PAUSED and not ended → `PAUSED`; otherwise `IN PROGRESS`.
- Budget L (Rule B): DONE → L = spent. Not ended → L = sum of ad-set lifetime budgets. No end date + daily budget → L = spent, S = `daily budget <cur> X, no end date`.
- H `=IFERROR(M{r}/F{r}*1000,"")` · I `=IFERROR(M{r}/G{r},"")` · N `=SUM(L{r}-M{r})`.

## 3. Write
- Existing campaign (match Q id, else exact C name): update E,F,G,J,K,L,M,P,Q (and clear S stale note) with `update_formulas`, nulls for untouched cells.
- New rows go at the end of the tab: `insertDimension` at startIndex T-1 (before TOTAL), `inheritFromBefore:true`.
  - Data rows: `copyPaste` from last data row (T-1) with `PASTE_FORMAT` and `PASTE_CONDITIONAL_FORMATTING` (extends existing rules, no duplicates).
  - New month divider: `repeatCell` A:I (bg #F3F4F6, Inter 10 bold #111827, top/bottom border #D1D5DB, left), `PASTE_FORMAT` cols J:S from an existing divider, `mergeCells` A:B only (merges across the frozen B|C boundary are rejected), clear DV on the row, row height 26. Label goes in col A.
  - Set Q of new rows to TEXT number format before writing ids.
- Values via `update_formulas` (USER_ENTERED): prefix month keys and divider labels with `'` (`'2026-10`, `'2026 OCTOBER`) or Sheets turns them into dates. Dates as serial numbers (days since 1899-12-30); cells format dd/mm/yyyy.
- Placeholder tab (only "MONTH YEAR" + 1 empty row): fill row 5 (divider) and row 6 (first campaign), then append.

## 4. Refresh formulas (every tab you inserted into)
Re-read col A:B, then write explicitly (don't rely on auto-adjust; single-row ranges collapse):
- Each divider r: J `=SUMIFS(M$6:M$last,$R$6:$R$last,$R{r},$A$6:$A$last,">0")`, L same with L, M same with M.
- TOTAL row T (last = T-1): E/F/G `=SUM(E6:Elast)`, H/I `=IFERROR(AVERAGEIF(H6:Hlast,">0"),"")`, J and M `=SUMIF($A$6:$A$last,">0",M6:Mlast)`, L `=SUMIF($A$6:$A$last,">0",L6:Llast)`, N `=SUM(N6:Nlast)`.
- Pinned row 3 and ADS Clients refs follow automatically.

## 5. ADS STATS
- Each client block = header row (currency in A) + one row per month. New month → insert row after the block's last month row (`inheritFromBefore`), write: A seq, B MONTH, C YEAR, D `=M{r}`, E/F/G/L/M `=SUMIFS('Tab'!X:X,'Tab'!R:R,$O{r},'Tab'!A:A,">0")`, H/I CPM/CPA, J `=IFERROR(1/(1/MINIFS('Tab'!J:J,…)),"")`, K same with MAXIFS on K, N `=SUM(L{r}-M{r})`, O `'YYYY-MM`.
- Rewrite that block's header J/L/M `=SUM(Mfirst:Mlast)` / L / M, the grand TOTAL row (`SUMIF($A$5:$A$last,">0",…)`) and SPENT USD / SPENT AED (`SUMPRODUCT(($A$4:$A$last="USD")*…)`).
- ADS Clients needs no edits (refs move with rows). Fill START/END only for new clients.

## 6. Tab order
After writing, sort client tabs by latest campaign end date, newest first (ongoing/no end = top): ADS Clients, ADS STATS, active…, new-empty, dormant, legacy, TEMPLATE (80), Data Sources.

## 7. Verify (must pass before reporting)
- Each touched month divider M = sum of Meta spend for that month's campaigns (to the cent).
- Pinned TOTAL = bottom TOTAL; ADS STATS grand total = SPENT USD + SPENT AED = ADS Clients TOTAL USD + AED.
- No `#REF!`/`#VALUE!`; ids exact (18 digits); keys are text.
- Log the run in today's backlog `claude/backlog/YYYY-MM-DD.md` (create or update — one file per day) and the objective/need log only if a build happened.

## Never
- Touch formatting beyond the steps above, delete rows with data, edit Meta objects, or guess a row number.