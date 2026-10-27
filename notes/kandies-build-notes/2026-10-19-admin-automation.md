# Oct 19, 2026 (late night): Admin automation + house ads

## Goal
Find every admin task in the Kandies code that can run on its own, and automate it.

## Audit results (read the code first)
Automated tonight:
1. Homepage "New music": was hand-picked → now auto-fills with the 10 newest public songs
2. Featured artists: was hand-picked → now auto-fills with the top 6 trending artists (7-day plays + 2× likes)
3. Browse featured video: was manual upload → now rotates weekly to an artist who has a music video
4. Promos / Browse ads: stayed up forever → now have optional end dates and auto-expire
Already automatic: Spotlight (done earlier tonight), Stats, ad click counts
Kept manual (judgment calls): creating/editing artists, Artist Development guides, reading bug reports, deleting comments

## Timeline (Eastern)
- 22:05  Audited admin features in index.html (grep for isAdmin + admin functions)
- 22:08  Exported table columns (information_schema → Export → Copy as markdown)
- 22:15  Wrote SQL + app changes
- 22:18–22:26  Automation SQL didn't apply (still only 1 cron job)
- 22:45  Re-ran in 4 small pieces → all 5 cron jobs active
- 22:20  Added house ads (Kandies promoting itself when no paid ad is running)
- 23:26  Downloaded the new index.html (646,897 bytes) and copied it into the project
- 23:34  Tested; Browse showed no house ads → confirmed by design (3 paid ads active)
- 23:41  Deployed via Netlify Drop ("Published at 11:41 PM"); git push succeeded

## Database changes
- New columns: featured_tracks.is_auto, featured_artists.is_auto (existing picks = "pinned"), promos.ends_at, browse_ads.ends_at
- New functions: auto_refresh_new_music, auto_refresh_featured_artists, auto_rotate_browse_video, auto_expire_ads
- Cron jobs (all active):
  - spotlight-weekly-rotation   5 * * * *    hourly :05
  - auto-new-music              15 * * * *   hourly :15
  - auto-featured-artists       10 10 * * *  daily 6:10 AM ET
  - auto-browse-video           25 * * * *   hourly :25 (switches weekly)
  - auto-expire-ads             35 * * * *   hourly :35

## App changes (index.html)
- Manage Home labels each item "pinned" (my pick, permanent) or "auto" (refreshes on schedule)
- Promos + Browse ads get an "Auto-remove after" date field; expired ads hidden immediately
- House ads: ⭐ Vote for the Spotlight, 🎤 Ohio artists upload free, 🛍️ Spotlight artist's merch
  - Shown only when no paid ad is running in that spot; tagged "Kandies" not "Ad"
  - House ad taps are not counted in ad click stats (keeps sponsor numbers honest)
- Load errors now logged instead of hidden

## SECURITY
1. FOUND + FIXED: the Browse video "Save" deleted the old video file without checking ownership. With auto-rotation sharing artist video files, it could have deleted an artist's own profile video. Fix: only delete files whose key starts with "browse-".
2. BUILT IN: the merch house ad only uses links starting with http:// or https://. Blocks "javascript:" links (a stored-XSS trick) if someone puts one in a merch field.
3. BUILT IN: the automation functions pin search_path and have execute revoked from public/anon/authenticated. Only the scheduler can run them (least privilege).
4. GAP NOTED (not fixed yet): comments have no user Report/Block. Google Play requires both for apps with user content.

## Troubleshooting
1. Big SQL block "succeeded," but the cron check still showed 1 job
   Diagnose: counted what actually exists instead of trusting the message:
     select count(*) from pg_proc where proname in (...)                → 0
     select count(*) from information_schema.columns where column_name in ('is_auto','ends_at') → 0
   Cause: the long block never fully pasted/ran.
   Fix: split into 4 numbered pieces, run one at a time, verify with cron.job → 5 jobs.
2. Copy-Item kept failing: "Cannot find path ...Downloads\index.html"
   Diagnose: Get-ChildItem on Downloads = empty; a whole-profile recursive search for index*.html in the last 2 hours = empty → the file never downloaded.
   Cause: clicking the file card in the Claude app opens a PREVIEW; it doesn't download.
   Fix: in the preview, the ⌄ arrow next to "Copy" → Download. Verified by byte size (646,897).
3. No "Kandies" card in Browse
   Diagnose: select title, active, ends_at from browse_ads → 3 active paid ads, no end dates.
   Result: working as designed (house ads fill in only when no paid ad is running).
4. Asked whether expiring ads leave empty space
   Answer: ads only expire if given an end date; remaining ads rotate in that slot; if none, house ads fill it; if a section has nothing, it disappears (no empty box).

## Lessons
- Verify database changes by querying what exists (pg_proc, information_schema, cron.job), not by trusting "Success"
- Break long SQL into small numbered pieces and check each one
- Verify downloads by file size; check where a file actually landed before copying
- Shared files need ownership rules: only delete files a feature created itself
- Only render user-supplied links with http/https schemes
- Keep humans in control: automation fills in around manual "pinned" picks, never replaces them

## Next up
- Report/Block for comments (Play Store requirement)
- Android signing key + release (.aab) build workflow
- Play Store listing: privacy policy, Data Safety form, screenshots, content rating
- Optional: mix house ads in alongside paid Browse ads
