# Kandies Build History: Inception to October 21, 2026

**Project:** Kandies (kandiesapp.com), a free independent music streaming app for Ohio artists, built solo
**Time zone:** America/New_York
**Covers:** everything before session-by-session notes began on October 22, 2026

> **How this was put together:** rebuilt from saved project memory, the project's "Code" document, files delivered during earlier sessions, and earlier session summaries. Dates before October 19 are approximate (month or version level). Times from October 19–21 come from a timestamped debug log kept during those sessions. There are no evidence photos for this period.

---

## Summary

| Phase | When | Highlights |
|---|---|---|
| 1. Idea and foundation | ~July 2026 | App concept, single-file web app on Netlify, Supabase backend, base schema |
| 2. Core features (v1–v74) | ~July–Sept 2026 | Artist profiles, uploads, streaming, likes, comments, Ohio Spotlight, admin tools, versions up to v74 |
| 3. Business and launch planning | Sept–early Oct 2026 | CFO prep, distribution strategy, LLC and trademark planning, marketing content, Play Store plan |
| 4. Native Android wrapper | ~mid Oct 2026 | Capacitor project, Raspberry Pi dev box, GitHub repo, GitHub Actions test APK |
| 5. Stabilization and security | Oct 19–21, 2026 | Spotlight rotation fix, scheduling, admin automation, stats, critical permission fix |

---

## Phase 1: Idea and foundation (~July 2026)

- **Idea:** a free streaming platform for independent artists, focused on Ohio, where artists keep ownership of their music and listeners pay nothing.
- **Architecture chosen:**
  - One self-contained web app file (`index.html`: HTML, CSS and JavaScript together), hosted on **Netlify**.
  - **Supabase Pro** for sign-in, database (Postgres with row-level security) and file storage.
  - Domain **kandiesapp.com**, with **kandies.netlify.app** as the fallback address.
- **Base database** (saved in the project's "Code" doc, July 23): `artists`, `tracks` (with private/public songs), `plays`, `track_likes`, with row-level security so anyone can read public content and only owners can change their own.

## Phase 2: Core features, v1 to v74 (~July–September 2026)

Built as numbered versions, each with its own SQL migration and deploy steps.

- **Listening:** browse and stream songs, likes, play counts, genres.
- **Artists:** profiles with bio, picture, website/Instagram/merch links, music videos; artist sign-up with a "I own the rights to my music" agreement; Ohio artist tag.
- **Community:** comments on artist pages; hearts for artists.
- **Ohio Spotlight:** weekly community vote, star ratings, featured artist page.
  - v72: artist profile effects (confetti, shimmer, pulsing badge) with per-artist admin override.
  - v73: tapping the Spotlight artist opens their full profile.
  - v74: fixed votes piling up forever by adding a cycle reset function.
- **Home page:** featured artists with animated glowing borders, new music, promos, merch.
- **Admin tools:** Manage Home, Manage Artists (showcase profiles for artists who haven't signed up yet), claim codes for artists to take over their profile, Browse ads, bug reports, developer guides/checklist/links.
- **Design fixes:**
  - Sign-in screen showed a ghosted, color-split logo. Cause: an image embedded in the file had the flaw baked in. Fix: replaced it with the clean candy image and drew the wordmark as styled text.
  - Sign-in sparkles didn't match the home page. Fix: reused the same sparkle style on both screens.

## Phase 3: Business and launch planning (September–early October 2026)

- **Funding:** prepared for a CFO advisor meeting (traction snapshot, funding breakdown, meeting checklist).
- **Distribution strategy:** decided Kandies should fund artists' own distribution accounts rather than act as a distributor, and stay out of the money path entirely.
- **Exclusive releases idea:** to be demand-tested with marketing before building; artists would be paid directly.
- **Business setup plan:** Ohio LLC, possible separate nonprofit arm, "Kandies" trademark, logo copyright, Meta business verification.
- **Name decision:** kept "Kandies" (branding already established).
- **Marketing:** CapCut editing guide, Reel scripts per tab, transition prompts, Spotlight/ad content pack; rule adopted to keep AI-generated video face-free and use real screen recordings for real artists.
- **Launch plan (Oct 8):** Android/Google Play first, then Apple; wrap the web app with Capacitor; LLC filing checklist and resequenced roadmap.
- **Oct 12:** recovered and saved the base schema; noted that the v72–v74 migrations had never been saved to the project.

## Phase 4: Native Android wrapper (~mid October 2026)

- **Raspberry Pi** (64-bit Debian, ~1 GB RAM) set up as a dev box; project folder `~/kandies-app`, Node 22.
- **GitHub:** private repo `duppy-c/kandies-app` holding the Capacitor project.
- **GitHub Actions:** builds a **test APK** on every push; installed and ran on an Android phone.
- Decided the final Android build runs in the cloud (GitHub), not on the Pi.

## Phase 5: Stabilization and security (October 19–21, 2026)

### October 19: Ohio Spotlight never rotated

| Time | Event |
|---|---|
| 16:16 | Confirmed the rotation functions exist in the live database |
| 16:24 | Evidence: the Spotlight had not changed in **39 days** (~5 missed weekly rotations) |
| 16:30 | Browser console showed the rotation request failing on every page load, hidden by the app |
| 16:34 | **Root cause:** a database safety rule blocks "delete all rows" without a condition, so the whole rotation rolled back |
| 16:40 | Fix applied (explicit condition on the delete) |
| 16:46 | Verified: Spotlight rotated to a new artist |
| 16:50 | Local test server had "frozen" ~15 minutes: clicking inside PowerShell had paused it (Esc resumed) |
| 19:19–19:45 | **New feature:** on rotation, Spotlight automatically shows the artist's music video, or their most-played song. Deployed. |
| 19:53–20:02 | Security fix: a privileged function didn't pin its search path (see security retrospective) |
| 20:10 | Admin "Set as Spotlight" now also pulls in media, through a new admin-checked function |
| 20:38 | **Scheduled rotation:** turned on the database scheduler (pg_cron) so rotation no longer depends on someone opening the app |
| 20:44 | Deployed (commit f992c63) |
| 21:50 | Verified the first automatic run succeeded at 9:05 PM |

### October 19 (late): admin automation

- Audited admin chores. Automated: homepage New music, Featured artists (by recent plays and likes), weekly Browse video, promo/ad expiry dates. Kept manual: artist create/edit, guides, bug reports, comment moderation.
- Fixed a bug where saving a new Browse video could delete an **artist's** own video file; deletes now only touch admin-uploaded files.
- **House ads:** when no paid promos run, Kandies promotes itself (vote for Spotlight, upload as an Ohio artist, Spotlight artist's merch).
- 22:45: automation applied in 4 pieces after the single large block silently failed to paste. **5 scheduled jobs** active.
- 23:41: deployed.

### October 20: GitHub and Parrot OS lab

- Set up the `cybersecurity-labs` GitHub repo and flashed Parrot OS onto a Raspberry Pi 4 (separate lab; full notes: `2026-10-20_parrot-os-pi-github-setup.md` and `session-notes-2026-10-20.md`).
- Adopted the Muse workflow: Claude writes end-of-session notes; Muse pushes them to GitHub.

### October 20: detailed admin Stats

- **Bug found:** Stats undercounted once plays passed **1,000 rows**, because the app downloaded raw rows and the database caps responses at 1,000.
- **Fix:** all counting moved into one admin-only database function (accounts, listening, engagement, top songs/artists with unique listeners, 30-day chart, genres, ads).
- Added drill-down lists: newest artists and how they joined, artists with no songs, songs with zero plays, artist accounts without a profile.
- **File mishap:** while transferring the new app file, `index.html` was overwritten with 120 bytes of command text (copying the command replaced the file on the clipboard). Recovered with `git restore`. New rule: never move a file through the clipboard.
- 18:23: Stats live. Real numbers: 42 artist profiles (17 claimed, 25 unclaimed, 18 Ohio).

### October 20: account type gap and a critical permission fix

- **Account type gap:** 3 listener accounts had artist profiles, because "Become an artist" created the profile without switching the account. Fixed those 3 accounts.
- **Decision:** switching from listener to artist is allowed but **one-way**, with a clear warning (built Oct 22).
- **Critical finding:** signed-in users could change protected fields on their own profile, including the admin flag. Fixed and tested the same evening (see security retrospective).
- Other gaps found and queued (all fixed Oct 22): artists could edit staff-only fields; any account could create an artist row; signup trusted the account type sent by the app.

### October 21

- No Kandies changes recorded.

---

## Lessons from this period

- Supabase blocks deletes and updates with no condition; use an explicit condition when you mean "all rows."
- The app must log database errors instead of hiding them; a silent failure hid the Spotlight bug for over a month.
- Anything scheduled should run in the database (pg_cron), not depend on visitors.
- Count in the database, not in the browser; responses are capped at 1,000 rows.
- Give long SQL in small pieces and verify with catalog queries, not "Success."
- Label every command with where it runs; never move files through the clipboard.
- Don't click inside a running server's PowerShell window.

## Notes index (from October 22 on)

| Date | Build note | Security note |
|---|---|---|
| Oct 20 | (Parrot OS lab) | `2026-10-20_parrot-os-pi-github-setup.md`, `session-notes-2026-10-20.md` |
| Oct 22 | `2026-10-22_kandies-build-notes.md` | `2026-10-22_kandies-security-session-notes.md` |
| Oct 23 + 26 | `2026-10-23_26_kandies-build-notes.md` | Not produced (see build note) |
| Oct 28 | `2026-10-28_kandies-build-notes.md` | `2026-10-28_kandies-security-session-notes.md` |
| Oct 29 | `2026-10-29_kandies-build-notes.md` | `2026-10-29_kandies-security-session-notes.md` |
