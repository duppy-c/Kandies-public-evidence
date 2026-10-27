# Oct 19, 2026 (evening): Spotlight auto-media, admin picks, scheduled rotation

## Goal
When the Ohio Spotlight changes artists, automatically feature their media:
1. their music video if one exists
2. otherwise their most popular song

## Timeline (Eastern)
- 18:55  Read before building: exported the table layout from Supabase (information_schema.columns, Export > Copy as markdown, no screenshots needed)
  - Music videos live on artists.video_key / video_poster_key / video_title
  - Popularity = count of rows in the plays table per track
  - spotlight table had video fields but no song field
- 19:13  DATABASE (SQL Editor):
  - alter table spotlight add column track_id uuid references tracks(id) on delete set null
  - new function spotlight_apply_media(p_artist): copies the artist's video + their most-played public song into the spotlight
  - revoked execute on that helper from public/anon/authenticated (only the rotation can call it)
  - run_spotlight_cycle now calls spotlight_apply_media(winner)
  - backfilled the current artist (cycothaking)
- 19:17  Sent index.html to Claude as a file (attach via the + button, file name box: C:\Users\<you>\kandies-app\www\index.html)
- 19:19  APP (index.html):
  - Spotlight page shows a "Featured Song" play button when there's no video
  - spotlight admin upload only deletes files it created (keys starting "spotlight-"/"poster-"), so it can never delete an artist's own profile video (artist files start "artist-")
  - boot-time rotation call now logs errors instead of hiding them
- 19:35  Tested at localhost: Featured Song showed and played ("Eureka!")
- 19:4x  Committed + pushed (08462cc); deployed the www folder via Netlify Drop → Published 7:45 PM
- 19:53  Read admin_reset_spotlight_cycle to copy its admin-check pattern (profiles.is_admin)
- 20:02  DATABASE: new admin_set_spotlight(p_artist): admin-only, sets artist + pulls media + restarts the 7-day cycle + clears votes in one call
- 20:04  APP: admin "Set as Spotlight" button now calls admin_set_spotlight and logs errors
- 20:10  Tested: picked another artist by hand → media loaded; set back to cycothaking
- 20:38  DATABASE: enabled pg_cron; job #1 "spotlight-weekly-rotation", schedule '5 * * * *' (5 past every hour), runs run_spotlight_cycle(); verified active = true
- 20:44  Committed (f992c63), pushed, deployed → Published 8:44 PM

## SECURITY FINDING (found + fixed)
- FOUND (19:53): admin_reset_spotlight_cycle() was SECURITY DEFINER (runs with elevated/owner rights) but did NOT pin its search_path.
  Risk: "search_path hijacking". A definer function without a fixed search path can be tricked into resolving a look-alike table or function from another schema, running attacker-controlled code with elevated rights.
- FIXED (20:02, confirmed "Success"): alter function public.admin_reset_spotlight_cycle() set search_path = public;
- Every new definer function written tonight pins search_path and checks profiles.is_admin inside (raises 'Not authorized' otherwise).
- Least privilege: internal helper spotlight_apply_media can't be called by any user; admin_set_spotlight is callable by logged-in users but refuses non-admins.

## Troubleshooting
1. Copy-Item "Cannot find path ...Downloads\index.html"
   Cause: the file wasn't downloaded yet (or got renamed).
   Diagnose: Get-ChildItem "$HOME\Downloads" -Filter "index*" (showed index.html, 641652 bytes, 8:04 PM, matching the size of the version sent, proving it was the right file)
   Fix: download first, then copy. Remove-Item the old download first so the new one keeps the name index.html instead of "index (1).html".
2. Verify a copy worked: Select-String -Path ...\www\index.html -Pattern "Featured Song" (found lines 3409 and 3544)
3. PowerShell froze mid-command (title bar "Select Windows PowerShell") → QuickEdit Select mode from clicking inside the window. Esc resumes. Click the title bar only.
4. "bck-i-search:" appeared → Ctrl+R reached PowerShell (reverse history search), from pressing Ctrl+Shift+R while PowerShell was in front. Esc exits.
5. Netlify folder picker: the path went into the search box ("No items match your search") and the Folder box said kandies-app.
   Fix: Folder box must be ...\kandies-app\www. Uploading the whole project would publish android/.git/config files. Uploading only index.html would wipe icons, sw.js and manifest from the live site.
6. Asked whether PowerShell needed restarting after SQL changes → No. SQL runs in Supabase (cloud); the local server only serves files. Just refresh the browser after file edits.

## Lessons
- Read the existing schema/code before building (export column lists instead of screenshotting)
- Every SECURITY DEFINER function: set search_path + check authorization inside + revoke execute from roles that don't need it
- Record every security gap and whether it was fixed
- Shared files need ownership rules: never delete a file another feature might be using
- Deploy the whole www folder, never a single file
- Keep the app-side check as a backup even with a server schedule (defense in depth)

## Pending
- ✅ VERIFIED 21:50: first automatic cron run succeeded at 9:05 PM (01:05 UTC). Scheduler confirmed working.
- Android app gets these changes in the next APK build (GitHub Actions builds on every push)
