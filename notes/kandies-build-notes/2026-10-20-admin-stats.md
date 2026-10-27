# Oct 20, 2026: Detailed admin Stats + account-type cleanup

## Goal
A much more detailed admin Stats section (listeners vs artists, exact numbers, name-by-name artist lists), then clean up the account-type issues the new stats uncovered.

## What shipped (live on kandiesapp.com, 6:23 PM)
- Stats page rebuilt on one admin-only database function (admin_stats):
  - Accounts: listener vs artist split bar, new listeners/artists this week and month
  - Listener activity: active listeners (7/30 days), plays, plays per listener, who's playing, likes, comments, hearts, Spotlight votes
  - Artist activity: claimed/unclaimed/Ohio profiles, uploaded this month, zero plays, no songs
  - Content: public/private songs, uploads, songs by genre
  - 30-day chart, top songs and artists with unique listener counts, ad clicks, bug reports
- Artist detail lists (admin_artist_lists), tap to expand, tap a name to open the profile:
  - New artists (last 30 days) and how they joined
  - Profiles with nothing uploaded
  - Songs uploaded, zero plays
  - Artist accounts with no profile set up

## Fixes
- Old Stats undercounted: it counted plays in the browser, and Supabase caps each request at 1,000 rows. Counting now happens in the database.
- 3 artists had listener account types (Dangerous Zygos, Bornking7, BLACK PYRAMID SYNDICATE). Cause: the "Become an artist" button made an artist profile but never updated the account type. Fixed at 6:32 PM.
- Locked down which profile fields users can change (see security note).

## Troubleshooting
1. Stats function didn't exist (count = 0) → the file was never pasted; pasted from the code block instead
2. Downloads didn't land → the file card opens a preview; Download is in the ⌄ menu next to Copy
3. PowerShell commands pasted into the SQL Editor → every code block now has a "Run in:" label
4. index.html overwritten to 120 bytes → copying the command replaced the file on the clipboard. Restored with git restore, then saved the new file through Notepad (Save As, UTF-8)
5. Browser showed the old version → hard refresh (Ctrl+Shift+R); check the server log for a new request

## Lessons
- Count in the database, not the browser
- Label where every command runs
- Never put a file on the clipboard and then copy a command
- Verify file size and contents before and after copying
- Git can restore any committed file

## Decisions
- Listeners can switch to artist themselves, one way only. Artists can't switch back (they'd need a new account with a new email). Show a clear warning before switching.

## Next up
- Build the one-way "switch to artist" flow with the warning (Become an artist + Claim profile)
- Make "Claim my artist profile" fully self-service (no admin claim codes)
- Tighten what artists can edit on their own profile (owner, claim code, Ohio tag)
- Report/Block for comments (Play Store requirement)
