# Meta Ads Playbook
Living doc. Claude updates it whenever a rule changes or something new is learned.
Automation and schedules: `claude/workflow.md`. ADS sheet: `claude/ads-sheet.md`. Reports: `claude/reports/`.

`<Operator>` = the name you put at the end of every object name (e.g. your first name). Set it once and keep it.

## Workflow
1. The operator names the client and the need (objective, budget, duration).
2. Claude reads `claude/clients/<client>.md` and the last run on that account.
3. Claude suggests either reusing the previous setup or trying something new, with a one-line reason.
4. Claude lists the full setup verbatim, in hierarchy order: account → campaign → ad set → ads.
5. The operator approves. Nothing runs before approval.
6. Claude builds, publishes, and runs the **Launch verification** (see HARD RULES and "Publish and verify"): every campaign, ad set and ad found live, nothing left in drafts, then a scheduled delivery check.
7. Claude updates the client doc and the log below.
8. Every action, decision and instruction (manual, skill or scheduled) is logged in `claude/backlog/YYYY-MM-DD.md`: one file per day. Scheduled runs update it and never create a second.

## HARD RULES
- **Never report a launch until it's verified live:**
  - After every publish, read back the **live** campaign, ad set AND every ad (filter by campaign id). Each must exist live with status ACTIVE and effective_status ACTIVE / IN_PROCESS / PENDING_REVIEW (or SCHEDULED for a future start).
  - Also read the **draft** state for the account: nothing from this build may remain in drafts.
  - "PUBLISHING" from the publish call, a configured ACTIVE, or drafts disappearing are NOT proof. Only the live read-back is.
  - If anything is missing live or still in drafts: don't report done. Debug immediately (see "If a publish fails silently"), fix, re-publish, read back again.
  - The launch message always states the read-back result per object (campaign / ad set / each ad: status + effective status).
  - **Delivery check:** schedule a follow-up (send_later) ~2–3h after the ad set's start time. Confirm impressions > 0 and ads out of review. If still 0 impressions or ads stuck in review/rejected, alert right away with the reason (ads_get_errors, delivery status). The daily check also flags any ad set with 0 impressions >6h after its start.
- **Editing a live ad set (budget, schedule, targeting) pauses it:**
  - `ads_update_entity` on an ACTIVE ad set returns `status_forced_to_paused: true`, meaning the tool pauses it with the edit.
  - The approved plan must say "edit → reactivate". Right after the edit, reactivate with `ads_activate_entity` and read back status ACTIVE + the new value. If reactivation is blocked, say so immediately; never leave it paused silently.
  - After a live edit the ad set and its ads go back to IN_PROCESS (re-review) for a while. Schedule a delivery check ~2h later.
- **Advantage+ audience: OFF unless the operator asks for it.**
  - Always send `targeting_automation.advantage_audience = 0`; the API turns it ON when the field is omitted.
  - Read it back after publishing.
- **Mirror audiences exactly.**
  - Copy every detailed-targeting group with its IDs: the "include" group plus each "must also include" group that narrows it. Never flatten them into one list.
  - Copy geo, location types, gender, ages and placements too.
- **Audience source = client doc targeting (with IDs), sent inline.** A saved audience is mirrored manually into the client doc (interests, groups, age, geo, exclusions) and sent inline. Passing `saved_audience_id` is accepted but its targeting is NOT applied. Verify the read-back matches the doc.
- **Exclude existing followers/engagers on prospecting ad sets** when the client has those custom audiences: `excluded_custom_audiences` with their IDs. Keep the IDs in the client doc.
- **WhatsApp vs Instagram DM: never confuse the two.**
  - A campaign whose target is WhatsApp is labeled "Whatsapp MSG". A campaign whose target is IG DM is labeled "Direct Message".
  - Go by the target the operator set, never relabel.
- **Every ad must carry the Facebook Page + Instagram account (identity).** Never send an ad with only the IG post / IG account: it lands with no page selected and won't publish.
- **Messaging ads (Engagement → Conversations → IG Direct) from reels: build the creative inline with `object_story_spec`**. See "Messaging-ad creative recipe" below. This is the only path that publishes cleanly from the draft publisher.
- **Messaging ad sets (Engagement → Conversations → IG Direct): attribution = 1-day click.** Don't send a 7-day `attribution_spec`: the draft shows zero errors but publish silently drops the ad set and ads (error 1885423 "Attribution window is invalid… use 1 days" only surfaces after a later draft edit). Safest: omit attribution_spec or send `[{"event_type":"CLICK_THROUGH","window_days":1}]`.
- **Traffic campaigns = Visit Instagram profile, for every client and every ad set:**
  - Objective OUTCOME_TRAFFIC · destination_type `INSTAGRAM_PROFILE` · optimization_goal `VISIT_INSTAGRAM_PROFILE` · highest volume · billed on impressions · 1-day click.
  - Never use `PROFILE_VISIT` (rejected on several accounts, subcode 2016153). No promoted_object needed.
  - Ads: existing IG posts (`source_instagram_media_id` + `instagram_user_id`), no call_to_action field.
  - A different traffic destination (website, landing page) only when the operator asks explicitly.
- **Operator's own ad accounts** (testing, customer issues): when a client's spend or campaigns aren't in their own account, check these before concluding anything. List them in `claude/clients/index.md`.
- **Scheduled runs are read-only on Meta.** Pausing/turning off ended campaigns only after the operator approves the list in chat.
- **App campaigns: judge cost per install on store data** (App Store Connect / Play Console), not Meta alone. Meta can under-count heavily (5× has been seen on Android after an SDK break).

## Messaging-ad creative recipe (IG Direct, from reels)
1. `ads_get_ig_media` on the client's IG → pick the reels.
2. Get each reel's `video_id`: run `ads_create_creative` once with `page_id` + `instagram_user_id` + `source_instagram_media_id` + `call_to_action_type INSTAGRAM_MESSAGE` (no link), then `ads_get_creatives` on it → `video_id`. Don't use that creative_id for the ad.
3. Thumbnail `image_hash`: the probe creative may not return one, and video `picture`/thumbnail URLs are 64–160px. Get a proper cover instead: on a local machine curl the reel's IG `media_url` and grab a frame with ffmpeg → `ads_creative_upload_local_image` (name, mime, size, sha256) → POST the bytes to the returned `upload_url` with headers `Authorization: OAuth <upload_authorization>`, `Offset: 0`, `X-Entity-Length`, `X-Entity-Type`, `X-Entity-Name` → `ads_finalize_local_ad_image_upload` with the `completion_token` → `hash`. (Cloud sandboxes may not reach Meta's upload host; a local machine can.)
4. Create each ad with an inline creative:
   `{"object_story_spec":{"page_id":"<PAGE>","instagram_user_id":"<IG>","video_data":{"video_id":"<VID>","image_hash":"<HASH>","message":"<caption>","call_to_action":{"type":"INSTAGRAM_MESSAGE","value":{"link":"https://www.instagram.com/","app_destination":"INSTAGRAM_DIRECT"}}}}}`
   - `image_hash` only, never `image_url` too.
   - `message` = the caption (fix typos).
5. Check `active_errors` = [] on every ad before publishing.
- What fails (don't use for messaging ads):
  - `{"creative_id": ...}` or `source_ad_id` copy → "ObjectStorySpecRedundant: Only one of image_url and image_hash…".
  - `source_instagram_media_id` + INSTAGRAM_MESSAGE as the ad creative → rejected by the draft publisher.
  - Same with a link → "Verified Whatsapp number isn't a business account".
  - Any version without `page_id` → no page/IG identity, won't publish.
- Multi-destination (IG Direct + WhatsApp in one ad set) isn't available through the MCP: `destination_type` takes one value. Use IG Direct only, or tick WhatsApp in Ads Manager.

## Default build (the fingerprint)
- **Structure:** 1 campaign → 1 ad set per geo → 4–7 ads.
- **Budget:** ABO (on the ad set, never the campaign), lifetime budget only, usually about 10 days.
  - Messaging runs: $150–200 for 10 days.
  - Tests: $20–50 for 4–6 days.
  - Some clients have a fixed monthly pot split across campaigns by need. See client doc.
- **Cadence:** a new dated campaign starts the day the last one ends. Never edit or extend old campaigns; rebuild with a new date instead.
- **Core objective:** Engagement → Conversations → messaging target (IG Direct or WhatsApp, per client). Sales-objective campaigns are also set up for Conversations.
- **Traffic:** Visit IG profile → Instagram profile (see HARD RULES). Often launched as a pair with Engagement on the same day.
- **Bid and billing:** Highest volume, billed on impressions.
- **Attribution:** 1-day click for Engagement → IG Direct messaging (7-day blocks publish, see HARD RULES); 1-day click for traffic, profile visits and link clicks.
- **Location types** (who counts as "in" the area; not a placement): home + recent + frequently in. API: `location_types ["home","recent","frequently_in"]`.
  - Clinics and local shops: radius pin around the business.
  - Retail: country or region.
- **Age and gender:** 22–55 by default. Luxury retail can be split by gender, 18/22–65.
- **Interests:** stacked "must also include" groups, often with Engaged Shoppers in one of them. Copy the client's stack exactly (see HARD RULES).
- **Advantage+ audience:** OFF (see HARD RULES).
- **Placements:** use every channel the ad can run on.
  - Instagram destination: Instagram feed, story, reels, explore home, profile feed, search, plus Threads.
  - WhatsApp in the destination: add WhatsApp Status.
  - Facebook only when the client doc says so.
- **Multi-geo:** one ad set per city or country, with a geo suffix.
- **Creatives from client reels:**
  - Traffic/Awareness: existing IG post (`source_instagram_media_id` + `instagram_user_id`), matched from the permalink via ads_get_ig_media.
  - Messaging (IG Direct): the recipe above (inline object_story_spec with page + IG).

## Naming (canonical; normalize every new build to this)
- **Campaign:** `DD/Month/YYYY [Objective] Campaign <Operator>`
- **Ad set:** `DD/Month/YYYY [Objective] Ad Set <Operator>`. With several ad sets, add a geo suffix, e.g. `... Ad Set <Operator> Dubai`.
- **Ad:** `DD/Month/YYYY [Objective] Ad 001 [Type] <Operator>`
  - Type: Reel / Post / Story / Carousel / Single.
  - Numbers are always 3 digits.
- **Month:** full English name, capitalised (October, not okt/oct/october).
- **Ads added mid-flight:** use the date they were added and continue the numbering.
- **App campaigns:** platform first, e.g. `iOS DD/Month/YYYY App promotion Campaign <Operator>`.
- **Several brands on one ad account:** brand prefix first, e.g. `<Brand> iOS DD/Month/YYYY …`.
- **Test builds:** `TEST NN <what's tested> - RESULT <pass/fail/note>` on ad sets, so drafts can be audited at a glance.
- **Never leave:** "- Copy", "نسخة", double spaces, or Meta default names.

## Audiences
- Each client has a doc at `claude/clients/<client>.md` holding the exact audience mirrored from their last live ad set (or from a saved audience), including IDs and group structure. Use that by default.
- Copy the last-used live audience only when it's the best option, when testing, or when there's no client doc yet. Then update the doc.
- **What the Meta Ads MCP can do with audiences (tested Nov 2026):**
  - Inline targeting: geo + radius, location types, age, gender, interests/behaviors/employers/job titles by ID in stacked groups → works.
  - Exclusions (`excluded_custom_audiences`) → works.
  - Custom audience include only (retargeting; A+A off reason CUSTOM_AUDIENCE_ONLY) → works.
  - Lookalikes: create (`ads_create_custom_audience` LOOKALIKE, 1%) and target → works. Don't seed from engagers whose followers are the wrong crowd.
  - A+A ON with the stack as suggestions (`age_range` + `age_min` ≤25, no hard `age_max`) → works; only on request.
  - Saved audience by ID → ID stored, targeting not applied. Don't use.
  - Custom audiences: list, read, create (customer list, website, lookalike, engagement), update, delete. IG engagement audience (`ig_business` source) is listed but untested; until proven, create engager/follower audiences in Ads Manager.
- **Gap:** no interest/targeting search. New interest IDs come from the operator or an existing ad set/saved audience in Ads Manager. Never invent IDs.

## Publish and verify (launch verification, mandatory on every build)
1. Build the full draft and check active_errors = [] on every object. Check every ad has the page + IG identity.
2. Publish (campaign + ad set + ads together with object_ids).
3. Wait ~15–30s, then read back **live** at all three levels, filtered by the campaign id:
   - campaign: status + effective_status
   - ad set: status, effective_status, budget, start/end, destination, attribution, targeting
   - ads: status + effective_status for every ad (count must match what was built)
4. Read the **draft** state for the account (ad set + ad level): nothing from this build may remain.
5. If an object is missing live or still in drafts → silent failure. Debug now (step below), fix, re-publish, repeat 3–4. Never report it as launched.
6. Check the read-back targeting matches the client doc: groups, exclusions, Advantage+ OFF, location types, placements, OS versions.
7. Report with the per-object read-back. "In process" / "pending review" = in review, not live yet; say so.
8. Schedule the delivery check (send_later, ~2–3h after start): impressions > 0, ads approved. Alert immediately if not.
- **If a publish fails silently** (PUBLISHING, then the object is still in drafts with active_errors []): re-run a harmless `ads_update_entity` on the stuck draft ad set (e.g. its start/end time). That re-validates it and surfaces the real error in active_errors. Fix it, publish again with object_ids. Try this before asking anyone to publish by hand.
- Before re-publishing a stuck draft, move its start_time forward if it's already in the past.
- Broken draft objects block their parent ad set from publishing even with allow_partial/ignore flags. Discard them (discard_draft_changes, preview → confirm) before publishing.

## Known blockers and fixes
- **Live ad set edits force-pause it** (`status_forced_to_paused: true` from ads_update_entity). Reactivate right after with ads_activate_entity and read back. Ad set + ads return to IN_PROCESS (re-review) after the edit.
- **IG Explore placement is deprecated** and blocks publishing. Use `explore_home` only.
- **Old multi-destination creatives** (WhatsApp/Messenger/IG) get rejected on single-destination ad sets. Rebuild them with a matching CTA, e.g. INSTAGRAM_MESSAGE.
- **IG account must be connected to the ad account for the API.** Ads Manager lets you pick any IG you can access (even in another BM), but the API only sees IG accounts connected to that ad account (ads_get_ig_accounts). If it returns empty, connect the IG to the ad account (BM › Instagram accounts › Connected assets, or ad account assets). No IG password needed.
- **Profile-visit goal:** `PROFILE_VISIT` is rejected on some accounts ("Ad account isn't eligible for Profile Visit ads", subcode 2016153). Always use `VISIT_INSTAGRAM_PROFILE` with destination INSTAGRAM_PROFILE.
- **IG-post creatives (traffic/awareness):** don't send a call_to_action with `source_instagram_media_id` ("The link field is required"). Send only the media id + instagram_user_id.
- **IG-post creatives (messaging / IG Direct):** don't use them. See the messaging-ad recipe. The draft publisher rejects every IG-post / creative_id / source_ad_id variant.
- **The draft publisher can publish the campaign but drop its children.** Usually a hidden validation error (see "If a publish fails silently"). Find it, fix it, then re-publish the ad set and ads explicitly with object_ids.
- **Attribution 7-day click on Engagement → Conversations → IG Direct:** invalid, only 1 day allowed (error 1885423). Draft shows no error; publish drops the ad set silently.
- **Draft creatives are immutable** (ads_update_entity rejects creative changes). To fix an ad's creative, discard the draft ad and create a new one.
- **WITH_ISSUES ads** are usually copyright rejections (music). Usually acceptable; don't chase them.
- **The API defaults Advantage+ audience to ON** when the field isn't sent. Always send 0.
- **Saved audiences:** can't be listed; `saved_audience_id` is accepted but its targeting isn't applied. Mirror inline from the client doc.
- **App campaigns: promoted_object reads back empty** on existing ad sets, so copying with source_adset_id fails ("APP_INSTALLS requires promoted_object"). Keep each app's Meta App ID + store URLs in the client doc and pass promoted_object explicitly.
- **iOS OS targeting:** `user_os ["iOS_ver_14.0_and_above"]` saves as iOS 14.0–14.4. Send `["iOS_ver_14.0"]` (= 14.0+) and read back.
- **Drive "Computers" folders:** never create or move files there from the cloud. Desktop sync bins cloud-added files. Keep the ADS master in My Drive.
- **Sheets writes:** USER_ENTERED turns "2026-10" / "2026 OCTOBER" into dates, so prefix with `'`. Merges across the frozen column boundary are rejected. Single-row ranges collapse (E6:E6 → E6), so rewrite totals explicitly. Full rules: `claude/ads-sheet.md` and the reconciliation skill.

## Not your work (ignore as a pattern source)
List here anything in the accounts built by others (client staff, old agencies, Easy Ads funnels, IG app boosts) so it's never copied as a pattern. Typical tells: "Objective - product" names, default ad set/ad names, CBO daily budgets.

## Objective → need log
| Date | Client | Need | Objective | Destination | Optimization | Result |
|---|---|---|---|---|---|---|
