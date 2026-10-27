# Oct 19, 2026 — Bug fix: Ohio Spotlight not rotating

## Problem
The Ohio Spotlight was supposed to rotate every 7 days (top-voted Ohio artist, or a random Ohio artist if nobody voted). It hadn't changed in about 3 weeks, and no error was ever shown.

## Timeline (Eastern)
- 15:59  Bug reported: spotlight not cycling, no auto-pick after 7 days
- 16:05  Searched index.html in VSCodium (Ctrl+Shift+F "spotlight", 118 hits). Found the app calls a Supabase function:
  `try{ await sb.rpc('run_spotlight_cycle'); }catch{}`  ← runs only when someone opens the app; errors hidden
- 16:16  SQL: select proname from pg_proc where proname like '%spotlight%'
  → run_spotlight_cycle, admin_reset_spotlight_cycle, bump_spotlight_video_play all exist
- 16:18  Read the function with pg_get_functiondef. The logic was correct.
  Suspect: last line "delete from spotlight_votes;" has no WHERE clause
- 16:24  SQL: spotlight row cycle_started_at = 2026-09-10, age = 39 days → ~5 missed rotations
- 16:30  Chrome DevTools Console showed: POST .../rpc/run_spotlight_cycle 400 (Bad Request) on every page load
- 16:34  ROOT CAUSE confirmed in Console:
  {code: '21000', message: 'DELETE requires a WHERE clause'}
- 16:40  FIX: recreated the function with
  `delete from spotlight_votes where true;`
  Verified saved: pg_get_functiondef(...) like '%where true%' → true
- 16:46  Ran select run_spotlight_cycle(); → new artist picked (d76a688f-...), age reset to 20 seconds
- 16:50  Confirmed in the app: new spotlight artist shows, works

## Root cause explained
- Supabase blocks DELETE/UPDATE without a WHERE clause when it comes through the app's API (a safety guard called "safeupdate", which stops accidental "delete every row" mistakes).
- The function errored on its last line, so Postgres rolled back the WHOLE function, including the new artist pick.
- Supabase's JS client RETURNS errors instead of throwing them, so the app's try/catch never fired. The failure was completely silent for 39 days.
- Running the function in the SQL Editor works even without the fix (the admin role skips the guard). That's why testing through the app's path (browser Console) was needed to see the real error.

## Side issues hit while debugging
1. Typed "allow pasting" WITH quote marks → Chrome treated it as a string; the paste warning stayed. Fix: type it with no quotes.
   Security note: this warning defends against "self-XSS" scams, where attackers trick people into pasting code into DevTools.
2. Mini keyboard has no F12 key → use Fn+F10 (blue F12 label), or Ctrl+Shift+J (opens the Console directly), or right-click > Inspect.
3. Console commands did nothing → the page was still loading (tab spinner), so commands queued.
4. Page stuck loading ~15 min → the server's PowerShell window was in QuickEdit "Select" mode (the title bar said "Select Windows PowerShell"). Clicking inside a console window pauses the running program. Fix: press Esc. Prevention: don't click in the server window; use a second window for commands.

## Lessons
- Silent catch blocks hide real failures. Always log errors.
- Test through the same path users take; the admin SQL editor can mask permission and safety-rule errors.
- Read the code before running anything that changes live data.

## Follow-ups
- Make the app log rpc errors (console.warn on res.error) so failures can't hide again
- Consider a Supabase cron job so rotation runs even with no visitors
- NEXT FEATURE: when the spotlight rotates, auto-feature media: the artist's music video if one exists, otherwise their most popular song
