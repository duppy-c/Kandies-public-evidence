# Kandies Security Sweep Lab: October 23 and 26, 2026

**Time zone:** America/New_York (EDT)
**Course context:** CYB/130T, applied practice on my own system
**System under test:** Kandies (my own app), Supabase Postgres + Auth, single-file app on Netlify

**Provenance note:** The original cybersecurity note for these sessions was cut off by an automatic safety filter while being written and was deleted. This note is rearranged only from the Kandies build notes for Oct 23 and 26, 2026 — timeline, database changes, and troubleshooting sections. No new facts have been added.

## Goal

Finish small pre-launch items (artist renames, Terms of Service), then run a full security sweep of the app code and the Supabase database before the Play Store release.

## Timeline

### Friday, Oct 23

| Time | Event |
|---|---|
| 8:03 PM | Session start; confirmed the 4 email-based artist renames + emails were already done |
| 8:19–8:32 PM | Terms of Service: added section 7 "Comments and community"; tested on localhost |
| 8:37 PM | Netlify publish: Terms update |
| 8:41 PM | Decided to do a full app once-over + security sweep before Android signing |
| 8:53–8:55 PM | Unsafe link finding tested (stored + shown; Safari and Chrome blocked it) |

### Monday, Oct 26

| Time | Event |
|---|---|
| 2:49 PM | Session start; resumed the link fix |
| 2:55–2:56 PM | Found 6 artists with non-https links; cleaned up |
| 3:03 PM | Database link rule added and verified |
| 3:21 PM | App link check verified on localhost |
| 3:22 PM | Netlify publish: safe links |
| 3:24–3:33 PM | Supabase Security Advisor: 0 errors, 39 warnings; reviewed every function |
| 3:40–3:44 PM | User-count functions closed to signed-out visitors; function access tightened |
| 3:51 PM | App check after function changes: all good |
| 3:53–3:58 PM | Plays/ad clicks locked to the signed-in user (test had to be redone, see Troubleshooting) |
| 3:59–4:00 PM | Listening history (plays) made private |
| 4:06–4:09 PM | Scanned all public tables for user IDs; private tracks confirmed safe |
| 4:10–4:20 PM | Likes, hearts, Spotlight votes and ratings made private (app + database) |
| 4:18 PM | Netlify publish: private likes/votes app |
| 7:11–7:18 PM | Leaked password protection turned on, minimum password length 6 → 8, tested |

## Security Findings and Hardening

### Finding 1: Unsafe links stored and shown
- **Risk:** A non-https / unsafe link could be stored as an artist, promo, ad, merch, or dev link and shown to users.
- **Before:** An unsafe link was tested and shown (browsers blocked it, but the app stored/shown it). A read-only check found 6 existing artists with non-https links (Instagram @handles, missing https://, a test link).
- **Fix:** Database rule `artists_links_web_only` — website, Instagram and merch links must start with `http://` or `https://`. App `safeUrl()` / `normalizeUrl()` — only http(s) links saved or shown. Instagram `@handle` auto-converts to a full link. Existing data was cleaned first (handles → full links, https:// added, test link cleared).
- **After:** App link check verified on localhost; published 3:22 PM.

### Finding 2: Function access too open (Security Advisor review)
- **Risk:** Signed-out visitors and direct callers could reach functions that should be restricted.
- **Before:** Supabase Security Advisor showed 0 errors, 39 warnings; every function was reviewed.
- **Fix:** `kandies_user_count` and `kandies_new_user_count` closed to app users. Signed-out access removed from `admin_reset_spotlight_cycle`, `admin_set_claim_code`, `claim_artist_profile`, `run_spotlight_cycle`. Direct access removed from 4 trigger functions.
- **After:** App check after the function changes: all good.

### Finding 3: Plays / ad clicks could be logged as someone else
- **Risk:** Impersonation / false activity — a signed-in user logging a play or click under another user's id.
- **Fix:** Plays / ad_clicks insert rules — a signed-in user can only log a play or click as themselves (or with no user).
- **After:** Fix confirmed after the test was redone correctly (see Troubleshooting — the first test was flawed).

### Finding 4: Listening history exposed who played what
- **Risk:** Privacy — plays table could reveal who listened to which track.
- **Fix:** Plays read access — only `track_id` is readable (counts still work); who played what is hidden.
- **After:** Applied and verified live.

### Finding 5: Likes, hearts, votes and ratings exposed other users' rows
- **Risk:** Privacy — other users' like/vote/rating rows were readable.
- **Before:** All public tables were scanned for user IDs; private tracks were confirmed safe.
- **Fix:** New totals-only functions — `track_like_counts`, `artist_heart_count`, `spotlight_rating_summary`, `spotlight_vote_tally` (return numbers only). `track_likes`, `artist_hearts`, `spotlight_votes`, `spotlight_ratings` — each user can only read their own rows. App (661,925 bytes, live) reads totals from the new functions.
- **Order note:** Add functions → deploy app → hide rows, so counts did not break during the change.

### Finding 6: Auth — leaked passwords and short minimum length
- **Risk:** Weak / previously leaked passwords accepted.
- **Fix:** "Prevent use of leaked passwords" turned on; minimum password length 6 → 8.
- **After:** Tested.

## Troubleshooting (from the build notes)

| Problem | Cause | Fix |
|---|---|---|
| Link rule would have failed | 6 existing links weren't https (handles, bare domains, a test link) | Read-only check first, cleanup, then the rule |
| localhost still showed the database error, not the app's message | New file hadn't been downloaded yet; old app still running | Checked with Select-String; downloaded via the file card's ⌄ menu |
| Get-ChildItem returned nothing | File not downloaded yet | Download first, then check size, then copy |
| "NetworkError … api.supabase.com" | Dashboard couldn't reach Supabase (connection/VPN), query never ran | Retried |
| Plays test passed both before and after the fix | Test looked up "someone else" while acting as the test user, who can't see other profiles, so it inserted a play with no user (allowed) | Looked up the other account first, then switched roles; fix confirmed |
| Final check showed only the first line's result | Last query returned zero rows | Rewrote it to always return a count row |
| Risk of wrong counts during the likes/votes change | Hiding rows before the new app was live would break counts | Order: add functions → deploy app → hide rows |

## Evidence

- Public-safe evidence set: 15 photos + README (4 from Friday, 11 from today). The full private set is 17 photos (4 from Friday, 13 from today, numbered by day).
- **Today's #13: the email typed into the signup test is blurred in the public set.**
- **Today's #06 and #07: left out of the public set on purpose. They come from the flawed plays test described above. Do not present them as proof the fix passed; the fix was only confirmed after the test was redone correctly (#08).**
- Supabase dashboard photos in the public set (#04, #05, #08, #09, #11, #12) have the browser address bar and dashboard header blurred (project slug and account name).
- The unblurred originals, including #06 and #07, stay in the private evidence set only.

## Key Lessons / Technical Facts

- Check existing data before adding a rule; a rule fails if old rows break it.
- A test that looks up data while acting as a limited user can be fooled by that user's own limits. Look up test values first, then switch roles.
- When tightening what the app can read, ship the app change first, then lock the database.
- A verification query should always return a row (a count), so "no result" can't be confused with "didn't run."

## Open Items / Next Steps (from the build notes)

- **Pay the Supabase invoice** (red banner) so the project isn't paused.
- App once-over list (fixes and polish) before release.
- Android signing and `.aab` release build; Google Play listing.
- Optional: listener profile names; captcha (needs app setup first).
